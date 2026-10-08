---
generated: '2026-10-07'
method: generated
name: enrich-contact-and-social
description: Resolve work email, personal email, phone and social profile fields for a known identity, with per-field billing and partial-failure handling.
api: openapi/ploid-openapi.yml
operations: [enrichPerson, enrichSocialProfile, getLinkedInProfile]
source: >-
  operationIds verified in openapi/ploid-openapi.yml; behaviour from
  https://ploid.com/documentation/api/enrichment and https://ploid.com/documentation/api/social.
---

# Enrich contact and social fields

## Auth
- API key, scope `people:enrich` for `enrichPerson` and `enrichSocialProfile`; `linkedin:read` for `getLinkedInProfile`.

## Steps
1. Use the strongest identity anchor you have (a canonical LinkedIn URL beats a name alone).
2. Call `enrichPerson` (`POST /v1/enrich`) with exactly one of `linkedin_url`, `person_id`, `email` (reverse lookup) or `name` + `company_domain`, and a `fields` array such as `["work_email", "personal_email", "phone"]`. Send an `Idempotency-Key`.
3. Every requested field comes back with `value`, `status` (`found`, `not_found`, `unavailable`), `confidence`, `source`, `last_seen`. Only successful reveals are billed (`meta.usage.billed[]`); transport or provider failures appear in `meta.warnings`.
4. If every field fails you get `503 upstream_unavailable` (or `upstream_timeout` after the 80-second provider deadline) with `error.retryable: true` and `Retry-After`: retry with the same `Idempotency-Key`.
5. For one public social profile call `enrichSocialProfile` (`POST /v1/socials`) with `platform` (linkedin, x, instagram, tiktok, youtube, github, reddit, facebook) and `identifier` (handle, slug or URL; a URL's domain must match the platform). `404 profile_not_found` and provider failures are not charged.
6. For a raw public LinkedIn read use `getLinkedInProfile` (`GET /v1/linkedin/profile?url=...`).

## Limits
- `/v1/socials` and `/v1/linkedin/*` are each capped at 60 requests per minute and 600 per hour on top of the organization and key buckets. See `rate-limits/ploid-rate-limits.yml`.
- Do not send cookies, proxies or provider credentials; the API does not accept them.
