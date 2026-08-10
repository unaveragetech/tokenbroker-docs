# The compute network

TokenBroker doesn't rely on a single data center. It routes work to a network
of **compute nodes** — desktop PCs contributed by members like you — plus
cloud-hosted models. This section explains how the network works, how to join
it, and how you get paid for participating.

## Why a network?

Some models are small enough to run on a laptop. Others need serious GPUs. A
network lets the service offer both: a node with an RTX-class GPU can serve
bigger local models, a modest node can serve smaller ones, and everything can
fall back to cloud models when no node is available.

The more nodes join, the more capacity the network has — which keeps the
service available around the clock and reduces reliance on any single piece
of hardware.

## How nodes are chosen

Every request is matched to the **best available node** for that model:

1. A node that already has the model pulled and meets its requirements —
   fastest path.
2. A node with enough disk space to pull the model on demand.
3. A cloud-hosted model (e.g. Ollama Cloud) when the request is for a cloud
   model or no capable local node is online.

Nodes only ever receive work they can actually do — the gateway checks GPU
VRAM, system RAM, and storage before assigning anything. A node never gets a
job for a model it can't handle.

## Joining the network with the worker

The **TokenBroker Worker** is a small Windows application you download from
your dashboard. Highlights:

- **Easy install** — download the bundle, double-click, done. No command line.
- **Easy uninstall** — a single uninstaller removes everything.
- **Visible, not hidden** — it lives in the taskbar tray with a live status
  window showing uptime, requests per minute, CPU/GPU use, and what it's
  working on. It can minimize to the tray and run 24/7.
- **Transparent** — you see redacted prompts, compute cost, payment for each
  completed job, and timestamps.
- **Private by design** — the worker shows a **redacted** view of the prompt
  and never displays the requester's identity. It cannot see who you are
  serving.

The worker connects outbound only — **no ports are opened** on your router.

### What the worker needs

- Windows 10/11, 64-bit.
- Ollama installed (the worker uses it as the local inference engine).
- A GPU is a big advantage (more model families, bigger jobs), but not
  strictly required — smaller models run on CPU.
- Storage you're willing to contribute. Models the worker pulls for the
  network are evicted automatically after 2 hours idle, so storage stays
  available for your own use.

### Optional: connect your Ollama Cloud account

If you sign in to your free Ollama account in the worker, the node can take on
**cloud-hosted models** too — including large models your hardware could never
run. The worker tracks your cloud usage and reports how much remains, and it
stops accepting cloud work the moment your free quota is exhausted (it
resumes when the quota resets).

## How workers earn credits

Every completed, accepted job pays **credits** into your account. Quality
matters:

- The network grades completed work (response length, speed, variety,
  consistency).
- **Premium, standard, and basic** credit tiers exist. Well-behaved,
  well-performing nodes earn more valuable credits; low-quality nodes earn
  less valuable ones.
- Credits can be redeemed for API usage. They are **not** cash — 1 credit is
  not 1 cent. Their value depends on the quality tier and current network
  conditions.

### Anti-exploit protections

The network is built to resist gaming:

- **Minimum contribution** before inference is accepted (tiny jobs are
  rejected).
- **Duration and speed gates** — instant/fabricated completions are caught.
- **Diversity checks** — canned or repeated responses are flagged.
- **One worker per account**, bound to a hardware fingerprint and the IP it
  registered from.
- **VM detection** — virtualized workers are rejected.
- **Cloud quota honesty** — a node that exhausts its cloud quota stops
  advertising cloud models until it resets.

These protections exist so that real users get real work done, and credits
mean something.

## Worker dashboard

Your dashboard's **Compute** page shows:

- Your node's status, hardware fingerprint (never how it's derived), and
  quality score.
- Expected performance based on the RAM, CPU, and GPU detected.
- A collapsible transaction history of every job and its earnings.
- Rejected worker attempts (and why) for full transparency.
- The worker download, the Ollama install link, and instructions.

## Removing the worker

Uninstall is one double-click. The worker leaves no background services, and
your account keeps whatever credits you've earned.
