# Security & privacy

TokenBroker is built with a security-first posture. This page describes the
controls in plain language. The implementation itself is intentionally kept
private — obscurity is not security, but exposing attack surface details to
the public helps nobody.

## What we protect

- **Your account** — passwords are stored using strong, salted hashing; never
  in plaintext. Sessions use hashed, expiring tokens. Login attempts are
  rate-limited per IP to blunt brute force.
- **Your keys** — API keys are shown once at creation, stored only as hashes,
  revocable at any time, and can carry per-key daily spend caps. If you
  contribute an external provider key (e.g. Google AI Studio) as a worker
  operator, it's encrypted at rest the same way, never shown back to you in
  plaintext, and only ever used to serve jobs on your own account's behalf.
- **Your money** — billing uses holds: funds are reserved per request and
  released when the request settles. No double-charging, no silent
  overcharges.
- **Your data** — requests are not used for training. Transcripts are
  retained for billing/abuse purposes only, and provider identity is masked
  in responses.

## How the network protects users from nodes

This is the heart of the design:

- **The requester's identity is never shown to the node.** A worker sees a
  redacted prompt and a job id — not who asked.
- **Private jobs** can be encoded end-to-end: the hub encrypts the payload
  before it ever reaches a worker, only a node the operator has explicitly
  marked as trusted (and that has opted into private work) ever receives
  one, and it's decrypted **only in that worker's memory** — never written
  to disk, never logged. The response is re-encrypted the same way before it
  travels back. A worker with private jobs turned off never sees an
  encrypted payload at all; it only ever requests public work.
- **The response path is masked** — users never learn which node fulfilled
  their request, so nodes stay anonymous to the people they serve.
- **Honest boundary, stated plainly**: encoding protects against casual
  exposure — logs, disk scans, network sniffing — not against a determined
  machine operator inspecting their own process memory while a job runs.
  True end-to-end confidentiality against the operator themselves would
  need trusted hardware execution, which isn't built yet. Sensitive traffic
  is steered toward trusted, operator-run nodes for exactly this reason.

## How the service protects itself from nodes

- **Hardware fingerprints** — each worker is bound to its hardware and the IP
  it registered from (one worker per account).
- **VM detection** — virtualized workers are rejected.
- **Tamper evidence** — worker binaries are hashed before distribution;
  modified workers don't receive jobs.
- **Quality gates** — minimum work, duration, and diversity checks make
  credit farming impractical.

## Infrastructure

- **No inbound ports** — the operator's network exposes nothing; all traffic
  is outbound through tunnels and Cloudflare's edge.
- **Admin isolation** — administrative APIs are restricted to trusted
  networks and require separate remote-access keys.
- **Encryption at rest & in transit** — TLS everywhere on the public surface;
  secrets encrypted at rest.
- **Rate limiting** across auth, API, and compute endpoints.
- **Self-healing operation** — supervisors keep services alive, restart
  crashed processes, and recycle the gateway daily.

## What we don't do

- We don't train on your prompts.
- We don't require a GPU, a server, or port forwarding from you.
- We don't hide what the worker does — it's a visible tray application with a
  transparent status view, not a hidden background process.

## Reporting issues

Found a vulnerability? Report it through the dashboard contact channel rather
than posting it publicly, so it can be fixed before it's exploited.

## Related reading

- [Tested & verified](tested-and-verified.md) — a real networking security
  issue (TLS fingerprint blocking) we found on the mobile client and fixed.
- [Compute network](compute-network.md) — the anti-farming and anti-abuse
  mechanics in full.
- [FAQ](faq.md#the-compute-network) — quick answers about worker safety.
