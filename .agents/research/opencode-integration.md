# OpenCode integration research

Research date: 2026-09-01. Primary sources only: official docs at opencode.ai, the OpenCode source at github.com/anomalyco/opencode (the `dev` branch, commit `ebece6e`; github.com/sst/opencode resolves to the same repository and commit), the official config schema at opencode.ai/config.json, the Agent Skills spec at agentskills.io, and the skills.sh / vercel-labs/skills CLI.

## Executive summary

- OpenCode has its **own plugin system** (JS/TS modules with event hooks). It does **not** read `.claude-plugin/plugin.json` or `.codex-plugin/plugin.json`, and has no plugin-manifest file at all. "Plugins" are either files dropped in `.opencode/plugins/` / `~/.config/opencode/plugins/` or npm packages listed under `"plugin": [...]` in `opencode.json`. There is no marketplace concept and no `npx opencode@latest init` scaffold (that command does not exist).
- OpenCode **does** support the Agent-Skills `SKILL.md` convention, and natively reads `.claude/skills`, `.agents/skills`, and its own `.opencode/skills` / `~/.config/opencode/skills`, so skills written once are picked up by OpenCode with no changes. It uses the `description` frontmatter for model auto-invocation through a `skill` tool.
- **`disable-model-invocation` and `agents/openai.yaml` + `policy.allow_implicit_invocation` are NOT supported.** OpenCode's own gate for "user-invoked vs model-invoked" is the `permission.skill` mechanism (deny hides a skill from the model; ask prompts the human). Skills are additionally auto-exposed as slash commands, which is how a human reaches a model-hidden skill.
- Config is `opencode.json`/`opencode.jsonc` (schema at `https://opencode.ai/config.json`), merged across remote, global (`~/.config/opencode/`), custom (`OPENCODE_CONFIG`), and project (`opencode.json`, `.opencode/`) layers. Agents live in the `agent` key or as markdown in `.opencode/agents/`.
- Custom slash commands are markdown files in `.opencode/commands/` or the `command` key. Every skill is automatically registered as a slash command (`/skill-name`).
- MCP servers are configured under the `mcp` key (local + remote + OAuth), with a CLI (`opencode mcp add`, `list`, `auth`, `logout`, `debug`).
- There is no OpenCode-native "marketplace" or `skills add` equivalent for skills, but there **is** a remote-skills-registry protocol (`skills.urls` + `index.json`) and an npm plugin installer (`opencode plugin`). **skills.sh supports OpenCode as a target harness** (`-a opencode`; installs to `.agents/skills/` project, `~/.config/opencode/skills/` global). This repo's own `scripts/link-skills.sh` already symlinks into `~/.agents/skills` and `~/.claude/skills`, both of which OpenCode reads.

---

## 1. Plugins

### Does OpenCode support plugins?

Yes, but "plugin" means something different from Claude Code/Codex. An OpenCode plugin is a **JavaScript/TypeScript module** that exports plugin functions; each function receives a context object and returns a hooks object that subscribes to events (tool execution, sessions, files, shell, permissions, etc.).

> "Write your own plugins to extend OpenCode... A plugin is a JavaScript/TypeScript module that exports one or more plugin functions. Each function receives a context object and returns a hooks object." - https://opencode.ai/docs/plugins

### Plugin manifest format

**OpenCode does not read `.claude-plugin/plugin.json` or `.codex-plugin/plugin.json`.** A repository-wide search of `packages/opencode/src`, `packages/core/src`, and `packages/cli/src` for `.claude-plugin`, `.codex-plugin`, and `plugin.json` returns zero hits (source, `dev` branch). There is **no manifest file**; plugins are declared two ways:

1. **Local files** in the plugin directories, auto-loaded at startup:
   - `.opencode/plugins/` - project-level
   - `~/.config/opencode/plugins/` - global
   
   (Source: the loader glob is `{plugin,plugins}/*.{ts,js}`, so the singular `plugin/` directory is also accepted - `packages/opencode/src/config/plugin.ts:21`.)
2. **npm packages** in the `plugin` key of `opencode.json`:

   ```json
   {
     "$schema": "https://opencode.ai/config.json",
     "plugin": ["opencode-helicone-session", "opencode-wakatime", "@my-org/custom-plugin"]
   }
   ```

   "npm plugins are installed automatically using Bun at startup. Packages and their dependencies are cached in `~/.cache/opencode/node_modules/`." - https://opencode.ai/docs/plugins

The config schema (https://opencode.ai/config.json) defines `plugin` as an array of either a string or a `[spec, options-object]` tuple.

### Plugin package structure (source-level detail)

From `packages/opencode/src/plugin/shared.ts`:

- Entrypoints are resolved from `package.json` `exports["./server"]` or `exports["./tui"]`; for server plugins `main` is a fallback (`INDEX_FILES`: `index.ts`, `index.tsx`, `index.js`, `index.mjs`, `index.cjs`).
- Plugin kinds are `"server"` and `"tui"`.
- npm plugins can declare a version-compatibility range via `package.json` `engines.opencode` (semver checked against the running OpenCode version).
- TUI theme files are declared via a package.json `oc-themes` array field.
- A plugin module is loaded either as the newer **v1 shape** (`export default { id, server(input, options) }` / `{ id, tui(input, options) }`) or the **legacy named-export shape** (`export const MyPlugin = async (ctx) => ({ "tool.execute.before": async (input, output) => {...} })`) - `packages/opencode/src/plugin/index.ts:99-125`.
- Plugins can register **custom tools** via the `tool` helper from `@opencode-ai/plugin`:

  ```ts
  import { type Plugin, tool } from "@opencode-ai/plugin"
  export const CustomToolsPlugin: Plugin = async (ctx) => {
    return {
      tool: {
        mytool: tool({
          description: "...",
          args: { foo: tool.schema.string() },
          async execute(args, context) { ... },
        }),
      },
    }
  }
  ```
  - https://opencode.ai/docs/plugins

### How a user installs a plugin

- `opencode plugin <module>` (alias `opencode plug`): installs the npm package, reads the package manifest, and patches `opencode.json`. Flags: `--global` / `-g` (install in global config `~/.config/opencode/`), `--force` / `-f` (replace existing version). - `packages/opencode/src/cli/cmd/plug.ts:178-199`
- For local plugins, just drop files in `.opencode/plugins/` or `~/.config/opencode/plugins/`; external npm deps for local plugins come from a `package.json` in the config directory (`.opencode/package.json`), installed by `bun install` at startup. - https://opencode.ai/docs/plugins

### Load order

"1. Global config (`~/.config/opencode/opencode.json`) 2. Project config (`opencode.json`) 3. Global plugin directory (`~/.config/opencode/plugins/`) 4. Project plugin directory (`.opencode/plugins/`)". Duplicate npm packages with the same name+version load once. - https://opencode.ai/docs/plugins

### Skills inside a plugin?

**There is no skills array in any OpenCode plugin manifest.** OpenCode plugins are code modules (hooks + custom tools + themes); skills are a separate, filesystem/config-driven feature (see Q2). Claude Code's `plugin.json` `skills: [...]` array and Codex plugin manifests have no OpenCode equivalent.

### Does `npx opencode@latest init` exist?

**No.** There is no `init` subcommand anywhere in `packages/opencode/src/cli` (command list: `run`, `generate`, `debug`, `account`, `providers`, `agent`, `upgrade`, `uninstall`, `serve`, `web`, `models`, `stats`, `export`, `import`, `github`, `pr`, `session`, `plugin`, `db`, `mcp`, `tui`, `attach`, `acp` - `packages/opencode/src/index.ts:81-103`), and no "init" scaffold is documented. The closest scaffolding commands are `opencode agent create` (https://opencode.ai/docs/agents) and `opencode mcp add` (https://opencode.ai/docs/mcp-servers).

---

## 2. Skills

### Does OpenCode support the Agent-Skills SKILL.md convention?

Yes. OpenCode is listed as a client of the Agent Skills spec (https://agentskills.io), which says skills are folders with a `SKILL.md` containing `name` and `description` frontmatter. OpenCode implements progressive disclosure: it lists available skills (name + description) in the system prompt via a native `skill` tool, and loads the full `SKILL.md` on demand.

> "Agent skills let OpenCode discover reusable instructions from your repo or home directory. Skills are loaded on-demand via the native `skill` tool." - https://opencode.ai/docs/skills

### Where OpenCode looks for skills

Official docs (https://opencode.ai/docs/skills) list six locations; the source (`packages/opencode/src/skill/index.ts:21-233`) confirms them:

| Location | Notes |
| --- | --- |
| `.opencode/skills/<name>/SKILL.md` | Project; walked up from cwd to the git worktree; also accepts singular `.opencode/skill/` (glob `{skill,skills}/**/SKILL.md`) |
| `~/.config/opencode/skills/<name>/SKILL.md` | Global (config directories) |
| `.claude/skills/<name>/SKILL.md` | Project, Claude-compatible, walked up to worktree (glob `skills/**/SKILL.md`) |
| `~/.claude/skills/<name>/SKILL.md` | Global Claude-compatible |
| `.agents/skills/<name>/SKILL.md` | Project, agent-compatible, walked up to worktree |
| `~/.agents/skills/<name>/SKILL.md` | Global agent-compatible |

Plus three config-driven sources:

- `"skills": { "paths": [...] }` in `opencode.json`: additional absolute/relative directories scanned recursively for `**/SKILL.md` (relative paths resolve from the project directory, `~` expands to home) - `packages/opencode/src/skill/index.ts:210-220`.
- `"skills": { "urls": [...] }`: URLs to remote skill registries (see Q6) - `packages/opencode/src/skill/index.ts:222-227`.
- A built-in `customize-opencode` skill is always registered (it teaches the model the opencode config schema) - `packages/opencode/src/skill/index.ts:32-35, 276-283`.

Discovery is recursive: the project/global patterns are `skills/**/SKILL.md` (external dirs) and `{skill,skills}/**/SKILL.md` (OpenCode config dirs), so `skills/<bucket>/<name>/SKILL.md` is found. Skills are de-duplicated by `name`; a duplicate logs a warning and the later one wins - `packages/opencode/src/skill/index.ts:125-131`.

### Does it read `description` frontmatter for auto-invocation?

Yes. The `skill` tool's description is built from each skill's `name` and `description` and injected into the model prompt as `<available_skills>`; the model invokes a skill by calling `skill({ name: "git-release" })`. - https://opencode.ai/docs/skills and `packages/opencode/src/tool/skill.ts`.

### Recognized frontmatter fields

"Only these fields are recognized: `name` (required), `description` (required), `license` (optional), `compatibility` (optional), `metadata` (optional, string-to-string map). Unknown frontmatter fields are ignored." - https://opencode.ai/docs/skills. The source stores only `name`, `description`, and the body content (`packages/opencode/src/skill/index.ts:37-42, 134-139`), so `license`/`compatibility`/`metadata` are parsed but not used at runtime.

`name` constraints (docs): 1-64 chars, lowercase alphanumeric + single hyphens, must match the directory name. Regex `^[a-z0-9]+(-[a-z0-9]+)*$`; `description` must be 1-1024 chars.

### `disable-model-invocation` / `agents/openai.yaml` / `policy.allow_implicit_invocation`

**None of these are supported.** There is no `disable-model-invocation`, no per-skill `agents/` subdirectory handling, and no `policy`/`openai.yaml` support anywhere in the OpenCode docs or source (repository-wide search for `disable-model-invocation`, `allow_implicit_invocation`, `implicit_invocation` returns zero hits). These are Claude Code and Codex mechanisms.

### OpenCode's own user-invoked vs model-invoked gate

Gating is done with **permissions on the `skill` tool** (https://opencode.ai/docs/skills, "Configure permissions"):

- `"permission": { "skill": { "*": "allow", "pr-review": "allow", "internal-*": "deny", "experimental-*": "ask" } }` in `opencode.json`.
- `allow` = skill loads immediately; `deny` = skill hidden from the model and access rejected; `ask` = the human is prompted for approval before loading.
- Wildcards supported; `deny` removes the skill from the model's `<available_skills>` list (source: `Skill.available()` filters by `Permission.evaluate("skill", name, agent.permission).action !== "deny"` - `packages/opencode/src/skill/index.ts:310-315`).
- Per-agent overrides: `agent.<name>.permission.skill` in `opencode.json`, or a `permission:` block in a markdown agent's frontmatter.
- Whole-tool disable: `tools: { skill: false }` per agent (or `"agent": { "plan": { "tools": { "skill": false } } }` in config).

The practical "user-invoked only" pattern in OpenCode is: set `permission.skill.<name> = "deny"` so the model never sees it, and rely on the skill being exposed as a slash command (`/skill-name`) for the human (see Q4). Denied skills still appear in the slash-command registry because command registration iterates `skill.all()` without a permission filter - `packages/opencode/src/command/index.ts:134-152`.

### "agents" subdirectory per skill

No. OpenCode has no per-skill `agents/` subdirectory (the Codex convention for per-agent skill variants). Skills are flat; per-agent differences are expressed with per-agent `permission`/`tools` config, not by files inside the skill folder.

### How a user installs or links skills into OpenCode

There is no built-in skill installer/linker. Options:

1. Symlink or copy each skill folder into any discovery location (`.opencode/skills/`, `~/.config/opencode/skills/`, `.agents/skills/`, `~/.agents/skills/`, `.claude/skills/`, `~/.claude/skills/`).
2. Add `"skills": { "paths": ["/path/to/skills"] }` to `opencode.json`.
3. Add `"skills": { "urls": ["https://example.com/.well-known/skills/"] }` to fetch from a remote registry (Q6).
4. Use `npx skills add <owner>/<repo> -a opencode` (skills.sh supports OpenCode; installs to `.agents/skills/` project or `~/.config/opencode/skills/` global - Q6).

Environment toggles that disable parts of discovery (source, `packages/opencode/src/effect/runtime-flags.ts`): `OPENCODE_DISABLE_EXTERNAL_SKILLS` (disables `.claude` + `.agents` scanning), `OPENCODE_DISABLE_CLAUDE_CODE` / `OPENCODE_DISABLE_CLAUDE_CODE_SKILLS` (disables just the `.claude` scanning).

**Note for this repo:** `scripts/link-skills.sh` already symlinks every skill into `~/.agents/skills` and `~/.claude/skills` - both are OpenCode discovery locations, so OpenCode picks up the repo's skills with no additional work. (`~/.config/opencode/skills` is the other valid global target.)

---

## 3. Agents / config

### `opencode.json` / `opencode.jsonc`

Both JSON and JSONC are supported. The runtime config schema is published at **https://opencode.ai/config.json** (`$schema` value `https://opencode.ai/config.json`). Documented top-level keys include: `model`, `small_model`, `provider`, `agent`, `command`, `mcp`, `plugin`, `skills` (`paths`, `urls`), `permission`, `tools`, `instructions`, `references` / `reference` (named git or local directory references), `server`, `shell`, `formatter`, `lsp`, `share`, `snapshot`, `autoupdate`, `compaction`, `watcher`, `disabled_providers`, `enabled_providers`, `default_agent`, `subagent_depth`, `attachment`, `experimental`, `$schema`. - https://opencode.ai/docs/config and https://opencode.ai/config.json

### Config locations and precedence

"Config sources are loaded in this order (later sources override earlier ones): 1. Remote config (from `.well-known/opencode`), 2. Global config (`~/.config/opencode/opencode.json`), 3. Custom config (`OPENCODE_CONFIG` env var), 4. Project config (`opencode.json`), 5. `.opencode` directories (agents, commands, plugins), 6. Inline config (`OPENCODE_CONFIG_CONTENT`), 7. Managed config files, 8. macOS managed preferences." Configs are **merged**, not replaced. - https://opencode.ai/docs/config

- Project config is found by starting in the current directory and walking up to the nearest git directory. TUI-specific settings go in `tui.json` (schema `https://opencode.ai/tui.json`).
- `OPENCODE_CONFIG_DIR` env var points at a custom config directory searched for agents, commands, modes, plugins "just like the standard `.opencode` directory"; it can override settings.
- The `.opencode` / `~/.config/opencode` subdirectories use **plural names**: `agents/`, `commands/`, `modes/`, `plugins/`, `skills/`, `tools/`, `themes/` (singular names accepted for backwards compat). - https://opencode.ai/docs/config

### Agents and subagents

Two agent types: **primary agents** (Build and Plan are the built-ins; cycled with Tab) and **subagents** (General, Explore, Scout are the built-ins; invoked by the Task tool or by `@mention`). - https://opencode.ai/docs/agents

Configured two ways:

1. **JSON**: the `agent` key in `opencode.json`:

   ```json
   {
     "agent": {
       "code-reviewer": {
         "description": "...",
         "mode": "subagent",
         "model": "anthropic/claude-sonnet-4-5",
         "prompt": "...",
         "permission": { "edit": "deny" },
         "temperature": 0.1,
         "tools": { "write": false }
       }
     }
   }
   ```

2. **Markdown**: files in `.opencode/agents/` (project) or `~/.config/opencode/agents/` (global). "The markdown file name becomes the agent name." Frontmatter fields: `description` (required), `mode` (`primary` | `subagent` | `all`, default `all`), `model`, `temperature`, `top_p`, `prompt`, `permission`, `tools` (deprecated in favor of `permission`), `steps` (replaces deprecated `maxSteps`), `disable`, `hidden`, `color`, plus pass-through provider options.

Agent creation: `opencode agent create` is an interactive scaffold that asks for scope (global vs project), description, generates a prompt and identifier, and lets you pick allowed permissions. - https://opencode.ai/docs/agents

Other relevant keys: `default_agent` (must be a primary agent; falls back to `build`), `subagent_depth` (default 1; `0` forbids all subagent launches). - https://opencode.ai/docs/config

Permissions per agent (schema `PermissionConfig`): keys `read`, `edit`, `glob`, `grep`, `list`, `bash`, `task`, `external_directory`, `todowrite`, `webfetch`, `websearch`, `lsp`, `skill`, `question`, `doom_loop`; each `allow` | `ask` | `deny`, with glob-pattern objects for the file/bash/skill-style keys. `permission.task` controls which subagents an agent may invoke.

---

## 4. Commands / slash commands

### Custom slash commands

Custom commands are markdown files in `.opencode/commands/` or `~/.config/opencode/commands/` (also scanned under singular `command/`), or the `command` key in `opencode.json`. "The markdown file name becomes the command name." - https://opencode.ai/docs/commands

Fields: `template` (required), `description`, `agent`, `model`, `subtask`. The body becomes the prompt template.

Placeholders and syntax:
- `$ARGUMENTS` and positional `$1`, `$2`, `$3`, ...
- `` !`command` `` injects shell output (e.g. `` !`git log --oneline -10` ``).
- `@file` includes a file's contents.

Built-ins include `/init`, `/undo`, `/redo`, `/share`, `/help`; custom commands with the same name override built-ins. - https://opencode.ai/docs/commands and https://opencode.ai/docs/config

### Can a skill be exposed as a slash command?

**Yes, automatically.** The command registry registers every skill as a command (`source: "skill"`), with the template being the skill's `SKILL.md` content plus its base directory. - `packages/opencode/src/command/index.ts:134-152`. So in the TUI a user types `/skill-name` to run a skill directly (this is the human-invocation path even for a `deny`-permissioned skill). MCP server prompts are likewise auto-registered as slash commands (`source: "mcp"`).

---

## 5. MCP

MCP servers are configured under the `mcp` key in `opencode.json`. Two types (schema at https://opencode.ai/config.json; docs at https://opencode.ai/docs/mcp-servers):

- **Local**: `{ "type": "local", "command": ["npx", "-y", "my-mcp-command"], "cwd": "...", "environment": {...}, "enabled": true, "timeout": 5000 }` (`type` and `command` required).
- **Remote**: `{ "type": "remote", "url": "https://...", "headers": {...}, "oauth": {...} | false, "enabled": true, "timeout": 5000 }` (`type` and `url` required).
- OAuth: automatic dynamic client registration (RFC 7591); tokens stored in `~/.local/share/opencode/mcp-auth.json`.

CLI: `opencode mcp add [name]` (interactive add of local/remote servers), `opencode mcp list`, `opencode mcp auth [name]`, `opencode mcp logout [name]`, `opencode mcp debug <name>`. - https://opencode.ai/docs/mcp-servers and `packages/opencode/src/cli/cmd/mcp.ts`

MCP tools are namespaced `<server-name>_<tool>` and can be gated with glob patterns via `tools` / `permission` (e.g. `"mymcp_*": false` or `"mymcp_*": "deny"`), globally or per agent. Remote organizational defaults can be provided via the `.well-known/opencode` remote config and overridden locally.

---

## 6. Marketplace / installer

### OpenCode-native skill marketplace?

There is **no OpenCode-native skill marketplace or installer** equivalent to `npx skills add` or `claude plugins install`. What OpenCode does have:

1. **npm plugin installer** for code plugins: `opencode plugin <module>` (alias `plug`) installs the package and patches `opencode.json`; `-g` for the global config. - `packages/opencode/src/cli/cmd/plug.ts`
2. **Remote skills registry protocol** via `"skills": { "urls": [...] }`: OpenCode fetches `<url>/index.json` and downloads each skill's files into `~/.cache/opencode/skills/`. The `index.json` format (from the source schema `IndexSkill` and the test fixture at `packages/opencode/test/fixture/skills/index.json`):

   ```json
   {
     "skills": [
       { "name": "agents-sdk", "description": "...", "files": ["SKILL.md", "references/callable.md"], "version": "1.0.0" }
     ]
   }
   ```

   Files are fetched from `<url>/<skill-name>/<file>`; a `version` field enables cache refresh; skills without a `SKILL.md` in `files` are skipped. - `packages/opencode/src/skill/discovery.ts:49-132`. The docs example URL is `https://example.com/.well-known/skills/` (https://opencode.ai/docs/skills; config schema `skills.urls` description). There is no central public registry published by OpenCode itself.

### Does skills.sh support OpenCode?

**Yes.** OpenCode is a first-class supported harness in the skills CLI (vercel-labs/skills):

- README: "Supports **OpenCode**, **Claude Code**, **Codex**, **Cursor**, and 73 more." - https://github.com/vercel-labs/skills
- Agent id: `opencode`; install via `npx skills add <owner>/<repo> -a opencode`.
- Install paths (from the supported-agents table): project `.agents/skills/`, global `~/.config/opencode/skills/`. Both are OpenCode discovery locations, so skills installed this way are picked up with no extra config.
- The agent page states: "OpenCode is an open-source AI coding agent that integrates with the skills CLI for repo-scoped skill installation... Run the command below from your project root, then start a new OpenCode session." - https://skills.sh/agent/opencode
- The skills.sh compatibility table lists OpenCode as supporting basic skills and `allowed-tools` (experimental), but not `context: fork` or hooks. **Caveat:** OpenCode's own docs and the `dev`-branch source recognize only `name`, `description`, `license`, `compatibility`, and `metadata` frontmatter and ignore unknown fields, so the `allowed-tools` claim is not verifiable against OpenCode's own primary sources as of this research date.

### Impact on this repo

- The repo's Claude Code plugin (`.claude-plugin/plugin.json`) is **invisible** to OpenCode; it is not read and its `skills: [...]` array does nothing for OpenCode.
- The repo's skills themselves are **already OpenCode-compatible**: they follow the `SKILL.md` + `name`/`description` frontmatter convention, and `scripts/link-skills.sh` symlinks into `~/.agents/skills` and `~/.claude/skills`, both scanned by OpenCode (project and global). A user gets the skills in OpenCode for free.
- "User-invoked" skills in this repo (Claude: `disable-model-invocation: true`; Codex: `agents/openai.yaml` + `policy.allow_implicit_invocation: false`) need a different gate in OpenCode: set `permission.skill.<name> = "deny"` (model-hidden) and rely on the auto-generated `/skill-name` slash command for the human. A model-invoked-only skill needs no config.
- Publishing: the repo could additionally serve `index.json` (the `skills.urls` protocol) or rely on skills.sh (`npx skills add mattpocock/skills -a opencode`), which installs to `.agents/skills/` (project) or `~/.config/opencode/skills/` (global).

---

## Sources

Primary sources consulted:

1. https://opencode.ai/docs/plugins - OpenCode plugin system, install, load order, events, custom tools
2. https://opencode.ai/docs/skills - Agent Skills support, discovery locations, frontmatter, permissions
3. https://opencode.ai/docs/config - config format, locations, precedence, schema overview
4. https://opencode.ai/config.json - official runtime config JSON schema (authoritative field list)
5. https://opencode.ai/docs/agents - agent types, JSON + markdown agents, options, `opencode agent create`
6. https://opencode.ai/docs/commands - custom slash commands, markdown + JSON, placeholders
7. https://opencode.ai/docs/mcp-servers - MCP local/remote/OAuth config and CLI
8. https://github.com/anomalyco/opencode - source (branch `dev`, commit `ebece6e`; `sst/opencode` resolves to the same HEAD):
   - `packages/opencode/src/skill/index.ts` (discovery patterns, frontmatter handling, permission filtering)
   - `packages/opencode/src/skill/discovery.ts` (remote `index.json` registry protocol)
   - `packages/opencode/src/tool/skill.ts`, `packages/opencode/src/tool/skill.txt`
   - `packages/opencode/src/command/index.ts` (skills auto-registered as slash commands)
   - `packages/opencode/src/config/plugin.ts`, `packages/opencode/src/plugin/{index,loader,shared}.ts` (plugin loading, entrypoints, no claude/codex manifest reading)
   - `packages/opencode/src/config/{paths,command}.ts` (config dirs, command markdown discovery)
   - `packages/opencode/src/cli/cmd/plug.ts`, `agent.ts`, `mcp.ts`, `packages/opencode/src/index.ts` (CLI command list; no `init`)
   - `packages/opencode/src/effect/runtime-flags.ts` (`OPENCODE_DISABLE_CLAUDE_CODE_SKILLS`, etc.)
   - `packages/opencode/test/skill/discovery.test.ts`, `packages/opencode/test/fixture/skills/index.json`
9. https://agentskills.io/specification - Agent Skills spec (frontmatter fields, structure); OpenCode listed as a client at https://agentskills.io
10. https://github.com/vercel-labs/skills - skills CLI README (OpenCode agent id, install paths `.agents/skills/` / `~/.config/opencode/skills/`, compatibility table, `.claude-plugin` manifest discovery during install)
11. https://skills.sh/agent/opencode - skills.sh OpenCode agent page