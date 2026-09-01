# Handbook tree

The scaffold layout with per-file purpose. Create a directory even when it will hold no content yet; create a file only when you have something real to write. Do not create empty files just to satisfy the structure.

```text
docs/
├── README.md                      ← main navigation index (humans + AI agents)
│
├── 00-system/                     ← why the system exists
│   ├── vision.md                  ← the reason the system exists
│   ├── scope.md                   ← what is in and out of scope
│   ├── goals.md                   ← business goals the system serves
│   └── constraints.md             ← non-negotiable limits
│
├── 01-domain/                     ← the problem space
│   ├── glossary.md                ← canonical terminology (ubiquitous language)
│   ├── personas/                  ← one doc per inferred persona
│   ├── use-cases/                 ← one doc per meaningful use case
│   ├── workflows/                 ← important business workflows (Mermaid where useful)
│   └── business-rules/            ← rules that affect system behavior
│
├── 02-product/                    ← what the product must do
│   ├── modules/                   ← one doc per real business module
│   ├── requirements/              ← meaningful existing behavior and capabilities
│   └── specifications/            ← behavior-focused specifications
│
├── 03-architecture/               ← how the system is built
│   ├── README.md                  ← architectural style, boundaries, communication
│   ├── workspace.dsl              ← Structurizr C4 model (canonical graph)
│   ├── decisions/                 ← ADRs (Structurizr-importable format)
│   ├── quality-attributes/        ← performance, availability, scalability, etc.
│   ├── security.md                ← actual security architecture
│   ├── reliability.md             ← failure behavior and guarantees
│   ├── data.md                    ← data ownership, storage
│   ├── integrations.md            ← external integrations
│   └── deployment.md              ← deployment architecture
│
├── 04-engineering/                ← how engineers and agents work here
│   ├── development.md             ← development workflow
│   ├── testing.md                 ← testing strategy
│   ├── observability.md           ← logging, metrics, tracing
│   └── conventions.md             ← important engineering conventions
│
└── 05-operations/                 ← how the system runs
    ├── deployment.md              ← how it ships
    ├── backup.md                  ← backup behavior
    ├── recovery.md                ← recovery behavior
    └── runbooks/                  ← one runbook per operational procedure
```

## Obsidian compatibility

The whole `docs/` directory must work as an Obsidian vault. Prefer Markdown, relative Markdown links, a clear document hierarchy, stable filenames, and meaningful document titles. YAML frontmatter is supported by Obsidian natively and may be used selectively for important entities (modules, specifications, decisions); do not add it merely for decoration. Obsidian-specific syntax may be used when it provides significant value, but the same files must remain useful through GitHub, Structurizr, standard Markdown viewers, AI agents, and code editors.

## Documentation relationships

Where useful, establish explicit relationships between personas, use cases, requirements, specifications, modules, architecture, ADRs, and implementation, through Markdown links, filenames, identifiers, or YAML metadata. Avoid maintaining the same relationship manually in multiple places unless necessary.