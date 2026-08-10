# API reference

The API is OpenAI-compatible. If you've used OpenAI's API, you already know
how to use TokenBroker.

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
| 401 | Missing/invalid API key |
| 402 | Insufficient balance or spend cap reached |
| 404 | Unknown model or endpoint |
| 429 | Rate limit exceeded |
| 500 | Upstream failure (never leaks provider details) |

## Compute jobs via the API

Some models are served through the compute network. When you request one:

- The gateway assigns the job to a capable node (or a cloud fallback).
- You pay per token at the model's listed price.
- The response is returned exactly like any other completion — you never see
  which node did the work.

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
