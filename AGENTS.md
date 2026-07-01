# BragAI

Harness agent setup with role-based multi-agent capabilities.
Initially a plugin with skills, evolving to an agent SDK.
Licensed Apache-2.0.

## Constitution -- Immutable Principles

1. **Universal compatibility.** BragAI must work with any
   agent or code assistant. Every skill, command, and
   agent must be installable via Lola and follow the
   agentskills.io specification. No vendor lock-in.

2. **Roles, not monoliths.** Agents behave as specialized
   roles (Engineer, PM, PO, Sales, etc.). Each role has
   distinct expertise, perspective, and decision
   authority. A role never claims competence outside its
   domain.

3. **Spec-driven development.** Every feature starts as a
   spec. No implementation without an approved spec.
   Specs live in `specs/` and are the source of truth
   for what was decided and why.

4. **English only.** All content -- code, docs, skills,
   agents, specs, comments -- is in English.

5. **Progressive disclosure.** Skills load name+description
   at startup (~100 tokens each), full body on activation
   (<5000 tokens), and resources on demand. Never
   front-load context.

6. **Commit discipline.** Follow the tpope 50/72 rule:
   subject line max 50 chars, body wrapped at 72 chars.
   Use Conventional Commits prefixes (`feat:`, `fix:`,
   `docs:`, `test:`, `spec:`). Enforced by pre-push
   hook. All markdown files wrap at 80 chars.

## Architecture

```
plugin/                   # Dual: Claude Code plugin + Lola module
  .claude-plugin/         # Claude Code plugin manifest
    plugin.json           # Plugin identity and version
    marketplace.json      # Marketplace catalog
  skills/                 # agentskills.io skills (cross-platform)
    <name>/SKILL.md       # One SKILL.md per skill
  agents/                 # Role-based agents
    <role>.md             # One file per role
  commands/               # Slash commands
  scripts/                # Shared executable scripts
  hooks/                  # Lifecycle hooks
  AGENTS.md               # Plugin persona and identity
  lola.yaml               # Lola module hooks
specs/                    # SDD spec documents
```

## Plugin Compatibility

The `plugin/` directory works two ways:

- **As a Claude Code plugin**: the `.claude-plugin/`
  manifest defines the plugin identity. Claude Code
  discovers skills, agents, and hooks from the plugin
  root. Install via `/plugin install` or marketplace.
  Ref: https://code.claude.com/docs/en/plugin-marketplaces

- **As a Lola module**: Lola reads `skills/`, `agents/`,
  `commands/`, and `lola.yaml` from the module content
  root and installs them into any supported assistant
  (OpenCode, Cursor, Gemini CLI, Copilot, etc.).
  Ref: https://github.com/LobsterTrap/lola

Install via Lola:
```bash
lola mod add <source> --module-content plugin
lola install bragai
```

Note: OpenCode "plugins" are JS/TS event-hook modules --
a different concept from what BragAI builds. BragAI's
skills and agents reach OpenCode through Lola's managed
AGENTS.md sections or native skill directories.

When compatibility gaps are found between Claude Code's
plugin system and Lola's module system, file issues or
PRs against Lola following its `.github/` templates.
Improving Lola is an explicit goal of this project.

## Skills vs Agents

BragAI uses both. They serve different purposes:

- **Skills** (agentskills.io): Procedural knowledge and
  workflows. A folder with `SKILL.md` containing
  instructions, scripts, and references. Skills activate
  on task matching and execute workflows.
  Path: `plugin/skills/<name>/`.

- **Agents**: Specialized AI assistants with custom
  prompts, model selection, and tool permissions. Each
  agent embodies a professional role with distinct
  expertise. Defined as markdown with YAML frontmatter.
  Path: `plugin/agents/<role>.md`.
  Ref: https://opencode.ai/docs/agents/

Skills teach *how to do something*.
Agents define *who is doing it*.

## Skill Standards

Per agentskills.io specification:

- `name`: kebab-case, matches directory, max 64 chars
- `description`: max 1024 chars, include trigger phrases
  and skip conditions
- Body: <5000 tokens, <500 lines
- Required sections: `## Trigger`, `## Flow`, `## Gotchas`
- Scripts: flags for input, `--help`, stderr diagnostics,
  stdout output, `--dry-run` for destructive ops
- `allowed-tools`: only tools used on every run
- Spec: https://agentskills.io/specification

## Conventions

- File and directory names: kebab-case
- Commits: tpope 50/72 rule + Conventional Commits
  (`feat:`, `fix:`, `docs:`, `test:`, `spec:`)
  Ref: https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html
- Markdown line length: 80 chars max
- Bash for orchestration; Python with PEP 723 inline
  deps and `uv run` for logic requiring libraries
- Tests must pass before any commit

## CI

OpenCode GitHub Action at
`.github/workflows/opencode.yml`, triggered by `/oc`
or `/opencode` comments on issues and PRs.
Model: `google-vertex/claude-sonnet-4-6@default`.

## Lineage and Inspiration

BragAI supersedes the original bragai project (personal
AI assistant for tracking accomplishments). It evolves
toward a role-based agent harness inspired by
[OpenHuman](https://github.com/tinyhumansai/openhuman).

The differentiator: multiple agents that each embody a
professional role (Engineer, PM, PO, Sales, etc.) with
distinct expertise and judgment. The plugin is the first
step -- skills prove the patterns, roles emerge as the
SDK matures.

Design references:
- [Lola](https://github.com/LobsterTrap/lola) -- module
  system and multi-assistant compatibility
- [agentskills.io](https://agentskills.io) -- skill
  manifest specification
