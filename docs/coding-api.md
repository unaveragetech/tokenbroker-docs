# Coding API: Anthropic Messages + safe file editing

TokenBroker speaks **two** wire formats on the same key, the same wallet and the
same model catalog:

| Format | Endpoint | Use it for |
| --- | --- | --- |
| OpenAI | `POST /v1/chat/completions` | Existing OpenAI SDKs and tools |
| Anthropic Messages | `POST /v1/messages` | Anthropic SDKs, agent/coding clients, file editing |

Every chat model in the catalog works on both. The catalog shows what each
model supports: **Text chat**, **Tools**, **File edit**, **Vision** (see
[Models catalog](models-catalog.md)).

## Quick start

```bash
curl https://tokenbroker.org/v1/messages \
  -H "x-api-key: $TOKENBROKER_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{
    "model": "gpt-oss:120b-cloud",
    "max_tokens": 512,
    "messages": [{"role": "user", "content": "Say hi"}]
  }'
```

`Authorization: Bearer tb_live_...` works too. With the Anthropic Python SDK:

```python
import anthropic
client = anthropic.Anthropic(base_url="https://tokenbroker.org", api_key="tb_live_...")
msg = client.messages.create(model="gpt-oss:120b-cloud", max_tokens=512,
                             messages=[{"role": "user", "content": "Say hi"}])
```

Any client that lets you set the base URL, API key and model name can use it
(set the base URL to `https://tokenbroker.org`; the client appends `/v1/messages`).

## Use it with Claude Code

Claude Code speaks the Anthropic Messages API, so it works against TokenBroker
with environment variables only (no proxy):

```bash
export ANTHROPIC_BASE_URL=https://tokenbroker.org
export ANTHROPIC_AUTH_TOKEN=tb_live_...        # your TokenBroker key
export ANTHROPIC_API_KEY=""                     # must be empty so the token above is used
export ANTHROPIC_MODEL=glm-5.3                  # any "Agent-ready" model
export ANTHROPIC_SMALL_FAST_MODEL=glm-5.3-flash # used for background tasks
export CLAUDE_CODE_MAX_OUTPUT_TOKENS=8192
claude
```

Things that decide whether it works well:

- **Pick an Agent-ready model.** Claude Code sends roughly 25-35k tokens of
  system prompt and tool definitions on *every* turn. Models with a small
  context window cannot even fit that, so the catalog marks models with a
  64k+ window and tool support as **Agent-ready** (filter for it on the Models
  page, or check `features.agent_ready` in `GET /v1/models`). Ollama Cloud
  models such as `glm-5.3`, `kimi-k2.7-code` and `gpt-oss:120b-cloud` have
  native tool calling and large windows and were verified end to end.
- **Context errors are recoverable.** If a prompt does not fit, the API returns
  `400 invalid_request_error` with `prompt is too long: N tokens > M maximum`,
  which is exactly what Claude Code looks for to compact the conversation and
  retry.
- **Cloudflare `cf-*` models** run on Workers AI. On a free Workers AI account
  they have a daily allocation (10,000 neurons) and some models are paid-only;
  when that is exhausted the gateway fails over to the same model on Ollama
  Cloud where one exists, and otherwise returns a `429`/`503` that clients
  retry. Small-context `cf-*` models (for example 24k-32k windows) are not
  Agent-ready.
- **Rate limits.** Each turn can be 30k+ tokens, so a key with a 200,000
  tokens/minute limit allows roughly six turns a minute. Raise the key's token
  limit if long sessions hit `429`.
- **Tool calls on models without native tools** are parsed by the gateway,
  including the raw `<|call|>` text some gpt-oss hosts return, and are always
  delivered to the client as normal `tool_use` blocks.

Want Claude Code to *delegate* work to TokenBroker models instead of running on
them? See [Claude as an orchestrator](claude-orchestrator.md).

## Tools work on every model

Send `tools` (Anthropic `input_schema` format) and handle `tool_use` /
`tool_result` blocks as usual.

- Models with **native** tool calling pass your tools straight through.
- Models without it use the gateway's **emulated** protocol: tools are described
  in the prompt, the reply is parsed, validated and returned as ordinary
  `tool_use` blocks. Your client cannot tell the difference. If the model
  produces a malformed call, the gateway asks it to fix it (up to 2 internal
  retries) before you see anything.

`GET /v1/models` reports each model's `features` and `tool_mode`
(`native` or `emulated`).

## Hashed whole-file editing

Plain chat resends every file every turn, makes your client guess how to apply
free-text edits, and cannot tell if a file changed underneath the model.
TokenBroker's file protocol fixes all three by content-addressing files.

**A file is its path plus the sha256 of its canonical text** (UTF-8, no BOM, LF
line endings). Send full content once; after that refer to it by hash.

### 1. Send files as `file` blocks

```json
{"type":"file","path":"src/app.js","sha256":"9f2c…","content":"…full text…"}
{"type":"file","path":"src/app.js","sha256":"9f2c…"}
```

The second form references a file the gateway already holds (cached per
account for 24 h, rolling). If it is not cached you get
`409 file_not_cached` with the path and hash; resend with `content`.
Only the **newest** version of each path is shown to the model in full, which
keeps long edit sessions cheap.

### 2. Give the model the file tools

Fetch them from `GET /v1/file-tools`, or set `"tb": {"file_tools": true}` on the
request (they are also added automatically whenever the request contains
`file` blocks).

| Tool | Purpose |
| --- | --- |
| `read_file {path}` | Ask the client to send a file |
| `edit_file {path, base_sha256, edits[{search, replace}]}` | Targeted search/replace edits (default for larger files) |
| `write_file {path, content, base_sha256?}` | Whole file; without `base_sha256` it is create-only |

### 3. The gateway applies and verifies the edit

Before a `tool_use` reaches you, the gateway validates the path, checks that the
model only touches files and hashes it was actually shown, applies the edits to
that exact version (exact match first, then whitespace-tolerant; ambiguous or
missing matches are fed back to the model, never guessed) and hashes the result.
You receive:

```json
{"type":"tool_use","name":"edit_file","input":{
  "path":"src/app.js",
  "base_sha256":"9f2c…", "result_sha256":"41ab…",
  "edits":[{"search":"…","replace":"…"}],
  "content":"…full new file (LF)…"}}
```

### 4. The client verifies, writes, and reports the new hash

1. Re-hash the file on disk. If it is not `base_sha256` → **do not write**; reply
   with an error `tool_result` containing the current file (`stale_base`).
2. Check that `content` hashes to `result_sha256` (else `hash_mismatch`).
3. Write atomically, restoring the file's original line endings and BOM, and
   re-check the hash.
4. Reply with a `tool_result` containing the new hash and a `file` block that
   references it **by hash only**. No re-upload on the next turn.

The reference client (`coding-sdk/` in the TokenBroker source: `TbFiles`, plus a
ready-made `codingSession` agent loop) does all of this.

## Safety

- **No path traversal.** Paths must be project-relative. The gateway rejects
  `..`, absolute and drive-letter paths, UNC paths, NUL/control characters,
  percent-encoded and Unicode-lookalike tricks, Windows device names, and
  anything under `.git`, `.ssh`, `.aws`, `.kube`, plus `.env*` and key files.
  The reference client independently re-validates, confines the resolved path to
  the project root, and refuses symlink escapes and writes through symlinks.
- **The model can only edit what it was shown.** Unknown paths and unknown or
  stale hashes are rejected and fed back to it.
- **Truncated output never becomes an edit.** If the model is cut off mid-call
  you get `stop_reason: "max_tokens"` and no `tool_use`.
- **Hashes are tenant-scoped.** Another account's hash and a nonexistent hash
  produce the identical `file_not_cached` response, so hashes cannot be probed.
- **Your code is not logged.** Request transcripts keep only file paths and
  short hashes, never contents.
- **Private compute jobs stay private.** Coding turns served by the community
  compute network travel as ordinary Hiro-encrypted private jobs; the stored
  job holds no plaintext source.

## Errors

`/v1/messages*` returns Anthropic-shaped errors for every failure (including auth
and rate limits):

```json
{"type":"error","error":{"type":"invalid_request_error","message":"…","code":"file_not_cached"}}
```

| HTTP | `error.type` | Typical `code` |
| --- | --- | --- |
| 400 | `invalid_request_error` | `invalid_request`, `model_not_found`, `model_not_chat`, `hash_mismatch`, `path_traversal`, `protected_path`, `binary_file` |
| 401 | `authentication_error` | `invalid_api_key` |
| 402 | `billing_error` | `insufficient_balance`, `minimum_balance_required` |
| 409 | `invalid_request_error` | `file_not_cached` (resend with `content`) |
| 413 | `request_too_large` | `file_too_large`, `body_too_large` |
| 429 | `rate_limit_error` | `rate_limit_exceeded` |
| 502/503 | `api_error` | `invalid_tool_call`, `no_endpoint_available`, `upstream_error` |

Error messages never contain provider names, upstream URLs, internal paths or
other accounts' data. Streaming requests get a real HTTP status for failures that
happen before generation starts (auth, balance, limits).

## Limits and behaviour worth knowing

- Files: text only, up to 2 MB each; 100 MB cached per account.
- `stream: true` returns the standard Anthropic event sequence. The turn is
  generated and validated first, then replayed (with `ping` events while waiting).
  Token-by-token streaming is not available on this endpoint.
- Internal retries are real, billed requests. See the `X-TokenBroker-Attempts`
  and `X-TokenBroker-Cost-Cents` response headers.
- `/v1/chat/completions` with `tools` on a model without native tool support is
  served through the same emulation.

