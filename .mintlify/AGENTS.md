# PARA API documentation agent rules

## Sources of truth

1. `openapi.json` is generated from FastAPI and is authoritative for the public
   contract.
2. `agent-manifest.json` summarizes capabilities and must match OpenAPI.
3. `llms.txt` is the curated navigation and directive layer.
4. `skill.md` is the installable capability and workflow summary.
5. `agents/handoff.mdx` defines operator inputs, approvals, checkpoints, and
   completion reporting.

## Required behavior

- Keep LinkedIn and Reddit coverage parallel where their contracts match.
- State platform differences explicitly instead of hiding them in generic prose.
- Describe side effects, required scopes, idempotency, terminal conditions, and
  safe retries for every agent workflow.
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
