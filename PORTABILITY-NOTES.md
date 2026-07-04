# Portability notes

How this repo's contents were chosen, for future reference when adding more.

## Full skill portability audit (all 47 installed skills, checked 2026-07-04)

**Portable as-is** (generic mechanics, no employer coupling): `algorithmic-art`,
`brand-guidelines`, `diagnose`, `doc-validate`, `find-skills`, `frontend-design`, `grill-me`,
`handoff`, `html-doc`, `html-prd`, `humanizer`, `linkedin`, `linkedin-analyzer`, `pdf`,
`personal-brand`, `power-bi-dax`, `power-bi-deployment`, `power-bi-diagnostics`,
`power-bi-docs`, `power-bi-filters`, `power-bi-modeling`, `power-bi-pages`,
`power-bi-partitions`, `power-bi-report`, `power-bi-security`, `power-bi-themes`,
`power-bi-visuals`, `powerbi-report-authoring.md`, `powerbi-report-design.md`,
`powerbi-report-planning.md`, `semantic-model-authoring.md`, `semantic-model-consumption.md`,
`setup-matt-pocock-skills`, `skill-creator`, `task-observer`, `teach`, `thread`,
`theme-factory`, `ui-ux-pro-max`, `wayfinder`, `web-artifacts-builder`, `writing-great-skills`,
`xlsx`. The `powerbi-modeling-mcp` MCP server itself is also generic (Tabular Object Model
operations, no client data baked in) — portable, not included here since it's a separate
install, not a skill file.

**Genericized and included in `skills/`**: `code-review`, `fabric-notebook`, `email`. Note
`fabric-notebook` needed more than a naming pass — the original had real DEV/PRD workspace and
dataset GUIDs plus a live Key Vault URL hardcoded in a reference table; those are placeholders
here.

**Deliberately excluded, employer-specific**: `daily-update`, `friday-update`, `weekly-update`,
`new-report`, `sustana-doc`, `fabric-proposal`, `html-build`, `html-review` — built for one
employer's stakeholder cadence and branding, not generally reusable.

## Why only 3 of the ~41 portable skills are here so far

These 3 were the ones that needed active genericizing (Sustana references or hardcoded values
to strip), so they were done first as the highest-value cleanup pass. The remaining ~41 already
generic skills can be added directly, no editing needed — that's a mechanical copy, not a
decision, so it's left for whenever the full sync happens rather than done piecemeal here.
