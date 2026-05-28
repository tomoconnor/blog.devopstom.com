---
title: Teeny Tiny Weather Service
date: "2026-05-28T21:30:01.123Z"
description: "A weather report in 9,808 bytes"
---

*Or: what "self-contained binary" actually means, once you have to do TLS yourself.*

I had an n8n workflow that ran at 05:00 every morning, fetched a five-day forecast from OpenWeatherMap, asked GPT-4.1-mini to write me a fun summary of the day, and posted it to a Mattermost channel. It worked. It was also doing maybe three syscalls' worth of useful work inside a n8n runtime.

So I rewrote it in Go. Single binary, static link, scratch container. 10.3 MB image, 7.7 MB binary. Comfortable. Done.

And then the question that derails every evening: Now what? Can I make it smaller?

A few hours later I had three implementations of the same daily weather workflow:
* The Go one at 10.3 MB. 
* A Rust one with `reqwest` and `rustls` and `chrono-tz`, at 5.97 MB. 
* And one more: an x86-64 NASM program that compiled to a 9,808-byte ELF binary inside a 22.5 KB scratch container. 

That last one was about 460× smaller than the Go binary. It also doesn't do TLS.

This is a post about how all three of those facts are true, and what the asm version taught me about the difference between binary size and shipping cost.

## Why bother

Honest answer: I wanted to find out what the smallest implementation was. Modern infrastructure work has its own particular kind of bloat-tolerance; we ship 100 MB containers for what's morally a `curl` pipeline because the abstractions are cheap and our compute budgets are obscene. 
I was thinking back to the old adage "640K should be enough for anyone" - Where does that sit in 2026?

The other answer: I wanted to see how a current LLM handled NASM. My initial guess was going to be "badly". 
I was wrong; Claude can write x86-64 assembly nearly as readily as Python.

The bugs aren't in the model's understanding of `repe cmpsb`; they're in the difficult interactions between protocols and arithmetic. Which is also where the bugs would be if a human wrote it.

I picked Claude because I haven't touched Assembly language in over a decade, and whilst I could read and understand the x86 instruction set, and figure it all out, why not just throw this at Claude for a laugh?

## The TLS problem

The first conversation I had with Claude about the asm version was about TLS.

OpenWeatherMap, OpenAI, and Mattermost are all HTTPS-only. Implementing TLS 1.3 in assembly is a multi-month project. The bulk crypto (AES-GCM, SHA-256) has dedicated x86 opcodes thanks to AES-NI and the Intel SHA extensions, but the handshake state machine, X.509 / ASN.1 cert parsing, ECDSA P-256 verification, HKDF, the trust store, SNI, and ALPN are all your code. A "minimal" pure-asm TLS client would be tens of thousands of lines and would need to keep up with Cloudflare's edge taking the BoringSSL position on every protocol detail. Not a challenge for an evening's experiment.

So we're not doing TLS in asm. The asm needs to speak plain HTTP. Something else has to do the TLS.

The standard reverse-proxy options (nginx, HAProxy, Caddy) are 15-30 MB each; using one of those would dwarf the asm binary that justified the exercise in the first place. What I wanted was the smallest possible thing that would terminate TLS on a localhost port and forward to the real upstream.

That thing is `socat`. About 14 MB on Alpine, no HTTP smarts; just a bytestream proxy with an OpenSSL backend. Three of them, one per upstream.

## The sidecar trick

The architecture I landed on uses Docker Compose's `network_mode: "service:weatherreport"` to put the three `socat` instances inside the asm container's network namespace. From the asm container's perspective, the sidecars are all bound to `127.0.0.1:8081`, `:8082`, `:8083`. From the sidecars' perspective, they're listening on what looks like loopback inside a namespace they share with the main program.

```yaml
services:
  weatherreport:
    build: .
    restart: unless-stopped
    environment:
      OPENWEATHER_API_KEY: ${OPENWEATHER_API_KEY:?}
      # ...

  tls-owm:
    image: alpine/socat
    network_mode: "service:weatherreport"
    command: TCP-LISTEN:8081,fork,reuseaddr OPENSSL:api.openweathermap.org:443

  tls-openai:
    image: alpine/socat
    network_mode: "service:weatherreport"
    command: TCP-LISTEN:8082,fork,reuseaddr OPENSSL:api.openai.com:443

  tls-mattermost:
    image: alpine/socat
    network_mode: "service:weatherreport"
    command: TCP-LISTEN:8083,fork,reuseaddr OPENSSL:${MATTERMOST_HOST}:443
```

This is the same model Kubernetes pods use for sidecars: shared netns, isolated everything else. The asm program is genuinely a 9.8 KB binary that dials three localhost ports; `socat` does the actual modern internet.

The honest "deployment footprint" therefore includes that 14 MB of `socat` image. Once you account for it, the asm version's image cost is roughly equal to the Rust version's. The asm program is small because it offloaded the hard work, not because TLS is free.

That, I think, is the actual lesson of the whole exercise. Anything you ship that talks HTTPS contains, somewhere, the same TLS implementation. The Go binary contains it as part of the `crypto/tls` package and the embedded Mozilla CA bundle. The Rust binary contains rustls and the `webpki-roots` crate. The asm binary contains a reference to `socat` sitting in a sidecar. Three different ways of accounting for the same thirty thousand lines of careful, edge-case-laden cryptographic code.

## Building the thing

The asm itself is straightforward once you commit to "raw syscalls, no libc". Linux x86-64 uses six syscall-arg registers in a fixed order; you load a syscall number into `rax` and execute `syscall`; the return value comes back in `rax`. The program needs:

- `socket(2)`, `connect(2)`, `read(2)`, `write(2)`, `close(2)` for the HTTP
- `nanosleep(2)` for the daily timer
- A walk through `envp` to read the API keys out of the environment
- String concatenation for building HTTP requests
- A naïve decimal formatter for the `Content-Length` header
- Substring search for finding `"content"` in the OpenAI response
- A small JSON unescaper for `\n`, `\\`, `\"`, `\/`, `\t`, `\r`
- A `repe cmpsb`-based string equality for the `-once` flag

About 1,100 lines of NASM. Claude wrote the first cut; I corrected the architecture across three or four passes ("no, the sidecars share the netns" / "wait, the OpenAI response is pretty-printed JSON, not compact").

The build is a two-stage Dockerfile. Stage one runs `nasm -felf64 weatherreport.asm -o weatherreport.o` then `ld -s -o weatherreport weatherreport.o` inside an Alpine container; stage two is `FROM scratch` and copies the ELF over. Container size: 22.5 KB. There's no libc, no ca-certs, no shell, no resolver config. Docker handles namespace-internal name resolution.

## The three bugs

The asm worked end-to-end after roughly five sessions of fighting. The bugs were in protocols and arithmetic, which is to say: in the places you'd expect.

**Bug one: chunked transfer encoding.** Cloudflare in front of OpenAI sends responses with `Transfer-Encoding: chunked`. The body that arrives is a sequence of `<hex-size>\r\n<bytes>\r\n` chunks terminated by a zero-sized chunk. Our HTTP parser was looking for `\r\n\r\n`, slicing the headers from the body, and then handing the raw "body" to the content extractor, which got confused by the hex chunk-size prefixes embedded in the byte stream.

Claude wrote a chunked decoder. It segfaulted. It was dechunking on responses that had no need to be dechunked, then dereferencing things that weren't pointers. I went down that hole for an hour before giving up.

The fix turned out to be a single byte: change `HTTP/1.1` to `HTTP/1.0` in the request line. HTTP/1.0 doesn't support chunked transfer encoding; the server is required to use `Content-Length` or close-delimited body. The dechunk code still exists, inert.

**Bug two: a register-source typo in `build_mm_request`.** Now the program got further and segfaulted later, somewhere between extracting the LLM content and posting to Mattermost. With no debugger and no `printf`, the way you find that bug is to add an instrumentation routine that writes a single character to stderr before each step, then read the last letter printed before the segfault.

The last letter was `d`. The crash was in the function called immediately after `d`. That function had this code:

```nasm
lea  rax, [rel mm_body]
sub  rdi, rax
mov  r12, rax                    ; r12 = mm_body length
```

The comment lies. `lea rax, [rel mm_body]` loads the base address of `mm_body`; `sub rdi, rax` puts the length into `rdi`; the third line stores the base address into `r12`. Later, `r12` is used as the byte count for `rep movsb`. With `r12` holding a BSS address (a few million), `rep movsb` tries to copy a few million bytes into a 256 KB buffer. SIGSEGV.

The fix is one register: `mov r12, rdi`. The bug had been in the code for a while because earlier failures meant the program never reached the function.

**Bug three: the model's responses are pretty-printed.** The content extractor was looking for `"content":"` and slicing forward. OpenAI's JSON is pretty-printed: `"content": "..."`, with a space after the colon. The marker never matched.

The fix was to look for `"content"` only, then skip optional whitespace, expect `:`, skip whitespace again, expect `"`, then start copying. About fifteen extra lines of asm.

All three fixes were small. None of them were findable without the program getting close enough to working to expose them.

## Prompt Critical

The asm version doesn't aggregate the OpenWeatherMap forecast. There is no asm code to compute `min`/`max`/`avg` of the temperature; there is, instead, a one-line decision to escape the entire 30 KB forecast JSON into the OpenAI user message and let the model do whatever it wants with it.

This was supposed to be a bodge, and yet it produced noticeably better reports than the Go version, which had been faithfully reproducing the n8n workflow's pre-aggregation step.

The Go and n8n versions take today's eight three-hour forecast slots, compute `min`, `max`, `avg`, `max_pop`, dominant condition string, and feed the model a 12-line summary. The model is then asked to write "how the WHOLE DAY will feel from morning to evening". The temporal structure that the model needs to write a morning-to-evening narrative has already been thrown away. The model can only generate plausible filler.

The asm version, by being too tedious to write JSON aggregation in, hands the model 40 three-hour slots with their `dt_txt` timestamps, conditions, wind, and precipitation probabilities. The model reads the structure it needs to produce time-banded paragraphs: morning is cool and overcast, afternoon warms up under broken cloud, evening cools again with light winds. The reports stopped reading like generic weather paragraphs and started reading like something an actual human had written looking at an hourly forecast.

Pre-digesting data before an LLM is a habit from the era of small context windows and expensive tokens. With 128k input and gpt-4.1-mini's pricing, it usually costs you more in output quality than it saves in input cost. 

"I cannot be bothered to do this in asm" was the right call, by accident.

I went back and rewrote the Go version to skip aggregation too. It produces the same quality of report as the asm one now. 

## Deploying with at(1)

The asm program has an internal scheduler: fire on startup, then `nanosleep(86400)`, then repeat. Every fire happens 24 hours after the previous one completed, with a few seconds of drift forward per day. For a morning weather report, drift in the seconds-per-day range is invisible.

To pick the initial fire time, I considered three options: cron firing a `docker compose restart` daily; a systemd timer; a one-shot `at` to start the stack at the chosen time.

Cron and systemd timers are right for pinning to a wall-clock time forever. They're also more setup than I wanted for a "type this once on my server" deployment. `atd` is what's right when you want to fire one command, once, at a specific time, and then have the consequences of that command persist by themselves.

```
$ at 04:55
warning: commands will be executed using /bin/sh
at> cd /opt/weatherreport && docker compose up -d --build
at> <Ctrl-D>
```

`atd` waits until 04:55, runs the command, exits. `docker compose up -d --build` brings the stack up; the asm container sleeps five seconds for `socat` to bind, then fires the cycle. The first report lands in Mattermost about ten seconds later. From then on, `restart: unless-stopped` keeps the stack running and the asm's own 24-hour timer drives the schedule.

If the container ever does crash and restart, the schedule shifts forward (it now fires 24 hours from the restart, not from the previous fire). For a daily weather report I will not notice and will not care; if it ever starts to matter I will put `0 5 * * * docker compose restart weatherreport` in cron to pin it.

## What I actually learned

The 9,808-byte binary is fun. The 22 KB container is fun. They're also a bit of a lie about how much code is involved in talking to the modern internet.

The Go binary's 7.7 MB contains, statically linked, a TLS 1.3 implementation, an HTTP/2 stack with chunked decoding, an ASN.1 parser, the Mozilla CA bundle, and a cryptographically secure RNG. The Rust binary's 1.4 MB contains the same things via a different implementation. The asm binary's 9,808 bytes contains none of them; the bytes that should contain them are sitting next to it on disk as 14 MB of `socat`.

The point isn't that one of these is "better". The point is that when we say "self-contained binary", we mean "self-contained except for the parts so universally needed that we've stopped noticing them". 

The other thing I learned is that an LLM can write a thousand lines of NASM that mostly works, and the bugs are in the same places they would be if I wrote them. 
The "Claude wrote x86-64 assembly" headline is the surprising bit; the un-surprising bit is that the debugging session looked exactly like one with a junior engineer who's confident with the instruction set and shaky on the protocols. We have, more or less, just acquired another colleague. 

The interesting question for the next decade is not whether they can write the assembly. They can. It's what we do with the time we used to spend writing it.

The asm version is at [github.com/tomoconnor/teeny-tiny-weather-reporter-asm](https://github.com/tomoconnor/teeny-tiny-weather-reporter-asm). The Go and Rust versions will be released eventually. The asm one is the interesting one though.
