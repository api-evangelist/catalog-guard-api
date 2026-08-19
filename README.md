# Catalog Guard API (catalog-guard-api)

A bounded, fail-closed catalog preflight and validation service for Shopify-shaped supplier product CSVs. Exposes an unauthenticated JSON HTTP API that validates raw CSV text or normalized product rows and returns a deterministic result: safe rows, blockers and warnings, each finding carrying a row index, field, stable code and message. Ambiguous input is refused rather than guessed — unclosed quotes, malformed rows, duplicate headers, duplicate normalized SKUs and incomplete variant pairs all become blockers, and supplier categories are treated as audit-only rather than mapped to a Shopify taxonomy. The service does not accept file uploads, credentials, payment data or store connections, holds no storage, and never imports or modifies a catalog; every successful response repeats those disclosures inline as machine-readable fields. Access is controlled by input bounds and a best-effort rate limit rather than by identity. Commercially it is fronted by a free in-browser CSV preflight, a $149 bounded human CSV Diagnostic, and a separate free Shopify store-launch referral path.

**APIs.json:** [https://catalog-guard-api.apievangelist.com/apis.yml](https://catalog-guard-api.apievangelist.com/apis.yml)

## Tags

- ecommerce
- catalog-validation
- shopify
- data-quality
- csv-validation
- product-data-qa
- data-preflight
- data-validation
- retail

## Timestamps

- **Created:** 2026-07-31
- **Modified:** 2026-08-09

## APIs

### Catalog Guard API Catalog API

The Catalog API from Catalog Guard API — 2 operation(s) for catalog.

- **Human URL:** [https://catalogguard.noahcortezj-c.workers.dev/api/v1/catalog/docs](https://catalogguard.noahcortezj-c.workers.dev/api/v1/catalog/docs)
- **Base URL:** `https://catalogguard.noahcortezj-c.workers.dev`

#### Tags

- Catalog

#### Properties

- [OpenAPI](openapi/catalog-guard-api-catalog-api-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Postman Collection](collections/catalog-guard-api-catalog-api.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/catalog-guard-api-catalog-api.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Documentation](https://catalogguard.noahcortezj-c.workers.dev/api/v1/catalog/docs)
- [API Reference](https://catalogguard.noahcortezj-c.workers.dev/openapi.json)
- [Getting Started](https://catalogguard.noahcortezj-c.workers.dev/api/v1/catalog/docs)

## Common Properties

- [Documentation](https://catalogguard.noahcortezj-c.workers.dev/api/v1/catalog/docs)
- [API Reference](https://catalogguard.noahcortezj-c.workers.dev/openapi.json)
- [Getting Started](https://catalogguard.noahcortezj-c.workers.dev/api/v1/catalog/docs)
- [Pricing](https://catalogguard.noahcortezj-c.workers.dev/diagnostic)
- [Terms of Service](https://catalogguard.noahcortezj-c.workers.dev/terms)
- [Privacy Policy](https://catalogguard.noahcortezj-c.workers.dev/privacy)
- [A P Is Json](https://catalogguard.noahcortezj-c.workers.dev/apis.json)
- [OpenAPI](openapi/_original/catalog-guard-api-catalog-check-openapi.json) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Open A P I Overlay](overlays/catalog-guard-api-catalog-check-overlay.yaml)
- [Authentication](authentication/catalog-guard-api-authentication.yml)
- [Conventions](conventions/catalog-guard-api-conventions.yml)
- [Error Catalog](errors/catalog-guard-api-problem-types.yml)
- [Lifecycle](lifecycle/catalog-guard-api-lifecycle.yml)
- [Rate Limits](rate-limits/catalog-guard-api-rate-limits.yml)
- [Conformance](conformance/catalog-guard-api-conformance.yml)
- [Data Model](data-model/catalog-guard-api-data-model.yml)
- [Examples](examples/catalog-guard-api-examples.yml)
- [M C P Server](mcp/catalog-guard-api-mcp.yml)
- [Tool Crosswalk](mcp/catalog-guard-api-tool-crosswalk.yml)
- [L L Ms Txt](llms/catalog-guard-api-llms.txt)
- [Agent Skill](skills/_index.yml)
- [Domain Security](security/catalog-guard-api-domain-security.yml)

## Maintainers

**FN:** Catalog Guard
**URL:** https://catalogguard.noahcortezj-c.workers.dev/
