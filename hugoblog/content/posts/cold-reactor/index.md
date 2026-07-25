---
title: "The com:LAAG Cold Reactor"
date: "2026-07-22T20:15:00.000Z"
description: "A paddling pool, a positive void coefficient, and 16 days. Also: a really badly made kettle."
---

*Or: how our EMF village ended up with a spent-fuel cooling pond you're allowed to sit in.*

A nuclear reactor is, if you squint, a really badly made kettle. You have a heat source, you have water, and the entire multi-billion-pound apparatus of pumps, control rods and cooling loops exists to stop the water doing the one thing water does when you heat it. It's a kettle where boiling is the failure mode. Everything else is plumbing and paperwork.

I decided, roughly 16 days before [Electromagnetic Field](https://www.emfcamp.org/), that our village — **com:LAAG** — needed one. Not a real one, obviously. A paddling pool dressed as a spent-fuel cooling pond: blue glowing water, fake fuel rods lying on the bottom, a fistful of laminated warning signs, and — because it's EMF and you can't help yourself — a QR code that turns the whole thing into a CTF.

The plan was for it to double as an actual pool, too, because EMF in July can be a scorcher and a village that provides somewhere to cool your feet is a popular village. The pond had other ideas about that. More on the pond.

This is the write-up. It's a paddling pool. It's a reactor. It's a really badly made kettle. Here's how all three of those are true.

## The concept

The reference image in my head was a spent-fuel pond: those eerie, still, impossibly-blue pools where used fuel assemblies sit underwater for years, glowing faintly from [Cherenkov radiation](https://en.wikipedia.org/wiki/Cherenkov_radiation). Deeply photogenic. Also, conveniently, the *safe-looking* part of a nuclear facility — no billowing towers, no exclusion zone, just a calm blue rectangle that happens to be lethally radioactive.

Ours is neither lethal nor radioactive. It's a round paddling pool. The blue is lights and a whisper of dye. The "fuel" is plastic pipe. But at dusk, with the string lights on the gazebo behind it and the blue glow coming up through the water, it reads *exactly* right. People do a double-take, then read the sign, then laugh, then reconsider the sign.

A word about the pool, because it's the funniest failure of the build. I designed the whole thing around a **150 cm** paddling pool: big enough that a couple of people could genuinely sit in it and cool off, fuel bundles arranged around the edges, room to spare. The pond that actually turned up — and the one I built everything around by the time I noticed — was **75 cm** across. Half the diameter, a quarter of the area. There is no sitting in a 75 cm pool. There is, at most, dangling your feet in and pretending. So it became almost entirely decorative: a *look-but-mostly-don't-soak* Cherenkov pond. The signage promising a soaking pool stayed up regardless, which is arguably funnier than if it had worked.

The design brief wrote itself:

- Blue glowing "Cherenkov" pool.
- Fake fuel bundles visible underwater — horizontal, sparse enough that the pool still reads as water you *could* dip into (spoiler: barely).
- Nuclear-warning signage doing about half the comedic and narrative work.
- Optional control-panel interactivity for the people who want to *do* something.
- Zero real hazard. Battery-only. No mains anywhere near the water.

"Spent fuel pond, but also forbidden foot spa" was the vibe I was aiming for. I hit "spent fuel pond" squarely; "foot spa" got downgraded to "foot bath, one foot, briefly," by the tyranny of the 75 cm pond. We'll get to that.

## Building fake fuel assemblies

Real fuel assemblies are bundles of long, thin metal rods held in a precise lattice by spacer grids. I didn't need Zircaloy cladding and enriched pellets; I needed *something that looks like a bundle of silver rods when it's underwater and lit blue*, that sinks, that doesn't rust, and that a barefoot person can safely dip a foot next to.

The recipe I landed on:

- **15 mm PEX pipe** for the rods. Cheap, comes in long straight-ish lengths, cuts with a pipe cutter, doesn't corrode.
- **Aluminium HVAC foil tape** wrapped around each length to make it metallic. This is the trick that sells the whole thing — bare PEX looks like plumbing; foil-taped PEX looks like a fuel rod.
- **3D-printed spacer plates** with a 3×3 grid of holes. Nine rods per bundle, three plates along the length of each bundle to hold the lattice square. Three bundles in the pool.
- **Sand ballast** to make them sink and stay put.

So: 3 assemblies × 9 rods = **27 fuel rods** in the pond. (Hold that number; it comes back in the CTF.)

The spacer plates are the only bespoke part. Nine holes sized for 15 mm PEX, printed flat, three per bundle. Slide the plates on, space them out along the rods, glue or clip them so the bundle holds its shape, and you've got a rigid 3×3 lattice that looks the part. I kept every edge rounded and smooth, because feet were still going to end up in this thing and nobody wants a sharp printed corner in the sole of their foot.

> **Why foil tape instead of paint?**
> PEX is a polyethylene-ish plastic, and paint hates adhering to it — especially when the finished object is going to live *underwater* for a weekend. You can prep and prime and still get flaking. Foil tape gives you a convincing brushed-metal look in one pass with no cure time. The one caveat: tape edges can lift, so you want the printed spacer plates trapping the tape ends where possible, and you *really* don't want a peeling foil edge where it can catch a toe. Test overnight (see below) before you trust it.

> **Why sand instead of rebar?**
> The obvious ballast is a length of steel rebar or threaded rod down the middle of a tube. It's also a bad idea in a paddling pool full of people: it rusts, it stains the water orange, and a cut end is a sharp bit of steel in a pool. Dry sand poured into the bottom row of tubes, capped at both ends with silicone, gets you the same sink-and-stay-put with none of that. Leave the *other* tubes open so they flood — a sealed, empty tube is a buoyancy problem that will roll your beautiful bundle over and float it to the surface at the worst possible moment.

Fill the bottom row with sand and cap it; leave the top rows open to flood. Low centre of mass, no trapped air up top, sits flat on the liner. That's the theory, anyway, which is exactly why you test it.

> **The overnight bucket test.**
> Before committing three bundles' worth of effort, build *one* complete assembly and leave it in a bucket of water overnight. You're checking five things: does it **sink**? does it **roll** (bad — means trapped air or a high centre of mass)? does the **foil tape lift** at the edges? does the water go **cloudy** (dye bleed, sand escaping, tape adhesive letting go)? and does it generally **behave** the way it needs to for two days unattended in a pool? Everything you learn here is cheap. Everything you learn on-site with 400 people around is not.

## Dye and lighting

The instinct is to reach for dye and make the water properly, saturatedly blue. Resist it. The blue should come mostly from **light**, with dye as a tint at most.

Lighting is battery-only waterproof puck/pin LEDs in blue and cyan, sitting on the bottom of the pool, pointing up through the water. No mains, no transformer, nothing with a plug anywhere near a pool full of people. This is non-negotiable and it's also just *better*: light glows and moves and catches the ripples in a way that dyed water simply doesn't. A blue pool liner helps the lights do their job.

> **Why no methylene blue?**
> If you go searching for "how to make water glow blue," methylene blue comes up fast, and it is a *terrible* idea here. It's a genuine pharmacological compound with real body-contact and cleanup concerns, and it stains ferociously — hands, feet, swimsuits, the liner, your tent, everything. In a pool people are going to put bare skin in, that's a hard no. If you want *any* dye at all, use a little food colouring, or a pond/pool dye that's *explicitly* labelled body-contact safe — and lean on the lights for the actual glow. The best-looking version of this is 90% lighting and 10% tint.

## Signage is half the project

I mean this sincerely: the signs are load-bearing. They set the joke, they carry the safety message (by being *obviously* a joke, they make it clear the thing is fake), and they're what people photograph. I built a set in the classic yellow-and-black nuclear-hazard style:

```
☢  com:LAAG COLD REACTOR  ☢
Thermal output: questionable
Coolant quality: ornamental
Tea suitability: zero
```

```
CAUTION
NO DIVING
NO SPLASHING
NO CRITICALITY EXCURSIONS
```

```
CHERENKOV SOAKING POOL
Sit in. Soak.
Don't overload.
```

Plus the standalone one-liners: **REALLY BADLY MADE KETTLE**. **LOW RADIATION / HIGH RELAXATION**. **DO NOT LICK THE FUEL RODS.** (You should not need that last one. You will need that last one.)

The name is a small in-joke: "Cold Reactor" is a nod to [Everything Everything](https://en.wikipedia.org/wiki/Everything_Everything), because a good chunk of the village are fans. "Cold" also happens to be the entire safety proposition — it's a reactor that is, thermodynamically, off. It's a pond.

## The tock

Nuclear facilities in fiction always have that ambient, faintly menacing periodic *tock*. I wanted one. The plan on the physical side is deliberately low-effort:

- **Simple version:** a cheap quartz clock movement hidden in a small resonant box. Free tick, no electronics.
- **Better version:** a microcontroller driving a small solenoid tapper once per second against something that resonates nicely.

The one hard requirement is that it has to be **quiet enough that the village doesn't want to murder it after 40 minutes**. Ambient menace, not a metronome from hell. The accompanying sign, naturally:

```
AUDIBLE CORE TIMING REFERENCE
Do Not Silence The Tock
```

As it happens, the tock also ended up *in software* — the web app synthesises its own, but I'll get to that.

## Turning it into a CTF

This is EMF. A static art installation is fine, but an art installation with a QR code that drops you into a fake reactor control panel is *correct*. So the Cold Reactor got a companion: the **com:LAAG Cold Reactor — Cooling Loop Simulator**.

The whole thing is built to the constraints of a field in Herefordshire with unreliable signal:

- **Mobile-first static React/Vite app.** No login, no backend, no APIs, no database.
- **Progress lives in `localStorage`**, so a phone can lock, or drop signal, or get closed mid-run, and pick straight back up.
- It's a **static site** — the production build is just files. It can be served off a laptop at camp, a USB stick, or a real host. (I wrapped it in a multi-stage Docker build behind [Caddy](https://caddyserver.com/) for automatic HTTPS when it's on a public domain, but that's a convenience, not a requirement.)
- Scan the QR code at the pool, and you're an operator.

The flow is a six-stage journey from "you scanned a code" to "show this screen to a human and claim a physical challenge coin":

1. **Landing / the brief.** Also the moment audio unlocks (browsers require a user gesture before they'll make noise — the "Begin" click *is* that gesture).
2. **Operator login / physical recon.** Questions you can only answer by *actually looking at the pool*: how many assemblies (3)? rods per assembly (9)? geometry (3×3)? coolant quality (ornamental)? tea suitability (zero)? Answer them → `comlaag{operator_has_eyes}`.
3. **Cooling loop simulator.** The main event. Hold the core stable for a few seconds → `comlaag{coolant_loop_nominal}`.
4. **SCRAM test.** An alarm fires — `UNAUTHORISED SPLASHING DETECTED` — and you slam the big red SCRAM button → `comlaag{no_criticality_excursions}`.
5. **Operator Certification Exam.** A gated multi-question exam. Pass *every* question → `comlaag{really_badly_made_kettle}`.
6. **Coin claim.** Collect all four flags in the Flag Vault; it generates the coin-claim flag.

Recon before the arithmetic is deliberate: `operator_has_eyes` is exactly what it sounds like. You cannot solve stage 2 from the sofa. You have to be *at the pool*, counting rods, to progress — which is the whole point of tying a CTF to a physical installation.

### The fake physics

The simulator is where I let the "badly made kettle" premise turn into an actual, if entirely invented, control problem. The core temperature updates once a second. You have **five coupled controls** — pump speed, coolant valve, secondary pump, control rod insertion, and pressure — and the trick is that they fight each other across two sub-problems (how much heat you make, and how much you remove), each with its own band.

> **The fake physics model.**
> Every tick, roughly:
> ```
> flow       = (pump/100) * (valve/100)          # pump drives, valve throttles — coupled
> presFactor = cavitation multiplier from pressure  # low pressure cuts cooling
> cooling    = flow * presFactor * (loopCooling + secondary*exchanger)
> boilPoint  = 100 + pressureGain*(pressure - 50)   # pressure raises the boiling point
> voidFrac   = clamp((temp - boilPoint) / voidSpan, 0, 1)
> reactivity = clamp((100-rods)/100 + voidCoeff*voidFrac, 0, cap)
> heatIn     = heatGain * reactivity
> dTemp      = heatIn - cooling + ambientPull * (ambientTemp - temp)
> ```
> None of this is real. All of it is *plausible enough* to be a genuine multivariable balancing act — only about 0.08% of the control space is actually stable, so there is no lazy "set everything to 80%" answer.

The behaviours that make it fun to fight:

- **Pump and valve are coupled.** Flow is their *product*. Shut either one and circulation stops, cooling stops, and the core heats up. 100% pump with a shut valve reads 0% flow — which is one of the exam questions.
- **The core drifts to ~21 °C on its own** when there's no active cooling, because it is, thermodynamically, a bucket of water sitting in a field.
- **It over-cools.** The heat exchanger dumps to a chilled secondary loop, so if you slam every slider you *overshoot down* through the stable band into a COLD failure (floored around 12 °C). Stabilising is genuinely two-sided: too little cooling runs hot, too much runs cold. You have to find the middle.
- **Positive void coefficient — the actual kettle.** Coolant boils at 100 °C. Above that it forms steam voids, and in this reactor voids *add* reactivity, which makes more heat, which makes more voids: a self-accelerating runaway. Once it's boiling hard, inserting the control rods **doesn't save you** — you have to restore cooling or SCRAM to knock the temperature back below boiling and collapse the voids. (Real reactors with positive void coefficients are a genuinely notorious design problem. This one is a paddling pool, so the stakes are lower.)
- **The tock wobble.** The tock module induces a tiny periodic thermal disturbance, so you can't just park a slider on the exact edge of a band and walk away — you have to hold a *centred* setpoint through the wobble.
- **Run it past ~130 °C** and it logs a fictional "criticality excursion" (kettle achieved), then snaps back to a recoverable temperature. Do it twice and you get a whiteout and a restart button.

Everything a marshal might want to retune on-site — the temperature bands, flow thresholds, rod ranges, hold time, excursion temperature — lives in one `constants.js` file, precisely so it can be made easier or harder from a laptop in a tent without touching the actual logic.

### The tock, again

The app's ambience is entirely synthesised at runtime with the Web Audio API — no audio files shipped. A mechanical **tock** every couple of seconds across every screen, a harsh **clang** on each excursion, and a warbling **klaxon** while the SCRAM alarm is up or the coolant is boiling. It's scheduled on the audio clock rather than a `setInterval`, so the beat stays even even when the main thread is busy. So the Cold Reactor tocks in two places: a physical box by the pool, and in your hand.

### Flags, the vault, and stopping screenshot fraud

The four stage flags collect in a slide-in **Flag Vault**. Two nice properties fell out of building it this way:

- Vault slots are **typeable**, not just auto-filled. So flags printed on *physical* signage around the installation can be entered by hand — the digital and physical layers share one flag space.
- The vault store is kept **separate from gameplay state**, so typing a flag into the vault can't be used to skip the simulation. You still have to actually stabilise the reactor.

When all four valid flags are present, the vault generates the master coin-claim flag:

```
comlaag{cold_reactor_online_<todays-claim-word>}
```

That claim word rotates by weekday — Thursday `BORON`, Friday `XENON`, Saturday `KETTLE`, Sunday `CHERENKOV`, otherwise `COOLANT`. It's deterministic from the local date, which means **yesterday's victory screenshot is obviously stale**. To claim the coin you show a com:LAAG operator both the flag *and* the day's claim word, and a screenshot from Friday doesn't help you on Saturday. Cheap anti-cheat for a physical prize, no server required.

The success screen signs off the way the whole thing started:

```
COLD REACTOR STABILISED
Cooling loop: nominal
Coolant quality: ornamental
Tea suitability: zero
Criticality excursions: none
Final flag: comlaag{really_badly_made_kettle}
```

### The posters

Two QR posters, because I couldn't pick:

1. **Industrial warning-sign style** — yellow and black, `com:LAAG COLD REACTOR`, "Scan to stabilise the ornamental coolant loop," "Win a custom challenge coin," and the obligatory `NO DIVING / NO SPLASHING / NO CRITICALITY EXCURSIONS`, with the QR in a big `SCAN ME` box.
2. **Sci-fi control-panel style** — dark blue and cyan, `CHERENKOV SOAKING POOL`, "com:LAAG Cold Reactor CTF," "Coolant quality: ornamental / Tea suitability: zero / A really badly made kettle," QR in a `SCAN HERE` frame.

Same QR on both. One for the people who like hazard tape, one for the people who like glowing consoles.

## The best bit, which I take no credit for

Here is the thing about building something at EMF: the field is full of people who will improve your project without asking, using materials you did not know were on site. At some point someone turned up with **dry ice** and lobbed a chunk into the pond.

I had spent 16 days engineering the *look* of a Cherenkov cooling pond. The blue uplighting, the foil-taped bundles, the whole illusion. And then a pellet of solid CO₂ hit the water and did, for free, the one thing I hadn't even attempted: a thick, low, blue-lit **fog** rolling off the surface and spilling over the edge of the pool. Sublimating dry ice in warm-ish water throws a dense cold mist that hugs the ground, and lit from below in cyan it looked *exactly* like the establishing shot of a reactor hall in a film that's about to go wrong.

It also, briefly, made the "Cold Reactor" name literally true. The signage said "thermal output: questionable" and for about a minute the pond was the coldest, foggiest, most convincingly nuclear-looking object in the village. Nobody sat in it — you couldn't have fit anyway — but a small crowd stood around watching a paddling pool pretend to be a spent-fuel pond, which is exactly the outcome I was after, achieved by someone else's contribution.

I cannot recommend planning around dry ice — it's someone-else's-chunk-of-CO₂ energy, not a build step. But if a pond of yours is ever handed some, get out of the way and let it happen.

## What makes it EMF

Every ingredient here is a bit silly on its own. Together they're exactly the kind of thing EMF is *for*: it's an art installation, a technically-a-foot-bath, a physical puzzle, a five-variable control-theory toy, and a tiny bit of accidental education about why a positive void coefficient is a bad thing to design into your kettle. It's over-engineered in every direction and it cost about the price of a paddling pool, some PEX, a roll of foil tape, and 16 evenings.

And the premise holds all the way down. You can heat water. You can boil water. The entire difference between a cup of tea and a bad afternoon is *control* — pumps, rods, cooling loops, and the discipline not to let the thing run away from you. Take all of that away and what you've got left is a pond you can dip a foot in when it's hot — assuming, unlike me, you check the diameter before you build for it.

It's a paddling pool. It's a reactor. It's a really badly made kettle.

Do not drink the coolant. Do not lick the fuel rods. Do not silence the tock.

<!-- Images to add (drop files in this folder and uncomment):
![The Cold Reactor at dusk](cold-reactor-dusk.jpg)
![A fuel bundle: 3x3 PEX wrapped in foil tape](fuel-bundle.jpg)
![The Cooling Loop Simulator running on a phone](simulator.jpg)
![Warning signage set](signage.jpg)
![The two QR posters](posters.jpg)
-->
