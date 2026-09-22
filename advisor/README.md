# Advisor

Advisor lets the agent consult a stronger second model at three checkpoints: before a major decision, when it is stuck, and before it declares a task done. The main model keeps doing the work. The advisor reads a briefing (and the conversation transcript when available) and returns a verdict with concrete guidance. You pay for the strong model only where it matters.

The advisor is a read-only subagent with its own model, so it can come from a different model family than the one you chat with. The default is Grok 4.6 at its highest reasoning effort (xhigh).

## Installation

```bash
/add-plugin advisor
```

## Quick start

```text
/advisor                     turn on with the default advisor
/advisor composer            turn on with a different model (a slug or a family name)
/advisor ask is this migration safe to run twice?
/advisor status
/advisor nudge off           stop the end-of-turn reminder
/advisor off
```

Then work as usual. At each checkpoint the agent consults the advisor and reports in a line or two:

```text
Advisor (<model>): proceed with changes. The retry wrapper hides the real
failure; the 401 comes from a stale token cache. Dropped the retry, fixed the cache key.
```

## Checkpoints

| Checkpoint | Trigger |
| --- | --- |
| Major decision | Choosing between approaches; migrations, deletions, public API or config changes, dependency swaps, auth or payment code; requests ambiguous enough to change the work. |
| Stuck | Same error or failing test after two real attempts; unexplained behavior; about to add a workaround (retry loop, `sleep`, broad `try/except`, skipped test, disabled check). |
| Before declaring done | Any task that changed logic or touched more than a couple of files. Trivial edits are skipped, and the agent says so. |
| On request | `/advisor ask ...`, or asking what the advisor thinks. |

The skill caps this at about four consults per task. A typical feature takes one to three.

## Parts

- **Skill `advisor`**: the `/advisor` command and the checkpoint protocol. See [SKILL.md](skills/advisor/SKILL.md) and the [briefing template](skills/advisor/references/briefing-template.md).
- **Agent `advisor-subagent`**: read-only, answers with `Verdict / Why / Recommendations / Risks / Answers / Confidence`.
- **Hooks** (Cursor plugin hook format): `afterFileEdit` marks unreviewed edits, `subagentStop` counts each consult and appends it to `log.md`, and `stop` posts one `[Advisor]` reminder when a turn ends with unreviewed edits. The reminder fires once per batch of edits and stays quiet when the turn ended with a question for you. Without hooks, the skill still runs the pre-completion consult itself, but no transcript path is recorded.

## Models

`/advisor <model>` takes any model slug available to subagents, or a family name such as `grok fast`, which resolves to that family's latest model at its highest reasoning tier. If the slug is rejected, the skill picks the closest valid one, saves it and tells you. Team model restrictions and plan limits apply as for any subagent.

## State

State lives in `.cursor/advisor/` at the project root. The hooks read and write this path. Deleting it is safe at any time. Add it to `.gitignore` to keep it out of the repository; the skill never stages it.

| File | Purpose |
| --- | --- |
| `state.json` | Mode, model, nudge setting, consult count, bound conversation, transcript path. |
| `log.md` | Every completed consult with its verdict. |
| `pending` | Marker: files changed since the last consult. |
| `last-response.txt` | Tail of the latest reply, used to skip the reminder when you were asked a question. |

State is per project and bound to the conversation that ran `/advisor`. Running `/advisor` in a second conversation re-binds it there: model and nudge settings carry over; consult history and advisor context start fresh. `/advisor status` looks without re-binding.

## License

MIT
