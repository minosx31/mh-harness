---
name: reflect
description: Spawn three parallel review subagents over the active transcript, surface learnings, and route each to a concrete edit on an existing skill. Use when the user says reflect.
disable-model-invocation: true
---

# Reflect

Mine the current conversation for durable learnings, then route them into skill edits.

Invoke when the user says "reflect" or "/reflect". Skip when the conversation is trivial, off-topic, or already covered by an existing skill the parent followed correctly. One-offs are not learnings.

## 1. Locate the active transcript

Before fanning out, find this conversation's transcript under `~/.claude/projects/<slug>/`, where `<slug>` is the workspace path with each "/" turned into "-". Your scratchpad path, when the system prompt names one, ends in this session's id, so `<slug>/<session-id>.jsonl` is the file. Do not glob across `~/.claude/projects/*/`. That crosses workspace boundaries and reads private chats from unrelated projects.

Without a session id, list the newest candidates:

```bash
ls -t ~/.claude/projects/<slug>/*.jsonl | head -10
```

Subagent transcripts sit under `<slug>/<session-id>/subagents/`. For each candidate, find the first record with `"type":"user"`. Take the file whose message text contains the conversation's opening user prompt. If none matches, pass a tight digest of the session instead.

## 2. Spawn three reviewers in parallel

One message, three Agent calls, `subagent_type: "general-purpose"`. Reviewers need MCP access to look up the tickets, chat threads, and observability traces the transcript references. Each template already tells the reviewer not to edit files.

| Lens | `model` | `effort` | Prompt template |
|---|---|---|---|
| Judgment | `opus` | `xhigh` | `references/judgment-reviewer.md` |
| Tooling | `sonnet` | `high` | `references/tooling-reviewer.md` |
| Divergent | `opus` | `xhigh` | `references/divergent-reviewer.md` |

Each seat gets a fresh context and its own lens, per the Subagents section of the **poteto-mode** skill. Pass each template verbatim, substituting the transcript path or digest where marked. Reviewers return findings in the Agent result.

## 3. Synthesize

One Agent call, `subagent_type: "general-purpose"`, `model: "opus"`, `effort: "xhigh"`. It spot-verifies citations through MCP. Pass `references/synthesizer.md` verbatim, with each reviewer's full output inlined where marked. It returns an Accepted / Rejected / Backlog list.

## 4. Structural enforcement check

Move any Accepted item that a lint rule, script, metadata flag, or runtime check would enforce more reliably to Backlog. See the **encode-lessons-in-structure** principle skill.

## 5. Apply

Present the synthesizer's full Accepted / Rejected / Backlog output and wait for explicit approval before applying any Accepted edit. The user picks the subset and may redirect routings. Skill changes affect every future agent in the org. Do not auto-apply.

List each Backlog item in the summary. File it to a tracker only when the user names one. Only the Accepted list waits for approval.

Never edit a skill under `~/.claude/plugins/cache/`. That copy is replaced on the next plugin update. Before approval, show each edit's destination:

- An `mh-harness` skill: its source in the harness repo, the folder the `mh-local` marketplace points at (`claude plugin marketplace list` shows it). After editing, bump `version` in `.claude-plugin/plugin.json`, per the repo README.
- A project skill: `.claude/skills/<skill>/`. A personal skill: `~/.claude/skills/<skill>/`.
- Any other plugin's skill: copy the folder to `~/.claude/skills/<skill>/` and edit the copy.

Never open a PR or push.

Follow each approved row's Routing exactly:

- Trivial existing-skill edit (a one-line bullet, a tightened sentence, a stale fact corrected): the parent does it directly.
- Substantive existing-skill edit (a new section, a new pattern table, more than ~10 lines): write it through the **writing-for-agents** skill.
- `tune description: <skill path>` (the skill exists but didn't trigger when it should have): rewrite the description through **writing-for-agents**. Skip skills with `disable-model-invocation: true`, which never trigger on their own; every `mh-harness` skill is one.
- `new skill via create-skill: <kebab-name>`: follow the Authoring a skill playbook (`skills/poteto-mode/playbooks/authoring-a-skill.md`). Do not invent the shape ad hoc.

If your environment ships a SKILL.md validator, run it on every touched skill before declaring done.

## 6. Summarize for the user

Short list, no preamble:

- Edits applied: `<skill path>`. What changed, one line each.
- New skills created: `<skill path>`. One line each (rare).
- Backlog: `<pattern>`, and where it was filed if anywhere. One line each.
- Dropped: one line per rejected finding + reason from the synthesizer.
