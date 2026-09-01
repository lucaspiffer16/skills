---
"mattpocock-skills": minor
---

Go **OpenCode-first** and drop the native Claude Code and Codex integrations.

- Remove the Claude Code plugin (`.claude-plugin/plugin.json` + `marketplace.json`), its version-sync script (`scripts/sync-plugin-version.mjs`), and the related `package.json` scripts. skills.sh (`npx skills add mattpocock/skills`) is now the single distribution route; it already targets OpenCode (`-a opencode`), installing to `.agents/skills/` or `~/.config/opencode/skills/`.
- Remove the Codex per-skill metadata (`agents/openai.yaml` beside every `SKILL.md`) and the `policy.allow_implicit_invocation` gate.
- Remove `disable-model-invocation: true` from user-invoked skills. OpenCode gates them instead with `permission.skill: "deny"` in `opencode.json`; a ready config ships at `.agents/opencode.json.example`.
- Retire the Claude-specific `claude-handoff` and `git-guardrails-claude-code` skills. `handoff` covers the general case; nothing replaced the git hooks utility.
- Update `scripts/link-skills.sh`, the invocation docs, READMEs, and docs pages to OpenCode-first wording.