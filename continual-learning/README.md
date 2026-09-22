# Continual Learning

Continual Learning keeps `AGENTS.md` up to date by mining your recent session transcripts for recurring corrections and durable workspace facts. It runs incrementally: only new or changed transcripts are read, and matching bullets are updated in place.

## Installation

```bash
/add-plugin continual-learning
```

## How it works

1. A `stop` hook (Cursor plugin hook format, run with `bun`) counts completed turns. When the cadence below is met, it posts a `followup_message` asking the agent to run the `continual-learning` skill.
2. The skill delegates everything to the `agents-memory-updater` subagent. It has `disable-model-invocation: true`, so it runs only from the hook or when you invoke it.
3. The subagent reads `AGENTS.md`, mines transcripts that are new or newer than its index, and updates two sections: `## Learned User Preferences` and `## Learned Workspace Facts`. Each holds at most 12 plain bullets. If nothing qualifies, it replies `No high-signal memory updates.`

State files, relative to the project root:

- `.cursor/hooks/state/continual-learning.json`: hook cadence state.
- `.cursor/hooks/state/continual-learning-index.json`: transcript index used by the subagent.

## Cadence

The hook triggers when all of these hold:

- at least 10 completed turns since the last run
- at least 120 minutes since the last run
- the transcript's mtime has advanced since the last run

Trial mode is off by default. Set `CONTINUAL_LEARNING_TRIAL_MODE=1` to use 3 turns and 15 minutes for the first 24 hours, then fall back to the defaults.

## Environment overrides

| Variable | Default |
| --- | --- |
| `CONTINUAL_LEARNING_MIN_TURNS` | 10 |
| `CONTINUAL_LEARNING_MIN_MINUTES` | 120 |
| `CONTINUAL_LEARNING_TRIAL_MODE` | off |
| `CONTINUAL_LEARNING_TRIAL_MIN_TURNS` | 3 |
| `CONTINUAL_LEARNING_TRIAL_MIN_MINUTES` | 15 |
| `CONTINUAL_LEARNING_TRIAL_DURATION_MINUTES` | 1440 |

Each also accepts the legacy `CONTINUOUS_LEARNING_*` name.

## License

MIT
