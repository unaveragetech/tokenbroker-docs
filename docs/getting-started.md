# Getting started

From zero to your first AI response in about five minutes.

## 1. Create an account

Go to **[https://tokenbroker.hopto.org](https://tokenbroker.hopto.org)** and
sign up. You can use:

- **Email + password** — you'll receive a verification email, and you can
  reset your password anytime with a one-time code sent to your inbox.
- **Google** — one click, no password to remember.

New accounts start with a wallet. To use paid models you'll top up with a
card or Cash App (processed through Square), or buy a plan.

## 2. Create an API key

From the dashboard:

1. Open **API Keys**.
2. Click **Create key**.
3. Copy the key and store it somewhere safe. You can set a **daily spend cap**
   on each key, and you can revoke a key at any time.

## 3. Make your first request

The API is OpenAI-compatible. Point any OpenAI client at the service with your
key:

```bash
curl https://tokenbroker.hopto.org/v1/chat/completions \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-oss:20b-cloud",
    "messages": [{"role": "user", "content": "Say hello in one sentence."}]
  }'
```

Or in Python:

```python
import openai

client = openai.OpenAI(
    base_url="https://tokenbroker.hopto.org/v1",
    api_key="YOUR_API_KEY",
)

resp = client.chat.completions.create(
    model="gpt-oss:20b-cloud",
    messages=[{"role": "user", "content": "Say hello in one sentence."}],
)
print(resp.choices[0].message.content)
```

You can also stream responses, and the same key works for embeddings and text
completions.

## 4. Browse the catalog

The dashboard's **Models** page lists every available model with:

- A human-readable description.
- Parameter size and VRAM/disk requirements (for local models).
- Per-model input/output pricing.
- A ready-to-copy Python snippet with your key and the model pre-selected.

See the [models catalog](models-catalog.md) for what's available.

## 5. Try the playground

No API client handy? The **Playground** lets you chat with any model directly
in the browser, see the calculated cost of each exchange, and copy the exact
code for what you just did.

## 6. Check your usage

The dashboard shows wallet balance, today's spend, and per-key usage. The API
also exposes a usage endpoint so your own tooling can track it.

## What's next?

- Want to **earn** instead of spend? Install the worker — see
  [compute network](compute-network.md).
- Want to understand pricing? See [pricing & credits](pricing-and-credits.md).
- Have a question? The [FAQ](faq.md) probably answers it.
