---
"mattpocock-skills": minor
---

Add the **`system-handbook`** skill (engineering, user-invoked): builds a documentation-first System Engineering Handbook in an existing project. It audits the repo, scaffolds a `docs/` tree across System, Domain, Product, Architecture, Engineering and Operations, models the current architecture in a Structurizr C4 `workspace.dsl`, writes the glossary and ADRs, and wires everything into `AGENTS.md` (and `CONTEXT.md` where used). The `docs/` tree opens as an Obsidian vault with no configuration.

- Composes with the existing skills instead of duplicating them: `domain-modeling` keeps the glossary and ADRs sharp (re-homed to `docs/01-domain/glossary.md` and `docs/03-architecture/decisions/`), `to-spec` stays the working-spec surface, `research` fills gaps, `improve-codebase-architecture` feeds the model.
- Carries a Structurizr-compatible ADR template (`NNNN-slug.md`, `# NNNN. Title` / `Date:` / `## Status` / `## Context`) so the `!adrs` directive imports the record, and a minimal `workspace.dsl` seed.
- Gated like the other user-invoked skills: `permission.skill."system-handbook": "deny"` in `.agents/opencode.json.example`, reached via `/system-handbook`.
- Routed in `ask-matt` under Precondition, listed in the top-level and Engineering READMEs, with a docs page at `docs/engineering/system-handbook.md`.