# Ship the skill set OpenCode-first via skills.sh; no native Claude or Codex plugin

These skills have always been installable via [skills.sh](https://skills.sh/mattpocock/skills) (`npx skills add mattpocock/skills`), which copies editable skill files into a user's project across OpenCode, Claude Code, Codex, and other Agent-Skills-standard harnesses. A recurring request is a **plug-and-play** distribution: subscribe to the set as a read-only, always-current bundle you don't edit, rather than a fork you own. That is exactly what native plugin systems provide.

We ship **OpenCode-first via skills.sh** and ship **no** native Claude Code or Codex plugin. OpenCode reads the skills the repo already produces with zero adaptation, and skills.sh already installs them into OpenCode's discovery locations, so a plugin buys nothing for the primary harness.

## Why OpenCode-first

OpenCode reads the Agent-Skills `SKILL.md` convention natively from `.agents/skills`, `~/.config/opencode/skills`, and `~/.claude/skills`, all of which this repo already targets (see [scripts/link-skills.sh](../scripts/link-skills.sh) and the research at [.agents/research/opencode-integration.md](../.agents/research/opencode-integration.md)). The skills carry only `name` and `description` frontmatter, which OpenCode uses for its `skill` tool, so no conversion is needed.

OpenCode does **not** read `.claude-plugin/plugin.json` or `.codex-plugin/plugin.json`, has no plugin manifest of its own, and has no native skill marketplace. Its supported install paths for skills are the filesystem discovery locations and the `skills.urls` remote-registry protocol. skills.sh targets OpenCode directly (`npx skills add mattpocock/skills -a opencode`, installing to `.agents/skills/` or `~/.config/opencode/skills/`).

## What was removed

- The **Claude Code plugin** (`.claude-plugin/plugin.json` + `.claude-plugin/marketplace.json`) and its version-sync machinery (`scripts/sync-plugin-version.mjs`). The plugin was invisible to OpenCode: its `skills: [...]` array did nothing there.
- The **Codex per-skill metadata** (`agents/openai.yaml` beside every `SKILL.md`) and its `policy.allow_implicit_invocation` gate. OpenCode has no `agents/openai.yaml` or `policy` support.
- The **`disable-model-invocation: true`** frontmatter flag from user-invoked skills. It is a Claude Code mechanism OpenCode ignores.

## The invocation gate in OpenCode

Claude's `disable-model-invocation` and Codex's `policy.allow_implicit_invocation` have no OpenCode equivalent. OpenCode gates a user-invoked skill with `permission.skill.<name>: "deny"` in `opencode.json` (or per-agent under `agent.<name>.permission.skill`): `deny` hides the skill from the model's `<available_skills>` list while the auto-generated `/skill-name` slash command still reaches it for the human. A ready-to-go config ships at `.agents/opencode.json.example`. See [.agents/invocation.md](../.agents/invocation.md).

## Decision

- Ship **OpenCode-first via skills.sh** as the headline distribution.
- Remove the **Claude Code plugin** and the **Codex metadata** (see What was removed).
- Keep skills.sh as the universal installer: it already serves Claude Code, Codex, and other harnesses, so no user is left without an install path.
- Revisit a native plugin only if a harness we care about gains a curated-subset mechanism that OpenCode lacks.

## Invariants this creates

- Every promoted skill has a reference in the top-level `README.md`.
- The user-invoked skills are gated in OpenCode through `permission.skill: "deny"`, enumerated in `.agents/opencode.json.example`.
- `scripts/link-skills.sh` links into `~/.config/opencode/skills` (OpenCode) and `~/.agents/skills` (other harnesses).
