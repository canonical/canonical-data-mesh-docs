# Canonical Data Mesh Documentation

Documentation for the [Canonical Data Mesh](https://charmhub.io/solutions/preview/0a55c2269e364269), a composable data platform solution built on trusted open source projects and operated with [Juju](https://juju.is).

## Scope

This repository is the **single home** for Data Mesh documentation: solution architecture, cross-charm workflows, platform operations, and the documentation of the individual charms. Content is organized with the [Diátaxis](https://diataxis.fr/) framework (`tutorials/`, `how-to/`, `reference/`, `explanation/`), with solution-level pages first and a subsection per charm inside each section (for example `how-to/datahub/`).

## Build the docs locally

The documentation is built with Sphinx from the Canonical starter pack:

```bash
cd docs
make run
```

This serves a live-reloading build at `http://127.0.0.1:8000`. Other useful targets: `make html`, `make spelling`, `make linkcheck`.
