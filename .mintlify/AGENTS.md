# PARA API documentation agent rules

## Sources of truth

1. `openapi.json` is generated from FastAPI and is authoritative for the public
   contract.
2. `agent.mdx` is the complete narrative operating contract and the only
   required documentation page for an executing agent.
3. `agent-manifest.json` summarizes capabilities and must match OpenAPI.
4. `llms.txt` is a minimal bootstrap that points to `agent.mdx` and OpenAPI.
5. `skill.md` is the installable capability summary and must point to
   `agent.mdx`.

Agent-readable documentation uses `para-linkedin-api.mintlify.site`. Mintlify's
generated skill indexes and A2A agent card use the direct
`para-linkedin-api.mintlify.app` machine origin. Legacy pages remain available
for old links but are not primary navigation or required reading.

## Required behavior

- Keep LinkedIn and Reddit coverage parallel where their contracts match.
- State platform differences explicitly instead of hiding them in generic prose.
- Describe side effects, required scopes, idempotency, terminal conditions, and
  safe retries for every agent workflow.
- Distinguish topics-only LinkedIn creation from `use_case` creation:
  compiler-native creation queues compilation and may auto-activate exactly ten
  real-extension searches after plan and webhook validation.
- Require agents to poll compilation and inspect the generated plan before
  manual lifecycle actions or advanced overrides.
- Document explicit hard-filter provenance, inferred soft preferences, neutral
  unknown evidence, first-pass-only enrichment, and deterministic final scoring.
- Keep collection extension-only. Never document Chromium, Playwright, fixture
  uploaders, or direct scraping as production collection paths.
- Keep generated drafts advisory. Never imply that the public API posts,
  comments, votes, or submits drafts automatically.
- Never add API keys, temporary passwords, webhook secrets, authorization
  headers, session material, or receiver tokens to documentation or examples.
- Use obvious placeholders and environment variables in commands.
- Treat archive, webhook-secret rotation, delivery replay, and API-key lifecycle
  operations as explicit-approval actions.
- Link to OpenAPI instead of manually duplicating large schemas.
- Preserve existing LinkedIn URLs when adding Reddit or agent content.
- Keep primary navigation limited to the canonical agent contract and OpenAPI.
- Keep `agent.mdx` compact enough for one-pass loading while retaining the
  complete lifecycle, platform differences, delivery, recovery, and safety
  contract.

## Validation

Run from the application repository:

```bash
npm run openapi:check
npm run docs:agents:check
npm run docs:validate
```

Do not publish when generated OpenAPI differs, an agent-required operation loses
its `Idempotency-Key` declaration, a curated link fails, or a credential-shaped
value is detected.
