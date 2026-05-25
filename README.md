# proactive-agent

A skill for [Claude Code](https://claude.ai/code) and compatible agentic coding assistants that replaces the default **stop-and-ask** loop with a **think-ahead-and-execute** loop.

## The problem

Default agent behavior:

```
do task → stop → report → wait → do task → stop → ...
```

Every step requires a human prompt. Complex work that should take one session takes ten.

## What this skill does

Before touching any code, the agent:

1. **Thinks deeply** — understands the full scope, looks 3–5 steps ahead, surveys the blast radius
2. **Generates three task chains** — each a genuinely different strategy for tackling the work
3. **Lets you choose** — pick a chain, ask for new ones, or describe your own direction
4. **Executes completely** — runs the chosen chain end-to-end without interruption

Guiding principle: **everything not explicitly forbidden is fair game.**

---

## Two modes

### Plan mode (default)

The agent presents three chains before executing. You pick one — or ask for alternatives.

```
Chain A — Surgical: minimal, controlled scope
  1. Add rate limiting middleware
  2. Add 429 error response
  3. Write targeted tests

Chain B — Complete: the "done done" version
  1. Add rate limiting middleware
  2. Add env config for limits
  3. Write tests for enforcement and bypass
  4. Add 429 error response
  5. Update API docs

Chain C — Forward-looking: complete + next problem
  1–5. (same as B)
  6. Add rate limit headers to all responses
  7. Add monitoring hook for limit events

Which chain? (A / B / C) — or tell me a different direction, or type "new" for three new options.
```

### Yolo mode

No plan shown. The agent picks the best chain internally and executes immediately.

Activate with `--yolo` or "just do it."

---

## Chain selection

After seeing the three chains, you have four options:

| Input | Result |
|-------|--------|
| `A` / `B` / `C` | Execute that chain |
| `new` / "none of these" | Regenerate three new chains with different angles |
| "I want something more focused on X" | Agent generates one targeted chain, confirms, executes |
| "B but skip step 3" | Agent adapts the chain, confirms, executes |

---

## Safety mechanisms

**Circuit breaker** — if any test, lint, type-check, or build fails mid-chain, execution halts immediately. The agent reports what failed and waits for instructions. It never continues building on a broken state.

**Code delivery rules** — complete files only. No snippets, no `// ...rest unchanged` placeholders, no partial configs.

**Context boundary control** — chains are capped at 7 steps. If a chain touches more than 3 core files or involves cross-module changes, the agent pauses, re-states the original task, and confirms before continuing.

**Risk tiers** — each step is classified (Low / Medium / High / Blocked) before execution. High-risk steps are flagged in Plan mode and always called out in the summary.

---

## Installation

### Claude Code

```bash
# Global (all projects)
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills"   # Windows
mkdir -p ~/.claude/skills                                                        # Mac/Linux

cp SKILL.md ~/.claude/skills/proactive-agent.md
```

Add to your global `~/.claude/CLAUDE.md`:

```markdown
## Skills
- proactive-agent: see ~/.claude/skills/proactive-agent.md
```

### Codex (custom instructions)

Paste the contents of `SKILL.md` into your Codex custom instructions. Applies to every session automatically.

### Any other agent (Cursor, Gemini CLI, etc.)

Paste this at the start of your conversation:

> "Please read the instructions at this URL and follow them for this session: https://raw.githubusercontent.com/2506468617xwh-cmyk/proactive-agent/main/SKILL.md"

---

## Summary format

Every run ends with:

```
✅ Done — [one-line description]

Chain executed: [A – Surgical / B – Complete / C – Forward-looking / Custom]

Steps completed:
1. [what] — [why]
2. ...

⚠️ Skipped / flagged:
- [anything skipped and why]

🔜 Natural next step (not done):
- [the obvious follow-on for your next session]
```

---

## License

MIT
