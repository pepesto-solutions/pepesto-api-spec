# Contributing

The canonical source is [`openapi.yaml`](./openapi.yaml). Everything else (bundled JSON, rendered HTML, the [pepesto-mcp](https://github.com/pepesto-solutions/pepesto-mcp) server, the [pepesto-api-examples](https://github.com/pepesto-solutions/pepesto-api-examples) snippets) is generated or hand-maintained downstream.

## Validate locally

The spec is validated against two parsers; both should pass before you commit a change.

```bash
# Redocly (recommended — clearer errors, also drives the doc preview)
npx --yes @redocly/cli@latest lint openapi.yaml

# Swagger / @apidevtools (the parser most import tools use under the hood)
npx --yes @apidevtools/swagger-cli@latest validate openapi.yaml
```

## Preview interactively

### Swagger UI ("try it out", via Docker)

```bash
docker run --rm -p 8081:8080 \
  -e SWAGGER_JSON=/spec/openapi.yaml \
  -v "$(pwd):/spec" \
  swaggerapi/swagger-ui
# open http://localhost:8081
```

To fire requests from the UI, paste your `pep_sk_…` key into the **Authorize** dialog (the `bearerAuth` scheme).

## Build outputs

### Bundle to JSON

```bash
npx --yes @redocly/cli@latest bundle openapi.yaml -o openapi.json
```

### Render to HTML

```bash
npx --yes @redocly/cli@latest build-docs openapi.yaml -o index.html
```
