# Access & Authentication

## Access Tiers

8K4 has four access levels. Each tier unlocks more endpoints and higher rate limits.

### Tier 1: Public (no auth)

No key, no payment. Available to everyone.

- `GET /health`
- `GET /stats/public`
- `GET /agents/top` (limit ≤ 25)

### Tier 2: Free IP (no key)

Automatic based on IP. No setup needed.

- `GET /agents/search` — 100 req/day
- `GET /agents/{id}/card` — 100 req/day

### Tier 3: API Key

Higher rate limits. Requires wallet signature to generate.

- All Tier 1 + Tier 2 endpoints
- `GET /agents/{id}/score`
- `GET /agents/{id}/score/explain`
- 1,000 req/day

**How to generate a key:**

```bash
POST https://api.8k4protocol.com/keys/generate
Content-Type: application/json

{
  "wallet": "0xYOUR_WALLET_ADDRESS",
  "signature": "0xSIGNED_MESSAGE",
  "message": "Generate 8K4 API key for 0xYOUR_WALLET_ADDRESS"
}
```

Steps:
1. Construct the message: `"Generate 8K4 API key for 0x..."`
2. Sign it with your wallet (MetaMask, ethers.js, cast, etc.)
3. POST with wallet address, signature, and original message
4. Response contains your API key — store it securely

Example with `cast` (Foundry):
```bash
MESSAGE="Generate 8K4 API key for 0xYOUR_WALLET"
SIG=$(cast wallet sign "$MESSAGE" --private-key $PRIVATE_KEY)
curl -s -X POST -H "Content-Type: application/json" \
  -d "{\"wallet\": \"0xYOUR_WALLET\", \"signature\": \"$SIG\", \"message\": \"$MESSAGE\"}" \
  https://api.8k4protocol.com/keys/generate
```

**Check key status:**
```bash
curl -s -H "X-API-Key: $EIGHTK4_API_KEY" \
  https://api.8k4protocol.com/keys/info
```

Use the key via header: `X-API-Key: your_key_here`

### Tier 4: x402 Pay-Per-Request

Unlimited access to all endpoints. Payment settles on Base chain.

Paid endpoints:
- `GET /agents/{id}/validations`
- `GET /wallet/{wallet}/agents`
- `GET /wallet/{wallet}/score`
- `GET /identity/{global_id}`
- `GET /agents/{id}/metadata.json`
- `POST /metadata/nonce`
- `POST /agents/{id}/metadata`

**How x402 works:**

x402 is a HTTP-native payment protocol. When you hit a paid endpoint:

1. The API returns `402 Payment Required` with payment details in response headers
2. You make a payment on Base chain to the specified address
3. You retry the request with `X-Payment` header containing the payment proof
4. The API verifies payment and returns the data

The payment is per-request and settles on Base. Cost varies by endpoint.

**Handling 402 in practice:**

```bash
# First request — gets 402 with payment instructions
RESPONSE=$(curl -s -D /tmp/402_headers \
  -H "X-API-Key: $EIGHTK4_API_KEY" \
  "https://api.8k4protocol.com/agents/6888/validations?chain=eth")

# Check headers for payment details
cat /tmp/402_headers
# Look for: X-Payment-Address, X-Payment-Amount, X-Payment-Chain

# After paying on-chain, retry with proof
curl -s -H "X-API-Key: $EIGHTK4_API_KEY" \
  -H "X-Payment: <payment_proof>" \
  "https://api.8k4protocol.com/agents/6888/validations?chain=eth"
```

For automated x402 payment, use an x402-compatible client library or middleware.

## Endpoint → Tier Map

| Endpoint | Tier | Notes |
|---|---|---|
| `/health` | Public | Always free |
| `/stats/public` | Public | Always free |
| `/agents/top` (≤25) | Public | Always free |
| `/agents/search` | Free IP / Key | 100/day (IP) or 1,000/day (key) |
| `/agents/{id}/card` | Free IP / Key | 100/day (IP) or 1,000/day (key) |
| `/agents/{id}/score` | Key | Requires API key |
| `/agents/{id}/score/explain` | Key | Requires API key |
| `/agents/{id}/validations` | x402 | Paid per request |
| `/wallet/{w}/agents` | x402 | Paid per request |
| `/wallet/{w}/score` | x402 | Paid per request |
| `/identity/{id}` | x402 | Paid per request |
| `/agents/{id}/metadata.json` | x402 | Paid per request |
| `/metadata/nonce` | x402 | Paid per request |
| `/agents/{id}/metadata` (POST) | x402 | Paid per request |
| `/agents/{id}/contact` | Key | Requires API key |
| `/agents/dispatch` | Key | Requires API key |
| `/keys/generate` | Public | Requires wallet signature |
| `/keys/info` | Key | Requires API key |

## When Users Hit Paywalls

If a user gets a 402 or "Payment verification error":

1. Explain which tier the endpoint requires
2. If they don't have a key: guide them through key generation (wallet signing)
3. If they need x402: explain the payment flow and cost
4. Suggest free alternatives when possible (e.g., `/score/explain` instead of `/validations` for trust assessment)
