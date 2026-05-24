# proactive-agent

A skill for [Claude Code](https://claude.ai/code) (and compatible agentic coding assistants) that replaces the default **stop-and-ask** loop with a **think-ahead-and-execute** loop.

## The problem

Default agent behavior:

```
do task → stop → report → wait for user → do task → stop → ...
```

Every iteration requires a human prompt. Complex work that should take one session takes ten.

## What this skill does

Before touching any code, the agent expands the task into a **natural chain of follow-on work** — the steps a senior engineer would predictably take next — then executes the whole chain.

Guiding principle: **everything not explicitly forbidden is fair game.**

---

## Two modes

### Plan mode (default)

The agent shows you the task chain before executing. You can strike anything you don't want. Then it runs the rest without interruption.

```
User: add rate limiting to the API

Agent:
  Here's what I'll do:
  1. Add rate limiting middleware
  2. Add config for limits (env vars)
  3. Add tests for limit enforcement and bypass
  4. Add error response for 429
  5. Update API docs

  Striking anything? Otherwise starting now.
```

### Yolo mode

No plan shown. The agent expands the chain internally and executes immediately. One consolidated summary at the end.

Activate with `--yolo` or just say "don't tell me, just do it."

---

## Risk tiers

The agent classifies each step before acting:

| Tier | Examples | Behavior |
|------|----------|----------|
| Low | Tests, lint, comments | Silent execution |
| Medium | New files, new logic | Execute, mention in summary |
| High | Delete files, rename interfaces, schema changes | Plan mode: flag first. Yolo: execute + call out in summary |
| Blocked | Anything in project off-limits list | Skip, explain why |

---

## Project context

The agent respects your project's own rules. It checks for:

- `CLAUDE.md` / `AGENT.md`
- `.claude/rules.md`

Put team conventions, forbidden patterns, and ownership boundaries here. The agent reads them before expanding any task chain.

---

## Installation

### Claude Code

Copy `SKILL.md` into your project or global Claude Code skills directory:

```bash
# Project-level (affects this repo only)
cp SKILL.md .claude/skills/proactive-agent.md

# Global (affects all projects)
cp SKILL.md ~/.claude/skills/proactive-agent.md
```

Then reference it in your `CLAUDE.md`:

```markdown
## Skills
- proactive-agent: see .claude/skills/proactive-agent.md
```

### Codex (custom instructions)

Paste the contents of `SKILL.md` into your Codex custom instructions. It will apply automatically to every session.

### Any other agent (Cursor, Gemini CLI, etc.)

Feed the raw GitHub link at the start of your conversation:

> "Please read the instructions at this URL and follow them for this session: https://raw.githubusercontent.com/YOUR_USERNAME/proactive-agent/main/SKILL.md"

The agent will fetch the content and apply the rules immediately.

### claude.ai (skill file)

Package and install via the `.skill` format if your interface supports it.

---

## Summary format

Every run ends with a structured summary:

```
✅ Done — [one-line overall description]

Steps completed:
1. [what] — [why it was included]
2. ...

⚠️ Skipped / flagged:
- [anything skipped and why]

🔜 Natural next step (not done):
- [the obvious next thing, for your next session]
```

The **Natural next step** field is intentional — it gives you a clean on-ramp for the next session without reconstructing context.

---

## License

MIT
