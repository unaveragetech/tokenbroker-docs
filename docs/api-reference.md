# API reference

The API is OpenAI-compatible on purpose: if you've used OpenAI's API, or any
of the dozen tools built to speak its dialect, you already know how to use
TokenBroker. Point the base URL here, use your TokenBroker key, and
everything else — SDKs, streaming, error shapes — behaves the way you
already expect.

## Base URL

```
https://tokenbroker.hopto.org/v1
```

## Authentication

Send your API key as a Bearer token:

```http
Authorization: Bearer YOUR_API_KEY
```

Create and manage keys from the dashboard. Keys are hashed at rest, can be
revoked instantly, and can carry daily spend caps.

## Endpoints

### `POST /v1/chat/completions`

Chat completions, streaming supported.

```bash
curl https://tokenbroker.hopto.org/v1/chat/completions \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-oss:20b-cloud",
    "messages": [
      {"role": "system", "content": "You are a concise assistant."},
      {"role": "user", "content": "Explain APIs in one sentence."}
    ],
    "stream": false
  }'
```

Set `"stream": true` to get Server-Sent Events, exactly like OpenAI's own
streaming format — each chunk is a `data: {...}` line, terminated by
`data: [DONE]`. Add `"stream_options": {"include_usage": true}` to receive a
final chunk with token usage once the stream completes.

```bash
curl https://tokenbroker.hopto.org/v1/chat/completions \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-oss:20b-cloud",
    "messages": [{"role": "user", "content": "Count to five."}],
    "stream": true
  }'
```

**A real, tested caveat:** structured/JSON-schema-constrained outputs
(`response_format: {"type": "json_schema", ...}`) are not supported by every
backend — Ollama Cloud specifically does not support them as of this
writing, confirmed against Ollama's own documentation rather than assumed.
If you need guaranteed JSON, prompt for it explicitly (e.g. "respond with
only a JSON object matching this shape: ...") and validate the response on
your side. See [Tested & verified](tested-and-verified.md) for more of these.

### `POST /v1/completions`

Legacy text completions.

### `POST /v1/embeddings`

Embedding models (e.g. `all-minilm`, `nomic-embed-text`).

### `GET /v1/models`

List available models.

### `GET /v1/usage`

Your balance and today's usage.

## Errors

Errors use OpenAI-style shapes:

```json
{
  "error": {
    "message": "Insufficient balance.",
    "type": "billing_error",
    "code": "insufficient_balance"
  }
}
```

Common status codes:

| Code | Meaning |
| --- | --- |
| 200 | Success |
| 400 | Malformed request, unknown model, or payload too large |
| 401 | Missing/invalid API key |
| 402 | Insufficient balance, spend cap reached, or a minimum-balance requirement not met |
| 404 | Unknown model or endpoint |
| 429 | Rate limit (per-key RPM/TPM, or daily spend cap) exceeded |
| 503 | No capable endpoint/worker available right now, or the compute queue is full |
| 500 | Upstream failure (never leaks provider details) |

## Real, tested limits

These aren't theoretical — they came from deliberately testing the edges of
the system (oversized prompts, request bursts, bulk job submission) rather
than being guessed at:

| Limit | Value |
| --- | --- |
| Request payload size | 100,000 characters of raw JSON |
| Output tokens per compute-network job | 4,096 max |

A request past either limit gets a clear, typed error (`payload_too_large`)
rather than an ambiguous failure. There's also a queue-depth safeguard that
returns a fast `503 compute_queue_full` during an unexpected spike rather
than letting the queue grow without bound — deliberately not detailed
further here. See [Tested & verified](tested-and-verified.md) for the story
behind these limits, including a real quality-gate rejection captured
during testing.

## Compute jobs via the API

Some models are served through the compute network. When you request one:

- The gateway assigns the job to a capable node (or a cloud fallback), in a
  strict order of preference — see the [dispatch tiers
  diagram](compute-network.md#how-nodes-are-chosen).
- You pay per token at the model's listed price.
- The response is returned exactly like any other completion — you never see
  which node did the work.
- If literally no node or cloud fallback can serve the model right now, you
  get a fast `503` instead of a request that hangs waiting for something
  that was never going to happen.

## SDKs & tools

Any OpenAI-compatible client works by overriding the base URL and key:

```python
import openai

client = openai.OpenAI(
    base_url="https://tokenbroker.hopto.org/v1",
    api_key="YOUR_API_KEY",
)
```

```javascript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://tokenbroker.hopto.org/v1",
  apiKey: "YOUR_API_KEY",
});
```

## Related reading

- [Getting started](getting-started.md) — the shortest path to a first request.
- [Models catalog](models-catalog.md) — what you can put in `"model"`.
- [Tested & verified](tested-and-verified.md) — real limits and a real
  provider caveat (structured outputs), tested rather than assumed.
- [Security & privacy](security-and-privacy.md) — what happens to your
  prompts and how keys are protected.
