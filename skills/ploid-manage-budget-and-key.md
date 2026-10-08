---
generated: '2026-10-07'
method: generated
name: manage-budget-and-key
description: Read remaining ACU and active key budgets before spending, and revoke the calling key when it leaks.
api: openapi/ploid-openapi.yml
operations: [getCredits, getAccountUsage, revokeCurrentApiKey]
source: >-
  operationIds verified in openapi/ploid-openapi.yml; behaviour from
  https://ploid.com/documentation/api/errors-and-limits and
  https://ploid.com/documentation/getting-started/authentication.
---

# Watch credits, budgets and the calling key

## Auth
- API key, scope `account:read`.

## Steps
1. Before a paid operation call `getCredits` (`GET /v1/account/credits`) for the workspace ACU balance.
2. Call `getAccountUsage` (`GET /v1/account/usage`) to read the key's active daily and monthly budget and usage context; `402 daily_budget_exceeded` / `monthly_budget_exceeded` mean the key, not the workspace, is exhausted.
3. Every billed response also reports `meta.usage` (`acu_used`, `billed[]`, `free[]`, `not_found[]`), so reconcile after each call rather than polling credits.
4. If the key appears in source control, logs, screenshots or chat, call `revokeCurrentApiKey` (`DELETE /v1/account/key`) with that key. The success response is the final response it can authorize; secrets cannot be redisplayed, so create a replacement key first.

## Rules
- One key per environment or integration, least-privilege scopes, budgets configured where available. See `authentication/ploid-authentication.yml` and `conventions/ploid-conventions.yml`.
