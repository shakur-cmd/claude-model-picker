# claude-model-picker

Teach Claude Code to pick the right model for each job — and to route bulk work to cheaper models (GPT via the Codex CLI) — using nothing but a `CLAUDE.md` file and three small skills.

Inspired by [Theo's post](https://x.com/theo/status/2072482460122964067) and [video](https://www.youtube.com/watch?v=8GRmLR__OGQ&t=1229s) on putting a model-selection rubric in `CLAUDE.md`. This repo is my working version of that setup.

## The idea

Claude Code can spawn subagents and workflows on different models, and it can shell out to other coding agents (like OpenAI's Codex CLI) through Bash. But by default it has no opinion about *which* model should do *what*. So you give it one: a small table ranking each model on **cost**, **intelligence**, and **taste**, plus rules for how to apply it.

From then on, when Claude orchestrates work, it routes automatically:

- **Bulk/mechanical work** (clear specs, implementations, migrations, data analysis) → the cheapest capable model, via `codex exec`.
- **Anything user-facing** (UI, copy, API design) → a high-taste model on the first pass.
- **Reviews of plans and implementations** → the smartest models, optionally with a second independent perspective from the other lab's model.
- **Escalation is pre-authorized**: if a cheap model's output doesn't meet the bar, Claude reruns with a stronger model without asking. Judge the output, not the price tag.

The result: your expensive frontier-model tokens go to orchestration, judgment, and taste; the cheap tokens do the typing.

## What's in this repo

| File | What it is |
|------|-----------|
| `CLAUDE.md` | The model-picker section itself: the ranking table and the application rules. Drop it into your own global or project `CLAUDE.md`. |
| `codex-implementation` | Skill: delegate a scoped code change to Codex, then have Claude review the diff and run verification itself. |
| `codex-review` | Skill: get an independent code review from Codex on uncommitted changes, a branch diff, or a commit. |
| `codex-computer-use` | Skill: hand Codex verification work that needs real computer use — launching apps, browser automation, simulators, screenshots. (This is the one Theo highlights: Codex is notably strong at UI/UX verification.) |

Each skill file also carries an append-only "known fixes / blind spots" log — when a skill misfires, Claude appends the smallest fix so it doesn't repeat the mistake.

## How to use it

### 1. Prerequisites

- [Claude Code](https://claude.com/claude-code)
- [Codex CLI](https://github.com/openai/codex) installed and logged in (`codex` on your PATH) — only needed for the cross-model routing; the table works for Claude-only routing too.

### 2. Install the CLAUDE.md section

Copy the **"picking the right models for workflows and subagents"** section from `CLAUDE.md` into your own global config at `~/.claude/CLAUDE.md` (applies to every project) or a project's `CLAUDE.md` (that project only).

### 3. Make the table *yours*

The rankings only work if they reflect **your** reality:

- **cost** — what *you* actually pay (plan, deals, rate limits), not list price. My table rates OpenAI models as near-free because of my plan; yours may be the opposite.
- **intelligence** — how hard a problem you can hand the model unsupervised.
- **taste** — UI, UX, code quality, API design, copy.

Update the model names and the exact `codex exec -m <model>` invocation to whatever your Codex CLI version and plan actually support (the skills' logs show how to find out fast: just run it and read the error).

### 4. Install the skills

```bash
mkdir -p ~/.claude/skills/codex-implementation ~/.claude/skills/codex-review ~/.claude/skills/codex-computer-use
cp codex-implementation ~/.claude/skills/codex-implementation/SKILL.md
cp codex-review ~/.claude/skills/codex-review/SKILL.md
cp codex-computer-use ~/.claude/skills/codex-computer-use/SKILL.md
```

### 5. Use it

Nothing else to do — it's ambient. Ask Claude Code for work as usual, and it consults the rubric when spawning subagents or delegating. You can also invoke routing explicitly:

- "Have codex implement this while you review"
- "Get an independent codex review of this branch"
- "Verify the login flow in the browser" (routes to codex-computer-use)

## Notes and gotchas

- **Claude-only models in workflows**: the subagent `model` parameter only accepts Claude models. To use a Codex model inside a workflow, spawn a thin low-effort Claude wrapper agent whose prompt shells out to `codex exec` — and label it (e.g. `gpt:review-auth`) so you can tell who really did the work.
- **Timeouts**: Codex runs can exceed Bash's 10-minute default. Pass an explicit timeout or run in the background and poll for the report file.
- **Parallel Codex agents** need `isolation: 'worktree'` so their edits don't collide.
- **Trust but verify**: the skills make Claude read every Codex diff and run verification itself. Keep that — it's the difference between delegation and abdication.

## Credits

- [Theo (t3.gg)](https://x.com/theo) for the original CLAUDE.md model-routing approach — [post](https://x.com/theo/status/2072482460122964067), [video](https://www.youtube.com/watch?v=8GRmLR__OGQ&t=1229s).
