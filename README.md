# Reservations API docs

[![Lint OpenAPI](https://github.com/miniskynet/reservations-api-docs/actions/workflows/lint.yml/badge.svg)](https://github.com/miniskynet/reservations-api-docs/actions/workflows/lint.yml)

A docs-as-code sample by [Desira Wijesundara](https://miniskynet.github.io): the complete documentation for a small, **fictional** REST API, written, versioned and checked the way engineering teams treat code.

**[Open the interactive API reference →](https://miniskynet.github.io/reservations-api-docs/)**

## What's here

| File | What it is |
|---|---|
| [`openapi.yaml`](openapi.yaml) | The OpenAPI 3.1 spec: the single source of truth. 8 operations, every field, error and example. |
| [`docs/getting-started.md`](docs/getting-started.md) | A task-based guide that takes a developer from an API key to a confirmed reservation. |
| [`STYLE.md`](STYLE.md) | The style rules every description in the spec follows. |
| [`index.html`](index.html) | Renders the spec as an interactive reference with [Redoc](https://github.com/Redocly/redoc). Served by GitHub Pages. |
| [`redocly.yaml`](redocly.yaml) | Lint rules for the spec. |
| [`.github/workflows/lint.yml`](.github/workflows/lint.yml) | Lints the spec on every push and pull request, so broken docs can't be merged. |

## Things worth looking at

- **Errors that tell you what to do.** Every 4xx response says what triggered it and how to fix it, and there's one consistent error shape across the API.
- **Nothing left implicit.** Defaults, limits, units, what `null` means, which fields are read-only, and what can't be undone.
- **Consistent examples.** The same customer, products and IDs appear across the spec and the guide, so a reader can follow one story end to end.
- **The guide and the reference do different jobs.** The guide teaches one workflow in order; the reference covers everything. They link to each other instead of repeating each other.

## Preview locally

The reference loads `openapi.yaml` over HTTP, so serve the folder rather than opening the file directly:

```bash
npx --yes http-server -p 8080 .
```

Then open <http://localhost:8080>.

## Lint

```bash
npx --yes @redocly/cli@latest lint
```

## License

The documentation in this repository is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
