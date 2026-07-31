# catalog-guard-api
A bounded, fail-closed catalog preflight/validation service for Shopify-shaped supplier product CSVs. Exposes an unauthenticated JSON HTTP API that validates CSV text or normalized product rows and returns deterministic validation results. Does not accept file uploads, credentials, payment data, or 
