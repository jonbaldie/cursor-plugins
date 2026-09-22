# PR Review Canvas

Renders a pull request diff as an interactive Cursor Canvas ordered by reviewer importance, not file-tree order.

## Requirements

- Cursor with Canvas enabled. The skill reads Cursor's built-in canvas skill (`~/.cursor/skills-cursor/canvas/SKILL.md`) first, so it runs only where that file exists.
- A diff source: a local branch or ref (`git diff`), a GitHub PR URL or number (`gh pr diff`), or a Graphite stack (`gt` or `gh`).

## When to use

- Reviewing a PR and you want the real risk surfaced first.
- Summarising a stack of PRs for a human reviewer.
- Any request for a diff walkthrough, change-set overview, or "what actually changed here".

Trigger phrases: "PR review canvas", "review this PR on a canvas", "walk me through this diff on a canvas", or a PR URL or branch ref.

## Layout

1. **Core logic**: new behaviour, algorithm changes, state transitions, API surface. Full diffs with context.
2. **Wiring and integration**: route registration, DI, config plumbing. Condensed.
3. **Boilerplate and mechanical**: imports, renames, generated code, formatting. A list; inline diffs only when directly relevant.

On top: pseudocode for dense logic, before/after example traces for behaviour that's hard to predict from the diff, and inline callouts (`Subtle`, `Breaking`, `Race condition`, `Perf`) for surprising or risky hunks.

## License

MIT
