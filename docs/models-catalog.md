# Models catalog

Most AI products give you one model, maybe two, and call it a feature.
TokenBroker's catalog is intentionally broad instead — hundreds of models
spanning tiny laptop-friendly checkpoints up to 120B-class hosted models,
because "the right model for this task" is a different answer for a chatbot
than for a coding assistant than for a document pipeline. Every model has a
description, its own pricing, and (for community-served models) its hardware
requirements, so choosing isn't guesswork.

**Where we're headed:** our goal is to support nearly every open model. The
catalog you see today is smaller than that goal on purpose — we re-evaluated
the list and only advertise what we've verified end to end — and it grows as
the network grows (more contributors, more contributed provider keys, more
sustainable access to large models). The live list is always on the
dashboard's Models page and in `GET /v1/models`.

## Where models come from

### 1. Hosted cloud models

Always-on models that don't depend on any contributor being online:

- **Ollama Cloud** — GPT-OSS 20B and 120B, Gemma 4 31B, and NVIDIA Nemotron 3
  (Nano, Super, Ultra).
- **Edge GPU cloud (the `cf-` prefix)** — models served on Cloudflare's edge
  GPU network, including GPT-OSS 20B/120B, Gemma 4 26B, Nemotron 3 120B,
  Llama 3.1/3.2/3.3 (including a 3.2 11B vision model) and Llama 4 Scout
  17B, Qwen3 30B, QwQ 32B, Qwen2.5-Coder 32B, a DeepSeek-R1 32B distill,
  Mistral Small 3.1 24B, and GLM-4.7 Flash.

Each model's listing says where it runs. Because these routes sit on
provider free tiers, they have daily limits — that's part of why the network
of contributed capacity exists (see [Capacity & the network](compute-network.md)).

### 2. The community compute network

Contributors' desktops (and, for cloud-relay work, phones) serve a library of
roughly 250 open-weights models — Llama, Qwen, Gemma, DeepSeek-R1 distills,
Mistral, specialist coders, OCR and SQL models, and more. A node pulls what it
has room for, runs the job, and releases the disk afterward. These models are
available **whenever a capable worker is online**; they're the long tail that
grows with the network, and the main way the catalog widens toward "nearly
every open model."

### 3. Contributed provider keys

Some models aren't hosted by us or run on any machine we know about — a
contributor's worker calls an external provider's API directly with their own
free-tier key. Google's fast Gemini model (`gemini-3.6-flash`) is the first of
these. Like the community library, it's available whenever a contributor who
holds that key has a worker online.

### 4. Specialists and select variants

Coding, OCR and SQL specialists, embedding models (`nomic-embed-text` is
served directly), and a small set of **uncensored** variants. "Uncensored"
means refusal training was removed; these models are still subject to the
terms of service and the law. The selection is deliberately small today and
labeled clearly.

## What's not here (yet)

We previously listed a handful of very large models — certain GLM, Kimi,
MiniMax, Mistral Large and 600B-class releases. After re-testing every route
we no longer advertise them: their upstream providers either retired them or
gate them behind paid plans our free-tier routes don't cover. We'd rather
show a shorter list that works than a longer one that doesn't. As access
becomes sustainable — through paid upstream plans, contributed capacity, or
open weights the network can serve itself — they're exactly the kind of model
we want to add back. See [Tested & verified](tested-and-verified.md) for how
we check.

## How much hardware a model actually needs

Local models scale with your GPU, and the relationship is roughly linear
until you run out of VRAM entirely and fall back to CPU:

![What your hardware is actually worth](../assets/charts/hardware-throughput.svg)

This is also why hosted models exist: a 120B-parameter model would need
dozens of gigabytes of VRAM no consumer card has — running it on a hosted
route instead means a laptop with no GPU at all can still request it and get a
real answer.

## Model naming

Model names look like:

- `gpt-oss:20b-cloud` — hosted on Ollama Cloud.
- `cf-llama-3.3-70b` — hosted on the edge GPU cloud (`cf-` prefix).
- `deepseek-r1:7b` — community library; served by a capable worker.
- `gemini-3.6-flash` — served with a contributed provider key.

`-cloud` / `:cloud` and `cf-` names are hosted; other names run on the
community compute network.

## How to choose

| You want... | Try... |
| --- | --- |
| Cheap, fast chat | `cf-llama-3.2-3b`, `cf-gemma-2b` |
| Strong reasoning | `cf-qwq-32b`, `cf-deepseek-r1-32b`, `gpt-oss:120b-cloud` |
| Coding help | `cf-qwen2.5-coder-32b` |
| Maximum quality | `gpt-oss:120b-cloud`, `cf-nemotron-3-120b`, `cf-llama-3.3-70b` |
| Long context, fast | `gemini-3.6-flash` (community-served) |
| Embeddings | `nomic-embed-text` |
| OCR / document extraction | `deepseek-ocr`, `glm-ocr` (community library) |
| Natural language → SQL | `sqlcoder`, `duckdb-nsql` (community library) |
| Fewer refusals | A select uncensored variant, e.g. `wizardlm-uncensored` (community library) |

## Requesting new models

The catalog is community-driven. If there's a model you want, ask — the
network regularly imports proven models from Hugging Face, rebuilds them as
native Ollama models, and publishes them so any node can serve them.

## Related reading

- [Tested & verified](tested-and-verified.md) — how we check that a listed
  model really answers, and real model-name retirements we caught by testing
  live instead of trusting a reference list.
- [API reference](api-reference.md) — how to list and call models
  programmatically.
- [Compute network](compute-network.md) — how a model actually gets served.
