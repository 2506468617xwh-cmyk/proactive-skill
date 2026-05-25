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

1. **Thinks deeply** — understands the full scope, traces 5–10 steps ahead, maps the blast radius
2. **Identifies three distinct directions** the task could naturally grow into
3. **Builds a full chain for each direction** — 3–6 steps each, all comparably ambitious
4. **Lets you choose a direction** — not how much to do, but where to push
5. **Executes the chosen chain completely** — no interruptions, full files, circuit breaker included

Guiding principle: **everything not explicitly forbidden is fair game.**

---

## The key idea: direction, not depth

The three chains are not "do a little / do medium / do a lot." They are three genuinely different directions the work could expand into — each one tracing several steps forward along its own axis.

For example, "add rate limiting to the API" might produce:

```
Chain A — Observability: instrument everything around the limiter
  1. Add rate limiting middleware
  2. Emit metrics per route (hits, rejections, latency)
  3. Add structured logging with request context
  4. Wire up alerting thresholds for sustained rejection spikes
  5. Add a /health endpoint exposing current limit state

Chain B — Security hardening: make the limiter part of a broader defense layer
  1. Add rate limiting middleware
  2. Add IP allowlist/blocklist config
  3. Add anomaly detection for burst patterns
  4. Add audit log for rejected requests
  5. Add per-user token bucket alongside per-IP limit

Chain C — Resilience: make the system degrade gracefully under load
  1. Add rate limiting middleware
  2. Add 429 response with Retry-After header
  3. Add client-side backoff guidance in API docs
  4. Add queue-based shedding for non-critical endpoints
  5. Add load test suite to verify behavior under sustained traffic

Which direction? (A / B / C) — or describe your own, or type "new" for three new options.
```

You choose the direction. The agent executes all of it.

---

## Two modes

### Plan mode (default)

The agent presents three chains before executing. You pick one — or ask for alternatives.

### Yolo mode

No plan shown. The agent picks the best direction internally and executes immediately.

Activate with `--yolo` or "just do it."

---

## Chain selection

After seeing the three chains, you have four options:

| Input | Result |
|-------|--------|
| `A` / `B` / `C` | Execute that chain |
| `new` / "none of these" | Regenerate three new chains along different directions |
| "I want to focus on X" | Agent generates one targeted chain, confirms, executes |
| "B but skip step 3" | Agent adapts the chain, confirms, executes |

---

## Safety mechanisms

**Circuit breaker** — if any test, lint, type-check, or build fails mid-chain, execution halts. The agent reports what failed and waits for instructions. Never continues on a broken state.

**Code delivery rules** — complete files only. No snippets, no `// ...rest unchanged` placeholders, no partial configs.

**Context boundary control** — chains capped at 7 steps. If a chain touches more than 3 core files or involves cross-module changes, the agent pauses, re-states the original task, and confirms before continuing.

**Risk tiers** — each step is classified (Low / Medium / High / Blocked) before execution. High-risk steps are flagged in Plan mode and always called out in the summary.

---

## Installation

### Claude Code — global (all projects)

```bash
# macOS / Linux
mkdir -p ~/.claude/skills
cp SKILL.md ~/.claude/skills/proactive-agent.md

# Windows (PowerShell)
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills"
Copy-Item SKILL.md "$env:USERPROFILE\.claude\skills\proactive-agent.md"
```

Add to `~/.claude/CLAUDE.md`:

```markdown
## Skills
- proactive-agent: see ~/.claude/skills/proactive-agent.md
```

### Claude Code — project level

```bash
cp SKILL.md .claude/skills/proactive-agent.md
```

Add to `CLAUDE.md` in your project root.

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

Direction taken: [chain letter] — [direction name]

Steps completed:
1. [what] — [why]
2. ...

⚠️ Skipped / flagged:
- [anything skipped and why]

🔜 Natural next step (not done):
- [the obvious follow-on within this direction, for your next session]
```

---

## License

MIT
