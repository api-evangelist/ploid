---
generated: '2026-10-07'
method: generated
name: search-people
description: Run a bounded synchronous people search with structured filters and an optional semantic query, then hand a chosen result to person research.
api: openapi/ploid-openapi.yml
operations: [syncPeopleSearch, getPerson]
source: >-
  operationIds verified in openapi/ploid-openapi.yml; behaviour from
  https://ploid.com/documentation/api/search.
---

# Search for people

Return up to 150 people in one call for autocomplete, small cohorts or low-latency agent tools.

## Auth
- API key, scope `people:search`. See `authentication/ploid-authentication.yml`.

## Steps
1. Call `syncPeopleSearch` (`POST /v1/search`) with a semantic `query`, at least one structured filter, or both; `category` must be `people`; `num_results` defaults to 10 and accepts at most 150. Send an `Idempotency-Key`.
2. Pick `type`: `instant` (index only), `fast` (overfetch and rerank), `auto` (resolve a named-person miss from the public web when thin), `deep` (always add bounded public-web grounding).
3. Supported filters today: `title`, `seniority`, `company`, `industry`, `location`. `company_size`, `company_domain`, `radius_km` and `tenure` return `422 unsupported_filters` rather than being ignored.
4. Every result carries `person.person_id`; pass it straight to `getPerson` (`POST /v1/person`) for reviewed evidence. `person.identity_verified` stays false until research runs. Contact fields are excluded (`contact_status: "requires_enrichment"`).
5. A `202` with a background Set means the search was too large for the synchronous path: poll the Set instead.

## Errors and limits
- `503 search_unavailable` / `search_timeout` / `search_failed` fail closed with no partial results.
- Rejected `429` calls are never billed; honour `Retry-After`. See `rate-limits/ploid-rate-limits.yml`.
- For durable, criteria-verified lists use People Sets (preview, `POST /v1/sets`, not yet in the OpenAPI document).
