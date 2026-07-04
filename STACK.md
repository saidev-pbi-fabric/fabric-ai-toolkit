# The stack

What actually goes into agentic Fabric/Power BI development, credited honestly — what
Microsoft ships and recommends, what the community built on top of it, and what's mine.

## Microsoft-official

- **[microsoft/skills-for-fabric](https://github.com/microsoft/skills-for-fabric)** — Microsoft's
  own agent-skill collection for Fabric, built for GitHub Copilot CLI, Claude Code, and
  compatible tools. Ships two plugins: `fabric-skills` (general Fabric workloads) and
  `powerbi-authoring` (semantic model authoring/deployment/AI-readiness + PBIR report
  authoring/design/planning). This is the primary skill layer this toolkit builds on top of,
  not around.
- **[microsoft/powerbi-modeling-mcp](https://github.com/microsoft/powerbi-modeling-mcp)** —
  Microsoft's MCP server that brings Power BI semantic modeling to AI agents directly: create
  DAX queries from natural language, execute and validate them against a live model, connect to
  either a local Desktop instance or a Fabric workspace, read/write TMDL. This is what turns
  "the agent wrote some DAX" into "the agent validated it against the real model" — the
  verification step this toolkit's [harness pattern](oss-harness/ARCHITECTURE.md) depends on.
- **Power BI Desktop** with live-connection/Bridge support, and **TMDL/PBIR** — Microsoft's own
  plain-text formats for semantic models and reports, the reason any of this is possible: a
  coding agent can only work like a coding agent if the thing it's editing is actually text.
- **[Tabular Editor](https://tabulareditor.com/)** — the established tool for direct TOM-level
  model editing outside Desktop; relevant background for anyone going deeper than the MCP
  server's surface.

## Community, built for exactly this use case

- **[MinaSaad1/pbi-cli](https://github.com/MinaSaad1/pbi-cli)** — a Power BI CLI purpose-built
  for Claude Code: semantic model + PBIR report operations, token-efficient by design, an
  interactive REPL. Used here as a fallback alongside the MCP server above, not a replacement
  for it.

## Mine

- **`skills/`** in this repo — genericized versions of my own custom skills, on top of the
  stack above, not duplicating what it already does.
- **`oss-harness/ARCHITECTURE.md`** — the pattern for combining all of the above into a
  reliable loop (plan → author → validate → reload → screenshot-verify), not just a faster way
  to generate DAX.

If you're setting this up yourself: install `skills-for-fabric` and `powerbi-modeling-mcp`
first (they do the heavy lifting), then layer this repo's `skills/` on top.
