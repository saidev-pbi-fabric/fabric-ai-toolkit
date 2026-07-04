# Personal toolkit staging

Local-only staging area, deliberately created **outside** `OneDrive - Sustana` (that folder syncs
to Sustana's managed cloud tenant — not a safe home for anything meant to be your own asset).
This folder lives at `C:\Users\schannamsetti\personal-toolkit-staging\` and does not sync
anywhere on its own.

**What this is for:** a holding area for content that's ready to become your personal dev-toolkit
repo, the moment the personal machine + GitHub account are confirmed ready (see
`Power Bi Cleanup\_Personal-Brand\ROADMAP.md`, Phase 3 Step 0). When that happens: copy this
folder's contents into the new repo (or clone the repo here and move things in), `git init`,
first commit, push. Then this local copy can be deleted — the repo becomes the source of truth.

## What's in here

- `skills/` — 3 skills genericized out of `~/.claude/skills/` (Sustana references stripped,
  including real workspace/dataset GUIDs and a Key Vault URL that were hardcoded in the original
  `fabric-notebook` skill — those are placeholders now). Ready to drop into a personal
  `~/.claude/skills/` directory as-is.
  - `code-review/SKILL.md`
  - `fabric-notebook/SKILL.md`
  - `email/SKILL.md`
- `oss-harness/ARCHITECTURE.md` — Phase 3 prep draft: the generalized pattern behind the agentic
  Fabric/PBI dev harness, written in the abstract, no client specifics. Raw material for the
  eventual public repo's README, not a locked design.

## Full skill portability audit (all 47 installed skills, 2026-07-04)

**Portable as-is** (generic mechanics, no Sustana coupling — copy straight over once the personal
repo exists): `algorithmic-art`, `brand-guidelines`, `diagnose`, `doc-validate`, `find-skills`,
`frontend-design`, `grill-me`, `handoff`, `html-doc`, `html-prd`, `humanizer`, `linkedin`,
`linkedin-analyzer`, `pdf`, `personal-brand`, `power-bi-dax`, `power-bi-deployment`,
`power-bi-diagnostics`, `power-bi-docs`, `power-bi-filters`, `power-bi-modeling`,
`power-bi-pages`, `power-bi-partitions`, `power-bi-report`, `power-bi-security`,
`power-bi-themes`, `power-bi-visuals`, `powerbi-report-authoring.md`,
`powerbi-report-design.md`, `powerbi-report-planning.md`, `semantic-model-authoring.md`,
`semantic-model-consumption.md`, `setup-matt-pocock-skills`, `skill-creator`, `task-observer`,
`teach`, `thread`, `theme-factory`, `ui-ux-pro-max`, `wayfinder`, `web-artifacts-builder`,
`writing-great-skills`, `xlsx`. The `powerbi-modeling-mcp` MCP server itself is also generic
(Tabular Object Model operations, no client data baked in) — portable.

**Genericized in this folder already** (see `skills/` above): `code-review`, `fabric-notebook`,
`email`. Note `fabric-notebook` needed more than a naming pass — the original had real DEV/PRD
workspace + dataset GUIDs and a live Key Vault URL hardcoded in a reference table; those are
stripped to placeholders in the copy here.

**Stays put, Sustana/DePere-specific, do not port**: `daily-update`, `friday-update`,
`weekly-update`, `new-report`, `sustana-doc`, `fabric-proposal`, `html-build`, `html-review`.

## Status
Created 2026-07-04 as unblocked prep work while Phase 3's real prerequisites (personal machine
readiness, IP-clause check) are still open. Nothing here has been pushed anywhere or shared
publicly — it's local staging only.
