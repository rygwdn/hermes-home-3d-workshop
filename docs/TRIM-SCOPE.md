# TRIM-SCOPE — carving home-3d-workshop down to one home

Base: fork of Wijt/hermes-home-3d-workshop v0.2.0 (upstream HEAD a708d10), MIT.
Owner decisions (2026-11-25): trim the household machinery **in code** (P10);
plate-clearing stays **manual** — no auto-eject configuration, no printer mods (P11);
the fork stays **public** (P9). Printer model: not needed for the trim (P12 dismissed).

## What this fork becomes

A single-owner Hermes plugin for one home's Bambu Lab printer, managed by Bambuddy:
the agent reads status, finds/imports/designs/slices/queues parts; the owner approves
every start, typed; a zero-token watcher follows prints and tells the owner; finished
parts are removed from the plate by hand.

## Safety spine — MUST NOT be weakened by any layer

- `gates.py`: owner-only starts, typed approval, single-use passes, unattended/cron/
  kanban runs fail closed, voice asks never start, post-failure plate confirmation.
- `hooks.py`: the bounded `pre_tool_call` (status + queue only, ≤12 s budget).
- `watcher.py`: zero-token passes, backoff re-checks, one light cycle per pass.
- `backends/`: fail-closed Bambuddy parsing; simulator parity.
- `notify.py` outbox mechanics (states, retries) — kept for **owner** notices.
- `design.py`: sandboxed OpenSCAD render (time/CPU/mem/file limits, allowlist).
- `/home3d` command surface; `checks.py` doctor.

## Auto-eject: kept but unconfigured

P11 = manual. `eject.py`, `gcode.py` and the `autoeject_*` settings stay present but
empty (upstream's "no auto-eject" mode). No hardware story. Revisit only on owner
request.

## Layers, in order — one branch each, suite green per commit

**L1 — groups + tone.** Remove the `groups` config, group-target routing, tone
(family/team) message variants, group labels in the owner overview. Touches:
`config.py`, `notify.py`, `messages.py`, `watcher.py`, `workshop.py`, `store.py`
(group fields), `cli.py`, `commands.py`, `plugin.yaml`, tests. TS front end keeps
compiling; its group labels degrade cosmetically until L5.

**L2 — household/guests/policies.** Remove `household`, `allow_guests`,
`start_policy` (owner-only start remains), `problem_notices` (owner-only notices),
`family_target` (`owner_target` stays). Collapse `stop_policy` to `[owner, anyone]`,
default `owner`. Touches: `gates.py` call sites only (no safety-logic edits),
`hooks.py`, `tools.py`, `store.py`, `messages.py`, `notify.py`, `cli.py`,
`commands.py`, `checks.py`, `plugin.yaml`, tests.

**L3 — language duality.** English only: drop `language` config, all `tr` strings,
`lang` parameters, Turkish morphology helpers (`dative`, vowel-harmony sets).

**L4 — persona, skills, docs.** `persona_name` default → `Ponder`;
`household-printing` skill → lean English home-printing skill; README rewritten for
solo use with clear upstream credit; `plugin.yaml` description updated.

**L5 — front-end pass.** `src/` group/persona remnants (chip, sections, hero,
types), then rebuild `desktop/plugin.js` per upstream's esbuild build.

## Commit policy

- Branches: `home-trim-l1`, `home-trim-l2`, … merged into `home-trim` only after
  review. Conventional commits: `trim(l1): remove groups and tone routing`.
- Every commit keeps the full suite green; tests of removed machinery are deleted
  in the same commit. No orphan tests, no skipped-as-broken tests.

## Verification

- Suite: `/home/ponder/.hermes/cache/scratch/home3d-venv/bin/python -m pytest`
  (uv venv, pytest + pillow).
- Baseline on a708d10 (2026-11-25): **983 passed, 15 skipped**.
- Before any install: `hermes plugins validate` + `hermes home3d doctor` on a clean
  copy; install into the live Hermes only with owner go-ahead.

## Known-conflict map (for future upstream merges)

- `messages.py`: heavily rewritten by L1/L3 — expect conflicts everywhere.
- `config.py` + `plugin.yaml`: removed schema keys — conflict when upstream adds
  settings; prefer taking upstream keys and re-trimming.
- `gates.py`: only policy call sites change (L2); safety logic untouched — small
  conflicts.
- `store.py`: group/persona fields removed — upstream schema migrations may need
  manual reconciliation.
- Upstream credit: keep LICENSE, keep attribution in the README.
