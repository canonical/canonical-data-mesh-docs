(contribute)=

# Contribute

This documentation is built with Sphinx from the [canonical-data-mesh-docs](https://github.com/canonical/canonical-data-mesh-docs) repository. Contributions are welcome.

## Where content lives

This site is the single home for Data Mesh documentation. Follow these placement rules:

- **Solution-level pages** (cross-charm workflows, architecture, concepts) live at the top level of each section.
- **Charm-specific pages** live in the charm's subsection of each section (for example `how-to/datahub/`).
- **Configuration options, actions, and integration endpoints are never duplicated here.** They are generated from each charm's `charmcraft.yaml` and published on its Charmhub page; link to the Charmhub tabs instead.

## Build the docs locally

```bash
cd docs
make run
```

This serves a live-reloading build at `http://127.0.0.1:8000`. Other useful targets: `make html`, `make spelling`, `make linkcheck`.

## Style

Pages are written in MyST Markdown and follow the [Diátaxis](https://diataxis.fr/) framework. Use sentence case for headings, imperative mood for instructions, no `$` prompts in code blocks, and `<UPPERCASE>` placeholders.
