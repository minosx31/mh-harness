# Using mh-harness

Adapted from pstack's guide (`pstack/docs/guide/`) for Claude Code and for the skills this repo ships. The README has the quick reference tables. This page is about habits: how to prompt, what to expect back, and when to reach for each skill.

The one habit that matters most: **give a goal and a way to check it, then get out of the way.**

    /mh-harness:poteto-mode the export writes duplicate rows when a retry lands mid-run. repro first, then fix and verify.

You don't name a playbook or list skills. "repro first" plus a checkable outcome is enough for poteto-mode to pick Bug fix, copy its steps into the todo list, and call the right skills as each step fires.

## 1. Before your first run

- Install (see README), then start a **new session**. Skills load at session start.
- Every skill is `disable-model-invocation: true`. Claude never picks one by itself. You type `/mh-harness:<skill>`, or poteto-mode routes to it.
- **Permissions decide how autonomous a run can be.** poteto-mode proceeds on reversible work without asking, but Claude Code's permission prompts still stop it. For long runs, use auto mode or allowlist the commands your project needs (`/fewer-permission-prompts` builds an allowlist from your transcripts). A run that waits on a prompt all night has done nothing.
- Models are fixed in poteto-mode's Subagents table: `sonnet` for code delegates and explorers, `opus` for judgment and prose, `haiku` for mechanical edits. There is no `setup-pstack`. To change a role's model, edit that table and bump the version (README, "Changing the harness").

## 2. Prompting poteto-mode

A good prompt carries up to five things, one sentence each:

| Part | Example |
|---|---|
| Goal | "the csv export drops its last row since yesterday's deploy" |
| Done check (must be able to fail) | "done means the 60k-row fixture exports every row" |
| Proof you want | "show me row counts before and after" |
| What you already know | "failing job id is 4812" |
| Real constraints | "repro first", "don't change any code yet", "zero behavior change", "let me review before proceeding" |

Leave out:

- **The how.** Say what to achieve. The playbook already has the steps.
- **Your theory of the cause, at first.** It narrows the search to where you pointed. Ask for a restatement first, then share your hunch:

      /mh-harness:poteto-mode read this thread. restate the underlying issue in plain english. don't change any code yet.

  For the restatement alone, with no playbook: `/mh-harness:in-your-own-words`.

- **A skill order.** "use /how, then /architect, then /arena" reorders or drops steps the playbook would have kept. Name a skill only to override a default.

### Phrases that change routing

| Phrase | Effect |
|---|---|
| "don't change any code yet" | Pins the task to Investigation (read-only). |
| "new task" | Re-match a playbook instead of continuing the last one. Use it whenever you switch subjects in a long chat. |
| "repro first" | Bug fix reproduces before touching code. |
| "zero behavior change" | Refactoring pins current output before moving anything. |
| "with checkpoint" / "let me review before proceeding" | Stops after the design, before code. |
| "going to bed" / "don't stop" / "run until done" | Full autonomy (Autonomous run + decision log). |
| "pause" / "I'm going offline" | Pause safely: checkpoint commit and a resume note. |

### Sharpen a prompt before sending it

When you're unsure a prompt is aimed right:

    /mh-harness:sharpen write a migration guide for the v2 config format.

It names what a top professional would do that you didn't ask for, the raw material they'd start from, their methods and tools, and what outside check would show the result is right. Then it rewrites your prompt and stops. It does not do the task. Send the rewrite through poteto-mode when it reads right.

### Follow up short

Once the context is in the conversation, `continue`, `do it`, or `keep going until done` is a complete prompt. The playbook holds the structure.

### When to invoke it again

Once per task, not once per reply. When the agent asks you something, just answer; poteto-mode is still in context.

- **Compaction:** no need. Claude Code re-attaches invoked skills after compaction, and poteto-mode then re-reads its active playbook and rebuilds the todo list.
- **New session** (including `/clear`): invoke it again. Use `/mh-harness:recall` or "take over this branch" to load the old context.
- **New subject in the same chat:** invoke it again, or say "new task".
- **Drift** (no todo list, no evidence, no measured/inferred/guessed labels): invoke it again with `continue`.

### Parallel work

Several agents in one working tree overwrite each other. poteto-mode already gives code-writing workers `isolation: "worktree"`. When you want a specific base, say so: "branch off `<base>` in a fresh worktree". Worktrees pile up; `/mh-harness:poteto-mode what's eating my disk? prune the worktrees that are safe to prune.` runs Worktree cleanup, which deletes only what merge state and transcripts clear.

## 3. Understand before you change

Agents fail by misreading the goal (section 2 fixes that) or by lacking context (this section).

    /mh-harness:poteto-mode investigate why background jobs time out every few hours. give me what we know, what data you used, and your best hypotheses. don't change any code yet.

Investigation runs `how`, adds `why` for motivation questions, and returns a cited answer, or a recommendation with a trade-offs table for "A or B". When the findings point at a fix, start the fix as a **new task**.

Direct skills:

- `/mh-harness:how do we dedupe notifications? is there an n+1 on subscriber lookup?` reads the code and explains the runtime flow, key types, and non-obvious parts. Fans out read-only explorers for big subsystems.
- `/mh-harness:why was the retry limit set to five? does the reason still hold?` starts from git history, then whatever your MCPs expose (Sentry is connected in this setup; Linear, Slack, Notion, Datadog sources exist in `skills/why/references/sources/`). It separates evidence from inference and reports "nobody wrote it down" as a real answer.
- "do why first, then how" is a good combination when you suspect history explains the mess.
- `/mh-harness:teach me how this PR changes retries. convince me it fixes the cause and not the symptom.` runs `how` and `why` and weaves them into one plain explanation that builds up diagram by diagram. "Convince me" turns the tour into an argument you can test. It works on the agent's own choices too: "teach me why you did it this way and not with a queue."

Rebuilding your own context:

    /mh-harness:recall catch me up on the export work from last week
    /mh-harness:recall my work on the virtualized list from yesterday, then read this bug report.

`recall` mines this workspace's transcripts in `~/.claude/projects/` (default window 7 days) with parallel `haiku` readers, sweeps the shared record through `why`'s sources, checks branches and PRs with `git` and `gh`, and returns a capsule, tagged thread statuses, recurring problems, and one next move. To resume one specific chat, use Session pickup instead.

Taking over someone's branch or an old session:

    /mh-harness:poteto-mode take over this branch. read the decision log, figure out what's done, and continue. don't redo finished work.

**Pitfall:** skipping this because "the agent reads the code anyway." An agent without a traced model fixes the symptom at the first plausible spot.

## 4. Design before code

Most changes need none of this. A rough ladder:

| Situation | Reach for |
|---|---|
| Small finished change you're unsure about | `interrogate` |
| Crosses function boundaries or moves ownership | `architect` (brings `arena` with it; poteto-mode triggers it by itself) |
| Standalone decision: naming, format, algorithm | `arena` |
| Coverage matrix, parallel checks, or a race | `swarm` |
| A question you could answer by running something | a prototype, not a debate |
| Contested and expensive to reverse | `architect`, then `interrogate` before shipping |
| Several PRs | a plan, written after the design settles |

Examples:

    /mh-harness:architect design the import pipeline. i care most about how callers use it. with checkpoint.
    /mh-harness:arena this, 5 candidates. the cache key format is expensive to change later.
    /mh-harness:swarm check every package under packages/ against its check.sh. one worker per package. one report.
    /mh-harness:interrogate the whole branch, skeptically. no nitpicks unless it's a real bug or regression.
    /mh-harness:poteto-mode prototype a few options for the settings layout. screenshots of each.
    /mh-harness:poteto-mode turn this design into a plan. small verifiable PRs, each with its own proof.

Notes:

- `arena` repeats **one** brief N times and grafts the best parts. `swarm` splits **different** slices and aggregates. Don't use arena for coverage.
- `interrogate` and `arena` panels are mixed opus/sonnet seats with fresh contexts, not different model families as upstream. Agreement between seats is weaker evidence than it is in pstack; read the dismissed findings too.
- Don't spend `interrogate` on a plan with no code. Reviewers invent risks. Prototype, then review what got built.
- For shared code, ask for the README or tutorial first; it forces the caller's view and becomes the target.

## 5. Build and clean

Prompt each build playbook with what you observed; the playbook supplies the discipline (repro before fix, pin behavior before refactor, profile before optimizing):

    /mh-harness:poteto-mode add a --json flag. text output stays byte-identical. verify both forms.
    /mh-harness:poteto-mode move parsing into one module, zero behavior change. record current output first and prove it's unchanged.
    /mh-harness:poteto-mode startup takes 1.8s on this fixture. trace it, fix the measured cause, show before and after.
    /mh-harness:poteto-mode here's a cpuprofile from the slow startup. where does the time go? no fix yet.

The last one routes to Trace forensics, which diagnoses without fixing. Hillclimb is for pushing one metric over many attempts.

Tests: `/mh-harness:tdd implement` writes the smallest test that fails for the intended reason, runs it, fixes, and reruns. If a test would need broad harness setup or brittle mocks, it says so and uses the closest executable check instead. The Bug fix playbook uses it when a cheap test path exists. Your user-level `/tdd` (red-green-refactor for features) is a different skill; both can stay installed because this one is namespaced.

TypeScript: type `/mh-harness:typescript-best-practices` when a task touches `.ts`/`.tsx`. It doesn't load by itself.

Cleanup before commit (Opening a PR runs these for you):

- `/simplify` cleans the code (stands in for Cursor's `/deslop`).
- `/mh-harness:no-comments the diff` hands comments to `comment-sicko`, a reviewer that didn't write them. It keeps license headers, public API docs, and comments about external constraints, and turns the rest into refactors or deletions.
- `/mh-harness:unslop the readme changes, no emdashes` cleans prose.

## 6. Verify and ship

"It compiles" is not evidence. Put the finish condition in the first prompt and expect the reply to carry the exact commands and outputs, or "inconclusive".

Match the check to the change: a CLI change runs the real command; a UI change walks the flow in the running app (Visual parity playbook for pixel-exact work); a migration replays saved input; perf compares profiles; storage reads the value back.

### Give the agent a way to drive your app

The UI bullet hides a requirement: the agent needs a scripted way to drive the app. Without one, poteto-mode falls back to `/run` or Playwright and improvises each time. Generate a project verification skill once:

    /mh-harness:create-verification-skill

It interviews the repo, not you: what a user touches, how the app launches, what can drive it (existing harness first, then browser/CDP, PTY or tmux, or plain HTTP), what evidence proves behavior, and whether two instances can run side by side. It writes `.claude/skills/verify-<app>/` with Launch, Doctor, Drive, Evidence, Cleanup, and Helpers sections, plus a feature map under `features/` (one file per feature). Before handing over, it runs the skill end to end once and confirms the evidence survives cleanup. If that proof failed, don't use the output.

Commit it so every agent drives the app the same way. From then on, name it when you want proof in a specific form:

    /mh-harness:poteto-mode build the bulk-archive action. use /verify-<app> to verify and show me screenshots as proof.
    /mh-harness:poteto-mode repro this with /verify-<app>. if it repros on main, fix it and show me proof.

Generated verify skills are model-invocable, so poteto-mode picks them up as "the project's verification skill" without being told. Once it works, `swarm` can split a full pass by feature-map entry.

Apps change and feature maps rot. Run this periodically, or after big UI changes:

    /mh-harness:maintain-verification-skill

It sends one read-only reader per feature, then drives every feature live, and ends in one of three outcomes: `clean`, `changed` (one PR of proven corrections confined to the verify skill's directory), or `blocked`. It never edits product code. A real regression gets reported, not papered over in the docs.

If agents keep writing throwaway scripts to drive your app, ask for a small control CLI the verify skill calls instead (few composable commands, `--dry-run`, JSON output, good `--help`).

- `/mh-harness:blast-radius` on a small diff you don't trust: finds the one fact that makes it safe and proves it by running code.
- `/mh-harness:benchmark-checklist vet this 40% speedup before it goes in the PR` asks seven questions (what limits the number, both sides tuned, physical limits, errors, reproducibility, end-to-end impact, work inside the timed region) and returns faster / slower / no difference / inconclusive.

Shipping:

    /mh-harness:poteto-mode open the pr. small ordered commits, evidence in the description.
    /mh-harness:poteto-mode babysit PR 142. get it green.
    /mh-harness:poteto-mode check on pr 142. anything outstanding?
    /mh-harness:poteto-mode land the stack.

Babysit takes blockers in order (conflicts, review threads, CI), batches fixes into one push, and stops at merge-ready. It never merges. Shipping verifies each PR with a fresh agent and lands only the contiguous verified run from the bottom of the stack.

## 7. Unattended runs

Earn trust before you loop. You've watched the task done once, the agent has the tools you'd use (verify path, profiler, logs), every stage can stop the line, and you've turned repeated failures into checks.

The overnight contract:

    /mh-harness:poteto-mode im going to bed. migrate every caller to the new parser in a fresh worktree off main.
    done means zero old callers, all parser fixtures pass, old api deleted.
    keep a decision log. don't ask me before committing.
    run until done. if you're truly stuck after a few hours, stop and write up why.

- "going to bed" is the autonomy override; "done means" gives each iteration a check; the worktree isolates the run; "don't ask before committing" pre-answers a permission; the escape hatch beats hours of goal reinterpretation.
- The Autonomous run playbook wakes itself with Claude Code's `/loop` and `ScheduleWakeup`, using watcher subagents for events like CI.
- **A duration is not a finish condition.** "work on this for 4 hours" gives the loop nothing to check.
- Check permissions first (section 1).

Morning audit:

- `/mh-harness:show-me-your-work catch me up on what you did last night` summarizes the TSV decision log (`decisions.tsv` or `.audit/<task>.tsv`) with an Attention section. Read that section first.
- `/mh-harness:ketchup` in the chat that ran the work: what changed since your last message, plus each decision waiting on you with a pick, evidence, and risk. Read-only.
- `/mh-harness:bro` restates the last reply in plain words.

Bigger than one task:

| Shape | Playbook | Prompt |
|---|---|---|
| Queue of independent PRs, merge authority granted | Autopilot-full | "full autopilot on this queue, each item independent, merged by morning" |
| Coupled changes, you land them | Autopilot-stack | "autopilot these five changes but stack them, don't ship" |
| Multi-day program, many PRs | Orchestrate | "orchestrate the store migration. own it until every package is merged" |

Work that must outlive the session runs as `claude --cloud` sessions (see poteto-mode's Subagents section). Orchestrate keeps its store under `~/.claude/projects/<workspace-slug>/orchestrate/`. If one agent could finish in a session, Orchestrate routes you back to the overnight contract.

## 8. Steer with principle names

Mid-run corrections are one line. The right name does more than a paragraph:

    i said the goal is to repro. i did not ask for a fix yet.
    apply prove it works. show me the real output, not the build log.
    /mh-harness:principle-laziness-protocol

`ls skills | grep principle-` lists all 24. The ones you'll use most: prove-it-works, fix-root-causes, subtract-before-you-add, laziness-protocol, attack-the-premise, guard-the-context-window, never-block-on-the-human.

## 9. Make it yours

- **A session that taught you something → `/mh-harness:reflect that took too long. capture what we learned.`** Three reviewers (judgment and divergent on `opus`, tooling on `sonnet`) read this session's transcript; a synthesizer sorts proposals into Accepted, Rejected, and Backlog. Nothing changes until you approve rows. Edits to harness skills land in this repo, so bump the version afterwards. Approve a row only if it would change a future decision.
- **Same mistake twice → `/mh-harness:correct`.** `reflect` improves skills from one session; `correct` changes the repo so the mistake class can't come back. It mines commits, reverts, and review comments for mistake classes, then fixes each at the strongest level that works: architecture, then types or lint, then a test, then a written rule. It proves each new check fails on a real past mistake.
- **Writing a skill → `/mh-harness:poteto-mode write a skill for verifying migrations in this repo`.** Routes to the Authoring a skill playbook (with `writing-for-agents`), which validates frontmatter and links. Don't write a `SKILL.md` freehand. A skill that drives your app is a verification skill: use `create-verification-skill` instead.
- **Testing a skill change → `/mh-harness:poteto-mode run the eval playbook on this skill change. same task for both variants, candidates stay blind.`** Candidates never see the words "eval" or "candidate"; one judge scores under neutral labels. Read the outputs yourself before trusting the verdict.
- **Docs → `/mh-harness:technical-writing review the readme changes`.**
- Don't edit a skill mid-task because it misbehaved. Fix it in its own change and bump the version.

## Pitfalls

- Listing skills in the prompt. State goal and constraints.
- A done check that can't fail ("make it better").
- Leading with your theory of the cause.
- Taking the first design instead of prototypes or `architect`.
- Adversarial review of a plan with no code.
- Parallel agents in one worktree.
- Looping before the loop can verify itself (get a verify skill working first), or before permissions allow it to run.
- Trusting an unvetted number. Run `benchmark-checklist`.
- Correcting the same mistake by hand instead of `correct`.
- `arena` for coverage (use `swarm`).
- Accepting every review comment. `interrogate` sorts and dismisses with reasons; you can override.
- Reporting success off a green build.
- Forgetting the `mh-harness:` prefix, or forgetting to bump the version after editing the harness.
