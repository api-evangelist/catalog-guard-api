---
name: Triage Catalog Guard blockers into a supplier fix list
description: >-
  Turn a Catalog Guard check response into an actionable, grouped remediation list for the
  supplier — separating mapping mistakes from genuine data defects from cross-row collisions.
api: openapi/catalog-guard-api-catalog-check-openapi.json
operations:
  - POST /api/v1/catalog/check
generated: '2026-08-09'
method: generated
source: >-
  Grounded in finding codes observed live from the running API on 2026-08-09 and the check
  families the provider documents on its homepage and /diagnostic page.
---

# Triage Catalog Guard blockers

A Catalog Guard response tells you *what* is wrong per row. This skill turns that into *who
fixes what*, which is a different question — most first-run blocker floods are one mapping
mistake, not hundreds of bad rows.

## Step 0 — rule out the mapping trap first

If **every** data row carries `missing_required_field` for **all five** of `supplier_sku`,
`title`, `price`, `stock`, `published`, stop. That is almost never a supplier data problem. It
means the CSV headers were not normalized — most often Shopify's own export headers were passed
through untouched. Fix the header mapping and re-run before sending anything to the supplier.

## Step 1 — group by code, not by row

Read `result.blockers[]` and bucket by `code`:

- **`missing_required_field`** — a required value is absent on that row. Owner: supplier, unless
  Step 0 applies. Report by field so the supplier sees "SKU missing on 12 rows", not 12 tickets.
- **`invalid_price`** — unparseable or ambiguous price. Owner: supplier. Usually currency symbols,
  thousands separators, ranges, or blank-as-zero.
- **`invalid_stock`** — unparseable stock. Owner: supplier. Usually text like "in stock", or a
  negative quantity.
- **`invalid_published`** — unparseable published flag. Owner: you, usually. The service wants a
  clean boolean; values like `MAYBE`, `Y`, or a locale word will not coerce, by design.
- **`duplicate_normalized_sku`** — two or more rows collapse to the same normalized SKU. Owner:
  **you and the supplier jointly** — this is the highest-risk finding on the list, because in an
  update-mode Shopify import a collision can overwrite the wrong product. The message names every
  colliding row; resolve which row is authoritative before importing anything.

## Step 2 — separate blockers from warnings

`result.blockers[]` must be resolved. `result.warnings[]` are for review — the documented warning
families include ambiguous weight values and incomplete option/variant pairs. Do not auto-fix a
warning; the whole point of the service is that ambiguity is surfaced rather than guessed.

## Step 3 — check `safeFixes[]` before hand-editing

`result.safeFixes[]` lists corrections the service considers unambiguous. Apply those first, then
re-run the check. Fewer hand edits means fewer new defects.

## Step 4 — convert row numbers to supplier-visible line numbers

`row` is 1-based and **includes the header row**. When you write the ticket, say "spreadsheet
line 2" for `row: 2` — that is what the supplier sees when they open the file. Do not subtract.

## Step 5 — re-run to zero

Re-submit the corrected file and confirm `result.blockerCount` is 0 across every chunk. Only then
consider the import. Before an update-mode import, take a store backup and confirm for each handle
whether it is meant to create or update a product — Catalog Guard cannot see your live catalog and
cannot tell you whether an overwrite is safe.

## Escalation

If the file is large or the findings are ambiguous, the provider sells a bounded human review: a
$149 CSV Diagnostic covering one CSV of up to 500 data rows, returning a written issue summary and
a recommended next action. It is not an audit, an import service, or an acceptance guarantee, and
you do not upload the catalog to request it.
