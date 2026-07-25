# freellmapi setup (Windows)

Verified against freellmapi `server@0.2.1` on Node 22: dependencies install in
~30s, `tsc` build is clean, and the server boots, seeds its catalog, mints a
unified key, and rejects unauthenticated `/v1` calls.

## Scope: when to use it

freellmapi is limited to two cases. It is not a general-purpose backend.

1. **Simple foodrebel tasks.** High-volume, low-stakes calls where a weaker
   model is good enough: ingredient parsing, tag and category extraction,
   description drafts, embeddings for recipe search, test fixtures.
2. **Token exhaustion.** A fallback when the primary paid API is rate-limited
   or out of quota, to keep working rather than to save money by default.

Everything else stays on the paid API. In particular, keep it away from:

- Anything user-facing in production. It is single-user and localhost-bound,
  so it cannot serve end users, and the providers' terms forbid sharing the
  endpoint with other people.
- Nutrition, allergen, and dietary-restriction output. Weak models get facts
  wrong, and in a food app that is a safety problem, not a quality one.
- Agentic coding through `ANTHROPIC_BASE_URL`. Tool-calling reliability is the
  first thing that degrades on free-tier models.

Effective quality also drops through the day as the strongest models exhaust
their daily caps, resetting at UTC midnight. Route accordingly: batch the
simple work early if it matters.

## Option A — desktop installer (simplest)

Download the `.exe` from
[Releases](https://github.com/tashfeenahmed/freellmapi/releases/latest). No
account to create; the dashboard signs itself in locally. Grab the unified key
from the tray popover (**Copy Key**).

State lives in `%APPDATA%\FreeLLMAPI\` — `freeapi.db` (keys, models, settings)
and `config.json`. Copy that folder to migrate to another machine.

## Option B — Docker

```powershell
git clone https://github.com/tashfeenahmed/freellmapi.git
cd freellmapi

$Bytes = New-Object Byte[] 32
[Security.Cryptography.RandomNumberGenerator]::Create().GetBytes($Bytes)
$ENCRYPTION_KEY = -join ($Bytes | ForEach-Object { "{0:x2}" -f $_ })
"ENCRYPTION_KEY=$ENCRYPTION_KEY`nPORT=3001" | Out-File -Encoding utf8 .env

docker compose up -d
```

Dashboard at http://localhost:3001. Add provider keys on the **Keys** page,
reorder the **Fallback Chain**, and copy the unified key from the page header.

## Option C — from source

```powershell
npm install
npm run build
npm run dev
```

Building the *desktop* app from source additionally needs Visual Studio Build
Tools ("Desktop development with C++") and Python, to compile native SQLite for
Electron. The server alone needs neither.

## Binding

`HOST` defaults to `::` (all interfaces). The Docker path publishes only on
`127.0.0.1`, but running the server directly exposes it on your LAN. On an
untrusted network set `HOST=127.0.0.1` in `.env`.

## Pointing clients at it

Base URL `http://localhost:3001/v1`, the unified key as the bearer token, model
`auto` to let the router choose.

| Client | Configuration |
| --- | --- |
| OpenAI SDK | `base_url="http://localhost:3001/v1"`, `api_key=<unified key>` |
| Claude Code | `ANTHROPIC_BASE_URL=http://localhost:3001`, `ANTHROPIC_AUTH_TOKEN=<unified key>` |
| Cline / Roo | provider "OpenAI Compatible", same base URL and key |
| Aider | `OPENAI_API_BASE=http://localhost:3001/v1`, then `aider --model openai/auto` |

## Limits worth knowing before you depend on it

- No frontier models. The catalog tops out well below Opus/GPT-5 class.
- Effective quality drops through the day as the best models exhaust their daily
  caps, resetting at UTC midnight.
- Single-user, no multi-tenant auth. It cannot serve an app's end users.
- Cohere's terms forbid personal use; Google, NVIDIA, and GitHub Models carry
  evaluation-or-prototyping-only scopes. One account per provider, no reselling.
- Upstream free tiers change without notice. Prototype on it; ship on a paid API.
