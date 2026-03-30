# Module 04 — Skill Development

> Skills are the difference between briefing an agent from scratch every time and having one that already knows how.

---

## Concept

A skill is a reusable package of procedural knowledge. It's not a prompt template — it's a structured guide that tells an agent *how* to do a specific class of task, every time, without re-explaining from scratch.

Skills are how you encode institutional knowledge. You figure out the right way to do something once, write it down as a skill, and every future agent — including future you — gets that knowledge for free.

---

## Anatomy of a Skill

```
skill-name/
├── SKILL.md          (required)
└── references/       (optional)
    ├── patterns.md
    └── examples.md
```

### `SKILL.md`

Two parts:

**Frontmatter (YAML):**
```yaml
---
name: my-skill
description: What this skill does and when to use it. This is the trigger — write it clearly.
---
```

The description is the triggering mechanism. OpenClaw reads it to decide when the skill applies. Be specific about *when* to use it, not just what it does.

**Body (Markdown):**
Instructions for executing the skill. Loaded only after the skill triggers — so keep the frontmatter tight and put the detail in the body.

### `references/`
Supporting material — patterns, examples, schemas, API docs — that the agent loads only when needed. Keeps `SKILL.md` lean.

---

## Progressive Disclosure

Skills use a three-level loading pattern:

1. **Frontmatter** — always in context, ~100 words, triggers the skill
2. **SKILL.md body** — loaded when skill triggers, keep under 500 lines
3. **References** — loaded on-demand when the agent needs depth

Never put everything in `SKILL.md`. If it's reference material, it goes in `references/`.

---

## Publishing Skills

Once a skill is ready:
1. Create a public GitHub repo named after the skill (lowercase, hyphens)
2. Push `SKILL.md` + any `references/` files
3. Write a `README.md` with install instructions for OpenClaw
4. Tag it so others can find it

OpenClaw users install skills by cloning into their skills directory and pointing the config at it.

---

## Exercises

**E4.1 — Identify a repeatable task**
Pick something you ask your agent to do regularly. Write the SKILL.md for it. Include: what triggers it, what the agent should do, what common mistakes to avoid.

**E4.2 — Add a reference file**
Take your skill from E4.1 and move the detailed reference material (patterns, examples, schemas) into `references/`. Link from `SKILL.md`. Verify the skill still works.

**E4.3 — Publish a skill**
Publish a skill to GitHub. Write a README. Make it something you'd actually want someone else to use.

**E4.4 — Review someone else's skill**
Find a published OpenClaw or Codex skill. Read the SKILL.md. Ask: Is the description a good trigger? Is the body tight? What would you change?

---

## Common Mistakes

- **Describing what the skill is instead of when to use it** — The description drives triggering. "Use when X" beats "This skill does X."
- **Putting everything in SKILL.md** — It gets loaded on every trigger. Reference material in the body is wasted tokens when the agent doesn't need it.
- **Not testing the trigger** — Write the description, then ask your agent something that *should* trigger it and something that *shouldn't*. Verify both.
- **Writing skills for yourself, not for agents** — Skills aren't documentation. They're operating instructions. Write for an agent that has no other context.
