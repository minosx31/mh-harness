# mingz-harness

A Claude Code plugin: an adapted core subset of [pstack](https://github.com/cursor/plugins/tree/main/pstack) by Lauren Tan (poteto), MIT licensed (see `LICENSE.pstack`). Imported from pstack 0.15.30; the first commit is the unmodified upstream copy, so `git diff <first-commit> -- skills agents` shows every adaptation.

## Usage

Every skill is `disable-model-invocation: true`: Claude never picks one on its own. You type it, or `poteto-mode` routes to it by reading its `SKILL.md`. Skills are namespaced: `/mingz-harness:<skill>`.

### Default: start with poteto-mode

    /mingz-harness:poteto-mode <goal>. Done when <pass/fail condition>. Evidence: <what proves it>. Constraints: <limits>.

poteto-mode picks a playbook, copies its steps into a todo list (skipped steps stay, marked `skip: <reason>`), routes to the skills it needs, delegates code to `poteto-agent` subagents, and labels each claim measured, inferred, or guessed before reporting done.

Prompting rules (from pstack):

- Give the goal, a falsifiable done condition, the evidence you want, clues, and constraints.
- Leave out the how-to and your guess at the cause.
- Don't write a skill order ("first /how, then /architect"). Name a skill only to override a default.
- It proceeds on reversible work without asking. It always stops for force-push to shared branches, deploys, data deletion, and outward-facing messages.
- "Don't stop" / "going to bed" / "run until done" switches to full autonomy.

Examples:

    /mingz-harness:poteto-mode Fix: /api/orders returns 500 when cart is empty. Done when a request with an empty cart returns 400 with a validation message and a regression test covers it.
    /mingz-harness:poteto-mode Add CSV export to the reports page. Done when the button downloads a file matching the table's filtered rows. Don't add dependencies.
    /mingz-harness:poteto-mode Babysit PR 142 to merge-ready.
    /mingz-harness:poteto-mode Migrate every caller of legacyFetch to apiClient and delete legacyFetch. I'm going to bed; run until done.

### Playbooks poteto-mode routes to

| Say something like | Playbook |
|---|---|
| "how does X work", "why is Y like this", "should we do A or B" | Investigation (read-only) |
| a bug report | Bug fix |
| "this is slow" | Perf issue, or Hillclimb for sustained metric work |
| "add / change feature X" | Feature |
| rename, extract, dedupe, move | Refactoring |
| "prototype", "mock it up", "try both" | Prototype |
| "babysit", "get it green", "check on PR X" | Babysit |
| "land / ship the stack" | Shipping |
| "run until done", "/loop until X" | Autonomous run |
| "autopilot this queue" (independent PRs, merge authority granted) | Autopilot-full |
| "build the stack, I'll land it" | Autopilot-stack |
| "own this whole migration/project" (multi-day, many PRs) | Orchestrate |
| "pick up where that session left off" | Session pickup |
| "pause", going offline | Pause safely |
| "clean up worktrees", "free up space" | Worktree and cache cleanup |
| large migration or no playbook fits | figure-it-out skill |

### Skills to call directly

| Skill | Use it when |
|---|---|
| `how` | You want a walkthrough of how code works, or where a change belongs. |
| `why` | You want the design rationale or history, with evidence cited. |
| `architect` | Settle types, signatures, and module shape before implementing. Add "with checkpoint" to stop before code. |
| `arena` | Compare N competing designs or implementations, pick a base, graft the best parts. |
| `swarm` | Fan out parallel workers for coverage (test matrix, many files, races). |
| `interrogate` | Adversarial multi-reviewer pass on a diff or design before shipping. |
| `blast-radius` | What could this small diff break outside itself? |
| `correct` | Agents keep making the same mistake here; make it impossible (lint, types, tests). |
| `no-comments` | Strip narration comments from a diff before review. |
| `benchmark-checklist` | Before you trust or report a perf number. |
| `show-me-your-work` | Keep a TSV decision log for long or unattended runs. |
| `figure-it-out` | Design a bespoke auditable procedure for work no playbook fits. |
| `unslop` | Remove AI tells from prose. |
| `technical-writing` | Docs, RFCs, READMEs, PR descriptions, commit messages. |
| `typescript-best-practices` | Reading or editing `.ts` / `.tsx`. |
| `ketchup` | Catch me up on what changed since my last message. |
| `bro` | Restate the last reply in plain words. |
| `principle-<name>` | Steer with one named rule, e.g. `/mingz-harness:principle-laziness-protocol`. Full list: `ls skills \| grep principle-`. |

### Changing the harness

The installed copy is a snapshot in `~/.claude/plugins/cache/`, and `plugin update` only installs a higher version. After editing this repo, bump `version` in `.claude-plugin/plugin.json`, commit, then:

    claude plugin marketplace update mingz-local && claude plugin update mingz-harness@mingz-local

New sessions pick up the change. To see what upstream pstack changed since the import, diff its current `pstack/skills/` against this repo's first commit.

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
