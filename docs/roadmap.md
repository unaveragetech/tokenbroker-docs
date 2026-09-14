# Roadmap

TokenBroker is a living service. Here's the direction of travel — in rough
order of priority:

## Just shipped

- Mobile workers (Android & iOS) — contribute compute from a phone, not just
  a desktop.
- External cloud provider support (Google AI Studio) — a worker can now
  contribute a free-tier API key it holds personally as a second, independent
  capacity pool alongside Ollama Cloud.
- Live network status page at [/network](https://tokenbroker.hopto.org/network)
  — real-time aggregate capacity, visible to everyone.

## Now

- Expanding the models catalog from Hugging Face (distills, uncensored
  variants, specialized models).
- Growing the cloud model family coverage for custom personas.
- Improving worker onboarding and earnings transparency.
- More external cloud providers as verified free-tier keys become available.
- A real measured probe for Google AI Studio's throughput, the same
  deliberately bounded method already used for Ollama Cloud — see [Tested &
  verified](tested-and-verified.md) for why the current figure is still
  labeled an estimate.

## Next

- Bigger-model splitting across multiple GPUs for large jobs.
- More worker telemetry (uptime graphs, request-rate overlays, storage pool
  health).
- Refined credit value model tied to real network economics.
- Mobile-friendly dashboard polish.

## Later

- A compute marketplace with explicit supply/demand pricing.
- Team/organization accounts with shared billing.
- More payment methods and automated invoicing.

The roadmap is a conversation, not a contract. If there's a feature you need,
tell us — the network is built by its members.
