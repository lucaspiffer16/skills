## What it does

`system-handbook` builds a documentation-first **System Engineering Handbook** in an existing project: the persistent source of system intent and architectural context for future AI-assisted development. It audits the repository, scaffolds a `docs/` tree across System, Domain, Product, Architecture, Engineering and Operations, models the current architecture in a Structurizr C4 `workspace.dsl`, writes the glossary and ADRs, and wires everything into `AGENTS.md` (and `CONTEXT.md` where the project uses one).

Its defining constraint: it documents what exists, never what the codebase should become. It inspects the actual implementation rather than assuming from filenames, invents no components, "improves" no architecture, and marks anything it cannot determine as `Unknown / Requires clarification` instead of guessing. It is documentation-first, so the project stays simple: Markdown, Structurizr DSL, ADRs, `AGENTS.md`, and optionally `CONTEXT.md`. No CLI, server, database, or new tooling.

The whole `docs/` tree is built to open as an Obsidian vault (plain Markdown, relative links, selective YAML frontmatter), while staying useful through GitHub, Structurizr, standard Markdown viewers, and agents.

## When to reach for it

Type `/system-handbook`, or the agent reaches for it automatically when a task fits. In OpenCode it is gated `permission.skill: "ask"`, so the agent proposes it and the human approves before it runs.

Reach for it once per repo, before the architecture and domain knowledge of a project are scattered and only lived in people's heads. It is the [setup](https://www.aihero.dev/ai-coding-dictionary/harness)-time companion to [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills): that skill configures the engineering workflow (issue tracker, triage labels, doc layout), this one creates the standing system knowledge the engineering skills then maintain. The agent offers it automatically right after setup completes when the repo has no handbook yet. A repo already midway through a project is a fine place to run it; the skill reads what is already there and preserves existing documentation rather than replacing it.

| Your situation | What to reach for |
| --- | --- |
| You want the architecture and intent written down, as a durable record for humans and agents | `system-handbook` |
| You want the engineering workflow configured (tracker, labels) | [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) |
| A term or decision is fuzzy mid-work | [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |
| A documentation gap needs primary-source research | [research](https://aihero.dev/skills-research) |

## Prerequisites

It writes into the repo you run it in. It needs nothing preinstalled: Structurizr models are plain `.dsl` files (rendered with `structurizr export` or the playground), and Obsidian opens a folder as a vault with no configuration. If you want the engineering skills to read the handbook's glossary and ADRs, run [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) first so its `docs/agents/domain.md` can point at the handbook homes.

## The lifecycle it encodes

The handbook follows the software lifecycle: Discovery → Domain → Requirements → Product → Architecture → Architecture Decisions → Specification → Implementation → Testing → Operations → Evolution. The handbook holds the persistent knowledge through the whole loop, Structurizr holds the architectural model, Markdown holds the human- and agent-readable knowledge, Obsidian provides the exploration interface, and `AGENTS.md` connects the repository knowledge to agents. The skills provide the engineering execution.

The part that makes it more than a snapshot is the **documentation-drift clause** written into `AGENTS.md`: when implementation changes domain behavior, business rules, architectural boundaries, integrations, data ownership, or reliability and security behavior, the relevant documentation must be reviewed and updated. That turns the handbook into a source of intent that stays honest, rather than a document that goes stale the day after it is written.

## How it composes, not duplicates

It deliberately does not copy, fork, or replace the skills. The skills remain responsible for engineering workflows; the handbook provides system context. The one genuine addition is the Structurizr C4 model: nothing in the set models architecture as a verifiable graph. The reconciliations that keep it from duplicating the skills' own outputs:

- **One glossary.** The glossary lives in `docs/01-domain/glossary.md`; `CONTEXT.md` becomes a concise pointer to the handbook, not a second copy.
- **One ADR record.** ADRs live in `docs/03-architecture/decisions/`, in a format Structurizr's `!adrs` imports, reusing `domain-modeling`'s bar for when an ADR is worth writing.
- **Specs stay on the tracker.** [to-spec](https://aihero.dev/skills-to-spec) keeps publishing working specs to the issue tracker; the handbook holds the durable record.
- **Two AGENTS.md blocks.** [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) owns the `## Agent skills` block; the handbook owns its own `## System Handbook` section.

## Common questions

**Do I need Structurizr installed to run it?**

No. The model is a plain `workspace.dsl` file, the durable artifact you commit. Rendering diagrams is an optional local step (`structurizr export` or the Structurizr playground); nothing in the skill requires a running tool. Note that Structurizr Lite, the CLI, cloud, and on-premises are end of life as of 2026; the maintained rendering path is `structurizr export` or `local`/`push`/`server`.

**It collides with my existing `CONTEXT.md` / `docs/adr/`.**

It preserves them. Existing glossary and ADR homes are kept and referenced rather than silently replaced; the skill re-homes the canonical record into the handbook structure and updates the pointers (`docs/agents/domain.md`, `AGENTS.md`) so the skills read from the handbook homes going forward. If migration would be risky, it keeps the existing document and references it.

**Will it invent facts?**

No. The audit phase inspects the actual implementation, and anything it cannot determine is written as `Unknown / Requires clarification`, which then becomes material for [grill-with-docs](https://aihero.dev/skills-grill-with-docs) to resolve. ADRs are only created for decisions that meet the bar (hard to reverse, surprising without context, a real trade-off), and never with fabricated historical claims.

## It's working if

- `docs/README.md` is a coherent entry point a fresh agent or human can navigate from.
- The glossary, modules, specifications, and `workspace.dsl` all describe the architecture that actually exists, with nothing invented.
- `AGENTS.md` explains the handbook and the source-of-truth relationship (intent vs code), and carries the documentation-drift clause.
- The `docs/` tree opens as an Obsidian vault with no configuration, and the same files render on GitHub.
- No CLI, server, database, or custom infrastructure was introduced.
- The closing report lists the unknowns and documentation debt instead of papering over them.

## Where it fits

`system-handbook` is a **run-once setup** like [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills), the precondition that makes the system's knowledge exist before the engineering flow runs, and it is **model-invoked** so the setup skill can offer it right after configuring the repo (gated `ask` in OpenCode, so the human approves before it runs). After it runs, its readers and maintainers are the skills themselves: [domain-modeling](https://aihero.dev/skills-domain-modeling) keeps the glossary and ADRs sharp, [system-handbook-maintain](https://aihero.dev/skills-system-handbook-maintain) keeps the model in sync, [to-spec](https://aihero.dev/skills-to-spec) and [to-tickets](https://aihero.dev/skills-to-tickets) drive the work, [research](https://aihero.dev/skills-research) fills gaps, and [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) feeds the model. For which skill to reach for next, [ask-matt](https://aihero.dev/skills-ask-matt) routes the whole set.