# Resources for development

## Agentic tool bundle for product repositories

This directory provides instructions, skills and agents that are useful for developing products of Data Mesh. For supporting diverse harnesses, we use [apm](https://github.com/microsoft/apm)

To reference the skills, place the following lines in `apm.yml`. Consider also including the template, provided below, in the `README`.

```
dependencies:
  apm:
  - git: canonical/canonical-data-mesh-docs
    path: resources/development
```

We recommend adding instruction/skill/agent directories for most harnesses in `.gitignore`.

## Instructions to include in the repositories that consume the bundle

```markdown
### Setting up the environment for agents

This repository uses [apm](https://github.com/microsoft/apm) for managing dependencies for agentic resources. 

\```sh
apm install --target {copilot,claude,codex,opencode} # places skill and agent files. See apm docs for full list of supported harnesses
\```

The agent `apm-expert` and `apm-usage` skills are available for FAQ and assistance with the tool.

Some harnesses do not support granular instruction/rule sets, and rely solely on an entrypoint like `AGENTS.md`. To generate a single file with all the instructions, use `apm compile`.

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