# The canonical install block

One install story, one wording. `README.md`, `.changeset/*`, and every page under `docs/` must say **this** and nothing else. Change it here first, then propagate.

`mattpocock-skills` is distributed as **editable skill files**, copied into your project by [skills.sh](https://skills.sh/mattpocock/skills) (`npx skills add mattpocock/skills`). There is no plugin: you own the files and can hack on them. The skills work with **OpenCode** out of the box, because OpenCode reads `.agents/skills` and `~/.config/opencode/skills`, both of which skills.sh writes to.

## OpenCode, and other agents: skills.sh

Use the whole-set form on `README.md`:

<canonical-block name="skills-sh-whole-set">

```bash
npx skills@latest add mattpocock/skills
```

Pick the skills you want, and which coding agents to install them on. **The installer lets you choose which skills to take: make sure `setup-matt-pocock-skills` is one of them.**

</canonical-block>

…and the single-skill form wherever one skill is named on its own. Note that **`docs/` pages are not a consumer of this block**: ai-hero renders the install widget above the body, so a page that writes the commands out duplicates it. See [writing-docs.md](./writing-docs.md).

<canonical-block name="skills-sh-one-skill">

```bash
npx skills@latest add mattpocock/skills --skill=<name>
```

```bash
npx skills@latest update <name>
```

</canonical-block>

`skills@latest` is the pinned spelling in all three. The pages under `docs/` used to carry their own copy of these commands; those blocks are now deleted rather than corrected, because the site renders the install commands itself.

## User-invoked skills in OpenCode

The user-invoked skills (the ones you reach by typing `/skill-name` rather than letting the model fire them) are hidden from the model with `permission.skill: "deny"`. A ready-to-go config ships at `.agents/opencode.json.example`; copy it into your project's `opencode.json` to gate exactly those skills.

## Not the install story

There is no plugin. The old Claude Code plugin (`.claude-plugin/`) was removed when the set went OpenCode-first; see [.agents/adr/0002-ship-as-an-opencode-first-skill-set.md](./.agents/adr/0002-ship-as-an-opencode-first-skill-set.md). The skills are OpenCode-first: the repo ships no Claude Code or Codex integration, and skills.sh is the distribution for OpenCode and any other Agent-Skills harness.
