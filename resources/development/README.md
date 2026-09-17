# Resources for development

## Agentic tool bundle for product repositories

This directory provides instructions, skills and agents that are useful for developing products of Data Mesh. For supporting diverse harnesses, we use [apm](https://github.com/microsoft/apm)

To reference the skills, place the following lines in `apm.yml`. Consider also including the template, provided below, in `CONTRIBUTING.md`.

```
dependencies:
  apm:
  - git: canonical/canonical-data-mesh-docs
    path: resources/development
```

We recommend adding instruction/skill/agent directories for most harnesses in `.gitignore`.

## Instructions to include in the repositories that consume the bundle

`CONTRIBUTING.md`:

```markdown
### Setting up the environment for agents

This repository uses [apm](https://github.com/microsoft/apm) for managing dependencies for agentic resources. 

\```sh
apm install # places skill and agent files for the harnesses configured in this repo's apm.yml `targets:` (or auto-detected). See apm docs for full list of supported harnesses and how to override with --target
\```

The agent `apm-expert` and `apm-usage` skills are available for FAQ and assistance with the tool.

Harnesses that support scoped, native instruction directories (Claude's `.claude/rules/`, Copilot's `.github/instructions/`) get instructions deployed there directly by `apm install`, and only load the ones relevant to the files being touched. Some harnesses (Codex, OpenCode, Gemini today) only read a single root entrypoint and have no such scoping. Any `.instructions.md` content whose `applyTo` pattern matches gets fully inlined into that file (`AGENTS.md`/`GEMINI.md`) wherever it's placed. If you would like to avoid an overpopulated single instruction file, consider adding a global manual directive to look for instruction files in `apm_modules`.  

You can use `apm.local.yml` for specifying additional personal resources.


> Please note, that generated artifacts for Copilot are still tracked in the repository. This ensures that agents launched in web applications (chat, IDE) of GitHub have the necessary instructions.

```

`.gitignore` contents:

```
# APM dependencies
apm_modules/
# ignored as apm takes care of populating the files
.claude
CLAUDE.md
.opencode
AGENTS.md
.codex
# .github/instructions and .agents/skills are not ignored
# as we would like for them to be available in the Web Chat/IDE 
# These skills are ignored because they are not necessary there.
.agents/skills/apm-usage
.github/agents/apm-*
```

## Migrating a repo off `copilot-collections`
If a consuming repo still has
`.copilot-collections.yaml`, some of them might already be included in the bundle. Otherwise, consider either including the resources in the bundle, or simply add them in target repository's `apm.yml`.