---
name: whereamiburningtokens
description: "Read-only OpenClaw token usage analyzer. Reads the known local sessions ledger and shows where tokens and estimated cost are going by session type."
---

# whereamiburningtokens

Use this skill when the user wants to understand where OpenClaw tokens or estimated model cost are being spent.

## What this skill does

This skill is a **read-only local analyzer**.

It reads the known OpenClaw sessions ledger at:
- `~/.openclaw/agents/main/sessions/sessions.json`

If the file is missing, report that exact missing path and stop.

## Hard boundaries

- **Read only.** Do not modify config, session files, logs, memory, or any project files.
- **No filesystem wandering.** Do not scan the wider filesystem for alternatives unless the user explicitly asks.
- **No external network calls.** Do not send local session data to third-party services.
- **No secret access.** Do not read keychain items, env secrets, auth tokens, or unrelated OpenClaw state.
- **No automation changes.** Do not change heartbeat frequency, model config, cron jobs, or agent settings unless the user separately asks for that after seeing the analysis.

## Allowed inputs

Primary input:
- `~/.openclaw/agents/main/sessions/sessions.json`

Optional secondary input only if the user explicitly provides it:
- a user-supplied exported JSON file with the same purpose

## Required behavior

1. Read only the sessions ledger file.
2. Analyze usage by session category where possible, for example: main, cron, subagent, paperclip, heartbeat, or other visible categories from session metadata.
3. Show totals for tokens, estimated cost, and number of sessions.
4. Support time windows when the user asks, such as:
   - today
   - last 7 days
   - last 30 days
   - all time
5. Flag suspicious patterns in plain English, for example:
   - high-token background tasks
   - expensive low-value cron runs
   - a cheap model consuming excessive total tokens
6. Give concise recommendations, but keep them separate from any action. Diagnose first.

## Output format

Prefer a compact table plus 2 to 5 short observations.

Include:
- time window used
- total sessions
- total tokens
- estimated total cost
- breakdown by category
- notable anomalies
- next-step recommendations

## Good prompts

- where am I burning tokens?
- token breakdown this week
- what is costing me the most in OpenClaw?
- show token sinkholes for the last 30 days
- which session type is burning the most money?

## Bad assumptions to avoid

- Do not assume every expensive session is bad.
- Do not assume session category names if the data does not support them.
- Do not fabricate cost numbers, categories, or recommendations.

## If the file is missing

Reply with:
- the exact missing path
- that the skill is intentionally constrained to that file
- what the user can provide instead if they want analysis
