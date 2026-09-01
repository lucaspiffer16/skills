# System Engineering Handbook: alignment with the Matt Pocock skills

> **Update, 2026-09-01:** section 6 originally proposed `system-handbook` as a user-invoked orchestrator gated `deny`. It is instead shipped **model-invoked, gated `ask`** in OpenCode: model-invoked so `setup-matt-pocock-skills` can chain to it (a user-invoked skill cannot be called by another user-invoked skill), and `ask` so the human approves the high-blast-radius run before it executes. A separate model-invoked `system-handbook-maintain` skill keeps the handbook in sync afterwards. This document otherwise stands as the design record.

Research date: 2026-09-01. This document aligns the "System Engineering Handbook" proposal (a Markdown `docs/` tree, Structurizr C4 model, ADRs, Obsidian vault, AGENTS.md integration) with the existing skill set in this repo, so it can be delivered "integrating with the current pattern": as a skill, reusing the conventions the repo already owns.

## 1. What the proposal is

A documentation-first System Engineering Handbook for an existing project. A `docs/` tree in six areas: `00-system`, `01-domain`, `02-product`, `03-architecture` (with a Structurizr `workspace.dsl` and `decisions/` ADR folder), `04-engineering`, `05-operations`. It must open as an Obsidian vault, integrate with `AGENTS.md` and optionally `CONTEXT.md`, and follow a lifecycle: Discovery → Domain → Requirements → Product → Architecture → ADRs → Specification → Implementation → Testing → Operations → Evolution. Explicitly: no CLI, server, database, or new tool; Markdown + Structurizr DSL + ADRs + AGENTS.md is the whole solution.

## 2. What the current skills already cover (the pattern to integrate with)

- **Glossary / ubiquitous language**: `domain-modeling` writes and sharpens `CONTEXT.md` (single-context) or a `CONTEXT-MAP.md` (multi-context). Format: `CONTEXT-FORMAT.md`. It is a glossary and nothing else, deliberately free of implementation detail.
- **ADRs**: `domain-modeling` records ADRs in `docs/adr/` (format: `ADR-FORMAT.md`), lazily, only for decisions that are hard to reverse, surprising, or a real trade-off. Deliberately minimal: "an ADR can be a single paragraph."
- **Specs**: `to-spec` synthesises a spec (Problem Statement, Solution, User Stories, Implementation Decisions, Testing Decisions, Out of Scope) and publishes it to the **issue tracker**, applying a triage label. It does not interview the user.
- **Tickets**: `to-tickets` splits a plan/spec into tracer-bullet tickets on the tracker with blocking edges.
- **Research**: `research` spins up a background agent, investigates primary sources, and leaves a cited Markdown file "wherever the repo keeps such notes."
- **Setup**: `setup-matt-pocock-skills` writes `docs/agents/*.md` (issue-tracker, triage-labels, domain) and an `## Agent skills` block into `AGENTS.md`/`CLAUDE.md`.
- **Architecture survey**: `improve-codebase-architecture` scans for deepening opportunities and produces an HTML report.
- **Steering**: `writing-for-agents` is the reference for writing AGENTS.md, skills, and pointed-at docs.

## 3. Overlap map: the handbook vs the current skills

| Handbook section | Current skill/artifact | Overlap | Verdict |
| --- | --- | --- | --- |
| `docs/01-domain/glossary.md` | `CONTEXT.md` (domain-modeling) | Both are a canonical glossary. Two glossaries would drift. | **Merge, one source of truth.** |
| `docs/03-architecture/decisions/` | `docs/adr/` (domain-modeling) | Both are ADRs. Two locations and two formats would split the record. | **Re-home ADRs; keep the discipline.** |
| `docs/02-product/specifications/` | `to-spec` (publishes to issue tracker) | `to-spec` produces specs already, but lands them on the tracker, not in the handbook. | **Complementary: tracker for work, handbook for the durable record.** |
| `docs/02-product/requirements/` | `to-spec` problem statement / user stories | Partial. | Handbook adds the standing home. |
| `docs/03-architecture/workspace.dsl` | nothing | **Genuinely new**: the skill set has no architectural model/graph. | The biggest genuine addition. |
| `docs/04-engineering/` | `writing-for-agents`, repo AGENTS.md conventions | Overlap on conventions, but handbook is project-specific. | Complementary. |
| `docs/05-operations/` | nothing | **New**: runbooks, backups, recovery. | Additive. |
| `docs/00-system/` (vision/scope/goals) | nothing | **New**: intent layer. | Additive. |
| AGENTS.md integration | `setup-matt-pocock-skills` (writes its own block) | Both edit AGENTS.md. | Coexist if each owns its block. |
| CONTEXT.md pointing at handbook | `domain-modeling` (owns CONTEXT.md) | Handbook section 21 says CONTEXT.md stays a concise entry point. | Compatible with the "glossary" rule if the glossary itself lives in the handbook. |
| Obsidian vault | the repo's `docs/` pages (published at aihero.dev) | Different thing: those are human-facing skill docs, not a project knowledge vault. | Additive. |

## 4. What the handbook genuinely adds (beyond the skills)

1. **A machine-checkable architectural model.** Structurizr `workspace.dsl` is the one capability the skill set lacks. C4 says Context and Container diagrams are "recommended for all teams"; Component only if it adds value; Code never for long-lived docs. A handbook should model people, software systems, containers, external systems, databases (as containers tagged `Database`), and relationships, at Context + Containers level.
2. **A standing home for intent and product knowledge.** Vision, scope, goals, personas, use cases, workflows, business rules, requirements, specifications, quality attributes, security, reliability. The skills document *during* work (grill-with-docs, domain-modeling, to-spec); nothing holds the accumulated system knowledge afterwards.
3. **Operations and engineering context.** Runbooks, backup/recovery, deployment, observability, conventions. Not covered by the skills.
4. **A documentation-drift guardrail.** The proposal's AGENTS.md clause ("when implementation changes domain behavior, business rules, boundaries, integrations, data ownership, reliability, or security, the relevant docs must be reviewed and updated") is a standing rule the skills do not enforce. This is what makes the handbook a *source of intent*, not a snapshot.
5. **Obsidian as a navigation layer.** Plain Markdown + relative links + YAML frontmatter open as a vault with no `.obsidian` folder required; Obsidian reads the same files GitHub, Structurizr, and agents read.

## 5. The reconciliations (do not duplicate)

### 5.1 One glossary, not two
`CONTEXT.md` and `docs/01-domain/glossary.md` must not both exist. Cleanest fit with the existing pattern: the glossary lives in the handbook (`docs/01-domain/glossary.md`), written in the `CONTEXT-FORMAT.md` style (`Term`, definition, `_Avoid_`), and `CONTEXT.md` becomes a short entry-point that points agents at the handbook glossary plus the other handbook roots. This preserves `domain-modeling`'s discipline (it writes the glossary, just in its new home) and satisfies the handbook's rule that CONTEXT.md stays concise.

Alternative, if the project prefers the agent-facing `CONTEXT.md` as the living glossary: keep `CONTEXT.md` and have the handbook `glossary.md` link to it, not copy it. Either way: one source of truth, one pointer.

### 5.2 One ADR location, Structurizr-compatible
Move the ADR home from `docs/adr/` to `docs/03-architecture/decisions/` so the handbook structure holds the record. Adopt Structurizr's `!adrs` expectations so the model can import them:

- Filename `NNNN-slug.md` (zero-padded sequential id), e.g. `0001-record-system-handbook.md`.
- First line `# NNNN. Title`, a `Date: YYYY-MM-DD` line, `## Status`, then `## Context`.

This is compatible with `ADR-FORMAT.md`'s minimalism: Structurizr's `AdrToolsDecisionImporter` only needs the filename + first-line/Date/Status/Context to parse; extra sections (Decision, Alternatives, Rationale, Consequences, Related) are optional and can stay as the "only when they add value" rule. Keep `domain-modeling`'s bar for *offering* an ADR (hard to reverse, surprising, real trade-off) unchanged. Structurizr imports ADRs in adr-tools, madr, or log4brains format; use the adr-tools naming so the `!adrs` directive just works.

In the DSL: `!docs docs` at the workspace or system level attaches the handbook Markdown, `!adrs docs/03-architecture/decisions` attaches the ADRs. Note `!docs`/`!adrs` are honoured by Structurizr `local`/`push`/`export`, not the browser editor.

### 5.3 Specs: tracker for work, handbook for the record
`to-spec` keeps publishing specs to the issue tracker (that is the working surface). When a spec is settled and implemented, its durable form lands in `docs/02-product/specifications/` (and relevant `requirements/`), linked from the tracker issue. The handbook specifications use the behavior-focused structure from the proposal (Purpose, Actors, Preconditions, Functional behavior, Business rules, Success criteria, Failure cases, Edge cases, Security considerations, Consistency requirements, Observability). `to-spec`'s template stays the *working* spec; the handbook holds the *standing* spec.

### 5.4 AGENTS.md: two blocks, two owners
`setup-matt-pocock-skills` owns the `## Agent skills` block (tracker, labels, domain docs). The handbook owns its own block (System Handbook entry point, Source of Truth, Architecture, Domain, ADRs, Specifications, Documentation Drift). Both edit AGENTS.md; neither overwrites the other's section. This is already how the repo edits AGENTS.md in place.

### 5.5 Stale skill names in the proposal
The proposal's section 20 names skills that no longer exist: `to-prd` → **`to-spec`**, `to-issues` → **`to-tickets`**, `diagnose` → **`diagnosing-bugs`**. `grill-with-docs`, `prototype`, `improve-codebase-architecture`, `handoff`, `domain-modeling` are current and correct. Any skill built from this proposal must use the current names.

## 6. How it becomes a skill (integration with the current pattern)

The repo delivers capabilities as skills. The natural shape is a **user-invoked orchestrator skill** in `engineering/` that scaffolds the handbook into an existing project, reusing the conventions above:

**Name**: `system-handbook` (a project, not this repo: it runs inside the target project, like `setup-matt-pocock-skills`).

**What it does** (one run):
1. Audit the repo (structure, apps, modules, services, databases, integrations, auth, deployment, CI/CD, tests, existing docs/ADRs/diagrams/AGENTS.md/CONTEXT.md/skills/terminology) without assuming from filenames and without inventing components.
2. Preserve existing documentation; reference rather than migrate when risky.
3. Scaffold the `docs/` tree (directories only where content exists; no empty files).
4. Write `docs/README.md` as the navigation index (System, Domain, Product, Architecture, Engineering, Operations), explaining intent vs code.
5. Produce the glossary (`docs/01-domain/glossary.md`) and reconcile `CONTEXT.md` to point at it (or link to the existing one).
6. Model the current architecture in `docs/03-architecture/workspace.dsl` (C4 Context + Containers), with `!docs` and `!adrs` directives.
7. Write ADRs only for decisions that meet the bar (hard to reverse, surprising, real trade-off), in the Structurizr-compatible format, preserving any existing ADRs.
8. Write quality attributes / security / reliability / engineering / operations docs only where the repo evidences them; mark unknowns explicitly ("Unknown / Requires clarification") instead of guessing.
9. Update `AGENTS.md` (its own block) and CONTEXT.md per sections 19-21.
10. Run the final validation checklist (section 26 of the proposal) and report (section 27), including unknowns and documentation debt.

**How it composes with the existing skills**: it is the *onboarding* that makes the handbook exist; after setup, `domain-modeling` maintains glossary + ADRs, `to-spec`/`to-tickets`/`implement` drive the work, `research` fills gaps, `improve-codebase-architecture` feeds the model, `writing-for-agents` keeps AGENTS.md sharp, and the handbook's drift clause is enforced as a standing AGENTS.md rule that any skill reads.

**OpenCode integration**: like the other user-invoked skills, gate it with `permission.skill."system-handbook": "deny"` in `opencode.json`; the human reaches it via `/system-handbook`.

**Repo packaging**: SKILL.md + template files (the scaffold layout, the ADR/glossary/spec templates, a minimal `workspace.dsl` seed), a docs page at `docs/engineering/system-handbook.md`, a route in `ask-matt`, a line in the engineering README and the top-level README, and a changeset. Follows every rule in `CLAUDE.md` for a promoted engineering skill.

## 7. What is deliberately NOT added

- No duplicate glossary, ADR record, or spec pipeline. One source of truth each, per section 5.
- No restructuring of the existing skills. `domain-modeling`, `to-spec`, `to-tickets`, `implement`, `research`, `improve-codebase-architecture` are unchanged; the handbook re-homes some of their outputs (glossary, ADRs) and adds the standing record around them.
- No new tooling: Markdown, Structurizr DSL (a text file rendered by `structurizr export` / the playground), Obsidian (opens a folder as a vault with no config), git. That is the whole stack.

## 8. Risks and open decisions

- **ADR home change is a repo-wide convention change.** `domain-modeling`'s `ADR-FORMAT.md` and `CONTEXT-FORMAT.md` currently say `docs/adr/` and root `CONTEXT.md`. If the handbook relocates them, those two format files (and any project's existing ADRs) must be reconciled. Decision needed: relocate (cleaner, breaks existing layout) vs. keep `docs/adr/` and have Structurizr `!adrs` point there (less structure change, slightly weaker handbook coherence).
- **How much is automatic vs grilling.** The handbook's audit must not invent facts. Whether setup grills the user (grill-with-docs style) for the unknowns or marks them "Unknown / Requires clarification" determines how much is scaffolded in one pass. The proposal leans "do not guess": mark unknowns, then let `grill-with-docs` close them.
- **Structurizr end-of-life status.** Lite, the CLI, cloud, and on-premises are EOL as of 2026; the maintained path is `structurizr export` (free, OSS) or `local`/`push`/`server`. The DSL file itself is the durable artifact in git; rendering is an optional local step. Worth stating in the skill so nobody builds the workflow on a dead tool.
- **Spec duplication.** `to-spec` already writes to the tracker; mirroring into `docs/02-product/specifications/` is a second copy unless the handbook spec is written once, on approval. Decide the handoff so "spec" has one life.

## 9. Sources

- Structurizr DSL: https://docs.structurizr.com/dsl , https://docs.structurizr.com/dsl/language , https://docs.structurizr.com/dsl/example , https://docs.structurizr.com/dsl/docs , https://docs.structurizr.com/dsl/adrs , https://docs.structurizr.com/dsl/parser , https://docs.structurizr.com/export , https://docs.structurizr.com/eol
- C4 model: https://c4model.com , https://c4model.com/diagrams , https://c4model.com/diagrams/system-context , https://c4model.com/diagrams/container , https://c4model.com/diagrams/component , https://c4model.com/diagrams/code
- Obsidian: https://help.obsidian.md/Files+and+folders/How+Obsidian+stores+data , https://help.obsidian.md/Files+and+folders/Manage+vaults , https://help.obsidian.md/Linking+notes+and+files/Internal+links , https://help.obsidian.md/Editing+and+formatting/Properties
- ADRs: Michael Nygard, Documenting Architecture Decisions (http://thinkrelevance.com/blog/2011/11/15/documenting-architecture-decisions); https://github.com/joelparkerhenderson/architecture-decision-record ; Structurizr `AdrToolsDecisionImporter` (https://github.com/structurizr/structurizr/blob/main/structurizr-import/src/main/java/com/structurizr/importer/documentation/AdrToolsDecisionImporter.java)
- Skills referenced: `domain-modeling` (CONTEXT-FORMAT.md, ADR-FORMAT.md), `to-spec`, `to-tickets`, `research`, `setup-matt-pocock-skills`, `improve-codebase-architecture`, `writing-for-agents`, `grill-with-docs`