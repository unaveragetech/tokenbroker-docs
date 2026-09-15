# Frequently asked questions

Answers organized by what you're actually trying to do — use the service,
run a worker, or understand the economics behind either. If your question
isn't here, the [full system breakdown](system-breakdown.md) goes deeper on
mechanics, and [Tested & verified](tested-and-verified.md) covers what's
been proven to actually work.

## Service basics

### What is TokenBroker?

TokenBroker is a public AI inference service. You get one account, one API
key, and one bill, and you can query hundreds of models — from small local
models to 600B+ cloud models — through a single OpenAI-compatible API at
[https://tokenbroker.hopto.org](https://tokenbroker.hopto.org).

### Do I need a GPU or a server?

No. To **use** the service you only need an internet connection and an API
key. GPUs belong to the compute network, not to you. If you want to *join*
the network and earn credits, then yes, your own hardware (ideally with a
GPU) becomes useful — see the compute network docs.

### Is this a wrapper around someone else's API?

Partly — and openly. The catalog mixes:

1. Open-weights models running on the distributed compute network;
2. Large models hosted on Ollama Cloud (free-tier friendly);
3. Community personas published as cloud variants;
4. Models served by contributed external-provider keys (Google AI Studio
   today), where a worker calls that provider's own API directly.

There is no hidden rebranding: each model's listing says where it runs, and
[the full breakdown](system-breakdown.md#multiple-providers-on-purpose)
explains why depending on more than one upstream is deliberate, not
incidental.

### Is the service free?

There's a free-ish path: new accounts get a wallet, and the network
occasionally runs promo/zero-cost models. But TokenBroker is a business —
paid access funds the network. You can also earn credits by running the
worker, which makes access effectively free for contributors.

### What models are available?

Hundreds, and the list grows weekly. Highlights include GPT-OSS 20B/120B,
Gemma 4, GLM 5.x, DeepSeek (R1 distills and V4 Flash), Qwen (2.5, 3, 3.5),
Kimi K2.6/K2.7 Code, MiniMax M2.7/M3, Mistral Large 3, Nemotron 3, Llama 3.x,
specialized coding/OCR/SQL models, and uncensored/abliterated variants. See
[models catalog](models-catalog.md).

### Does "uncensored" mean illegal content is allowed?

No. "Uncensored"/"abliterated" refers to models whose refusal training was
removed — they don't lecture you or refuse on moralistic grounds. They are
still subject to TokenBroker's terms of service, the laws of the jurisdictions
involved, and the hosting providers' policies. The service can revoke access
for abuse.

## Accounts & login

### How do I sign up?

Email + password, or Google. New email accounts get a verification link/OTP.
See [getting started](getting-started.md).

### I forgot my password.

Request a reset from the login page. A one-time code (OTP) is emailed to you;
enter it and choose a new password. The code expires quickly, and attempts
are rate-limited.

### Can I log in with Google?

Yes — Google OAuth is supported alongside standard email/password.

### Can I have multiple accounts?

You can, but the compute network enforces **one worker per account** and
hardware fingerprints, so you can't multiply your worker earnings by creating
accounts.

## API keys & usage

### How do I get an API key?

From the dashboard: API Keys → Create. It's shown once — copy it. Keys are
hashed at rest; if you lose it, revoke and create a new one.

### Is the API really OpenAI-compatible?

Yes. `chat/completions`, `completions`, `embeddings`, and `models` endpoints
follow OpenAI conventions. Most SDKs work by changing the base URL and key.

### Can I set a spending limit?

Yes — each key has a configurable **daily spend cap**. When it's hit, that key
stops working until the next day (or until you raise the cap).

### What happens when my balance runs out?

Requests return HTTP 402. Nothing is charged, nothing breaks. Top up from the
dashboard and continue.

### Are my requests logged?

For billing and abuse prevention, yes — request metadata and transcripts are
retained. They are not used for training, and provider identity is stripped
from responses. See [security & privacy](security-and-privacy.md).

## Payments

Money moves in one direction here — from your wallet to a request's exact
cost, never more, never before it's actually run.

### How do I pay?

Square-processed payments: credit/debit cards or Cash App. You can top up a
wallet, buy a plan, or use a static payment link from the dashboard.

### What is a payment hold?

Before a request runs, the estimated cost is held from your balance. When the
request settles, the hold is released and the actual amount is charged. No
double charges.

### Can I get a refund?

Wallet credits can be refunded at the operator's discretion per the terms of
service. Contact the operator through the dashboard.

### Why do I see ads?

Ads (Google AdSense and similar) keep entry costs low. Ads never appear in
API responses.

## The compute network

The questions people actually ask before they trust a stranger's software
enough to install it on their own machine.

### What is a compute node?

A computer — usually a desktop PC — running the TokenBroker worker
application. Nodes contribute GPU/CPU time to fulfill API requests and earn
credits.

### How do I become a node?

Download the worker from your dashboard's Compute page and install it. It
runs in the tray, connects outbound only (no port forwarding), and starts
earning when idle capacity exists.

### Will the worker slow down my PC?

The worker is designed to use **idle** capacity and to respect your hardware.
You can pause it, and the network only assigns jobs the node can handle. The
status window shows live CPU/GPU usage so you can see exactly what it's doing.

### Is the worker safe? Is it spyware?

It's a visible tray application, not a hidden process. It shows redacted
prompts, job payments, timestamps, and resource use. It cannot read your
personal files, and it never displays who requested the job. Uninstalling is
one double-click. Workers are hash-verified before distribution, and modified
workers don't receive jobs.

### What does the worker see?

For normal jobs: a redacted prompt, the model name, and job metadata. For
private jobs, the payload is encoded and only decrypted in memory. Either
way, the node never sees the requester's identity.

### How much can I earn?

It depends on your hardware, uptime, and quality tier. The base rate is 10
credits per 1,000 real output tokens your node completes; what that's worth
in redeemable value ranges from $0.05 (basic tier) to $0.20 (premium tier)
per 1,000 tokens — see [pricing & credits](pricing-and-credits.md#credit-quality-tiers)
for the full breakdown. Don't expect a salary — think of it as covering your
own usage and supporting the network. There's also a daily earnings cap per
node so no single machine can dominate payouts.

### What is a "quality tier"?

The network grades nodes and pays premium/standard/basic credits. Higher
quality nodes accrue more value per credit. Your dashboard shows your tier
and score.

### Can I run the worker on a VM or a cloud server?

Virtualized workers are detected and rejected — the network wants real
hardware from real people, and VM farms are how credit systems get gamed.

### Does the worker need Ollama?

Yes — Ollama is the local inference engine the worker uses. The dashboard
links the installer, and signing into your free Ollama account lets the node
also serve large cloud models (with your quota tracked and reported).

### I have limited storage. What happens?

You declare how much storage you're contributing. Models the worker pulls for
the network are automatically evicted after 2 hours idle, so your disk
doesn't fill up.

### The worker seems idle. Why?

The network only assigns jobs a node can actually serve. If no matching jobs
are queued — or the node's cloud quota is exhausted — it waits. Idle nodes
still show as online and are first in line when work appears.

### Do I need a GPU to contribute anything at all?

No. Two GPU-free ways to contribute real capacity: enable Ollama Cloud (your
node relays to Ollama's own infrastructure, so your hardware doesn't run the
model), or add a free-tier external provider key (Google AI Studio today) —
your node calls that provider's API directly with your key. Either one turns
a GPU-less laptop or a phone into real network capacity.

### How is capacity from my hardware different from capacity from my API key?

Hardware capacity is measured directly from real completed jobs. A
provider-key's capacity is credited using a "known floor" the moment you add
it — a conservative baseline so a brand-new contribution counts immediately
instead of looking like zero. See [Tested & verified](tested-and-verified.md)
for exactly how that floor was derived for each provider, and why they're
labeled differently (one measured, one estimated).

## Mobile workers (Android & iOS)

### Can I really contribute compute from my phone?

Yes. A companion mobile app runs the same relay loop as the desktop worker —
heartbeat, claim a job, run it, report back — using a foreground service so
it keeps working with the screen off and the app swiped away. It's a real,
tested path: a live job was dispatched to and completed by a real phone
during development, with the desktop worker paused, calling Google's API
directly and returning a genuine response.

### Why can't my phone run local models like a desktop can?

Phones don't have the VRAM or the sustained thermal headroom for it, so
mobile workers only ever serve cloud-hosted and external-provider work
(Ollama Cloud, or a provider key you've added) — never a locally pulled
model. This is a deliberate scope limit, not a missing feature.

### How do I connect my phone?

Generate a one-time pairing code from your dashboard's Compute page, and
enter it in the app. The code carries your account's Ollama key (if you have
one) along automatically, so a phone setup never has to ask you to type an
API key on a phone keyboard.

### Can I run both a desktop and a phone worker on one account?

Yes — one of each. Two desktops or two phones on the same account is not
allowed (the one-worker-per-platform-per-account rule), but one of each kind
is exactly the setup the account-wide key system was built for.

## Referrals

### How do referrals work?

Every account has a personal referral code, visible on your dashboard. When
someone signs up with it and becomes a real active user — registers a worker
node, or completes a real payment, whichever happens first — **you both get
$5.00** credited automatically. See [pricing &
credits](pricing-and-credits.md#referrals-5-for-you-5-for-them) for the full
mechanics.

### Is there a limit to how many people I can refer?

No cap. Every genuinely new, independent member strengthens the network for
everyone already using it, not just for whoever referred them.

## Models

The catalog spans four very different serving mechanisms behind one
consistent naming scheme — here's how to read it.

### Why are some models marked "cloud"?

`-cloud`/`:cloud` models run on Ollama's hosted infrastructure rather than
the compute network. They're useful when no local node has the hardware for a
large model, or when you want maximum quality on any device.

### Can I request a model that isn't in the catalog?

Yes — requests are welcome. Popular Hugging Face models get imported, rebuilt
as native Ollama models, and published so any node can serve them.

### What's the difference between a model and its "cloud variant"?

The variant keeps the same personality/prompt behavior but runs on a
different base model (e.g. a creative persona on Gemma 4 vs. GLM 5). Same
character, different engine — different price and quality.

### Are embeddings supported?

Yes — embedding models like `all-minilm` and `nomic-embed-text` are available
through the embeddings endpoint.

### Does TokenBroker support structured outputs / JSON mode?

Not on every backend. Ollama Cloud specifically does not support
structured/JSON-schema-constrained outputs as of this writing — confirmed
directly against Ollama's own documentation, not assumed. If your workflow
needs guaranteed JSON, prompt for the exact shape explicitly and validate the
response yourself rather than relying on a `response_format` parameter
against a cloud model. See [Tested & verified](tested-and-verified.md).

### What's the biggest prompt I can send?

Requests are capped at 100,000 characters of raw request JSON, and
compute-network jobs cap output at 4,096 tokens per request. These aren't
arbitrary — they came from deliberately testing oversized requests and bulk
job submission to find real limits. Past either limit you get a clear, typed
error rather than a silent failure or a hang.

### Why did a model I used yesterday disappear or get renamed?

Upstream providers retire and rename models with little warning — this has
already happened at least twice with real models in this catalog. Rather
than trust a static reference list, new provider models are verified live
against the real API before being published. See [Tested &
verified](tested-and-verified.md#what-we-found-doesnt-work-yet) for two real
examples this caught.

## Infrastructure & uptime

### Is the service up 24/7?

That's the goal and the design: supervisors keep services alive, a watchdog
re-checks every few minutes, crashed processes restart automatically, and the
gateway recycles daily. Real-world outages still happen. The live [network
status page](https://tokenbroker.hopto.org/network) shows real-time capacity
and activity to anyone, and [Tested & verified](tested-and-verified.md)
documents real incidents and fixes rather than pretending they never
happened.

### How is the site exposed without port forwarding?

Outbound tunnels to Cloudflare's edge. The public domain
[tokenbroker.hopto.org](https://tokenbroker.hopto.org) resolves to the stable
edge URL, which always points at the current tunnel — so the name never goes
stale even when the tunnel rotates.

### Can I self-host or white-label TokenBroker?

Not today. The codebase is private for security reasons. If you're interested
in enterprise licensing, contact the operator through the dashboard.

## Misc

### Why is this repo documentation-only?

Releasing the full source would expose attack surface and secrets-handling
details for no user benefit. The docs explain how the service behaves, what
it protects, and how to use it — without revealing implementation.

### How do I contact support?

Through the dashboard's contact channel. For security issues, report
privately rather than publicly.

### Where do I find the terms of service and privacy policy?

Linked from the website footer. The short version: your prompts aren't
trained on, payments are real, abuse isn't tolerated, and the service is
provided as-is with best-effort availability.

## Didn't find your answer here?

- [A full breakdown of the system](system-breakdown.md) — every component,
  in depth, with diagrams.
- [Tested & verified](tested-and-verified.md) — real measurements, real
  limits, and honest caveats about what doesn't work yet.
- [Take back your compute](take-back-your-compute.md) — the why behind all
  of this.

*More questions? Ask through the dashboard — the FAQ grows with the network.*
