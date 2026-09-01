---
name: system-handbook-maintain
description: Keep the project's System Engineering Handbook in sync with the code as it evolves. Use when you make changes that affect architecture, modules, integrations, data ownership, reliability, security, or documented behavior, and the repo has a System Handbook (docs/README.md).
---

Maintain the project's **System Engineering Handbook** so it stays an accurate source of system intent as the code evolves. This is the model-invoked maintenance counterpart to `/system-handbook` (also model-invoked, gated `ask` in OpenCode, so the human approves before it builds). You reach for this automatically whenever your work changes something the handbook documents.

Call the Skill tool with "domain-modeling" when the change touches **domain vocabulary or a hard-to-reverse decision**: `domain-modeling` owns the glossary (`docs/01-domain/glossary.md`) and the ADRs (`docs/03-architecture/decisions/`). Do not duplicate its job here.

## When this fires

The handbook is present when `/docs/README.md` exists (see the repo's `AGENTS.md` "System Handbook" section if present). Fire this whenever your implementation changes:

- architectural boundaries or the container/system model
- modules, their responsibilities, or their dependencies
- external integrations
- data ownership or storage
- important reliability behavior (retries, idempotency, consistency, dead-letter handling)
- security behavior (auth, authorization, roles, trust boundaries, secrets)
- documented behavior in `docs/02-product/` (requirements, specifications) or business rules

This is the "Documentation Drift" clause from the handbook's `AGENTS.md` integration, made operative: if implementation changed one of these, the relevant documentation must be reviewed and updated in the same change.

## Process

### 1. Locate the handbook and what changed

Read `/docs/README.md` for the navigation structure. Identify the specific sections your change touches (a container, a module doc, a specification, a business rule, an integration). Inspect the actual diff you just made; do not re-audit the whole repo.

### 2. Update the architectural model (`docs/03-architecture/workspace.dsl`)

If your change added, removed, or renamed a container, software system, external integration, or database, update the Structurizr C4 model to match. Keep it at Context + Containers level. Update relationships only when the communication intent actually changed; do not rewrite the model for cosmetic reasons. Follow the existing model's style (element naming, tags, autolayout, `!docs`/`!adrs` directives).

### 3. Update product and behavior docs

Update the module doc under `docs/02-product/modules/` when responsibilities, owned domain concepts, or dependencies changed. Update the relevant specification under `docs/02-product/specifications/` or a business rule under `docs/01-domain/business-rules/` when documented behavior changed. Keep specifications behavior-focused, not implementation-level.

### 4. Update reliability, security, integrations, data

Touch `docs/03-architecture/reliability.md`, `security.md`, `integrations.md`, or `data.md` only when the change actually altered the documented behavior or guarantee. If the change merely reimplemented the same behavior, leave them alone. Distinguish `Current behavior` from `Recommended future improvement` in security docs.

### 5. Delegate domain vocabulary and ADRs to `domain-modeling`

When the change introduces a new domain term, resolves an ambiguity, or meets the ADR bar (hard to reverse, surprising without context, a real trade-off), call the Skill tool with "domain-modeling" so it writes to the glossary and ADR homes correctly. Do not write glossary entries or ADRs yourself here; that is `domain-modeling`'s job and it keeps them consistent.

### 6. Compose, do not duplicate

Do not create a second handbook structure, a second glossary, or a second ADR record. Update the existing handbook in place. Do not over-document: if the change is obvious from the code and has no architectural or business significance, leave the docs untouched.

## Don't over-document

Same rule as `/system-handbook`: prefer accurate, concise, useful. If the change does not alter any documented behavior or structure, do nothing. If you are unsure whether a doc is stale, flag it to the user rather than guessing.

## Done when

- Every handbook section your change touches is accurate against the current code.
- `docs/03-architecture/workspace.dsl` matches the current containers and integrations.
- `domain-modeling` was called (or already handled) for any new terms or decisions.
- No duplicated structure was introduced.
- Any documentation you could not confidently update is flagged to the user as an unknown.