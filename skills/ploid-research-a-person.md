---
generated: '2026-10-07'
method: generated
name: research-a-person
description: Resolve one person to a permanent person_id, then run Ploid's reviewed deep research as a durable run and poll it to completion.
api: openapi/ploid-openapi.yml
operations: [resolvePerson, getPerson, getPersonRun, cancelPersonRun]
source: >-
  operationIds verified in openapi/ploid-openapi.yml; behaviour from
  https://ploid.com/documentation/api/person and https://ploid.com/documentation/api/errors-and-limits.
---

# Research a person

Get a source-reviewed person report with professional history, evidence links and field-level provenance.

## Auth
- API key as `Authorization: Bearer $PLOID_API_KEY` (or `x-api-key`), scope `people:enrich` for `getPerson`. See `authentication/ploid-authentication.yml`.

## Steps
1. If you only have a name plus a clue (company, role, location, email or phone), call `resolvePerson` (`POST /v1/resolve`) to obtain a permanent `person_id`. It is free when nothing is found.
2. Call `getPerson` (`POST /v1/person`) with exactly one identifier: `person_id`, `linkedin_url`, or `name` + `company_domain`. Send an `Idempotency-Key`.
3. A `200` is an eligible reviewed snapshot (`meta.stale: true` means a refresh was queued). A `202` returns `data.run_id` and `poll_url`: poll `getPersonRun` (`GET /v1/person/runs/{id}`) instead of retrying the POST. Research can take minutes; the deep call has a 15-minute deadline.
4. Stop polling on a terminal result. `410 run_expired` means the run is older than seven days.
5. Only call `cancelPersonRun` (`DELETE /v1/person/runs/{id}`) when the user asks to stop; cancellation does not refund work already done.

## Reading the result
- `summary`, `source_links` (up to 25), `presence.platforms[]`, `professional.history[]`, `provenance` keyed by field path.
- `data: null` with `meta.resolution: "unsure"` (`ambiguous_identity` or `insufficient_evidence`) is not "not found": retry with a `linkedin_url`.
- Search snippets are discovery leads, not supporting evidence.

## Cost
- First successful profile: 25 ACU on every plan; the same organization rereads free for 90 days. Uncertain, not-found, validation and failed outcomes use 0 ACU. See `plans/ploid-plans-pricing.yml`.

## Errors and limits
- `402 insufficient_acu`, `403 insufficient_scope`, `409 idempotency_conflict`, `429 rate_limited` (honour `Retry-After`). See `errors/ploid-problem-types.yml` and `rate-limits/ploid-rate-limits.yml`.
