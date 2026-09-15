# Resources for development

## Agentic tool bundle for product repositories

This directory provides instructions, skills and agents that are useful for developing products of Data Mesh. For supporting diverse harnesses, we use [apm](https://github.com/microsoft/apm)

To reference the skills, place the following lines in `apm.yml`. Consider also including the template, provided below, in the `README`.

```
dependencies:
  apm:
  - git: canonical/canonical-data-mesh-docs
```

We recommend adding instruction/skill/agent directories for most harnesses in `.gitignore`.

## Instructions to include in the repositories that consume the bundle

```markdown
### Setting up the environment for agents


\```sh
apm install --target {copilot,claude,codex,opencode} # places skill and agent files. See apm docs for full list of supported harnesses
apm compile --target ... # generates AGENTS.md, CLAUDE.md files from custom instructions
\```

> Please note, that generated artifacts for Copilot are still tracked in the repository. This ensures that agents launched in web applications (chat, IDE) of GitHub have the necessary instructions.

```


`.gitignore` contents

```
# APM dependencies
apm_modules/
# ignored as apm takes care of populating the files
.claude
CLAUDE.md
.opencode
AGENTS.md
# .github/instructions and .agents/skills are not ignored
# as we would like for them to be available in the Web Chat/IDE 
# These skills are ignored because they are not necessary there.
.agents/skills/apm-usage
.github/agents/apm-*
```