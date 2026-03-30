# Module 05 — Production Ops

> An agent that only responds to messages is a chatbot. An agent that proactively manages its environment is infrastructure.

---

## Concept

Production OpenClaw deployments don't just answer questions — they run scheduled tasks, check system health, fire reminders, and monitor external services. This module covers the operational layer: crons, heartbeats, health checks, and handling the failure modes that show up in long-running deployments.

---

## Cron Jobs

OpenClaw's cron scheduler supports three schedule types:

**One-shot:**
```json
{ "kind": "at", "at": "2026-04-01T09:00:00Z" }
```

**Recurring interval:**
```json
{ "kind": "every", "everyMs": 1800000 }
```

**Cron expression:**
```json
{ "kind": "cron", "expr": "0 8 * * 1-5", "tz": "Europe/London" }
```

**Payload types:**
- `systemEvent` — injects text into main session (for reminders, alerts)
- `agentTurn` — spawns an isolated agent to run a task (for AutoDream, health checks, research)

**Delivery:**
- `announce` — sends result to a chat channel
- `webhook` — HTTP POST to a URL
- `none` — silent (the job runs but doesn't notify)

---

## Heartbeats

Heartbeats are lightweight periodic checks that run in the main session context. They're cheaper than crons for simple checks because they don't spawn isolated sessions.

Configure `HEARTBEAT.md` with a checklist:
- Check open-items for urgent items
- Verify daily log exists
- Check cron job health
- Flag anything time-sensitive

**Heartbeat vs Cron:**
- Heartbeat: batch checks, can drift slightly, uses main session context
- Cron: exact timing, isolated session, own model and thinking level

---

## Health Checks

A production health check cron typically covers:
- Are critical services reachable? (SSH, API endpoints, databases)
- Are VPN connections active? (if remote services depend on them)
- Are Docker containers running?
- Are there pending alerts or error logs?

Run every 30 minutes. On failure: alert via Telegram/Discord, not just a log entry.

**Pattern:**
```
Schedule: every 30 min
Payload: agentTurn — SSH to server, check services, report status
Delivery: announce to ops channel (bestEffort: true — don't fail the job if delivery fails)
```

---

## Context Limit Management

Long sessions accumulate context. When sessions approach token limits:

- OpenClaw triggers compaction — summarising old history
- Configure `compaction` in `agents.defaults` to tune behavior
- Force context limits on sub-agents via `contextTokens`

**Practical rule:** If a session handles heavy tool output (SSH, exec, large file reads), plan for a context reset every ~3-4 hours of heavy use.

---

## Exercises

**E5.1 — Set up a daily brief cron**
Create a cron job that fires every morning at 08:00 and delivers a brief covering: open items, calendar events in the next 24h, and any overnight alerts.

**E5.2 — Wire up a health check**
Write a health check cron for one service you depend on. It should: SSH in, verify the service is running, and fire an alert if not.

**E5.3 — Build HEARTBEAT.md**
Write a HEARTBEAT.md that covers 3-5 useful periodic checks. Run it 10 times over two days. Refine based on what was actually useful vs. noise.

**E5.4 — Survive a context limit**
Deliberately run a session heavy enough to hit the context limit. Observe what happens. Document your recovery procedure.

---

## Common Mistakes

- **Cron jobs that silently fail** — Always include delivery. A job that fails and doesn't notify is worse than no job.
- **Too many cron jobs** — Each job costs tokens when it fires. Batch related checks into one job.
- **Heartbeat doing too much** — If heartbeat output is longer than 3-4 lines, it's doing cron work. Move it to an isolated cron.
- **Not handling VPN/network dependencies** — If your crons SSH to remote servers, they'll fail silently when the VPN drops. Add connectivity checks before the main task.
- **Forgetting `bestEffort: true` on delivery** — If the delivery target is unavailable (Telegram rate limit, channel archived), a non-bestEffort job will mark itself failed. Usually you want the job to succeed even if notification fails.
