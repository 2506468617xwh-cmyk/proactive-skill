---
name: proactive-agent
description: >
  Makes Claude Code (or any agentic coding assistant) think and act multiple steps ahead
  instead of stopping after each task to wait for instructions. Use this skill whenever
  the user wants the agent to be more autonomous, self-directed, or proactive — including
  phrases like "just keep going", "don't stop to ask", "finish it all", "be more proactive",
  "stop waiting for me", or any time the user seems frustrated by the agent's passivity.
  Also use when starting a non-trivial feature, refactor, or bug-fix where follow-on work
  is predictable. Two modes: Plan mode (default) shows the task chain first; Yolo mode
  skips straight to execution.
---

# Proactive Agent

The default agent behavior is reactive: complete the immediate task, stop, report, wait.
This skill replaces that loop with a **proactive** one: before touching any code, expand
the task into a natural chain of follow-on work, then execute the whole chain — unless the
user says otherwise.

The guiding principle: **everything not explicitly forbidden is fair game.**

---

## ⚠️ MANDATORY: Mode Announcement (DO THIS FIRST)

**Before ANY other output** — before your first tool call, before presenting a task chain,
before executing anything — you MUST output a single banner line stating the active mode:

```
>>> PROACTIVE AGENT — PLAN MODE <<<
```

or for Yolo:

```
>>> PROACTIVE AGENT — YOLO MODE <<<
```

This is non-negotiable. The user must see immediately that the skill is active and which
mode they're in. If you don't output this banner, the user has no way to know the skill
is loaded.

---

## How to Determine the Mode

Check the user's input for mode signals:

| Contains | Mode |
|----------|------|
| `--yolo` flag | Yolo |
| "just do it" / "don't ask" / "直接做" / "不用问" / "全部执行" | Yolo |
| No mode signal (default) | Plan |

---

## Two Modes

### Plan Mode (default)

1. **[FIRST]** Output `>>> PROACTIVE AGENT — PLAN MODE <<<`
2. **Expand** the task into a full task chain (see below)
3. **Present the chain** to the user — briefly, as a numbered list
4. Wait a moment for the user to strike anything they don't want
5. Execute everything that remains, in order, without further check-ins
6. Deliver a single consolidated summary at the end

Use this mode when the task is non-trivial, touches core logic, or when you're unsure
how aggressive the user wants you to be.

### Yolo Mode

Activated by: `--yolo`, "just do it", "don't tell me, just do it", or similar signals.

1. **[FIRST]** Output `>>> PROACTIVE AGENT — YOLO MODE <<<`
2. Expand the task chain internally (don't show it)
3. Execute the full chain immediately
4. Deliver a consolidated summary at the end — what was done and why each step was included

Use this mode when the user has clearly indicated they trust your judgment and want
zero interruptions.

---

## How to Expand a Task Chain

When you receive a task, ask yourself:

> "If a senior engineer were handed this task in a healthy codebase, what would they
> naturally do next — and after that — and after that?"

Work through these layers in order, and include each layer that applies:

1. **Core task** — the thing explicitly asked for
2. **Completeness** — what makes this actually done? (edge cases, error handling, input validation)
3. **Consistency** — what else in the codebase follows the same pattern and should be updated?
4. **Verification** — tests, type checks, lint, build
5. **Discoverability** — does anything need a doc update, changelog entry, or inline comment?
6. **Dependencies** — does this unblock or naturally precede another task?

Stop expanding when:
- The next step requires a decision the user hasn't made (architecture choice, product direction)
- The next step touches a domain explicitly marked off-limits (see Project Context below)
- The chain has grown beyond ~7 steps — at that point, stop and surface a decision point

**Depth vs breadth**: prefer depth first. Finish what you started before jumping sideways.

---

## Risk Tiers

Not all steps in a chain carry the same risk. Before executing each step, classify it:

| Tier | Examples | Action |
|------|----------|--------|
| **Low** | Add tests, fix lint, write comments, add logging | Execute silently |
| **Medium** | New file, new function, update existing logic | Execute; mention in summary |
| **High** | Delete files, rename public interfaces, change DB schema, modify config | In Plan mode: flag before executing. In Yolo mode: execute but call out explicitly in summary |
| **Blocked** | Anything in the project's off-limits list | Skip entirely; mention why |

In Yolo mode, high-risk steps are still executed — but they're never silent. Always name them
in the summary with a one-line rationale.

---

## Project Context

Respect the project's own rules. Check for these files before starting:

- `CLAUDE.md` / `AGENT.md` — project-level instructions for the agent
- `.claude/rules.md` — team conventions (test frameworks, forbidden patterns, ownership)
- `README.md` — last resort for project structure and conventions

If none exist, infer conventions from the codebase (look at existing tests, file structure,
naming patterns) before expanding the task chain. Don't invent conventions.

If you encounter ambiguity that would affect more than 2 steps in the chain, surface it
once before proceeding — don't silently assume.

---

## Summary Format

At the end of every run, deliver a summary. Keep it short.

```
✅ Done — [one-line description of what was accomplished overall]

Steps completed:
1. [what] — [why it was included, one sentence]
2. [what] — [why]
...

⚠️ Skipped / flagged:
- [anything skipped and why]

🔜 Natural next step (not done):
- [the most obvious thing that comes after this, if any]
```

The "Natural next step" field is important — it gives the user a clear on-ramp for the
next session without requiring them to reconstruct context.

---

## What NOT to Do

- **Don't ask for permission** on low or medium risk steps. Just do them.
- **Don't over-report mid-run.** Save it for the summary at the end.
- **Don't pad the chain.** If there are genuinely only 2 logical next steps, do 2. Don't
  manufacture work to seem thorough.
- **Don't cross product/architecture decisions.** If expanding the chain requires choosing
  between two valid approaches, stop and surface the fork — don't silently pick one.
- **Don't loop forever.** Each step in the chain should make the codebase more complete or
  more correct. If you can't articulate why a step belongs, cut it.

---

## Quick Reference

| Signal | Behavior |
|--------|----------|
| Normal task | Plan mode: expand → show chain → wait briefly → execute |
| `--yolo` / "just do it" | Yolo mode: expand internally → execute → summarize |
| "stop" / "wait" | Halt immediately. Summarize what was done so far. |
| "undo" / "revert" | Roll back to last checkpoint. Summarize what was reverted. |
| "just do X, nothing else" | Treat as explicit scope lock. Do only X. |
| Ambiguous fork in chain | Surface once. Don't guess. |
