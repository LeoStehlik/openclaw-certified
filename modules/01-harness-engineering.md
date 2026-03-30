# Module 01 — Harness Engineering

> The workspace is not a folder. It is a mind.

---

## Concept

Most people treat OpenClaw as a chatbot with file access. That's not what it is.

OpenClaw is a persistent agent runtime. The workspace is the agent's long-term state — its identity, its memory, its rules of engagement. Get this right and everything downstream gets easier. Get it wrong and you end up with an agent that forgets things, contradicts itself, and needs hand-holding on every session.

This module covers the foundational files that define what your agent is and how it behaves.

---

## Core Files

### `SOUL.md`
Defines personality, tone, and behavioral rules. This is the agent's operating contract with itself — not just "be helpful" but *how* to be helpful, when to push back, when to ask questions, and what the red lines are.

**What to put in it:**
- Voice and tone (direct? warm? formal?)
- Decision-making rules (ask before acting on X, just do Y)
- Red lines (never exfiltrate data, always ask before deleting)
- Domain-specific triggers (calendar check, close session, LinkedIn post)

### `AGENTS.md`
Operational instructions for the agent and any sub-agents it spawns. Covers sprint discipline, agent prompt checklists, team structure, and mandatory rules that apply to every run.

**What to put in it:**
- Session startup sequence
- Sub-agent briefing checklists
- Mandatory status update rules
- Red lines for destructive actions

### `IDENTITY.md`
Name, emoji, avatar, creature. Sounds trivial — it isn't. An agent with a coherent identity behaves more consistently than one without.

### `USER.md`
Who is this agent helping? Name, timezone, contact, preferences. Keeps the agent oriented toward a real person, not a generic "user".

### `MEMORY.md`
Index pointing to memory files. Not the memory itself — just the map. (Module 03 covers memory in depth.)

### `HEARTBEAT.md`
Instructions for what to check on every heartbeat poll. Keeps the agent proactive without burning context on every tick.

---

## How It Works

On session start, OpenClaw injects these workspace files into the system prompt. The agent reads them before responding to anything.

Order matters. The agent reads in this sequence:
1. `SOUL.md` — who it is
2. `USER.md` — who it's helping
3. `IDENTITY.md` — its name and persona
4. `AGENTS.md` — operational rules
5. `MEMORY.md` + relevant memory files — what it remembers
6. `HEARTBEAT.md` — only on heartbeat polls

---

## Exercises

**E1.1 — Write your SOUL.md**
Write a SOUL.md for an agent you actually want to use. Include at minimum:
- 3 behavioral rules (not generic — specific to how *you* want to work)
- 2 domain triggers (e.g., "when I say X, do Y")
- 1 hard red line

**E1.2 — Define your identity**
Create IDENTITY.md. Give your agent a name that fits its role. Pick an emoji. Write one sentence on its vibe.

**E1.3 — Bootstrap a fresh agent**
Create BOOTSTRAP.md, start a new session, watch the agent read it and build its own identity through conversation. Delete BOOTSTRAP.md after. Never need it again.

**E1.4 — Measure drift**
Start two sessions with identical questions but different SOUL.md content. Observe how behavior changes. Iterate until the agent behaves the way you actually want.

---

## Common Mistakes

- **Writing SOUL.md like a policy document** — Rules that read like HR memos produce HR-memo behavior. Write it like you're talking to someone.
- **Forgetting USER.md** — Agents without user context default to generic helpful mode. That's not what you want.
- **Making AGENTS.md too long** — It gets injected on every session. Every kilobyte costs tokens. Keep it surgical.
- **Not updating files when behavior needs to change** — If you keep correcting the agent in conversation, that correction belongs in a file. Conversation corrections don't persist.
