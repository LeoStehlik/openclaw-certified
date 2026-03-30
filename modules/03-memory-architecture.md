# Module 03 — Memory Architecture

> Sessions are ephemeral. Files are not.

---

## Concept

Every OpenClaw session starts fresh. The agent has no memory of previous conversations unless you explicitly give it one.

Most people solve this badly — they either dump everything into a single giant memory file (expensive, noisy) or rely on the agent to "remember" things in conversation (it won't, next session). 

The right approach is a layered memory system: raw daily logs for capture, curated long-term memory for recall, and an index to navigate between them.

---

## The Memory Stack

### Layer 1 — Daily Logs (`memory/YYYY-MM-DD.md`)
Raw notes from each session. Everything that happened, decisions made, things to follow up on. Written *during* the session, not after.

**Rule:** If you want to remember it, write it to a file. "Mental notes" don't survive session restarts.

### Layer 2 — Long-Term Memory (`MEMORY.md` or `memory/*.md`)
Curated distillation of what matters. Not everything from the daily log — just the decisions, lessons, preferences, and context that future sessions need.

**Structure:** `MEMORY.md` as an index pointing to focused files:
- `memory/person.md — person, preferences, background
- `memory/infrastructure.md` — hardware, SSH, services
- `memory/decisions.md` — significant calls and reasoning
- `memory/open-items.md` — live homework tracker

### Layer 3 — Open Items (`memory/open-items.md`)
The canonical homework tracker. If it's not in this file, it doesn't exist. Every assigned task, every pending decision, every thing-to-follow-up-on goes here.

**Checked at:** session start (always), heartbeat (always).

---

## AutoDream — Nightly Memory Consolidation

A cron job that runs nightly (e.g., 03:00 local time) to:
1. Review recent session transcripts and daily logs
2. Update long-term memory files with significant new information
3. Close stale open items
4. Prune outdated content from MEMORY.md

This is the difference between a memory system that degrades over time and one that gets better.

**Setup:**
```json
{
  "schedule": { "kind": "cron", "expr": "0 3 * * *", "tz": "Europe/London" },
  "payload": { "kind": "agentTurn", "message": "Run AutoDream memory consolidation..." }
}
```

---

## Vector Search

OpenClaw supports semantic memory search via `memory_search`. Requires a local embedding model (e.g., `nomic-embed-text` via Ollama) configured in `agents.defaults.memorySearch`.

```yaml
memorySearch:
  provider: ollama
  model: nomic-embed-text
  paths:
    - MEMORY.md
    - memory/*.md
```

This enables fuzzy recall — "what did we decide about X?" — without loading all memory files into context.

---

## Exercises

**E3.1 — Build your memory index**
Create `MEMORY.md` as a table of contents pointing to 3-4 focused memory files. Populate each with real context about your setup.

**E3.2 — One week of daily logs**
Commit to writing `memory/YYYY-MM-DD.md` entries for 5 sessions. At the end of the week, identify what's worth promoting to long-term memory and what can be discarded.

**E3.3 — Set up open-items tracking**
Create `memory/open-items.md` with at least 5 real open items. Practice closing and adding items during a session. Check it at the start of your next 3 sessions.

**E3.4 — Wire up AutoDream**
Write a cron job that runs nightly and consolidates your daily logs into long-term memory. Run it manually first, verify the output, then schedule it.

---

## Common Mistakes

- **One giant MEMORY.md** — Gets injected on every session, costs tokens on everything, even unrelated tasks. Split into focused files and load only what's relevant.
- **Not writing open items immediately** — If you say "I'll remember that", you won't. Write it to `open-items.md` in the moment.
- **Forgetting to update memory on session close** — The close trigger (sign-off phrase) should automatically flush the session log and update open items. Wire this into SOUL.md.
- **Memory without pruning** — A memory system that only adds and never removes grows stale. AutoDream or periodic review is mandatory.
