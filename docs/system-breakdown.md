# A full breakdown of the system

The other pages explain each part of TokenBroker on its own. This page puts
the whole picture in one place — every moving part, how they fit together,
and why the system is built this way instead of the obvious alternative
(one company, one data center, one meter running).

## The big picture

![How a request flows through TokenBroker](../assets/charts/architecture.svg)

There are really only two kinds of thing happening at any moment:

1. Someone is **asking** for an AI response (through the API, the
   playground, or a browser session).
2. Someone is **answering** it — either a hosted cloud model the gateway
   calls directly, or a real, contributed device somewhere in the compute
   network.

Everything else in this document exists to make that exchange fair, safe,
and honest for both sides.

## Component by component

| Component | What it does | Who runs it |
| --- | --- | --- |
| **Gateway** | Authentication, billing, request routing, provider masking | The service operator |
| **Dashboard** | Sign up, manage keys, top up a wallet, watch usage and worker earnings | The service operator |
| **Hosted cloud models** | Large models the gateway calls directly on your behalf | Upstream providers (Ollama Cloud, and others as they're added) |
| **Compute network** | Real devices — desktops, phones — that claim and run queued jobs | **You**, and everyone else who joins |
| **External provider keys** | A worker's own free-tier key (Google AI Studio today), used to serve that provider's models | Individual worker operators, per account |
| **Live network status page** | Public, real-time capacity and activity numbers | The service operator, generated from real data |

No single piece of this is "the whole service." The gateway without a
compute network is just another AI reseller. A compute network without a
gateway has no billing, no catalog, and no way for a stranger to safely use
it. The two need each other.

## Two ways a request gets answered

**Path A — hosted cloud model.** The gateway calls a hosted provider's API
directly (Ollama Cloud, and other providers as they're integrated) and
relays the answer back. This is the simplest path and the one used for
models too large for any single contributed device to run.

**Path B — the compute network.** The gateway hands the job to whichever
online worker is best suited for it, in a strict order of preference:

![Dispatch tiers - which worker gets the job](../assets/charts/dispatch-tiers.svg)

A worker never receives a job outside its own capability — RAM, VRAM, and
storage are all checked before anything is assigned, and a worker only ever
serves an external provider's models if it (or its account) actually holds a
working key for that exact provider. There is no "just try it and see" mode
for that path — either a node is confirmed to have what a job needs, or it
never sees that job.

## Every dollar and every credit is accounted for

Money and credits move through the same kind of accounting discipline a bank
uses: nothing is created or destroyed, only moved and logged.

- A **wallet** balance is prepaid. Every request places a **hold** for its
  worst-case cost before it runs, then settles for the exact cost once it
  completes — so a long-running request can never overdraw the account, and
  a failed request never gets charged at all.
- A **compute job** follows the same discipline in the other direction:

![Life of one job, from creation to credit](../assets/charts/job-lifecycle.svg)

The requester pays once, up front, the moment the job is created — this is
what stops the queue from being used to mint free credits. The worker is
paid once, only after the response clears every quality gate and is actually
accepted — this is what stops fabricated or garbage responses from earning
anything. Both halves are **idempotent**: replaying the same completion or
acceptance event twice never double-charges or double-pays anyone.

## Multiple providers, on purpose

TokenBroker deliberately doesn't depend on one upstream. Today that means
Ollama Cloud (subscribed to by contributors) and Google AI Studio (a second,
completely independent free-tier quota pool a contributor can add), with
more providers planned as verified, working keys become available. See the
[capacity comparison](compute-network.md#optional-bring-your-own-cloud-api-key)
for how these two stack up today, and why they're honestly labeled
differently — one is a direct measurement, the other a conservative
estimate pending its own measurement.

This matters beyond redundancy: **no single company's rate limit, pricing
change, or outage can take the whole network down.** A contributor who adds
a second provider key isn't just adding capacity for themselves — they're
adding a second, independent path to serving the whole community.

## Trust, without asking you to trust blindly

Three separate audiences need to trust three separate things, and the system
is built to earn each one on its own terms rather than asking for blanket
trust:

- **A requester** needs their prompt handled safely and never linked back to
  whichever stranger's machine answered it. See
  [Security & privacy](security-and-privacy.md).
- **A contributor** needs assurance that running this on their own hardware
  won't expose them to abuse, legal risk, or a hidden background process
  they can't see. The worker is a visible tray app, never hidden, and every
  private job is end-to-end encoded so the operator's machine never sees
  plaintext it doesn't need to.
- **The community at large** needs proof the numbers on the [network status
  page](https://tokenbroker.hopto.org/network) are real, not marketing. That
  page is deliberately aggregate-only (no per-node, no per-person detail)
  and explains its own math, including where a number is a measurement and
  where it's an honest estimate.

## Where to go deeper

- [Overview](overview.md) — the short version.
- [Tested & verified](tested-and-verified.md) — real measurements, real
  limits, and a real networking bug we found and fixed.
- [Compute network](compute-network.md) — how to actually join and earn.
- [Models catalog](models-catalog.md) — what you can run today.
- [Pricing & credits](pricing-and-credits.md) — the economics in detail.
- [Security & privacy](security-and-privacy.md) — the honest controls and their limits.
- [API reference](api-reference.md) — if you're building against it.
- [FAQ](faq.md) — quick answers, organized by topic.
- **[Take back your compute](take-back-your-compute.md)** — why any of this exists.
