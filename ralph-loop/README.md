# Ralph Loop

Ralph Loop feeds the agent the same prompt after every turn until the task is done or an iteration limit is hit. The agent sees its own previous edits in the working tree and git history and improves on them: the prompt stays fixed, the code changes. It implements Geoffrey Huntley's [Ralph Wiggum technique](https://ghuntley.com/ralph/).

Use it for tasks with a verifiable finish line: tests passing, a migration complete, a feature built to spec. Keep tasks that need human judgment or have fuzzy goals out of the loop.

## Installation

```
/add-plugin ralph-loop
```

## Quick start

> Start a ralph loop: "Build a REST API for todos. CRUD operations, input validation, tests. Output COMPLETE when done." --completion-promise "COMPLETE" --max-iterations 50

- `--completion-promise <text>`: the loop ends when a response contains `<promise><text></promise>`.
- `--max-iterations <N>`: the loop ends after N iterations. Default: unlimited, so always set it.

Other skills: **cancel-ralph** stops the loop by deleting its state; **ralph-loop-help** explains the technique.

## How it works

Two hooks in the Cursor plugin hook format drive the loop, with state in `.cursor/ralph/`:

1. `afterAgentResponse` checks each response for the `<promise>` tag.
2. `stop` fires at the end of each turn. If no promise was seen and the limit isn't reached, it sends the original prompt back as a `followup_message`.

## Writing good prompts

Give the agent explicit completion criteria it can check:

```markdown
Build a REST API for todos.

When complete:
- All CRUD endpoints working
- Input validation in place
- Tests passing (coverage > 80%)
- Output: <promise>COMPLETE</promise>
```

Break large tasks into phases, and include a test-and-fix cycle in the prompt.

## Learn more

- [Original technique by Geoffrey Huntley](https://ghuntley.com/ralph/)
- [Ralph Orchestrator](https://github.com/mikeyobrien/ralph-orchestrator)

## License

MIT
