---
name: system-handbook
description: Build a System Engineering Handbook in the current project: audit the codebase, scaffold the docs tree, model the architecture in Structurizr, write ADRs and the glossary, and wire it into AGENTS.md.
---

Build a documentation-first **System Engineering Handbook** in the existing project you are working in. It becomes the persistent source of system intent and architectural context for future AI-assisted development. Run it once per repo, like `/setup-matt-pocock-skills`.

The project must remain simple. Do **not** create a CLI, server, database, custom documentation application, synchronization service, or new AI framework. The solution is Markdown files, a Structurizr DSL model, ADRs, `AGENTS.md`, and optionally `CONTEXT.md`. The `docs/` tree must also work as an **Obsidian vault** (plain Markdown + relative links + optional YAML frontmatter opens as a vault with no configuration).

## Compose with the other skills

This skill builds the handbook; the other skills maintain it afterwards:

- **`system-handbook-maintain`** is the model-invoked counterpart that keeps the handbook in sync as the code evolves: it updates `workspace.dsl`, module docs, and behavior docs when implementation changes them, and delegates glossary/ADR work to `domain-modeling`.
- **`domain-modeling`** writes and sharpens the glossary and ADRs once the handbook exists. Point it at the handbook homes (`docs/01-domain/glossary.md`, `docs/03-architecture/decisions/`) through the repo's `docs/agents/domain.md` when it exists, or an `AGENTS.md` pointer.
- **`grill-with-docs`** closes the unknowns this setup marks "Unknown / Requires clarification".
- **`to-spec`** keeps publishing specs to the issue tracker (the working surface); the handbook holds the durable record.
- **`research`** fills documentation gaps against primary sources.

## Process

### 1. Audit the existing project

Before creating or modifying documentation, inspect the repository thoroughly. Understand: project structure, applications, backend/frontend/mobile components, modules, packages, services, databases, external integrations, messaging systems, authentication/authorization, deployment, CI/CD, tests, existing documentation, existing ADRs, existing architecture diagrams, existing `AGENTS.md`, existing `CONTEXT.md`, existing AI-agent configuration, existing skills, existing terminology.

Inspect the actual implementation. Do not assume the architecture from filenames alone. Do not invent components that do not exist. Do not "improve" the architecture during setup unless explicitly necessary to describe what already exists. The first objective is to document the current system accurately.

### 2. Preserve existing documentation

Identify existing documentation, ADRs, architecture docs, and project conventions. Do not duplicate information unnecessarily. If existing documentation is useful, integrate it into the new structure where practical. Do not delete valuable existing documentation. If migration would be risky, keep the existing document and reference it from the handbook.

### 3. Scaffold the handbook structure

Create the `docs/` tree:

```text
docs/
├── README.md
├── 00-system/
│   ├── vision.md
│   ├── scope.md
│   ├── goals.md
│   └── constraints.md
├── 01-domain/
│   ├── glossary.md
│   ├── personas/
│   ├── use-cases/
│   ├── workflows/
│   └── business-rules/
├── 02-product/
│   ├── modules/
│   ├── requirements/
│   └── specifications/
├── 03-architecture/
│   ├── README.md
│   ├── workspace.dsl
│   ├── decisions/
│   ├── quality-attributes/
│   ├── security.md
│   ├── reliability.md
│   ├── data.md
│   ├── integrations.md
│   └── deployment.md
├── 04-engineering/
│   ├── development.md
│   ├── testing.md
│   ├── observability.md
│   └── conventions.md
└── 05-operations/
    ├── deployment.md
    ├── backup.md
    ├── recovery.md
    └── runbooks/
```

Do not create empty files just to satisfy the structure. If a category has no meaningful information yet, create its directory but avoid fabricating content.

### 4. Write the main index (`docs/README.md`)

Act as the main navigation page for both humans and AI agents. Use Markdown links compatible with normal Markdown and reasonably friendly to Obsidian (relative links). The structure includes: System, Domain, Product, Architecture, Engineering, Operations. Include a short explanation of the handbook's purpose and explain the relationship between: Intent, Domain, Requirements, Architecture, Specifications, Code, Operations.

### 5. Document the system (`docs/00-system/`)

Write `vision.md`, `scope.md`, `goals.md`, `constraints.md`. Do not invent business goals. If the repository does not contain enough information to confidently determine something, mark it `Unknown / Requires clarification` rather than guessing.

### 6. Document the domain (`docs/01-domain/`)

- **Glossary** (`glossary.md`): the canonical terminology, the project's ubiquitous language. For every important domain concept prefer: Term, Definition, Aliases, Related concepts. Avoid introducing multiple names for the same concept. Use the `CONTEXT-FORMAT.md` style from `domain-modeling` (`_Avoid_` lists) when it helps.
- **Personas**: only for personas reasonably inferred from the system (role, goals, responsibilities, relevant permissions, important interactions). Do not invent organizational roles without evidence.
- **Use cases**: actor, goal, preconditions, main flow, relevant alternative flows, failure scenarios.
- **Workflows**: important business workflows. Prefer concise Mermaid diagrams when a visual explanation materially improves understanding; do not create diagrams merely for decoration.
- **Business rules**: rules that affect system behavior. Separate `Business rule` from `Implementation detail`.

### 7. Document the product (`docs/02-product/`)

Each meaningful business module gets its own document under `modules/` (e.g. `authentication.md`, `billing.md`, `users.md`) with: Purpose, Responsibilities, Owned domain concepts, Dependencies, Main use cases, Business rules, Specifications, Architectural decisions. Only document modules that actually exist or are clearly established.

Under `requirements/` and `specifications/`, capture meaningful existing behavior and important system capabilities. Specifications describe behavior rather than implementation. A useful structure: Purpose, Actors, Preconditions, Functional behavior, Business rules, Success criteria, Failure cases, Edge cases, Security considerations, Consistency requirements, Observability requirements. Do not rewrite the entire application as artificial requirements.

### 8. Model the architecture in Structurizr (`docs/03-architecture/workspace.dsl`)

Use the **C4 model**. The model must reflect the actual current architecture. Model where applicable: people, software systems, containers, important components, external systems, databases (as containers tagged `Database`), message brokers, external integrations. Keep it at Context + Containers level; do not create excessive component-level detail. Use meaningful relationships that communicate architectural intent:

```dsl
workspace {

    model {
        user = person "User"
        api = softwareSystem "API" {
            web = container "Web App"
            db = container "Database Schema" {
                tags "Database"
            }
        }
        user -> api "Uses"
        web -> db "Reads from and writes to"
    }

    views {
        systemContext api {
            include *
            autolayout lr
        }
        container api {
            include *
            autolayout lr
        }
    }

    documentation {
        api !docs docs
    }

    adrs {
        api !adrs docs/03-architecture/decisions
    }
}
```

Attach the Markdown documentation and ADRs to the model with `!docs` and `!adrs` (honoured by Structurizr `local`/`push`/`export`, not the browser editor). The architectural model stays a plain `.dsl` file; the Markdown stays regular Markdown. Do not create a second documentation format.

### 9. Write architecture documentation (`docs/03-architecture/README.md`)

Explain: architectural style, major system boundaries, major containers, important dependencies, data ownership, communication patterns, synchronous/asynchronous interactions, external integrations, major architectural constraints. Use the Structurizr model as the canonical architectural graph.

### 10. Write ADRs (`docs/03-architecture/decisions/`)

Only create ADRs for meaningful architectural decisions (database selection, architectural style, message broker, event-driven architecture, synchronization strategy, consistency model, authentication architecture, external integration strategy, deployment architecture, significant infrastructure decisions). Do not create ADRs for trivial implementation choices. Use the concise ADR template in [adr-template.md](./adr-template.md), numbered sequentially (`NNNN-slug.md`), in a format Structurizr's `!adrs` can import: first line `# NNNN. Title`, a `Date: YYYY-MM-DD` line, `## Status`, then `## Context`.

If the project already has ADRs, preserve their history and integrate them rather than replacing them. ADRs are immutable: supersede rather than rewrite.

### 11. Document quality attributes (`docs/03-architecture/quality-attributes/`)

Document meaningful non-functional requirements: performance, availability, scalability, reliability, security, maintainability, observability. Do not invent numerical targets. If the repository does not establish an SLA, latency target, throughput target, or availability target, mark it as undefined.

### 12. Document reliability, security, data, integrations, deployment

In `security.md`, document the actual security architecture: authentication, authorization, roles, permissions, secrets, trust boundaries, external integrations, sensitive data, security-critical workflows. Do not invent security mechanisms. Clearly distinguish `Current behavior` from `Recommended future improvement`.

In `reliability.md`, capture meaningful failure behavior: retries, timeouts, idempotency, dead-letter handling, transaction boundaries, consistency guarantees, failure recovery, synchronization behavior, eventual consistency, duplicate message handling, external dependency failures. Focus on behavior and guarantees.

In `data.md` and `integrations.md`, document data ownership, storage, and external integrations as evidenced by the code.

### 13. Write engineering and operations docs

In `docs/04-engineering/`, document actual project conventions: development workflow, testing strategy, code organization, observability, important engineering conventions. Only document information that helps an engineer or AI agent make correct decisions. Do not duplicate every linting rule or framework convention.

In `docs/05-operations/`, document actual operational behavior: deployment, infrastructure, backups, recovery, monitoring, alerts, operational runbooks. Do not invent infrastructure that is not present.

### 14. Integrate `AGENTS.md`

Inspect the existing `AGENTS.md`. If it exists, modify it carefully instead of replacing it. If it does not exist, create it. Establish these principles:

- **System Handbook**: `/docs/README.md` is the entry point. Before meaningful architectural or behavioral changes, inspect the relevant documentation.
- **Source of Truth**: the handbook represents system intent; the code represents implementation. If documentation and code disagree, identify the discrepancy instead of silently assuming which one is correct.
- **Architecture**: the Structurizr model is `/docs/03-architecture/workspace.dsl`. Architectural changes should update the model.
- **Domain**: the canonical terminology is defined in `/docs/01-domain/glossary.md`. Avoid introducing synonyms for established domain concepts.
- **ADRs**: meaningful architectural decisions must be documented in `/docs/03-architecture/decisions/`.
- **Specifications**: behavioral changes should have an appropriate specification.
- **Documentation Drift**: when implementation changes domain behavior, business rules, architectural boundaries, integrations, data ownership, important reliability behavior, or security behavior, the relevant documentation must be reviewed and updated.

Add this as its own section (e.g. `## System Handbook`) so it does not collide with the `## Agent skills` block that `/setup-matt-pocock-skills` owns.

### 15. Reconcile `CONTEXT.md`

Check whether the project already uses `CONTEXT.md`. If it does, preserve it. If appropriate, update it to point agents toward `docs/README.md`, `docs/01-domain/glossary.md`, `docs/03-architecture/README.md`, `docs/03-architecture/workspace.dsl`, and `docs/03-architecture/decisions/`. Do not turn `CONTEXT.md` into another copy of the handbook: it stays a concise entry point. The glossary lives in the handbook; `CONTEXT.md` points at it. Where the repo uses `docs/agents/domain.md` (written by `setup-matt-pocock-skills`), update it so `domain-modeling` reads the glossary and ADRs from the handbook homes.

### 16. Compose, do not duplicate

Do not copy, fork, replace, or recreate the skills. The existing skills remain responsible for engineering workflows; the handbook provides system context. Use the appropriate existing skills: `grill-with-docs` when requirements, domain behavior, or architectural decisions are unclear; `prototype` when significant design uncertainty exists; `to-spec` when requirements need to become an actionable product specification; `to-tickets` when work needs decomposition; `diagnosing-bugs` when debugging complex behavior; `improve-codebase-architecture` when evaluating architectural improvements; `handoff` when transferring work between agents. Integrate through repository context and instructions, not duplication.

## Don't over-document

This setup is not an excuse to generate hundreds of pages. Prefer accurate, concise, structured, navigable, useful over large, verbose, repetitive. If information is unknown, say so. If something is obvious from the code and has no architectural or business significance, do not document it.

## Final validation

After creating the structure, verify:

1. `docs/README.md` provides a coherent entry point.
2. The domain glossary reflects actual terminology.
3. Personas are grounded in the existing system.
4. Product modules reflect actual architecture.
5. Specifications reflect actual behavior.
6. `workspace.dsl` accurately models the current architecture.
7. Important architectural decisions have ADRs.
8. ADRs do not contain fabricated historical claims.
9. Markdown links are valid.
10. Structurizr documentation references are valid.
11. `AGENTS.md` correctly explains the handbook.
12. Existing skills are not duplicated.
13. `CONTEXT.md` is concise and points toward the handbook if applicable.
14. The documentation can be opened directly as an Obsidian vault.
15. No CLI, service, database, or custom infrastructure was introduced.

## Final report

At the end, give the user a concise report containing:

- **Existing Architecture**: what you discovered.
- **Documentation Created**: the important new files/directories.
- **Structurizr Model**: what is represented.
- **ADRs**: the ADRs created and why each was necessary.
- **AI Integration**: how `AGENTS.md`, `CONTEXT.md`, the handbook, Structurizr, and the skills interact.
- **Unknowns**: important architectural or product questions that could not be determined from the repository.
- **Documentation Debt**: areas where the project lacks sufficient information.

Do not silently invent answers for these gaps.

## Reference

- [adr-template.md](./adr-template.md): the Structurizr-compatible ADR template.
- [docs-tree.md](./docs-tree.md): the scaffold layout with the per-file purpose notes.
- [workspace.dsl.example](./workspace.dsl.example): a minimal C4 workspace to adapt.