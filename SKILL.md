---
name: whereamiburningtokens
description: "Reads OpenClaw session data and shows exactly where tokens and money are going — by session type, across all sessions. Surfaces sinkholes like runaway logging sessions, expensive crons, or bloated sub-agents. Use when the user asks: 'where are my tokens going?', 'what's my biggest token cost?', 'token breakdown', 'where am I burning tokens this week/today/all-time', 'am I burning money on background tasks?', or 'token audit'."
---

# whereamiburningtokens

Reads `~/.openclaw/agents/main/sessions/sessions.json` and shows a token + cost breakdown by session category.

## Data source

```
~/.openclaw/agents/main/sessions/sessions.json
```

Each session entry has:
- `totalTokens`, `inputTokens`, `outputTokens`, `cacheRead`, `cacheWrite`
- `estimatedCostUsd`
- `model`, `modelProvider`
- `startedAt`, `updatedAt` (Unix ms timestamps)

Session keys: `agent:main:<category>:<optional-id>`

Categories (3rd segment of key):
- `main` — interactive chat
- `cron` — heartbeats + scheduled tasks
- `subagent` — spawned sub-agents
- `paperclip` — Paperclip logging (if installed)
- anything else — plugins, integrations

## Time windows

Detect from user's phrasing:
- "this week" / default → last 7 days
- "today" → last 24 hours
- "this month" → last 30 days
- "all time" / "all-time" → no filter

Filter by `updatedAt >= cutoff_ms`.

## Steps

1. Read and parse sessions.json. If missing, say so and stop.

2. Filter by time window based on user's phrasing.

3. Group by category (3rd `:` segment of key). Sum `totalTokens` and `estimatedCostUsd`.

4. Sort by tokens descending. Calculate % of total for both tokens and cost.

5. Flag anomalies:
   - **⚠️ SINKHOLE**: category >40% tokens but <15% cost (cheap model, high volume — likely a logging/cron drain)
   - **⚠️ EXPENSIVE**: non-main category >35% cost but <15% tokens (expensive model, few calls — check model config)

6. Output the table and 1-2 insight lines.

## Output format

```
🔥 WHERE AM I BURNING TOKENS? (last 7 days)
66 sessions | 2.8M tokens | $100.65 est.

Category        Sess    Tokens    Tok%     Cost   Cost%
──────────────────────────────────────────────────────────
paperclip         26      997k   36.1%  $  5.79    5.8%  ⚠️ SINKHOLE
subagent          22      795k   28.8%  $  9.49    9.4%
cron              16      692k   25.1%  $ 48.61   48.3%  ⚠️ EXPENSIVE
main               1      274k   10.0%  $ 36.76   36.5%

💡 paperclip is eating 36% of tokens on a cheap model.
   High volume, low cost = lots of context for little output. Consider disabling.
💡 cron costs 48% of spend. Verify heartbeat model is Haiku or a local model, not Sonnet.
```

Format token counts: `1.2M` / `692k` / `344`. Keep table tight, no padding.

## Notes

- `estimatedCostUsd` is OpenClaw's estimate — not exact billing
- `main` being expensive is expected (interactive Sonnet sessions) — don't flag it
- Sessions.json is cumulative — grows over time, no automatic reset
- Don't read individual session `.jsonl` files — sessions.json has everything needed
