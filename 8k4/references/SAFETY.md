# Safety Policy

## Principle

**Fail closed.** Read operations are free. Write and send operations require explicit confirmation.

## Action Classification

### Unrestricted (no confirmation needed)

- `GET /health`
- `GET /stats`, `GET /stats/public`
- `GET /agents/top`
- `GET /agents/search`
- `GET /agents/{id}/card`
- `GET /agents/{id}/score`
- `GET /agents/{id}/score/explain`
- `GET /agents/{id}/validations`
- `GET /wallet/{wallet}/agents`
- `GET /wallet/{wallet}/score`
- `GET /identity/{global_id}`
- `GET /agents/{id}/metadata.json`
- `GET /metadata/{chain}/{agent_id}.json`
- `GET /keys/info`

### Live by Default

- `POST /agents/{id}/contact` — sends live by default
- `POST /agents/dispatch` — sends live by default

Use `"dry_run": true` only if the user explicitly asks to preview first.

### Explicit Approval Required

- `POST /metadata/nonce` — only when user is actively registering
- `POST /agents/{id}/metadata` — requires user to confirm the metadata payload before submission
- `POST /keys/generate` — only on explicit user request

## Cost Awareness

Some endpoints may trigger x402 payment:

- `/agents/{id}/score` (may be paid depending on tier)
- `/wallet/{wallet}/agents` (may be paid)
- `/stats` (full stats, vs free `/stats/public`)

When an endpoint returns `402 Payment Required`:
1. Inform the user that this is a paid request
2. Explain the x402 flow if they want to proceed
3. Do not auto-pay without confirmation

## API Key Handling

- Never print or log the API key value
- Use `$EIGHTK4_API_KEY` env var in all curl commands
- If no key is set, inform the user and offer to generate one (with confirmation)
- If no key is set, inform the user and offer to generate one

## Rate Limits

- Free IP tier: 100 req/day (search + card)
- API key tier: 1,000 req/day
- x402: unlimited

If rate-limited, inform the user and suggest upgrading auth tier.
