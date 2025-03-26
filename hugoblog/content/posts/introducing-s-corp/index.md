---
title: "Introducing: S-Corp™"
date: "2025-03-26T23:00:01.123Z"
description: "Using Comnoco to automate startup nonsense at scale."
---

## 🚀 How I Used a No-Code Backend to Power a Fake Startup Certification Platform

Recently I launched [S Corp™](https://s-corp.lol) — a parody certification for startups that want to look ethical without the burden of actually doing anything. You type in your company name, click a button, and get a downloadable certificate that looks suspiciously legit. It’s satire. It’s fast. It’s 100% nonsense.

Behind the scenes, the app is powered by React, Tailwind, and a healthy disrespect for real sustainability certifications.

But the interesting bit — and the reason I’m writing this — is the backend.

I didn’t build one.

Instead, I used [Comnoco](https://comnoco.com/), a no-code logic engine that let me create a surprisingly flexible backend without writing any actual server code. And despite the project being deeply unserious, Comnoco made some genuinely serious things possible:

* 💬 Generating random testimonials on demand
* 🧾 Rendering PDF certificates from a .docx template
* ⚡️ Keeping the entire site stateless, fast, and free from maintenance overhead
* 💡 The Frontend Was Easy. The Logic? Not Worth Coding.

S Corp is a mostly-static React site with a handful of interactions. The testimonials and certificate download are the only dynamic bits — and honestly, they weren’t worth spinning up a real backend for.

The frontend is so boring it's available on [Github](https://github.com/S-Corp-lol/s-corp-lol/), and hosted by Cloudflare Pages.

### ⚡️ What I needed:
* An endpoint to generate 5 fresh testimonials (random startup name, employee, title, templated quote)
* A certificate generator that accepts a company name and founder name, inserts them into a DOCX file, and returns a PDF

In a typical backend, that’d mean:
* Setting up a server
* Writing routing and templating logic
* Hosting, securing, monitoring
* Probably dealing with Puppeteer, LaTeX, or wkhtmltopdf hell

I didn’t want to do any of that. So I used Comnoco.

### 💬 Dynamic Testimonials from a No-Code Flow 
In Comnoco, I created a data store (actually a single variable containing JSON) with:
* 150 startup names
* 150 fake people
* 150 hilarious job titles
* 60+ Go-style text/template testimonial patterns

Then I created a logic flow called GetTestimonials that:
1. Picks 5 templates at random
2. Fills in the variables
3. Returns them in JSON

No servers. No Postgres. Just a weirdly delightful API I can call from the React frontend.

### 🧾 Certificate Generation That Feels Too Easy 
Here’s where Comnoco **really** shined.

I uploaded a .docx template with {{ CompanyName }} and {{ FounderName }} placeholders. Then I built another Comnoco flow that:

1. Accepts a POST payload with those values
2. Injects them into the DOCX
3. Converts it to a PDF
4. Returns it with the correct Content-Disposition header so the browser downloads it

I didn’t need to mess with a single PDF library. The whole thing took less time than fighting with any open-source PDF generator I’ve ever touched.

### 🧠 What Worked (and Why It Matters)
Comnoco gave me the ability to:

* Build two fully functional backend endpoints
* Keep all logic version-controlled, editable, and visual
* Avoid deploying anything

It’s not just for jokes. I could see myself using it for internal tools, prototypes, customer onboarding flows — anywhere I’d normally duct-tape together Lambda functions and Zapier.

And if you’re someone like me who builds weird stuff on weekends (or out of spite), Comnoco hits the sweet spot of fast, functional, and actually fun to use.

### 🧾 TL;DR 
S Corp is a parody startup certification site

It has no backend in the traditional sense

Comnoco handles all dynamic logic:
* Random testimonials
* PDF certificate generation
* The project shipped fast and cost nothing to host

You can try it (and download your own fake badge of ethical superiority) here: https://s-corp.lol

And if you’re curious about [Comnoco](https://comnoco.com/), give it a spin — especially if you’re working on anything where the backend doesn’t need to be your job.

