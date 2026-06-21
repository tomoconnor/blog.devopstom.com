---
title: "On frustrating interview practices"
date: "2026-06-21T00:00:00.000Z"
description: "Why I declined a CoderPad challenge, what happened when I did, and what the response revealed."
---

# Stop wasting senior engineers' time

I declined a coding challenge recently. Not because I couldn't do it; because I shouldn't have to.

The brief was to parse a multi-document Kubernetes YAML file in Python, find resource policy violations across Deployments, and group them by team and service labels. There was a CoderPad link and a timer. The role was Staff Platform Engineer.

Here's the thing: no platform engineer solves that problem in Python. You solve it once, at admission time, with a Validating Admission Webhook or an OPA/Gatekeeper policy. The violation never reaches the cluster. The YAML never needs parsing after the fact. The exercise doesn't test platform engineering; it tests whether you know PyYAML's `safe_load_all` and can iterate nested dicts without hitting a KeyError under pressure. That's a different skill set, and frankly, a less useful one.

I've been doing this for fifteen years. I run Kubernetes in production. I run it at home for fun, on bare metal, because I find it interesting. I hold three AWS specialty certifications. I have shipped real systems at real companies that real people depend on. The idea that a timed Python exercise in a browser tab is a meaningful signal about any of that is, at this point, an insult dressed up as a process.

## The problem with coding challenges at senior level

Coding challenges made a kind of sense once. Early in a career, when someone's resume is thin and their claims are untested, a structured exercise gives you something concrete to evaluate. Fine. But the same logic doesn't hold when someone has a decade and a half of verifiable work history, open-source projects you can read, and a track record you can check with a phone call.

At senior and staff level, the signal you actually want is judgement. Can this person look at a problem and immediately identify that the framing is wrong? Can they propose a better solution than the one asked for? Do they understand the operational consequences of the choices they make? None of that is visible in a CoderPad session. What you see instead is how well someone performs under artificial time pressure on a problem that's been deliberately simplified to fit in 45 minutes.

The two things are not the same.

## It's 2026

I want to be direct about something that nobody in hiring seems willing to say out loud: AI writes this code now. If you give me a YAML parsing challenge, I will use a tool to help me write it, because that's what I'd do on the job. The fiction that a coding exercise measures some pure unassisted ability is dead. It's been dead for a while. Most companies just haven't updated their process to reflect that.

What hasn't changed is whether someone can read the output, understand what it's doing, identify where it's wrong, and make a call about whether it's the right approach at all. That's the skill. That's what you should be testing. A timed box on CoderPad doesn't get anywhere near it.


## What good looks like

The best technical interviews I've had were conversations. Someone walked me through a real system they'd built, told me where it had gone wrong, and asked what I'd have done differently. Or they described an incident and asked how I'd have approached the post-mortem. Or they asked me to design something and then pushed on the edges of whatever I proposed.

Those conversations took about the same time as a coding challenge. They told the interviewer vastly more. And they respected the fact that I'd turned up with experience worth discussing.

A take-home can work too, if it's scoped honestly. Give me a real problem, give me reasonable time, let me use the tools I'd actually use on the job. Then talk to me about what I built and why. That's a conversation with some substance behind it.

## The self-selection problem nobody mentions

Companies that use CoderPad for Staff-level roles are selecting for a specific kind of candidate: one who either doesn't know their own worth, or who is desperate enough to comply anyway. Engineers who can afford to walk away from an insulting process do exactly that. The pipeline fills with the ones who can't.

If your interview process consistently drives away experienced engineers, the problem is not the engineers.

## On refusing

I declined the challenge. I explained why. I said I was happy to discuss platform engineering in whatever depth they wanted, but I wouldn't be doing a timed Python exercise, and here was my reasoning.

What happened after I stopped is the point. We spent twenty minutes talking about gRPC keepalive issues with ALB, Karpenter node pool strategy, spot instance interruption handling, ingress controller migrations. Real problems, real tradeoffs; the kind of conversation where you can actually tell whether someone knows what they're doing.

At one point the interviewer mentioned that writing code is a minority of his day. So the primary assessment method tests a minority activity, in an artificial environment, under time pressure, in a language neither of us primarily uses. Nobody stopped to ask whether that made sense.

That twenty-minute conversation told both of us more than the CoderPad session ever would have. That's the point.

They closed the process. No alternative offered, no conversation about how else they might assess someone at this level. Just closed.
That tells you exactly how they think about seniority.

## The part that settled it

I explained my position on Admission Webhooks and OPA. They weren't interested.

That's boldest red flag in the whole process. Not the timed exercise, not the language mismatch; the fact that a detailed, relevant answer on a genuinely interesting problem landed in silence.

I've been doing this long enough to stop pretending these exercises tell anyone anything worth knowing. And I'm not doing them.