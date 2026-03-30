# Module 06 — Sprint Engineering

> An agent that declares "done" without proof is not done. It's optimistic.

---

## Concept

Running a multi-agent sprint — Builder, Reviewer, Tester, Verifier — sounds good in theory. In practice it collapses without discipline. The Builder builds on the wrong base. The Reviewer approves without running anything. The Tester only tests the new feature. Nobody catches the regression. The sprint ships broken.

Sprint engineering is the discipline that prevents this. It's the combination of spec freezing, role separation, explicit acceptance criteria, blast radius awareness, and mandatory regression testing.

---

## The Proof Loop

The canonical sprint workflow:

```
1. Spec freeze (Orchestrator) — ACs written, reviewed, locked before any code
2. Blast radius check — which files/functions does this change affect?
3. Builder — implements against frozen spec
4. Reviewer — code review + live browser test, verdicts per AC
5. Tester — full regression suite, all previous sprints
6. Orchestrator — only marks sprint done when Tester is green
```

**Critical:** Steps cannot be reordered or skipped. A sprint isn't done until all three agents (Builder, Reviewer, Tester) have signed off in sequence.

---

## Acceptance Criteria

Every sprint has explicit, testable ACs before the Builder starts:

```
AC1: German user sees all nav items in German
AC2: Prompt titles display in the user's selected language
AC3: Form field labels translate when locale is set to DE
AC4: English fallback when no translation exists
```

The Reviewer verdicts each AC: `PASS / FAIL / UNKNOWN`.
The Tester re-verifies all ACs against the live app, not just the code.

**Rule:** If an AC can't be verified by a human or a test, it's not a real AC.

---

## Blast Radius

Before briefing the Builder, run a blast radius analysis:
- Which files does this change touch?
- Which existing features depend on those files?
- Which tests cover those features?

Without this, the Builder may refactor a shared component and silently break three other features. The Reviewer catches it in code review — or doesn't. The Tester catches it in regression — or doesn't if they only run new tests.

**Tools:** `code-review-graph` (local knowledge graph), `git log -- <file>`, manual inspection.

---

## Regression Gate

Every sprint includes full regression — all previous sprint tests, every time. Not just the new sprint's tests.

**Tester's checklist (non-negotiable):**
- [ ] Run full cumulative test suite
- [ ] All previous sprint tests must pass
- [ ] Any previously-passing test that now fails = CRIT blocker
- [ ] Sprint does NOT pass until regression is clean

No shortcuts. No "just the new stuff". A sprint is not done until the Tester says so.

---

## Campaign Files

For multi-sprint projects, maintain:
- `org/campaigns/[SPRINT_ID]/campaign.md` — goal, phases, current state, discovery log
- `org/campaigns/[SPRINT_ID]/spec.md` — frozen ACs

Every agent on the sprint reads `campaign.md` first. The discovery log prevents amnesia when sessions restart mid-sprint.

---

## Exercises

**E6.1 — Write a frozen spec**
Pick a feature you want to build. Write the spec with explicit, testable ACs before writing any code. Get them reviewed. Then build.

**E6.2 — Run a blast radius check**
Before your next code change, manually trace which files it touches and which existing features depend on those files. Document it. Did it change the scope of the change?

**E6.3 — Separate Builder and Reviewer**
For your next coding task, deliberately use two separate agents — one to build, one to review. No shared session. Observe what the Reviewer catches.

**E6.4 — Write a regression suite**
Write 3-5 tests that cover the features you've already built. Run them after your next sprint. Make them part of your standard acceptance process.

---

## Common Mistakes

- **Building before the spec is frozen** — The most common failure mode. If the ACs change mid-build, the Builder has to redo work and the Reviewer is reviewing a moving target.
- **Reviewer doing code review only** — Code review doesn't catch "the UI is broken in German". Live browser tests are mandatory.
- **Only testing new features** — Every regression suite must include all previous sprints. New feature passing while old feature is broken = sprint failure.
- **Building on the wrong base** — Always verify the working directory and git branch before the Builder starts. Five minutes of confirmation saves hours of rebuilding.
- **Declaring done before the Tester runs** — "Reviewer approved it" is not done. Done means Tester ran the full suite and it's green.
