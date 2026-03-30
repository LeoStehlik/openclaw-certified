# Module 02 — Agent Orchestration

> One agent is a tool. Many agents, well-coordinated, are a team.

---

## Concept

OpenClaw supports spawning sub-agents — isolated sessions that run a task and report back. This is how you scale beyond what a single context window can hold, separate concerns (builder vs. reviewer vs. tester), and run work in parallel.

The hard part isn't spawning agents. It's briefing them well enough that they don't build the wrong thing, review the wrong version, or declare success without evidence.

---

## Core Primitives

### `sessions_spawn`
Spawns an isolated sub-agent. The key parameters:

```
task: What to do (the brief)
mode: "run" (one-shot) or "session" (persistent)
runtime: "subagent" (OpenClaw) or "acp" (external harness like Codex/Claude Code)
label: Name this agent so you can track it
streamTo: "parent" to see output in real time
```

### `sessions_yield`
End your current turn and wait for sub-agents to complete. Results come back as the next message.

### `subagents`
List, steer, or kill running sub-agents. Use for intervention — not for polling in a loop.

### `sessions_send`
Send a message into another session. Used for async steering.

---

## The Four-Agent Pattern

Most production workflows follow this shape:

```
Francis (orchestrator)
  └── Builder (implements)
  └── Reviewer (code review, no live test)
  └── Tester (live browser/integration test)
  └── Verifier (reads spec, reads verdicts, final call)
```

**Why separation matters:** An agent that builds its own work will approve its own work. Role separation is the only reliable quality gate.

---

## ACP Harnesses

For tasks requiring a full coding agent (Codex, Claude Code), use `runtime: "acp"`:

```json
{
  "runtime": "acp",
  "agentId": "codex",
  "mode": "session",
  "thread": true,
  "task": "..."
}
```

Thread-bound sessions (`thread: true`) persist across messages in the same chat thread — useful for iterative coding work where you want to steer mid-run.

---

## Exercises

**E2.1 — Spawn your first sub-agent**
Write a brief for a simple task (summarise a file, run a script, check a URL). Spawn it with `sessions_spawn`, yield, receive the result.

**E2.2 — Builder + Reviewer split**
Take any coding task. Spawn a Builder to implement it. Spawn a separate Reviewer to review the output. Observe what the Reviewer catches that the Builder didn't flag.

**E2.3 — Write a sub-agent brief checklist**
Before spawning any agent, run through this:
- [ ] First instruction: update status to `working`
- [ ] Last instruction: update status to `done` + fire system event
- [ ] Scope is clean — no bundled out-of-scope items
- [ ] Success criteria are explicit and testable
- [ ] Agent is told to ask if anything is unclear before proceeding

**E2.4 — Steer a running agent**
Spawn a long-running task. Mid-run, use `subagents(action=steer)` to redirect it. Observe how it responds to in-flight steering.

---

## Common Mistakes

- **Vague briefs** — "Build the login page" is not a brief. Acceptance criteria, file locations, constraints, and test instructions all need to be explicit.
- **Builder verifying its own work** — Never. Always separate roles.
- **Not including status update instructions** — Agents that don't update `status.json` become invisible mid-run. You won't know if they're done, blocked, or looping.
- **Polling `subagents list` in a loop** — Don't. Use `sessions_yield` and wait for push notification.
- **Forgetting the 95% confidence rule** — If the agent isn't 95% sure it understands the task, it should ask. If your brief doesn't make this explicit, the agent will guess.
