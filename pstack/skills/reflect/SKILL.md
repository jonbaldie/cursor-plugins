---
name: reflect
description: Spawn three parallel review subagents over the active transcript, surface learnings, and route each to a concrete edit on an existing skill. Use when the user says reflect.
disable-model-invocation: true
---

# Reflect

Mine the current conversation for durable learnings, then route them into skill edits.

## When to invoke

Invoke when the user says "reflect" or "/reflect". Skip when the conversation is trivial, off-topic, or already covered by an existing skill the parent followed correctly. One-offs are not learnings.

## Process

### 1. Locate the active transcript

The parent finds its own transcript file before fanning out. Use the active workspace's session transcripts, wherever your agent stores them (check the system prompt or the agent's docs; most write one JSONL file per session, one message per line). Do not glob across other workspaces' transcripts. That crosses workspace boundaries and reads private chats from unrelated projects.

```bash
ls -t <transcripts-dir>/*.jsonl <transcripts-dir>/*/*.jsonl 2>/dev/null | head -10
```

Layouts vary by agent: flat (`<id>.jsonl`), nested per session (`<id>/<id>.jsonl`), and sometimes separate subagent files under the parent session.

For each candidate, read the first user message and check that it contains the conversation's opening user prompt. Take the matching path. If no path resolves, write a tight digest of the session and pass that instead.

### 2. Spawn three reviewers in parallel

Read `~/.config/pstack/models.md` when present (written by `/setup-pstack`); a missing line falls back to the default here.

One message, three general-purpose subagent calls, explicit model on each, full tool access, not read-only. Reviewers need MCP access for context lookups (tickets, chat threads, observability traces referenced in the transcript). Some agents strip MCP access from read-only subagents.

| Lens | Model | Prompt template |
|---|---|---|
| Judgment | your configured reflect-judgment model (default Claude Fable 5.1 (max reasoning)) | `references/judgment-reviewer.md` |
| Tooling | your configured reflect-tooling model (default GPT-5.6 Sol (max reasoning)) | `references/tooling-reviewer.md` |
| Divergent | your configured reflect-judgment model (default Claude Fable 5.1 (max reasoning)) | `references/divergent-reviewer.md` |

Pass each template verbatim, substituting the transcript path or digest where marked. Reviewers return findings in the subagent response body.

### 3. Synthesize

One general-purpose subagent call, using your configured reflect-judgment model (default Claude Fable 5.1 (max reasoning)), full tool access, not read-only. The synthesizer's quality check includes spot-verifying citations, which can require MCP access. Some agents strip MCP access from read-only subagents. Use `references/synthesizer.md` verbatim, with each reviewer's full output inlined where marked. The synthesizer returns a structured Accepted / Rejected / Backlog list.

### 4. Structural enforcement check

Sanity-check the synthesizer's Accepted list. For any item that would be enforced more reliably by a lint rule, script, metadata flag, or runtime check, move it from Accepted to Backlog. See the **encode-lessons-in-structure** principle skill.

### 5. Apply

Before applying any Accepted edit, present the synthesizer's full Accepted/Rejected/Backlog output to the user and wait for explicit approval. The user picks which subset to apply and may redirect routings. Skill changes affect every future agent in the org. Do not auto-apply.

Backlog items file to whatever devex / backlog tracker your team uses automatically. Only the Accepted list waits for approval.

For each approved Accepted item, follow the Routing field exactly:

- Trivial existing-skill edit (a one-line bullet, a tightened sentence, a stale fact corrected): parent does directly.
- Substantive existing-skill edit (a new section, a new pattern table, more than ~10 lines): hand to your agent's skill-authoring skill, if it has one, and run its draft / test / iterate loop.
- `tune description: <skill path>` (the skill exists but didn't trigger when it should have): hand to the skill-authoring skill, if your agent has one, and run its description-optimization loop.
- `new skill: <kebab-name>`: hand creation to the skill-authoring skill, if your agent has one. Do not invent the shape ad hoc.

If your environment ships a SKILL.md validator, run it on every touched skill before declaring done. Skip this step if it doesn't.

### 6. Summarize for the user

Short list, no preamble:

- Edits applied: `<skill path>`. What changed, one line each.
- New skills created: `<skill path>`. One line each (rare).
- Backlog filed to the devex tracker: `<issue title>` (`<tags>`). One line each.
- Dropped: one line per rejected finding + reason from the synthesizer.
