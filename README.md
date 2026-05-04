# Pepesto OpenAPI Definition

Single-file OpenAPI 3.0.3 specification for the [Pepesto Grocery API](https://www.pepesto.com/ai-grocery-shopping-agent/) — recipe → matched grocery cart across 26 European supermarkets.

The canonical source is [`openapi.yaml`](./openapi.yaml). It covers all 10 public endpoints (`/link`, `/credits`, `/oneshot`, `/parse`, `/suggest`, `/products`, `/session`, `/checkout`, `/catalog`, `/promotions`) with request/response schemas, examples drawn from the [pepesto-api-examples](https://github.com/pepesto-solutions/pepesto-api-examples) repo, and the full enum of supported supermarket domains.

## Layout

```
.
├── openapi.yaml
└── README.md
```

# Contributing

## Validate locally

The spec is validated against two parsers; both should pass before you commit a change.

```bash
# Redocly (recommended — clearer errors, also drives the doc preview)
npx --yes @redocly/cli@latest lint openapi.yaml

# Swagger / @apidevtools (the parser most import tools use under the hood)
npx --yes @apidevtools/swagger-cli@latest validate openapi.yaml
```

### Swagger UI (interactive "try it out", via Docker)

```bash
docker run --rm -p 8081:8080 \
  -e SWAGGER_JSON=/spec/openapi.yaml \
  -v "$(pwd):/spec" \
  swaggerapi/swagger-ui
# open http://localhost:8081
```

To actually fire requests from the UI, paste your `pep_sk_…` key into the **Authorize** dialog (the `bearerAuth` scheme).

### Swagger Editor (live-edit + preview, via Docker)

```bash
docker run --rm -p 8082:8080 \
  -e SWAGGER_FILE=/spec/openapi.yaml \
  -v "$(pwd):/spec" \
  swaggerapi/swagger-editor
# open http://localhost:8082
```

The editor hot-reloads on save, so you can edit `openapi.yaml` in your IDE and see the rendered docs / try-it-out panel update in the browser.

# Formats

## Bundle to JSON

```bash
npx --yes @redocly/cli@latest bundle openapi.yaml -o openapi.json
```

## Export to html

```bash
npx --yes @redocly/cli@latest build-docs openapi.yaml -o index.html
```