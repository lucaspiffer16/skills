## What it does

`system-handbook-maintain` keeps a project's **System Engineering Handbook** accurate as the code evolves. Where [system-handbook](https://aihero.dev/skills-system-handbook) builds the handbook once, this skill is the ongoing maintenance pass: whenever implementation changes something the handbook documents (architecture, modules, integrations, data ownership, reliability, security, or documented behavior), it updates the relevant docs in the same change.

Its defining constraint: it updates the existing handbook in place, never a copy. There is one glossary and one ADR record, so it does not duplicate structure, and it delegates domain vocabulary and ADRs to [domain-modeling](https://aihero.dev/skills-domain-modeling) rather than writing them itself. It also holds the line against over-documentation: if a change is obvious from the code and alters no documented behavior, it leaves the docs untouched.

It is the "Documentation Drift" clause from the handbook's `AGENTS.md` integration, made operative: the handbook only stays a source of intent if something enforces that drift, and this skill is that something.

## When to reach for it

Type `/system-handbook-maintain`, or let the agent reach for it automatically when a task fits.

The agent fires it whenever the current work changes what the handbook documents, and `/docs/README.md` exists. You would reach for it directly after a change that touched architecture, a module, an integration, data ownership, or a documented behavior, when you want the docs brought back in sync. For a change to domain vocabulary or a hard-to-reverse decision, reach for [domain-modeling](https://aihero.dev/skills-domain-modeling) instead, which this skill calls internally anyway.

| Your situation | What to reach for |
| --- | --- |
| You just changed architecture, a module, or documented behavior, and want the handbook accurate | `system-handbook-maintain` |
| You want the handbook built in a repo from scratch | [system-handbook](https://aihero.dev/skills-system-handbook) |
| A term is fuzzy, or you made a hard-to-reverse decision | [domain-modeling](https://aihero.dev/skills-domain-modeling) |
| A documentation gap needs primary-source research | [research](https://aihero.dev/skills-research) |

## Prerequisites

It needs the handbook to exist (`/docs/README.md`) and the change to be in the working tree. It does not re-audit the whole repo; it reads the navigation index, identifies the sections your diff touches, and updates only those.

## How it composes, not duplicates

The split is clean: [system-handbook](https://aihero.dev/skills-system-handbook) creates, `system-handbook-maintain` maintains, [domain-modeling](https://aihero.dev/skills-domain-modeling) owns glossary and ADRs. This skill updates the architectural model (`docs/03-architecture/workspace.dsl`), the module docs under `docs/02-product/modules/`, and the behavior docs (specifications, business rules, reliability, security, integrations, data). Everything terminological or decision-shaped goes to `domain-modeling`, so the glossary and ADR record never have two authors.

## Common questions

**Will it rewrite my whole handbook every time?**

No. It reads the diff, updates only the sections the change touches, and does nothing when the change alters no documented behavior. The "don't over-document" rule applies: if it is obvious from the code and carries no architectural or business significance, it leaves the docs alone.

**Does it create ADRs and glossary entries itself?**

No. Those belong to [domain-modeling](https://aihero.dev/skills-domain-modeling), and this skill calls it rather than duplicating the discipline. That is what keeps one glossary and one ADR record.

**Is it useful without a handbook?**

No. It is the maintenance counterpart to [system-handbook](https://aihero.dev/skills-system-handbook); without the handbook there is nothing to maintain, and `domain-modeling` already covers the glossary/ADR case on its own.

## It's working if

- After an architecture-affecting change, `docs/03-architecture/workspace.dsl` matches the current containers and integrations.
- Module and behavior docs that the change touched are accurate against the code, and untouched docs are left alone.
- New terms and hard-to-reverse decisions show up as `domain-modeling` calls, not as ad-hoc glossary/ADR edits from this skill.
- No second handbook, glossary, or ADR structure was introduced.
- Anything it could not confidently update is flagged to you as an unknown rather than guessed.

## Where it fits

`system-handbook-maintain` is a **model-invoked maintenance** skill, the ongoing companion to the [system-handbook](https://aihero.dev/skills-system-handbook) run-once setup. It sits alongside [domain-modeling](https://aihero.dev/skills-domain-modeling) (glossary + ADRs) and [research](https://aihero.dev/skills-research) (gap-filling) as the set that keeps system knowledge alive after the handbook exists. For which skill to reach for next, [ask-matt](https://aihero.dev/skills-ask-matt) routes the whole set.