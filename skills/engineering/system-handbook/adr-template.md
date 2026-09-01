# ADR Template

ADRs live in `docs/03-architecture/decisions/` and use sequential numbering: `NNNN-slug.md` (zero-padded, e.g. `0001-record-system-handbook.md`). This layout and first-line format are what Structurizr's `!adrs` directive imports, so keep the filename and the `# NNNN. Title` / `Date:` / `## Status` / `## Context` skeleton stable.

## Template

```md
# NNNN. {Short title of the decision}

Date: YYYY-MM-DD

## Status

Accepted

## Context

Why this decision was necessary.

## Decision

What was decided.

## Alternatives

What alternatives were considered.

## Rationale

Why this option was selected.

## Consequences

### Positive

...

### Negative

...

## Related

- Related requirements
- Related modules
- Related specifications
```

## Numbering

Scan `docs/03-architecture/decisions/` for the highest existing number and increment by one. Numbers are sequential and monotonic; they are not reused. ADRs are immutable: when a decision is reversed, keep the old ADR and add a new one marked `superseded by ADR-NNNN`, never rewrite history.

## When to offer an ADR

All three of these must be true (the same bar as `domain-modeling`):

1. **Hard to reverse**: the cost of changing your mind later is meaningful.
2. **Surprising without context**: a future reader will look at the code and wonder "why on earth did they do it this way?".
3. **The result of a real trade-off**: there were genuine alternatives and you picked one for specific reasons.

If a decision is easy to reverse, skip it. If it is not surprising, nobody will wonder why. If there was no real alternative, there is nothing to record. Do not create ADRs for trivial implementation choices.

What qualifies: architectural shape, integration patterns between contexts, technology choices that carry lock-in, boundary and scope decisions, deliberate deviations from the obvious path, constraints not visible in the code, and rejected alternatives when the rejection is non-obvious.

## Integration

If the project already has ADRs elsewhere (e.g. `docs/adr/`), preserve them: keep the existing files, and reference or re-home them under `docs/03-architecture/decisions/` with a note pointing at the original so history is not lost.