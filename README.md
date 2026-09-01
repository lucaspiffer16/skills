<p>
  <a href="https://www.aihero.dev/s/skills-newsletter">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://res.cloudinary.com/total-typescript/image/upload/v1777382277/skills-repo-dark_2x.png">
      <source media="(prefers-color-scheme: light)" srcset="https://res.cloudinary.com/total-typescript/image/upload/v1777382277/skill-repo-light_2x.png">
      <img alt="Skills" src="https://res.cloudinary.com/total-typescript/image/upload/v1777382277/skill-repo-light_2x.png" width="369">
    </picture>
  </a>
</p>

# Skills For Real Engineers: OpenCode Fork

> **Fork notice.** This is a fork of [Matt Pocock's `skills`](https://github.com/mattpocock/skills), the original author of these agent skills. All credit for the skill design and content goes to him and his contributors. This fork reorients the set exclusively around **OpenCode** and drops the Claude Code and Codex harness integrations. The install commands below point at this fork, not at the upstream.

[![skills.sh](https://skills.sh/b/lucaspiffer16/skills)](https://skills.sh/lucaspiffer16/skills)

Agent skills for real engineering - not vibe coding. Built for **OpenCode** first, installable through [skills.sh](https://skills.sh/lucaspiffer16/skills), and usable by any harness that reads the Agent-Skills convention.

Developing real applications is hard. Approaches like GSD, BMAD, and Spec-Kit try to help by owning the process. But while doing so, they take away your control and make bugs in the process hard to resolve.

These skills are designed to be small, easy to adapt, and composable. They're based on decades of engineering experience. Hack around with them. Make them your own. Enjoy.

If you want to keep up with changes to the original skills, and any new ones Matt creates, join ~60,000 other devs on his newsletter:

[Sign Up To The Newsletter](https://www.aihero.dev/s/skills-newsletter)

## Why this fork exists

The upstream ships as a Claude Code plugin and carries Codex per-skill metadata. This fork is **OpenCode-only**:

- It removes the Claude Code plugin, the Codex `agents/openai.yaml` metadata, and the Claude `disable-model-invocation` frontmatter flag.
- It gates user-invoked skills with OpenCode's `permission.skill` mechanism (`deny` / `ask`) instead.
- It drops the `CLAUDE.md` steering file entirely: `AGENTS.md` is the single agent-instructions file the set writes to.
- It routes installation through skills.sh for OpenCode (`.agents/skills`, `~/.config/opencode/skills`), with no native plugin.

The skills themselves are unchanged in spirit: they follow the Agent-Skills `SKILL.md` convention, so they keep working on any harness that reads that convention.

## The System Engineering Handbook

The second reason this fork exists is the **System Engineering Handbook**: a documentation-first practice that makes a project's architecture and intent a persistent, machine-readable knowledge base before any implementation happens. The engineering skills in the upstream set document *during* the work (`grill-with-docs` writes `CONTEXT.md` and ADRs, `to-spec` writes specs). This fork adds a pair of skills that make the system's knowledge exist *up front* and stay alive afterwards.

### How it works

- **`/system-handbook`** (model-invoked, gated `ask` in OpenCode) builds the handbook in an existing project in one pass: it audits the repository, scaffolds a `docs/` tree across System, Domain, Product, Architecture, Engineering and Operations, models the current architecture in a Structurizr C4 `workspace.dsl`, writes the glossary and ADRs, and wires everything into `AGENTS.md` (and `CONTEXT.md` where the project uses one). The whole `docs/` tree opens as an Obsidian vault with no configuration.
- **`/system-handbook-maintain`** (model-invoked) keeps the handbook in sync as the code evolves: when implementation changes architecture, modules, integrations, data ownership, reliability, security, or documented behavior, it updates `workspace.dsl`, module docs, and behavior docs, and delegates glossary and ADR work to `domain-modeling` so there stays a single glossary and a single ADR record.

### Where the lifecycle runs

The handbook follows the software lifecycle: Discovery → Domain → Requirements → Product → Architecture → Architecture Decisions → Specification → Implementation → Testing → Operations → Evolution. The handbook holds the persistent knowledge through the whole loop; Structurizr holds the architectural model; Markdown holds the human- and agent-readable knowledge; Obsidian provides the exploration interface; `AGENTS.md` connects the repository knowledge to agents. The skills provide the engineering execution.

The part that makes it more than a snapshot is the **documentation-drift clause** written into `AGENTS.md`: when implementation changes domain behavior, business rules, architectural boundaries, integrations, data ownership, or reliability and security behavior, the relevant documentation must be reviewed and updated. That is what `system-handbook-maintain` enforces.

### How it composes, not duplicates

- **One glossary.** The glossary lives in `docs/01-domain/glossary.md`; `CONTEXT.md` becomes a concise pointer to the handbook, not a second copy.
- **One ADR record.** ADRs live in `docs/03-architecture/decisions/`, in a format Structurizr's `!adrs` imports, reusing `domain-modeling`'s bar for when an ADR is worth writing.
- **Specs stay on the tracker.** `to-spec` keeps publishing working specs to the issue tracker; the handbook holds the durable record.
- **Two AGENTS.md blocks.** `setup-matt-pocock-skills` owns the `## Agent skills` block; the handbook owns its own `## System Handbook` section.

## Installation (30-second setup)

These skills are built for **OpenCode** first, and install anywhere via **[skills.sh](https://skills.sh/lucaspiffer16/skills)**, which copies editable skill files into your project. You own the files and can hack on them: nothing updates behind your back.

### 1. Get the skills

```bash
npx skills@latest add lucaspiffer16/skills
```

Pick the skills you want, and which coding agents to install them on. **The installer lets you choose which skills to take, so make sure `setup-matt-pocock-skills` is one of them.**

The skills are Agent-Skills `SKILL.md` files, so they work with any harness that reads that convention (`.agents/skills`, `~/.config/opencode/skills`, and the Agent-Skills standard). The repo ships no Claude Code or Codex integration: skills.sh is the distribution, OpenCode-first. Pull the latest changes whenever you want them with `npx skills update`.

### 2. Gate the user-invoked skills in OpenCode (optional)

The user-invoked skills (the ones you type rather than let the model fire) are hidden from the model with `permission.skill: "deny"`. Copy `.agents/opencode.json.example` into your project's `opencode.json` to gate exactly those skills. `system-handbook` is gated `ask`, so the agent proposes it and the human approves before it runs.

### 3. Run `/setup-matt-pocock-skills`

In your agent, run it once per repo. It will:

- Ask you which issue tracker you want to use (GitHub, Linear, or local files)
- Ask you what labels you apply to tickets when you triage them (`/triage` uses labels)
- Ask you where you want to save any docs we create

When setup completes, the agent offers **`/system-handbook`** to build the System Engineering Handbook (gated `ask`, so you approve before it runs). Accept it if you want the architecture, domain language, and decisions written down up front.

### 4. Bam - you're ready to go.

## Why These Skills Exist

Matt built these skills as a way to fix common failure modes he saw with coding agents. This fork keeps that mission, aimed at OpenCode.

### #1: The Agent Didn't Do What I Want

> "No-one knows exactly what they want"
>
> David Thomas & Andrew Hunt, [The Pragmatic Programmer](https://www.amazon.co.uk/Pragmatic-Programmer-Anniversary-Journey-Mastery/dp/B0833F1T3V)

**The Problem**. The most common failure mode in software development is misalignment. You think the dev knows what you want. Then you see what they've built - and you realize it didn't understand you at all.

This is just the same in the AI age. There is a communication gap between you and the agent. The fix for this is a **grilling session** - getting the agent to ask you detailed questions about what you're building.

**The Fix** is to use:

- [`/grill-me`](./skills/productivity/grill-me/SKILL.md) - for non-code uses
- [`/grill-with-docs`](./skills/engineering/grill-with-docs/SKILL.md) - same as [`/grill-me`](./skills/productivity/grill-me/SKILL.md), but adds more goodies (see below)

These are the most popular skills. They help you align with the agent before you get started, and think deeply about the change you're making. Use them _every_ time you want to make a change.

### #2: The Agent Is Way Too Verbose

> With a ubiquitous language, conversations among developers and expressions of the code are all derived from the same domain model.
>
> Eric Evans, [Domain-Driven-Design](https://www.amazon.co.uk/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215)

**The Problem**: At the start of a project, devs and the people they're building the software for (the domain experts) are usually speaking different languages.

Matt felt the same tension with his agents. Agents are usually dropped into a project and asked to figure out the jargon as they go. So they use 20 words where 1 will do.

**The Fix** for this is a shared language. It's a document that helps agents decode the jargon used in the project.

<details>
<summary>
Example
</summary>

Here's an example [`CONTEXT.md`](https://github.com/mattpocock/course-video-manager/blob/076a5a7a182db0fe1e62971dd7a68bcadf010f1c/CONTEXT.md), from Matt's `course-video-manager` repo. Which one is easier to read?

- **BEFORE**: "There's a problem when a lesson inside a section of a course is made 'real' (i.e. given a spot in the file system)"
- **AFTER**: "There's a problem with the materialization cascade"

This concision pays off session after session.

</details>

This is built into [`/grill-with-docs`](./skills/engineering/grill-with-docs/SKILL.md). It's a grilling session, but that helps you build a shared language with the AI, and document hard-to-explain decisions in ADRs.

It's hard to explain how powerful this is. It might be the single coolest technique in this repo. Try it, and see.

> [!TIP]
> A shared language has many other benefits than reducing verbosity:
>
> - **Variables, functions and files are named consistently**, using the shared language
> - As a result, the **codebase is easier to navigate** for the agent
> - The agent also **spends fewer tokens on thinking**, because it has access to a more concise language

### #3: The Code Doesn't Work

> "Always take small, deliberate steps. The rate of feedback is your speed limit. Never take on a task that's too big."
>
> David Thomas & Andrew Hunt, [The Pragmatic Programmer](https://www.amazon.co.uk/Pragmatic-Programmer-Anniversary-Journey-Mastery/dp/B0833F1T3V)

**The Problem**: Let's say that you and the agent are aligned on what to build. What happens when the agent _still_ produces crap?

It's time to look at your feedback loops. Without feedback on how the code it produces actually runs, the agent will be flying blind.

**The Fix**: You need the usual tranche of feedback loops: static types, browser access, and automated tests.

For automated tests, a red-green-refactor loop is critical. This is where the agent writes a failing test first, then fixes the test. This helps give the agent a consistent level of feedback that results in far better code.

Matt built a **[`/tdd`](./skills/engineering/tdd/SKILL.md) skill** you can slot into any project. It encourages red-green-refactor and gives the agent plenty of guidance on what makes good and bad tests.

For debugging, there's also a **[`/diagnosing-bugs`](./skills/engineering/diagnosing-bugs/SKILL.md)** skill that wraps best debugging practices into a disciplined loop, gated phase by phase.

### #4: We Built A Ball Of Mud

> "Invest in the design of the system _every day_."
>
> Kent Beck, [Extreme Programming Explained](https://www.amazon.co.uk/Extreme-Programming-Explained-Embrace-Change/dp/0321278658)

> "The best modules are deep. They allow a lot of functionality to be accessed through a simple interface."
>
> John Ousterhout, [A Philosophy Of Software Design](https://www.amazon.co.uk/Philosophy-Software-Design-2nd/dp/173210221X)

**The Problem**: Most apps built with agents are complex and hard to change. Because agents can radically speed up coding, they also accelerate software entropy. Codebases get more complex at an unprecedented rate.

**The Fix** for this is a radical new approach to AI-powered development: caring about the design of the code.

This is built in to every layer of these skills:

- [`/to-spec`](./skills/engineering/to-spec/SKILL.md) quizzes you about which modules you're touching before creating a spec

And crucially, [`/improve-codebase-architecture`](./skills/engineering/improve-codebase-architecture/SKILL.md) surveys a codebase for deepening opportunities and hands you the candidates. It is a survey, not a rescue: on a genuinely old codebase it will find real candidates, but it won't untangle the mud for you.

### Summary

Software engineering fundamentals matter more than ever. These skills are a best effort at condensing these fundamentals into repeatable practices, to help you ship the best apps of your career. Enjoy.

## Reference

These split on one axis: who can invoke them. **User-invoked** skills are reachable only when you type them (e.g. `/grill-me`); their job is to orchestrate. **Model-invoked** skills can be invoked by you _or_ reached for automatically by the agent when the task fits; they hold the reusable discipline. A user-invoked skill may invoke model-invoked skills, but never another user-invoked one.

### Engineering

Skills used daily for code work.

**User-invoked**

- **[ask-matt](./skills/engineering/ask-matt/SKILL.md)**: Ask which skill or flow fits your situation. A router over the user-invoked skills in this repo.
- **[grill-with-docs](./skills/engineering/grill-with-docs/SKILL.md)**: Grilling session that also builds your project's domain model, sharpening terminology and updating `CONTEXT.md` and ADRs inline.
- **[triage](./skills/engineering/triage/SKILL.md)**: Move issues through a state machine of triage roles.
- **[improve-codebase-architecture](./skills/engineering/improve-codebase-architecture/SKILL.md)**: Scan a codebase for deepening opportunities, present them as a visual HTML report, then grill through whichever one you pick.
- **[setup-matt-pocock-skills](./skills/engineering/setup-matt-pocock-skills/SKILL.md)**: Configure this repo for the engineering skills (issue tracker, triage labels, domain doc layout). Run once per repo before using the other engineering skills.
- **[to-spec](./skills/engineering/to-spec/SKILL.md)**: Turn the current conversation into a spec and publish it to the issue tracker. No interview, just synthesizes what you've already discussed.
- **[to-tickets](./skills/engineering/to-tickets/SKILL.md)**: Break any plan, spec, or conversation into a set of tracer-bullet tickets, each declaring its blocking edges, written as text in a local file, or as native blocking links on a real tracker.
- **[implement](./skills/engineering/implement/SKILL.md)**: Build the work described by a spec or set of tickets, driving `/tdd` at pre-agreed seams and closing out with `/code-review` before committing.
- **[wayfinder](./skills/engineering/wayfinder/SKILL.md)**: Plan a huge chunk of work, more than one agent session can hold, as a shared map of decision tickets on the issue tracker, and resolve them one at a time until the way to the destination is clear.

**Model-invoked**

- **[prototype](./skills/engineering/prototype/SKILL.md)**: Build a throwaway prototype to answer a design question, either a single shareable HTML file for state/logic questions, or several radically different UI variations toggleable from one route.
- **[diagnosing-bugs](./skills/engineering/diagnosing-bugs/SKILL.md)**: Disciplined diagnosis loop for hard bugs and performance regressions: build a feedback loop that goes red on this bug → minimise → hypothesise → instrument → fix → regression-test.
- **[research](./skills/engineering/research/SKILL.md)**: Investigate a question against high-trust primary sources and capture the findings as a cited Markdown file in the repo, run as a background agent.
- **[tdd](./skills/engineering/tdd/SKILL.md)**: Test-driven development with a red-green-refactor loop. Builds features or fixes bugs one vertical slice at a time.
- **[domain-modeling](./skills/engineering/domain-modeling/SKILL.md)**: Actively build and sharpen a project's domain model: challenge terms against the glossary, stress-test with edge-case scenarios, and update `CONTEXT.md` and ADRs inline.
- **[codebase-design](./skills/engineering/codebase-design/SKILL.md)**: Shared discipline and vocabulary for designing deep modules: a lot of behaviour behind a small interface, placed at a clean seam, testable through that interface.
- **[code-review](./skills/engineering/code-review/SKILL.md)**: Two-axis review of the diff since a fixed point: **Standards** (does it follow the repo's documented coding standards, plus a Fowler smell baseline?) and **Spec** (does it faithfully implement the originating issue/spec?), run as parallel sub-agents so neither pollutes the other.
- **[resolving-merge-conflicts](./skills/engineering/resolving-merge-conflicts/SKILL.md)**: Work through an in-progress git merge or rebase conflict hunk by hunk, resolving by intent traced to each side's primary source, then finish the operation (never `--abort`).
- **[system-handbook](./skills/engineering/system-handbook/SKILL.md)**: Build or rebuild a System Engineering Handbook in the current project: audit the codebase, scaffold the docs tree, model the architecture in Structurizr, write ADRs and the glossary, and wire it into AGENTS.md. Model-reached after setup or on request; gated `ask` in OpenCode so the human approves before it runs.
- **[system-handbook-maintain](./skills/engineering/system-handbook-maintain/SKILL.md)**: Keep the project's System Engineering Handbook in sync with the code as it evolves, updating `workspace.dsl`, module docs, and behavior docs, and delegating glossary/ADR work to `domain-modeling`.
- **[wizard](./skills/engineering/wizard/SKILL.md)**: Generate an interactive bash wizard that walks a human through steps only they can perform: provisioning infrastructure, setting up credentials or CI secrets, walking an unfamiliar third-party dashboard, or running a one-off migration or cutover.

### Productivity

General workflow tools, not code-specific.

**User-invoked**

- **[grill-me](./skills/productivity/grill-me/SKILL.md)**: Get relentlessly interviewed about a plan or design until every branch of the design tree is resolved.
- **[handoff](./skills/productivity/handoff/SKILL.md)**: Compact the current conversation into a handoff document so another agent can continue the work.
- **[teach](./skills/productivity/teach/SKILL.md)**: Teach the user a new skill or concept over multiple sessions, using the current directory as a stateful teaching workspace.
- **[to-questionnaire](./skills/productivity/to-questionnaire/SKILL.md)**: Turn a decision you can't answer alone into a Markdown questionnaire for the one person who can, filled in async, or together over a meeting. It grills you about the send (who it's for, what you need back), not the subject.
- **[wait-what](./skills/productivity/wait-what/SKILL.md)**: Fire this the moment a message doesn't land. The agent re-pitches it with the context you're missing, in plain English, using your `CONTEXT.md` vocabulary.

**Model-invoked**

- **[grilling](./skills/productivity/grilling/SKILL.md)**: Interview the user relentlessly about a plan, decision, or idea until every branch of the design tree is resolved. The reusable interview primitive behind `grill-me`, `grill-with-docs`, `triage`, `wayfinder` and `improve-codebase-architecture`.
- **[writing-for-agents](./skills/productivity/writing-for-agents/SKILL.md)**: Writing documents for agents: skills, AGENTS.md, and any doc an agent reaches by a pointer.

## Credits

The skills in this repo are the work of [Matt Pocock](https://www.aihero.dev) and the contributors to [mattpocock/skills](https://github.com/mattpocock/skills). This fork adapts the distribution and the invocation model to OpenCode; the skill content remains theirs. License: MIT (see [LICENSE](./LICENSE)).