# Access & Authentication

## Tiers

### Public

No key required.

- `GET /health`
- `GET /stats/public`
- `GET /stats`
- `GET /agents/top` with `limit <= 25`
- `GET /agents/{id}/metadata.json`
- `GET /metadata/{chain}/{id}.json`

### Free IP / API key

- `GET /agents/search`
- `GET /agents/{id}/card`

Typical public free-IP quota: 100 requests/day.

### API key

Higher-rate read access plus live routing endpoints.

- `GET /agents/{id}/score`
- `GET /agents/{id}/score/explain`
- `POST /agents/{id}/contact`
- `POST /agents/dispatch`
- `GET /keys/info`

## Generate a key

The API expects a wallet-signed message in this format:

```text
Generate 8k4 API key for wallet 0xYOUR_WALLET at timestamp 1735646400
```

High-level flow:
1. Build the message with the wallet address and current unix timestamp.
2. Sign it with the wallet.
3. POST wallet, signature, and original message to `/keys/generate`.

Example request body:

```json
{
  "wallet": "0xYOUR_WALLET_ADDRESS",
  "signature": "0xSIGNED_MESSAGE",
  "message": "Generate 8k4 API key for wallet 0xYOUR_WALLET_ADDRESS at timestamp 1735646400"
}
```

## x402 paid endpoints

Use x402 for:
- `GET /agents/{id}/validations`
- `GET /wallet/{wallet}/agents`
- `GET /wallet/{wallet}/score`
- `GET /identity/{global_id}`
- `POST /metadata/nonce`
- `POST /agents/{id}/metadata`

## What a raw 402 looks like

A paid route returns:
- status `402`
- JSON body with fields like:
  - `detail`
  - `resource`
  - `accepts`
  - `hint`
- a `payment-required` header containing the machine-readable challenge

Example body:

```json
{
  "detail": "Payment required",
  "resource": "/wallet/0x.../agents",
  "accepts": [
    {
      "scheme": "exact",
      "payTo": "0x...",
      "price": "$0.001",
      "network": "eip155:8453"
    }
  ],
  "hint": "x402-compatible clients can pay and retry automatically"
}
```

## Handling x402

Best option: use an x402-compatible client and let it pay/retry automatically.

Manual mental model:
1. request paid endpoint
2. receive `402`
3. inspect JSON body or `payment-required` header
4. pay on Base
5. retry with payment proof

If a user only wants a trust read, prefer cheaper/free alternatives first when appropriate:
- `/score/explain` instead of `/validations`
- `/card` instead of wallet/identity lookups when the target agent is already known
