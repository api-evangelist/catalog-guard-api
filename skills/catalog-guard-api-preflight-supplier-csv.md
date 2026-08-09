---
name: Preflight a supplier catalog CSV before a Shopify import
description: >-
  Run a supplier product CSV through Catalog Guard's fail-closed validator and turn the
  deterministic blockers and warnings into a go/no-go decision, without ever connecting to a
  store or performing an import.
api: openapi/catalog-guard-api-catalog-check-openapi.json
operations:
  - POST /api/v1/catalog/check
  - GET /api/v1/catalog/health
generated: '2026-08-09'
method: generated
source: >-
  Grounded in the two operations the provider's live OpenAPI 3.1.0 actually declares. The spec
  publishes no operationIds, so steps bind by method + path. Behaviour verified by probe
  2026-08-09.
---

# Preflight a supplier catalog CSV

Use this before importing a supplier's product CSV into Shopify. Catalog Guard tells you which
rows are safe and which are ambiguous. It does **not** import anything, does not touch a store,
and does not accept credentials.

## Before you start

- **No authentication.** No key, no token, no signup. Do not send credentials, payment data or
  customer data — the service explicitly does not accept them.
- **Normalize the headers first.** This is the step people get wrong. The `csv` branch requires
  `supplier_sku,title,price,stock,published`. If you feed it Shopify's own export headers
  (`Handle,Title,Variant SKU,Variant Price,Variant Inventory Qty,Published`) the request
  succeeds but every row comes back blocked with `missing_required_field`, which looks like a
  data problem and is actually a mapping problem.
- **Respect the bounds.** 250 rows per request, or 98304 characters of CSV text, inside a
  131072-byte body. Chunk larger catalogs client-side.

## Steps

1. **(Optional) Check the service is up and read its current limits.**
   `GET /api/v1/catalog/health` returns `{schemaVersion, status, service, version, storage,
   rateLimit}`. Confirm `status` is `ok` and note `rateLimit` — currently "best-effort 20
   requests per minute per Cloudflare isolate".

2. **Normalize the supplier file.** Map the supplier's columns onto `supplier_sku`, `title`,
   `price`, `stock`, `published`, plus any of `vendor`, `category`, `weight`, `tags`. Leave
   supplier categories alone — Catalog Guard treats `category` as audit-only and will not claim
   a verified Shopify taxonomy mapping, and neither should you.

3. **Submit one chunk.** `POST /api/v1/catalog/check` with
   `content-type: application/json` and a body containing **exactly one** of:
   - `{"csv": "supplier_sku,title,price,stock,published\nSKU-1,Widget,12.50,3,true\n"}`
   - `{"rows": [{"supplier_sku": "SKU-1", "title": "Widget", "price": "12.50", "stock": 3, "published": true}]}`

   Sending both keys, neither, or an extra top-level key returns 400 `invalid_shape`.

4. **Read `result`, not the HTTP status.** A fully-blocked catalog still returns 200. The
   decision lives in the body:
   - `result.safeRows` — rows that passed everything.
   - `result.blockerCount` / `result.blockers[]` — must be resolved before import.
   - `result.warningCount` / `result.warnings[]` — review, not necessarily fix.
   - `result.safeFixes[]` — corrections the service considers unambiguous.

5. **Map each finding back to the source row.** Each finding is `{row, field, code, message}`.
   `row` is 1-based **including the header row**, so a finding on `row: 2` is your first data
   row. Finding codes observed in practice: `missing_required_field`, `invalid_price`,
   `invalid_stock`, `invalid_published`, `duplicate_normalized_sku`. A SKU collision is emitted
   once per participating row and names every colliding row in the message.

6. **Repeat per chunk and aggregate.** Sum `safeRows`, `blockerCount` and `warningCount` across
   chunks. Do not paginate — there is no cursor; chunking is the caller's job.

7. **Decide.** Import only if `blockerCount` is 0 across every chunk. Anything else goes back to
   the supplier or to a manual fix pass. Catalog Guard is a preflight, not an import validator,
   and it makes no guarantee that Shopify will accept the file.

## Error handling

| Status | `error.code` | What to do |
|---|---|---|
| 400 | `invalid_shape` | Send exactly one of `csv` or `rows`, and no other top-level key. |
| 400 | `invalid_rows` | More than 250 rows. Chunk smaller. |
| 405 | `method_not_allowed` | Use POST, not GET. |
| 413 | `body_too_large` | Body over 131072 bytes. Chunk smaller. |
| 415 | `unsupported_media_type` | Add `content-type: application/json`. |
| 429 | (unobserved) | Rate limited. Back off — there is no `Retry-After` header to read. |

Branch on `error.code`, never on `error.message`. Errors are **not** RFC 9457 problem details;
the envelope is `{"schemaVersion": "...", "error": {"code": "...", "message": "..."}}`.
The discriminator between success and failure is whether the body has `result` or `error`.

## Retries

The endpoint is stateless and stores nothing, so a retry is harmless and returns the same result.
There is no `Idempotency-Key` header and no published idempotency guarantee — retry safety here
is a property of the design, not a promise.

## What this skill will not do

Connect to a Shopify store, compare a CSV against live handles, decide whether an overwrite is
safe, perform or schedule an import, or replace a store-level backup. Before any update-mode
import, export and retain a backup and review the import preview yourself.
