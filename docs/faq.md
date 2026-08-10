# Frequently asked questions

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
3. Community personas published as cloud variants.

There is no hidden rebranding: each model's listing says where it runs.

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

It depends on your hardware, uptime, and quality tier. Premium nodes that run
reliably earn more valuable credits. Don't expect a salary — think of it as
covering your own usage and supporting the network.

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

## Models

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

## Infrastructure & uptime

### Is the service up 24/7?

That's the goal and the design: supervisors keep services alive, a watchdog
re-checks every few minutes, crashed processes restart automatically, and the
gateway recycles daily. Real-world outages still happen — that's why a public
status page is on the roadmap.

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

*More questions? Ask through the dashboard — the FAQ grows with the network.*
