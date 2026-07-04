# fabric-ai-toolkit

![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)
![Microsoft Fabric](https://img.shields.io/badge/Microsoft-Fabric-0078D4)
![Power BI](https://img.shields.io/badge/Power-BI-F2C811)

**New here and not a coder? → [Start with GETTING-STARTED.md](GETTING-STARTED.md)** — plain
language, no jargon, a real first result before you need to understand anything else on this
page.

> An AI assistant with real, hands-on access to Power BI and Microsoft Fabric — one that checks
> its own work against the live data before calling anything done.

Personal toolkit for AI-agent-driven development on Microsoft Fabric and Power BI: reusable
skills for a coding agent (Claude Code or equivalent), plus the harness pattern that makes it
reliable, not just fast.

No client or employer data, names, or configuration anywhere in this repo — everything here is
generic and safe to run against any Fabric/Power BI project.

## What's in here

- **`skills/`** — my own Fabric/PBI-specific coding-agent skills, ready to drop into
  `~/.claude/skills/` (or the equivalent directory for another agent CLI):
  - `code-review/` — structured, severity-ranked review for PySpark, SQL, DAX, Power Query/M,
    and Fabric pipeline expressions.
  - `fabric-notebook/` — generates a well-structured Fabric PySpark notebook: standard 5-cell
    layout, logging, error handling, naming conventions baked in.
- **`oss-harness/ARCHITECTURE.md`** — the general pattern behind agentic Fabric/PBI development:
  the plan → author → validate → reload → screenshot-verify loop that catches bugs a code-only
  review would miss, and an honest list of what it doesn't solve.
- **`STACK.md`** — the full toolchain this builds on, credited honestly: what's Microsoft-official
  (skills-for-fabric, the Power BI Modeling MCP server, Tabular Editor), what's community-built
  for this exact use case (pbi-cli), and what's actually mine.

This repo is scoped to Fabric/Power BI agentic development specifically — general-purpose
skills (writing, docs, etc.) live elsewhere, not here.

## Using a skill

Copy the folder you want into your agent's skills directory, e.g.:
```
cp -r skills/code-review ~/.claude/skills/
```
Each skill is a single `SKILL.md` with YAML frontmatter (`name`, `description`) the agent reads
to decide when to use it — no build step, no dependencies beyond the agent itself.

See `PORTABILITY-NOTES.md` for the reasoning behind what did and didn't make it in here.

## License
MIT — see `LICENSE`.
