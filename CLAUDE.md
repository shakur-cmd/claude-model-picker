# Global

- I'm AbdulShakur Abdullah, solo entrepreneur at CodeMyVibe (codemyvibe.com). I'm called Shakur for short
  Projects live under ``.
  Clients work live under ``.
  Business information live under ``.

## Package Managers
- Use pnpm if the project already uses it, otherwise use bun.
- Never use npm or yarn. If there is no other way, explain it and wait for permission from the user

## Tech Stack Preferences
- No default stack or language. When proposing a stack or language, choose per project. 
- Fit to project's goals and needs (platform, users, performance, timeline), then explain pick.
- Among stacks that fit, prefer ones where AI tooling works well: one
  consistent "right way", docs close to the source, strong testability, fast
  feedback loops (compiler errors, tests). Popularity/training-data volume
  alone is not the criterion.

## Code Style
- Always strive for concise, simple solutions.
- If a problem can be solved in a simpler way, propose it.

## Git
- Never use the `gh` CLI; use plain git.

## General Preferences
- if asked to do too much work at once, stop and state that clearly.
- if computer use is helpful for completing or verifying work, shell out to GPT 5.5 with codex for it.

## Rules for every project
- Each project has its own CLAUDE.md — read it first; it wins over this
  file on any conflict.
- Don't add dependencies, frameworks, or services a project isn't already
  using without asking me first.
- If a spec/PRD file exists in the repo (e.g. `FULL_PRD.md`), it's the
  source of truth for product behavior.

## picking the right models for workflows and subagents

Rankings higher = better. Cost reflects what I actually pay (OpenAI is near free for me due to a deal), not the list price. Intelligence is how hard a problem you can hand the model unsupervised. Taste covers UI, UX, code quality, API design, and copy.

| model         | cost | intelligence | taste |
|---------------|------|--------------|-------|
| gpt-5.6-terra | 9    | 8            | 5     |
| sonnet-5      | 5    | 5            | 7     |
| opus-4.8      | 4    | 7            | 8     |
| fable-5       | 2    | 9            | 9     |

How to apply:
- These are defaults, not limits. You have standing permission to override them: if a cheaper model's output doesn't meet the bar, rerun or redo the work with a stronger model without asking. Judge the output, not the price tag. Escalating costs less than shipping mediocre work.
- Don't let costs prevent you from using the right model for the job. Instead, take advantage of cheaper options to get more information and try things before moving the work to a more appropriate option.
- Bulk/mechanical work (i.e. clear specs, implementation, data analysis, migrations): gpt-5.6-terra - it's effectively free, and with a strong, self-contained prompt it produces high-quality code fast.
- Anything user-facing, (i.e. UI, copy, API design), needs taste > 6 in the first pass. A model with lower taste can make suggestions.
- Reviews of plans/implementations: Fable 5 or Opus 4.8. Optionally GPT-5.6-terra as an independent perspective
- Never use Haiku.
- Mechanics: GPT-5.6-terra is only reachable through the Codex CLI - `codex exec` / `codex review`. Pass the model explicitly: `codex exec -m gpt-5.6-terra` (verified working on codex-cli 0.144.1, 2026-07-12; the config.toml default may differ). Use the codex-implementation, codex-review and codex-computer-use skills; for work they don't cover (investigation, data analysis), run `codex exec -m gpt-5.6-terra -s read-only` directly with a self-contained prompt.
- Using GPT-5.6-terra inside workflows and subagents (the model parameter only takes claude models, so use a wrapper):
 - Spawn a thin Claude wrapper agent with `model: 'sonnet', effort: 'low'` whose prompts instruct it to write a self-contained `codex exec` via Bash, and return the report (use `schema` on the wrapper to get structured output back). 
 - Always label these agents with a `GPT-5.6:` prefix, e.g. `{label: 'gpt-5.6:review-auth'}`. The workflow UI shows the wrapper's Claude model, so the label is the only indication the real worker is GPT-5.6-terra
 - Codex can exceed Bash's 10-minute timeout: pass an explicit timeout, or run in the background and poll for the report file.
 - Parallel GPT-5.6-terra implementation agents must use `isolation: 'worktree'` so codex edits don't collide with the shared checkout.
 - Workflow token budgets only count Claude tokens; Codex work is free and invisible to `budget.spend()`. 
## Keapstone (personal memory OS)
- Keapstone MCP is connected (search_memory, get_context_packet, remember, propose_memory, search_sources, capture_source).
- At the start of substantial tasks about my projects/business, call `get_context_packet` for relevant context.
- When you learn a durable new fact/decision/preference during work, file it with `propose_memory` (I review in the inbox at localhost:3100). Use `remember` only when I explicitly say to remember something.
- Also propose action types when they surface in work: `blocker`, `next_action`, `goal`, `loop`, `automation` — one standalone sentence naming the project. They land in the Action Center after my approval.
- **Checklists**: ANY time I say "make a keapstone checklist" (or "keepstone checklist" — I dictate, spelling varies), immediately call `propose_memory` with type `checklist` — no clarifying questions. If I don't name a project, infer it from what we're working on. Format the content exactly as:
  - First line: `Checklist — <project-folder-name>: <short title>` (folder name, e.g. `tax-trail`, so it attaches to the right project card).
  - Then one `- [ ] Item title [tag]` line per task, concrete and ordered. Optional one-word tag in brackets at line end: `[blocker]`, `[decision]`, `[quick]`, `[keys]`, `[legal]`, `[client]`.
  - Under each item, indented step lines with concrete instructions (commands, URLs, exact clicks): `  - how: <step>`, and optionally one `  - why: <one-line reason>`.
  I accept it in the inbox, then work it from the Checklists tab — items expand to show the how-steps.
