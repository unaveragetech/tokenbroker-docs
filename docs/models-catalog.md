# Models catalog

Most AI products give you one model, maybe two, and call it a feature.
TokenBroker's catalog is intentionally broad instead — hundreds of models
spanning tiny laptop-friendly checkpoints to 600B-class reasoning models,
because "the right model for this task" is a different answer for a chatbot
than for a coding assistant than for a creative-writing tool. Every model
has a description, its own pricing, and (for local models) its hardware
requirements, so choosing isn't guesswork.

The catalog isn't fixed — it grows as the network grows. Every new provider
integration, every self-hosted model a contributor's node can run, and every
community-imported model adds to the same list, kept live and up to date on
the dashboard's Models page.

## Where models come from

### 1. Local open-weights models

Classic, proven open models (Llama, Qwen, Gemma, DeepSeek, Mistral, and more)
run on compute nodes that have the right hardware. These are the workhorses of
the network — cheap, fast, and always available when a capable node is
online.

### 2. Distilled and compact variants

Distilled versions of big models (for example, DeepSeek-R1 at 1.5B/7B/8B,
Llama 3.2 at 1B/3B, Qwen2.5-Coder at 1.5B/7B, Gemma 3 at 1B/4B) bring
surprising capability to hardware with only 8 GB of VRAM or less. The catalog
is continuously expanded with these proven variants.

### 3. Uncensored / abliterated variants

Some users want models without refusals. The catalog includes **abliterated,
uncensored, and "heretic" variants** (Qwen, Gemma, Llama, Mythos, and more),
clearly labeled so you know exactly what you're choosing. Same tooling, same
API — different behavior.

### 4. Native cloud models

Large models hosted on Ollama Cloud — GPT-OSS 20B/120B, Gemma 4 31B, GLM 5.x,
DeepSeek V4 Flash, Kimi K2.6/K2.7 Code, MiniMax M2.7/M3, Mistral Large 3 675B,
Nemotron 3 (Nano/Super/Ultra), Qwen3.5 397B, and more. These run on
Ollama's infrastructure and are served through the network, so even a laptop
can request a 600B-class model.

### 5. Custom model cloud variants

Community-built personas (simulation companions, creative writers, coding
specialists, business tools) are published as **cloud variants on multiple
bases** — so the same personality is available on light, medium, and heavy
cloud models depending on your quality/cost preference.

### 6. External provider models

Some catalog models aren't hosted on Ollama's infrastructure at all — they're
served by a compute-network worker calling an external provider's own API
directly with a contributor's personal free-tier key. Google's fast Gemini
model (`gemini-3.6-flash`) is the first of these, contributed this way rather
than run on any single machine.

## How much hardware a model actually needs

Local models scale with your GPU, and the relationship is roughly linear
until you run out of VRAM entirely and fall back to CPU:

![What your hardware is actually worth](../assets/charts/hardware-throughput.svg)

This is also why native cloud models exist: a model like a 120B-parameter
GPT-OSS variant would need dozens of gigabytes of VRAM no consumer card has —
running it on Ollama's infrastructure instead means a laptop with no GPU at
all can still request it and get a real answer.

## Model naming

Model names look like:

- `gpt-oss:20b-cloud` — native cloud model.
- `deepseek-r1:1.5b` — local open-weights model.
- `qwen2.5-7b-abliterated` — uncensored variant.
- `nova-sim-gemma4-cloud` — a custom persona variant running on the Gemma 4
  cloud base.

`-cloud` / `:cloud` names are cloud-hosted; everything else runs on the
compute network.

## How to choose

| You want... | Pick... |
| --- | --- |
| Cheap, fast, reliable chat | A small local model (1B–4B) |
| Strong reasoning on modest GPU | DeepSeek-R1 distill (1.5B/7B/8B) |
| Coding help | Qwen2.5-Coder, Shirdel Coder, Kimi K2.7 Code |
| Creative writing / roleplay | Parable, FableForge, Mythos, Nova personas |
| No refusals | Any model labeled uncensored/abliterated |
| Maximum quality, any hardware | A native cloud model (Gemma 4, GPT-OSS 120B, GLM 5.2...) |
| OCR / document extraction | olmOCR 7B |
| Natural language → SQL | Qwen3 4B text2SQL |

## Requesting new models

The catalog is community-driven. If there's a model you want, ask — the
network regularly imports proven models from Hugging Face, rebuilds them as
native Ollama models, and publishes them so any node can serve them.

## Related reading

- [Tested & verified](tested-and-verified.md) — real model-name retirements
  we caught by testing live instead of trusting a reference list, and what
  structured-output support actually looks like today.
- [API reference](api-reference.md) — how to list and call models
  programmatically.
- [Compute network](compute-network.md) — how a model actually gets served.
