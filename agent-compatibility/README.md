# Agent Compatibility

Scores how well a repo holds up when an agent works in it. Run `check-agent-compatibility` for the full pass: it runs the published `agent-compatibility` CLI, fans out to four review agents, and returns one score plus the highest-leverage fixes.

```md
## Agent Compatibility Score: 72/100

Top fixes
- First issue
- Second issue
```

Ask for a breakdown to see the component scores and the reasoning behind them.

## Components

| Component | Kind | Checks |
|:----------|:-----|:-------|
| `check-agent-compatibility` | skill | Full pass; runs the four agents below |
| `compatibility-scan-review` | agent | Raw CLI-backed scan |
| `startup-review` | agent | Cold start and bootstrap |
| `validation-review` | agent | Verifying a small change |
| `docs-reliability-review` | agent | Docs against the real setup path |

## Score model

| Score | Meaning |
|:------|:--------|
| Agent Compatibility Score | Final blended score shown to the user |
| Deterministic Compatibility Score | Raw score from the CLI |
| Startup Compatibility Score | Guesswork needed to boot the repo |
| Validation Loop Score | How practical it is to verify a small change |
| Docs Reliability Score | How closely the docs match the real setup path |

```text
Agent Compatibility Score = round((deterministic * 0.7) + (workflow * 0.3))
```

The CLI also reports an accelerator layer for committed agent tooling. It shapes recommendations and leaves the deterministic score unchanged.

The scanner is heuristic: it scores repo signals and surfaces likely friction. Treat the score as a friction estimate, not a code-quality verdict.

## CLI

The plugin runs the published npm package on demand; it bundles no scanner.

```bash
npx -y agent-compatibility@latest .          # terminal dashboard
npx -y agent-compatibility@latest --json .   # also --md, --text
npx -y agent-compatibility@latest . --config ./agent-compatibility.config.json  # ignored paths, weight overrides
```

## Local install

For Cursor, symlink this directory to `~/.cursor/plugins/local/agent-compatibility`. For other agents, copy `skills/` and `agents/` into the directories your agent loads.
