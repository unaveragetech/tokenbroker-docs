# Models catalog

TokenBroker's catalog is intentionally broad. Every model has a description,
its own pricing, and (for local models) its hardware requirements, so you can
pick the right tool for the job.

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
