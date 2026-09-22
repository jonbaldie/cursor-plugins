---
name: setup-pstack
description: Configure which models pstack uses per role and at what reasoning budget. Detects your available models and writes a config file that overrides the skill defaults. Use for /setup-pstack, "configure pstack models", "pstack budget", or changing pstack's model choices.
---

# Setup pstack

Write `~/.config/pstack/models.md`, a plain config file that sets pstack's model per role. The skills read it when they delegate and fall back to their inline defaults when a line is absent.

## Steps

### 1. Detect available models

Enumerate the models you can pass to a subagent in this session. That is the dependable source. If your agent also exposes a models API or CLI that lists the user's entitled models, prefer it for completeness. If you cannot detect any, ask the user to paste the model identifiers they have access to. Never write a real identifier you have not confirmed is available. The aliases `inherit-parent` and `auto` are always valid even though they are not detected identifiers.

### 2. Load current state

The default role-to-model mapping is the file shape shown in step 5 below. The defaults are model names, not identifiers: resolve each to the identifier your agent uses for that model and reasoning level. If a default is unavailable, pick the closest available model (prefer the highest reasoning tier of the same family) or `inherit-parent`. If `~/.config/pstack/models.md` already exists, read it and treat its `# budget` line and its role values as the current choices. Otherwise start from those defaults.

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Prefer a structured question tool if your agent has one, otherwise a chat question with concrete options. Offer these four options with these exact labels, and name the current budget when the config records one.

- `unlimited — keep max`
- `large — xhigh reasoning`
- `medium — high reasoning`
- `small — medium reasoning`

**(b) Apply it.** Build the working table from the skill defaults, and on a re-run keep any role you changed by family, list, or alias (`inherit-parent`, `auto`). `unlimited` leaves every reasoning level as in that table. `large`, `medium`, and `small` set the reasoning level of every real model, panel entries included, to `xhigh`, `high`, or `medium`, on the ladder `max` > `xhigh` > `high` > `medium` > `low`. If the result is not a detected model, use the same family's detected model with the highest reasoning level at or below the target, else mark the role as needing a choice. `inherit-parent` and `auto` do not change. So `small` turns Claude Fable 5.1 (max reasoning) into Claude Fable 5.1 (medium reasoning), and Grok 4.6 Fast (xhigh reasoning) into Grok 4.6 Fast (medium reasoning), each written as whatever identifier your agent uses for that model and level.

**(c) Show the roles and confirm.** Show every role with its model, marking any real identifier not in the detected set as needing a choice. Ask whether to accept as-is or change specific roles, offering the detected models plus `inherit-parent` and `auto` (both mean: this role runs on the parent chat model, which keeps users of an auto or router model on it) as the options. Prefer a structured question tool if your agent has one, otherwise a chat question with concrete options. For panel roles (arena runners, architect runners, interrogate reviewers) the value is a list, and one subagent runs per entry, alias entries included, so the list length sets the count. `arena cross-judge pool` is also a list, but Arena selects one value from it whose model family differs from the parent's when possible. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

### 4. Validate

Every real identifier written must be in the detected set. `inherit-parent` and `auto` always pass. If a chosen real identifier is not available, stop and ask again.

### 5. Write the config

Create `~/.config/pstack/` if needed and write `~/.config/pstack/models.md` with a `# budget` line with the chosen label and its target reasoning level, and one line per role, using the same labels poteto-mode uses. Values are the real identifiers your agent accepts, resolved in step 2. Overwrite the whole file so re-runs stay idempotent. Shape, shown with the default model names:

```
# pstack model configuration. One line per role. Delete a line to fall back to the skill default.
# `inherit-parent` or `auto` as a value: the role runs on the parent chat model (omit the subagent `model`). Alias entries in a panel list still count toward its fan-out.
# budget: unlimited (max)
feature, refactoring: Grok 4.6 Fast (xhigh reasoning)
bug-fix: Grok 4.6 Fast (xhigh reasoning)
perf-issue: Grok 4.6 Fast (xhigh reasoning)
hillclimb: Grok 4.6 Fast (xhigh reasoning)
judgment and prose: Claude Fable 5.1 (max reasoning)
hardest tasks: Claude Fable 5.1 (max reasoning)
how explorer: Grok 4.6 Fast (xhigh reasoning)
how explainer: Claude Fable 5.1 (max reasoning)
why investigators: Grok 4.6 Fast (xhigh reasoning)
why synthesizer: Claude Fable 5.1 (max reasoning)
reflect tooling: GPT-5.6 Sol (max reasoning)
reflect judgment, divergent, synthesizer: Claude Fable 5.1 (max reasoning)
arena runners: Claude Fable 5.1 (max reasoning), GPT-5.6 Sol (max reasoning), Grok 4.6 Fast (xhigh reasoning), Claude Opus 5 (xhigh reasoning)
arena cross-judge pool: Claude Fable 5.1 (max reasoning), GPT-5.6 Sol (max reasoning), Grok 4.6 Fast (xhigh reasoning), Claude Opus 5 (xhigh reasoning)
swarm workers: Grok 4.6 Fast (xhigh reasoning)
architect runners: Claude Fable 5.1 (max reasoning), GPT-5.6 Sol (max reasoning), Grok 4.6 Fast (xhigh reasoning), Claude Opus 5 (xhigh reasoning)
interrogate reviewers: Claude Fable 5.1 (max reasoning), GPT-5.6 Sol (max reasoning), Grok 4.6 Fast (xhigh reasoning), Claude Opus 5 (xhigh reasoning)
```

### 6. Confirm

Tell the user the config was written and that pstack skills read it each time they delegate. Re-running this skill updates it.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke `/create-verification-skill` (resolves wherever pstack is installed: workspace, user, or plugin). On no, move on without pushing.
