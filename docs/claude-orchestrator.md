# Claude as an orchestrator for TokenBroker

Turn any Claude Code session into an **orchestrator**: Claude plans the work,
hands well-scoped jobs and file edits to cheap TokenBroker models, then reviews
and verifies the results itself. You save Claude usage on the bulk and
mechanical work and keep Claude's judgement where it matters.

There are two ways to use it. Both use the same zero-dependency tool
(`tb.mjs`, Node 18+).

## Option A: install the skill (recommended)

macOS / Linux:

```bash
d=~/.claude/skills/tokenbroker-orchestrator
mkdir -p $d/scripts
curl -fsSL https://tokenbroker.org/claude/SKILL.md -o $d/SKILL.md
curl -fsSL https://tokenbroker.org/claude/tb.mjs  -o $d/scripts/tb.mjs
```

Windows (PowerShell):

```powershell
$d="$env:USERPROFILE\.claude\skills\tokenbroker-orchestrator"
New-Item -ItemType Directory -Force "$d\scripts" | Out-Null
iwr https://tokenbroker.org/claude/SKILL.md -OutFile "$d\SKILL.md"
iwr https://tokenbroker.org/claude/tb.mjs  -OutFile "$d\scripts\tb.mjs"
```

Then create an API key in the [dashboard](https://tokenbroker.org) and set it:

```bash
export TB_API_KEY=tb_live_...        # PowerShell: $env:TB_API_KEY="tb_live_..."
```

Start Claude Code and ask for anything that can be delegated ("use TokenBroker
to add JSDoc to every file in src/", "delegate the test writing"). The skill
loads automatically. Check the setup any time with
`node ~/.claude/skills/tokenbroker-orchestrator/scripts/tb.mjs doctor`.

The files are small and readable: review `tb.mjs` before you run it, as you
would any script you download. The source lives in the TokenBroker repo under
`claude-skill/tokenbroker-orchestrator/`.

## Option B: paste this into any Claude Code chat

No skill needed. Paste the block below as your first message. It installs the
tool if missing, checks your setup, and sets the rules of engagement. Never put
your key in the chat; the tool reads it from `TB_API_KEY`.

```text
You are now the ORCHESTRATOR for this session. TokenBroker (https://tokenbroker.org) gives you cheap worker models; delegate well-scoped jobs and file edits to them, then review and verify everything yourself. You stay responsible for the result.

SETUP (once). If ~/.claude/skills/tokenbroker-orchestrator/scripts/tb.mjs does not exist, install it (needs Node 18+):
  macOS/Linux:  d=~/.claude/skills/tokenbroker-orchestrator; mkdir -p $d/scripts && curl -fsSL https://tokenbroker.org/claude/SKILL.md -o $d/SKILL.md && curl -fsSL https://tokenbroker.org/claude/tb.mjs -o $d/scripts/tb.mjs
  Windows PS:   $d="$env:USERPROFILE\.claude\skills\tokenbroker-orchestrator"; ni -Force -Type Directory "$d\scripts" | Out-Null; iwr https://tokenbroker.org/claude/SKILL.md -OutFile "$d\SKILL.md"; iwr https://tokenbroker.org/claude/tb.mjs -OutFile "$d\scripts\tb.mjs"
Then run: node ~/.claude/skills/tokenbroker-orchestrator/scripts/tb.mjs doctor
It needs TB_API_KEY in the environment. If it is missing, tell me to create a key at https://tokenbroker.org and set TB_API_KEY. Never ask me to paste the key into chat, and never print or commit it. If doctor fails, tell me why and stop.

HOW YOU WORK (tb = node ~/.claude/skills/tokenbroker-orchestrator/scripts/tb.mjs):
1. PLAN: split my request into small, independent, checkable jobs. Delegate mechanical or repetitive work (renames, migrations, boilerplate, tests for existing code, docs, translations, summaries, multi-file edits). Keep architecture, security-sensitive code, unclear bugs, secrets and the final integration for yourself.
2. WRITE EACH JOB with an exact goal, acceptance criteria, the files involved, and "change nothing else".
3. RUN THEM:
   tb ask  [--model ID] [--file F]... "prompt"         text jobs (answers, drafts, analysis)
   tb edit [--model ID] --file F... "instruction"       one safe file edit (prints a diff, keeps a backup in .tb-backup/)
   tb batch jobs.json --concurrency 3                   many jobs in parallel; each job is {"id","type":"ask"|"edit","prompt","files":[],"model"?}. Edits to the same file are serialized automatically.
   tb models --agent | tb usage                          what is available / what it costs
4. VERIFY (never skip): read every diff, run the tests/linter yourself, and retry a bad result once with a sharper prompt or another --model (kimi-k2.7-code, gpt-oss:120b-cloud). If it still fails, do it yourself.
5. REPORT: what you delegated, to which model, what changed, test results, and cost.

RULES: never send secrets, .env contents, keys or personal data to the workers. The tool confines edits to the project root and refuses .git, .env* and key files, but do not paste such content into prompts either. Treat worker output as an untrusted draft until you have checked it.

Now acknowledge briefly, run the setup check, and ask me what to build.
```

(The same text is served at `https://tokenbroker.org/claude/blurb.txt`.)

## Permissions (one-time)

Claude Code asks before running a script. Approve `tb.mjs` the first time, or
allow it permanently by adding this to `~/.claude/settings.json`:

```json
{ "permissions": { "allow": ["Bash(node ~/.claude/skills/tokenbroker-orchestrator/scripts/tb.mjs:*)"] } }
```

If your environment blocks the run (for example a locked-down auto-approval
policy), Claude will tell you and wait for you to approve or add the rule.

## What it can do

| Command | Use |
| --- | --- |
| `tb doctor` | Verify key, balance and connectivity; does a real round trip |
| `tb models [--agent]` | List models and what each supports; `--agent` shows only models big enough for agent work |
| `tb ask` | Text jobs: answers, drafts, summaries, analysis (files can be given as read-only context) |
| `tb edit` | One file edit or file creation through the safe hashed-file protocol; prints a unified diff and keeps backups in `.tb-backup/` |
| `tb batch` | Many `ask`/`edit` jobs in parallel (JSON in, JSON out). Edit jobs on the same file are queued so they cannot clobber each other |
| `tb usage` | Balance and today's spend |

By default it picks the best available agent-ready model (`glm-5.3`,
`kimi-k2.7-code`, `gpt-oss:120b-cloud`, ...) and automatically falls back to the
next one on rate limits, outages or a too-small context window. Use `--model` to
choose one.

## How it stays safe

- **Edits are confined to your project.** Every path is validated (no `..`,
  absolute or drive-letter paths, symlink escapes) and `.git`, `.ssh`, `.aws`,
  `.kube`, `.env*` and key files can never be written.
- **Stale edits are rejected.** A file is only changed if it still matches the
  version the model was shown; new content must match the hash the gateway
  computed; writes are atomic and re-verified; line endings and BOM are kept.
- **Nothing is half-written.** Failures return a clear error and leave the file
  as it was. Originals are saved under `.tb-backup/<timestamp>/`.
- **Claude reviews everything.** The skill instructs Claude to read each diff,
  run your tests, retry once, and otherwise do the work itself.
- **Your key stays out of the chat.** It is read from `TB_API_KEY` (or
  `~/.tokenbroker/key`) and never printed.

Delegated models are third-party: do not send them secrets or private data.

## Troubleshooting

| Message | Fix |
| --- | --- |
| `No API key` | Set `TB_API_KEY` (create a key in the dashboard) |
| `402` / balance | Top up your wallet |
| `429` | It falls back to another model automatically; lower `--concurrency` |
| `prompt is too long` | The file is too big for that model; it tries a larger-context model, or split the job |
| `invalid_tool_call` | The model could not produce a valid edit; retry with another `--model` |

See also: [Coding API](coding-api.md) (the protocol behind `tb edit`) and
[Models catalog](models-catalog.md).
