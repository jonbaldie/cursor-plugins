# Docs Canvas

Renders documentation (architecture notes, design docs, API references, runbooks, codebase walkthroughs) as a navigable Cursor Canvas instead of a flat markdown file.

**Status: initial scaffold.** The skill structure is complete; the body is a starting outline, not a tuned playbook.

## Requirements

- Cursor with Canvas enabled. The skill reads Cursor's built-in canvas skill (`~/.cursor/skills-cursor/canvas/SKILL.md`) first, so it runs only where that file exists.
- Source material: a directory of markdown files, a single doc URL, an inline outline, or a codebase question to answer.

## When to use

- Turning a docs directory, or one large doc, into a Canvas with jump navigation.
- Answering a codebase question with sections, diagrams, tables, and callouts rather than one reply.

Trigger phrases: "docs canvas", "documentation overview", "architecture walkthrough", "API reference page", "render this doc as an interactive canvas".

## Layout

Every docs canvas includes, in order:

1. **Overview**: purpose, scope, audience.
2. **Table of contents**: pinned list of jump targets.
3. **Body sections**: one per logical unit (architecture, API, examples, gotchas), mixing prose, code, diagrams, callouts.
4. **References**: related docs, source files, RFCs, external material.

That is the minimum. Add whatever representation helps the reader for the topic: diagrams, tables, decision trees, worked examples.

## License

MIT
