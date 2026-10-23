# mingz-harness

A Claude Code plugin: an adapted core subset of [pstack](https://github.com/cursor/plugins/tree/main/pstack) by Lauren Tan (poteto), MIT licensed (see `LICENSE.pstack`). Imported from pstack 0.15.30; the first commit is the unmodified upstream copy, so `git diff <first-commit> -- skills agents` shows every adaptation.

## Use

Start a task with `/mingz-harness:poteto-mode <goal, done condition, constraints>`. Other skills (`how`, `why`, `architect`, `arena`, `swarm`, `interrogate`, `blast-radius`, `correct`, `unslop`, ...) are routed by poteto-mode or typed directly. Every skill is `disable-model-invocation: true`, so it runs only when called.

## What changed from upstream

- Cursor `Task` tool → Claude Code Agent tool. `readonly` roles use `subagent_type: "Explore"`; code-writing parallel workers use `isolation: "worktree"`.
- Grok/Opus slugs and `setup-pstack` model rules → one Claude-only table in poteto-mode's Subagents section (`sonnet` code, `opus` judgment and prose, `haiku` mechanical). Cross-family panels become mixed opus/sonnet seats with fresh contexts.
- `cursor-team-kit` skills → local equivalents: `/deslop` → `/simplify`, `control-ui`/`control-cli` → project verification skill, Playwright, or `/run`. `create-skill` → `writing-for-agents`.
- Transcript paths → `~/.claude/projects/<workspace-slug>/`.
- Cursor cloud agents → local background subagents with `isolation: "worktree"`, or cloud sessions (`claude --cloud`) for work that must outlive the session (Orchestrate, Autopilot). Orchestrate's store lives at `~/.claude/projects/<workspace-slug>/orchestrate/<project>/` (`ORCH_STORE`).
- Worktree cleanup targets Linux/WSL: iOS simulator and Cursor cache steps → bun/npm/pnpm/uv/Docker caches; `worktree-audit.sh` scans Claude Code transcripts. It needs `jq` and a real `rg` binary on PATH.
- `ketchup` and `bro` imported (Autopilot replies use ketchup); `/visualize` → Mermaid charts.
- Not imported: reflect, teach, recall, sharpen, in-your-own-words, tdd (local `tdd` skill used instead), verification and formal-verification skills, setup-pstack, automate-me, make-bot-ui, statechart, poteto-help.

## Install

    claude plugin marketplace add ~/Projects/mingz-harness
    claude plugin install mingz-harness@mingz-local
