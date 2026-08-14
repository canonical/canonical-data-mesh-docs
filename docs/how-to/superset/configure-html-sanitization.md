(how-to-superset-configure-html-sanitization)=

# Configure HTML sanitization

Superset renders user-supplied HTML in markdown components, dashboard headers, and chart labels. Sanitizing that HTML prevents cross-site scripting attacks, and the charm enables sanitization by default. This guide describes how to extend the sanitization schema when a feature you need is blocked by it.

```{caution}
Every extension widens what users can inject into a page that other users open.
Extend the schema only with the specific tags and attributes a feature needs,
and never disable sanitization on a deployment with untrusted users.
```

## Extend the schema

Some functionality requires HTML that the default schema strips, most commonly the styling capabilities of the Handlebars plugin, which need the `style` and `class` attributes.

Write the additions to a JSON file:

```json
{
  "attributes": {
    "*": ["style", "className"]
  },
  "tagNames": ["style"]
}
```

Apply it to the charm:

```bash
juju config superset-k8s html-sanitization-schema-extensions=@sanitization-extensions.json
```

The value is merged into Superset's default sanitization schema, which follows the [hast-util-sanitize](https://github.com/syntax-tree/hast-util-sanitize) format. Apply the same configuration to the worker application if your reports render the same content.

## Verify

Open a dashboard that uses the markup you enabled. The element now renders instead of being stripped. If it is still stripped, check the charm log for a configuration parsing error:

```bash
juju debug-log --include superset-k8s
```
