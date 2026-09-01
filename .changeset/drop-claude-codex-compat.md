---
"mattpocock-skills": minor
---

Remove all remaining Claude Code and Codex harness compatibility, keeping the set OpenCode-first.

- `AGENTS.md` is now the real agent-instructions file at the repo root; the `CLAUDE.md` file and the `AGENTS.md -> CLAUDE.md` symlink are removed. The repo no longer ships a Claude-named steering file.
- `setup-matt-pocock-skills` now writes its `## Agent skills` block to `AGENTS.md` only (creating it with confirmation if absent), instead of preferring a leftover `CLAUDE.md`.
- Every live instruction that named Claude Code or Codex as a way to reach a skill or configure a project now says the generic thing: `writing-for-agents`, `handoff`, `grill-me`, `grilling`, `wait-what`, `wayfinder`, `tdd`, `ask-matt`, `wizard`, `code-review`, `resolving-merge-conflicts`, `codebase-design`, `improve-codebase-architecture`, `diagnosing-bugs`, `research`, `to-tickets`, and `teach` docs pages updated. Bug-report evidence that happened to name a harness was kept as evidence or reworded to "across harnesses".
- README and `.agents/install-block.md` now state the set is OpenCode-first with no Claude Code or Codex integration, instead of saying the skills still run there.