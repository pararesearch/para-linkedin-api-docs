---
name: para-social-research
description: Use the PARA Social Research API to manage owner-scoped LinkedIn and Reddit research registrations, request real-extension collection, consume immutable relevant-post batches, and recover signed webhook delivery.
license: Proprietary
compatibility: Requires HTTPS access to the public API and an owner-scoped Bearer API key supplied through an environment variable.
metadata:
  author: PARA
  version: "1.2.0"
---

# PARA Social Research API

Use this skill for API-driven LinkedIn and Reddit research. The API controls
registration intent, lifecycle, and result delivery. Collection is asynchronous
and runs only through the already-paired real Chrome extension.

## Canonical context

- Agent index: `https://para-linkedin-api.mintlify.site/llms.txt`
- Agent handoff: `https://para-linkedin-api.mintlify.site/agents/handoff.md`
- Machine manifest: `https://para-linkedin-api.mintlify.site/agent-manifest.json`
- A2A agent card: `https://para-linkedin-api.mintlify.app/.well-known/agent-card.json`
- OpenAPI 3.1: `https://para-linkedin-api.onrender.com/openapi.json`
- API origin: `https://para-linkedin-api.onrender.com`
- Readiness: `https://para-linkedin-api.onrender.com/health/ready`

OpenAPI is authoritative for methods, paths, schemas, and webhook contracts.

## Required inputs

Read these from the operator handoff:

- platform: `linkedin` or `reddit`
- mode: `inspect`, `manage`, `run`, or `recover_delivery`
- API key environment-variable name, normally `PARA_API_KEY`
- stable registration `external_id`
- name and either a detailed LinkedIn `use_case` or legacy topics/context
- platform-specific sources or queries
- public HTTPS webhook URL for activation
- mutation approvals
- completion target and deadline

Do not guess missing business inputs, webhook destinations, communities, target
users, or approvals.

## Secret policy

- Read credentials from the named environment variable.
- Never print, log, serialize, or return API keys, temporary passwords, webhook
  secrets, raw authorization headers, or receiver inspection tokens.
- Route one-time secrets directly to an approved secret store.
- Stop on `401` or `403`; do not attempt credential discovery or repair.

## Action policy

Safe without additional approval:

- health and readiness
- list and get
- registration status
- batch and post polling
- delivery and attempt inspection

Allowed only when the handoff authorizes the mutation:

- create or patch a draft
- activate, pause, restore, or run
- test a webhook

Require explicit operator approval:

- archive a registration
- rotate a webhook secret
- replay a delivery
- create, rotate, or revoke API keys

Unsupported:

- direct LinkedIn or Reddit scraping
- Playwright, Chromium, or fixture collection as a production substitute
- posting, commenting, voting, or automatic draft submission
- real-world identity resolution for Reddit authors

## Deterministic workflow

1. Call `GET /health/ready`; stop if it is not successful.
2. Load OpenAPI and select operations under `/v1/{platform}`.
3. List registrations and match the exact operator-supplied `external_id`.
4. Patch only explicitly supplied fields, or create one registration when approved.
5. Persist each one-time webhook secret outside the transcript.
6. For LinkedIn `use_case` creation, poll compilation, inspect `/plan`, require
   exactly ten unique searches, and observe auto-activation when enabled.
   Query compatibility is server-owned: activation additionally requires
   complete queries of at most 120 characters with balanced Boolean syntax.
7. Activate a legacy or non-auto-activating registration only with a ready
   requirement, a public HTTPS webhook, and approval.
8. Request a run only with approval and persist returned cycle and run IDs.
9. Poll status with bounded intervals until all requested runs are terminal.
10. Respect `delivery_time`; a manual run does not force an immediate batch.
11. Read the immutable batch by webhook or polling.
12. Treat `posts: []` as successful completion.
13. Return a redacted checkpoint with resource IDs, states, counts, request IDs,
    and remaining operator actions.

Topics-only LinkedIn and all Reddit creation remain side-effect free. A
LinkedIn registration containing `use_case` queues server-side compilation and,
when `auto_activate` is true and the webhook is valid, may atomically queue one
cycle of exactly ten real-extension searches.

## Idempotency

Use one stable `Idempotency-Key` for each intended mutation and reuse it only for
an identical retry.

Examples:

```text
create:linkedin:founder-workflows:v1
activate:reddit:customer-pain:v1
run:reddit:customer-pain:2026-07-29
```

Never reuse a key with a different body or operation.

## Retry decisions

| Result | Action |
| --- | --- |
| `2xx` | Persist the redacted checkpoint and continue |
| `400`, `409`, `422` | Correct the request; do not repeat unchanged |
| `401` | Stop and request a valid credential |
| `403` | Stop and request the missing scope |
| `404` | Refresh owner-scoped state; do not infer cross-owner existence |
| `429` | Honor bounded `Retry-After` and reuse the idempotency key |
| `5xx` or transport failure | Use bounded backoff and reuse the idempotency key |

## Platform differences

LinkedIn:

- one detailed `use_case` can compile into exactly ten searches, post and
  author/ICP rubrics, prompts, hard filters, thresholds, and scoring weights;
- collection expands post content and visible media in the extension;
- only initially relevant authors receive Parallel enrichment;
- unavailable author evidence is neutral `UNKNOWN`, not an automatic rejection;
- final scoring is deterministic from persisted components and weights;
- final batches include plan revision, source provenance, both scoring stages,
  enrichment coverage, persona hash, and optional advisory comment drafts.

Reddit:

- collection includes selftext, media, engagement, top comments, and provenance;
- public author context is extension-only and cached for seven days;
- no Parallel or real-world identity resolution is allowed;
- reply drafts are advisory and participation remains dashboard-only.

## Webhook consumption

For both platforms:

1. Read the raw body before JSON parsing.
2. Verify `v1=HMAC-SHA256(timestamp + "." + raw_body)` with constant-time
   comparison.
3. Deduplicate on `X-Webhook-Id`.
4. Persist durably before returning `2xx`.
5. Reconcile from batch polling when delivery is unavailable.
6. Replay only after the destination is fixed and approval is present.

Events:

- `linkedin.relevant_posts.daily.v1`
- `reddit.relevant_posts.daily.v1`

## Final report

Return platform, external ID, registration ID and state, mutations and
idempotency keys, run/batch/delivery IDs, collected/matched/selected counts,
empty-batch status, webhook or polling outcome, request IDs, and remaining
approval needs. Never include secrets or full post bodies.
