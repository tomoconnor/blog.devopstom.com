---
title: Right-Sizing EKS in Practice 
date: "2026-01-01T15:00:01.123Z"
description: "From 14 Nodes to 8 Without Breaking Production"
---

EKS clusters rarely start inefficient.

They drift there.

What begins as a reasonable number of small nodes slowly becomes a fixed cost floor. HPAs are set optimistically. Storage is over-provisioned “just in case.” Instance types are chosen for safety, not fit.

Recently, I revisited an EKS environment that had evolved into:

* 14 × `t4g.medium` nodes
* Predominantly gp2 volumes
* HPA minimum replicas set to 3 or 4 across multiple services

Nothing was on fire. Services were stable. But the cost profile didn’t match the actual workload behaviour.

This wasn’t about aggressive cost cutting. It was about aligning costs to the actual workload profile.

---

## Step 1: gp2 → gp3 (The Quiet Win)

The first optimisation wasn’t Kubernetes at all.

It was storage.

gp2 ties IOPS directly to volume size. If you want more IOPS, you increase disk size — whether you need the capacity or not.

gp3 decouples performance from capacity.

Several volumes had been sized historically to guarantee I/O headroom. Migrating them to gp3 preserved performance while reducing baseline cost. No workload changes. No downtime. Just correcting a structural inefficiency.

It's a reminder: infrastructure decisions linger longer than the assumptions that created them.

---

## Step 2: The 14 Node Illusion

The cluster ran on 14 × `t4g.medium` nodes.

On paper:

* 2 vCPU each
* 4GB RAM each
* 28 vCPU total
* 56GB RAM total

It looked fine.

In practice, smaller nodes multiplied overhead:

* DaemonSets replicated across every node
* kube-system components scaled horizontally
* Fixed per-node memory consumption reduced usable capacity
* Fragmentation made bin packing inefficient

There’s a common assumption that “more small nodes = better efficiency.”

In reality, Kubernetes bin packing isn’t magic. If pods request awkward CPU/memory ratios, fragmentation increases. The scheduler can’t always consolidate effectively.

The cluster wasn’t overloaded — it was structurally inefficient.

---

### The Hidden Constraint: ENI and Pod Density Limits

It’s easy to think in terms of CPU and memory when evaluating bin packing.

In EKS, that’s only part of the story.

Pod density is also constrained by the underlying instance’s Elastic Network Interface (ENI) limits. Each ENI supports a limited number of secondary IP addresses, and therefore a limited number of pods.

For smaller instance types like t4g.medium, this ceiling can become the dominant constraint before CPU or memory are exhausted.

The result?

Nodes appear underutilised from a resource perspective, but the scheduler cannot place additional pods because the IP limit has been reached.

This creates a subtle inefficiency:
* CPU available
* Memory available
* No remaining pod slots

At that point, scaling out becomes mandatory — not because the workload needs more compute, but because the networking model demands more ENIs.

Increasing instance size improves not just raw CPU and RAM capacity, but also pod density limits. Larger instances support more ENIs and more IP addresses per ENI, raising the effective pod ceiling per node.

This was a significant factor in moving away from many small nodes toward fewer, larger ones.

## Step 3: Rethinking Instance Shape

Instead of scaling down node count blindly, the approach shifted to shape.

The cluster moved to:

* 3 × `t4g.large` (on-demand baseline stability)
* 5 × `c6g.xlarge` (Spot burst capacity)

Why this mix?

The `t4g.large` nodes provided predictable baseline capacity for core workloads.

The `c6g.xlarge` Spot instances provided compute-dense capacity at lower cost, tolerable to interruption.

This did two things:

1. Reduced node count.
2. Increased per-node capacity, improving scheduler flexibility.

Fewer nodes meant:

* Less DaemonSet overhead.
* Better packing density.
* Lower per-node system resource tax.

The cluster didn’t lose much total CPU capacity — it redistributed it more efficiently.

---

## Step 4: The HPA Adjustment

One of the quiet contributors to constant over-provisioning was HPA configuration.

Several deployments had:

```yaml
minReplicas: 3
```

Reducing this to:

```yaml
minReplicas: 2
```

Across multiple services had a non-trivial impact on node utilisation.  Some services were drastically overprovisioned at minReplicas: 4 or 5. 

HPAs are often set conservatively and forgotten. Over time, minimum replicas become a fixed cost multiplier.

After observing real traffic patterns and utilisation metrics, lowering the minimum replica count did not meaningfully degrade service behaviour.

The cluster had been permanently scaled for peak assumptions that no longer reflected reality.

---

## The Result

The environment moved from:

* 14 × `t4g.medium`

To:

* 3 × `t4g.large`
* 5 × `c6g.xlarge` (Spot)

Without major service loss.

No outages.
No cascading failures.
No emergency rollbacks.

Just a system more aligned with its actual workload profile.

---

## What This Reinforced

A few things became clear:

* gp2 → gp3 is still one of the easiest structural and finops wins available.
* Node count is not a proxy for efficiency.
* Kubernetes bin packing improves when instance shapes are chosen deliberately.
* HPAs quietly define your cost floor.
* Mixed on-demand + Spot strategies work well when baseline capacity is separated from burst capacity.

Cost optimisation is often framed as an aggressive trimming exercise. In practice, it’s architectural hygiene, and a good opportunity for a spring clean.

---

## The Subtle Lesson

Security, reliability, and cost are not separate concerns.

Over-provisioning can mask fragility.
Under-provisioning can create incident risk.
Poor bin packing increases noisy neighbour potential.
Spot without a stable baseline invites instability.

Right-sizing is not just about reducing spend. It’s about understanding how the system actually behaves under load.

That understanding is what makes infrastructure predictable.

And predictable systems are safer systems.

## Footnote

There was a time when running servers below 60% CPU utilisation felt irresponsible.

In the bare-metal world, capacity was fixed. You bought hardware upfront, racked it, powered it, and then squeezed every drop of utilisation out of it. Idle capacity was wasted capital.

Cloud infrastructure changes the economics.  Perhaps we overcorrected.

Cloud flexibility solved capacity anxiety, but it also removed the discipline of optimisation.

Bare-metal thinking wasn’t about suffering.
It was about understanding the system deeply enough to trust it near its limits.

Kubernetes doesn’t remove that requirement.
It just changes the constraints:

* ENI limits
* Pod density ceilings
* DaemonSet tax
* HPA floors
* Instance shape fragmentation

What if we aimed for 60–80% sustained utilisation with intentional headroom? 

How many instances would quietly disappear?

