# Exploratory pass: pstack skills under `~/Code/pstack` (2026-09-22)

## Scope and setup

- **Question:** do the de-Cursored pstack skills work when installed as `.agents/skills` with a `.claude/skills -> ../.agents/skills` symlink?
- **Source:** local fork `jonbaldie/cursor-plugins` at upstream `53e579f` (pstack 0.15.2), plus the uncommitted de-Cursor edits (60 files).
- **Install:** `~/Code/pstack/.agents/skills` (47 skills, byte-identical to `pstack/skills/` per `diff -rq`), plus the symlink `~/Code/pstack/.claude/skills`.
- **Agents:** Claude Code (`claude -p`, headless) and Codex (`codex exec`, read-only sandbox).
- **Personal config:** ordinary personal skill folders present (`~/.claude/skills`, `~/.agents/skills`, `~/.codex/skills`). No `~/.config/pstack/models.md`.
- **Evidence:** [`2026-09-22-folder-structure/`](./2026-09-22-folder-structure/) holds the Codex `bro` run, the test output and the checked plan template. Session transcripts and the `how` stream logs stayed local in `/tmp/pstack-explore-2026-09-22/`, because they contain full agent context.

## Journeys

| # | Journey | Result | Evidence |
|---|---------|--------|----------|
| 1 | Claude Code invokes pstack skills by slash through the symlink (`/architect`, `/recall`, `/principle-fix-root-causes`). A nonexistent skill is the negative control. | Pass. The negative control was rejected. | local only |
| 2 | Codex discovers pstack skills from `.agents/skills`. | Pass | local only |
| 3 | `how` answers a model-selection question with no models.md present. Codex (`$how`) and Claude (`/how`). | Pass in both. References resolve through the symlink. Claude spawned a general-purpose subagent. | local only |
| 4 | `bro` (the user's report): a jargon-heavy turn 1, then `bro` on the same session. Codex (`$bro`) and Claude (`/bro`). | Pass in both. The skill body was injected from `~/Code/pstack/.agents/skills/bro` (Codex) and `~/Code/pstack/.claude/skills/bro` (Claude). The output restated turn 1 plainly. | [turn 1](./2026-09-22-folder-structure/k1-codex-turn1.txt), [`$bro`](./2026-09-22-folder-structure/k2-codex-bro.txt); Claude session `a6e892b3…` local only |
| 5 | `worktree-audit.sh` with and without `PSTACK_TRANSCRIPTS_DIR`, in a scratch repo with one worktree. | Pass. Unset: warns, LAST_CHAT is `-`, bucket `review`. Set: LAST_CHAT is dated, bucket `verify-recent-chat`. | transcript of this session |
| 6 | `check-plan.mjs` against the plan template in `multi-phase-plan.md`. | Pass (0 problems) after the fix below. The upstream pair also passes. | [`tpl-plan.md`](./2026-09-22-folder-structure/tpl-plan.md) |
| 7 | poteto-mode scripts: `bun test` and `tsc`. | 52/52 pass in 4 consecutive runs. Typecheck is clean. | [`bun-baseline.txt`](./2026-09-22-folder-structure/bun-baseline.txt) |

## Confirmed findings

### F1. Personal skills shadow 4 pstack skills with different content

- **Impact:** in Claude Code, `/tdd`, `/teach`, `/create-verification-skill` and `/maintain-verification-skill` run your personal skill, not pstack's. You get no warning.
- **Conditions:** the same names exist in `~/.claude/skills` with different content. They also exist in `~/.agents/skills`, and `tdd` and `teach` also in `~/.codex/skills`.
- **Replay:** `cd ~/Code/pstack && claude -p "/tdd"`. The "Base directory for this skill" line reads `~/.claude/skills/tdd`.
- **Expected:** the project's pstack skill, or at least a visible clash.
- **Actual:** the personal skill wins. This is Claude Code's documented precedence, so it is a local configuration conflict, not a pstack bug. Codex lists several `tdd` entries with no clear precedence.
- **Repeat observations:** seen twice on the old copy. The same 4 clashes (and no others) were re-checked by name and content against the new 47-skill copy.

### F2. `check-plan.mjs` pinned a Cursor model slug that the de-Cursored template no longer contains (fixed)

- **Impact:** every plan written from the de-Cursored `multi-phase-plan.md` template would fail with `Verify, live lacks "Ten lanes on grok-4.6-fast-xhigh…"`.
- **Cause:** the checker's `LANES` constant duplicates template prose.
- **Fix:** the constant now matches the template: `Ten lanes on Grok 4.6 Fast (xhigh reasoning) at the PR head`. Verified by journey 6.

## Rejected / explained

- **"`/bro` doesn't work in Codex":** the fork was behind upstream, and `bro` didn't exist in the installed copy. After the merge and re-install, `$bro` works (journey 4).
  - Note that Codex invokes skills as `$bro`. A literal `/bro` in Codex's TUI was not tested headlessly.
- **First `bun test` run showed 44 tests, 2 failures and 1 error.** It did not reproduce in 4 later runs (52/52). That run overlapped with 4 agents editing files in the same tree. Unresolved, but most likely interference.

## Observations (not bugs)

- **Headless models.md read blocked:** Claude Code in `-p` mode blocked reading `~/.config/pstack/models.md` because it is outside the working directory. The skills fall back to their defaults as designed. Interactive users will see a permission prompt.
- **Codex description shortening:** Codex warns that it shortened skill descriptions to fit its context budget. There are 47 skills plus personal ones.
- **Frontmatter names don't match directories:** `poteto-mode` (`name: Poteto Mode`) and `make-bot-ui` (`name: Make Bot UI`). Both agents still load them. This is upstream's naming.
- **`codex exec resume --last`** picked another session that was open at the time and refused ("already has an active writer"). I resumed by explicit session ID instead. That refusal is Codex protecting the other session, not a failure of the skills under test.
- **Unrelated directory:** `~/Code/pstack/fuzz-lab/` appeared at 15:05–15:07, after the install. It was not created by this pass, so I left it untouched.

## Deliberate de-Cursor exceptions

- **`watch-pr/github.ts`** still recognises Bugbot (the `cursor` login and `CURSOR_AUTOMATION_ID`). Bugbot is a third-party GitHub review bot, and detecting its threads is interop with that bot, not coupling to the agent.
- **Kept:** the README author bio, the `.cursor-plugin/` packaging, "cursor location" in `why`, and GraphQL `endCursor`.

## Not explored

- Long-running orchestration skills (`poteto-mode` autopilot, `swarm`, `arena`), benny automations, and `setup-pstack` writing models.md end to end. These cost many subagent runs, and the question here was folder structure, not workflow quality.
