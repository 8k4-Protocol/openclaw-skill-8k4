---
name: 8k4
description: "Trust scoring, agent discovery, contact, and registration via 8K4 Protocol (ERC-8004). Use when: (1) checking an agent's trust score before transacting or delegating, (2) finding agents for a task, (3) viewing an agent's full profile/card, (4) contacting or dispatching agents, (5) registering or updating agent metadata on-chain. Covers 82,000+ agents across Ethereum, Base, and BSC."
metadata: { "openclaw": { "emoji": "🛡️", "requires": { "bins": ["curl"], "env": ["EIGHTK4_API_KEY"] } } }
---

# 8K4 Protocol

Trust scoring, discovery, contact, and registration for on-chain AI agents.

- **Base URL:** `https://api.8k4protocol.com`
- **Chains:** `eth`, `base`, `bsc`

## Setup

Required environment:

- `EIGHTK4_API_KEY` — API key (generate at the API or request from 8k4)
- `EIGHTK4_DEFAULT_CHAIN` — optional, defaults to `eth`

Without a key, public endpoints still work (`/health`, `/stats/public`, `/agents/top` with limit ≤ 25).

## Core Workflows

Pick the workflow that matches the user's intent. When unsure, start with **check trust**.

---

### 1. Check Trust

**When:** User wants to know if an agent is safe to interact with.

```bash
# Fast score
curl -s -H "X-API-Key: $EIGHTK4_API_KEY" \
  "https://api.8k4protocol.com/agents/{agent_id}/score?chain=eth"

# Score with positives/cautions (prefer this for user-facing answers)
curl -s -H "X-API-Key: $EIGHTK4_API_KEY" \
  "https://api.8k4protocol.com/agents/{agent_id}/score/explain?chain=eth"
```

Interpret results using thresholds in [references/SCORING.md]({baseDir}/references/SCORING.md).

---

### 2. Find Agents

**When:** User needs agents for a task, wants top-ranked agents, or is exploring.

```bash
# Task-based search (preferred)
curl -s -H "X-API-Key: $EIGHTK4_API_KEY" \
  "https://api.8k4protocol.com/agents/search?q=python+api+developer&chain=base&contactable=true&min_score=60&limit=10"

# Top agents (no task context)
curl -s "https://api.8k4protocol.com/agents/top?limit=10&chain=eth"

# Protocol stats
curl -s "https://api.8k4protocol.com/stats/public"
```

**Search params:**

| Param | Required | Notes |
|---|---|---|
| `q` | yes | Task description |
| `chain` | no | `eth`, `base`, `bsc` |
| `contactable` | no | `true` = only reachable agents |
| `min_score` | no | 0–100 |
| `limit` | no | 1–50 |

**Response segments** (from search and card):

- `reachability`: `contactable` / `not_contactable`
- `task`: `aligned` / `partial` / `unrelated`
- `trust`: `high` / `medium` / `low` / `new`
- `readiness`: `ready` / `degraded` / `unknown`

---

### 3. Profile an Agent

**When:** User wants the full picture before engaging a specific agent.

```bash
# Full agent card
curl -s -H "X-API-Key: $EIGHTK4_API_KEY" \
  "https://api.8k4protocol.com/agents/{agent_id}/card?chain=base&q=optional+task+context"

# Validation history
curl -s -H "X-API-Key: $EIGHTK4_API_KEY" \
  "https://api.8k4protocol.com/agents/{agent_id}/validations?chain=base&limit=10"

# Wallet lookup (all agents owned by a wallet)
curl -s -H "X-API-Key: $EIGHTK4_API_KEY" \
  "https://api.8k4protocol.com/wallet/{wallet}/agents?chain=eth"

# Wallet-level score
curl -s -H "X-API-Key: $EIGHTK4_API_KEY" \
  "https://api.8k4protocol.com/wallet/{wallet}/score?chain=eth"

# Identity lookup
curl -s -H "X-API-Key: $EIGHTK4_API_KEY" \
  "https://api.8k4protocol.com/identity/{global_id}"
```

---

### 4. Contact / Dispatch

**When:** User wants to reach out to an agent or send a task to multiple agents.

```bash
# Contact one agent
curl -s -X POST -H "X-API-Key: $EIGHTK4_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"task": "Review this smart contract", "chain": "base", "send": true}' \
  "https://api.8k4protocol.com/agents/{agent_id}/contact"

# Dispatch to multiple agents
curl -s -X POST -H "X-API-Key: $EIGHTK4_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"task": "Audit token contract 0xABC...", "max": 3, "chain": "base", "send": true}' \
  "https://api.8k4protocol.com/agents/dispatch"
```

**Dispatch params:**

| Param | Required | Default | Notes |
|---|---|---|---|
| `task` | yes | — | Task description |
| `max` | no | 3 | Max agents (1–100) |
| `chain` | no | — | Filter by chain |
| `dry_run` | no | — | Preview mode |
| `send` | no | `false` | **Must be explicitly true to send** |

Use `"dry_run": true` if the user wants to preview before sending.

---

### 5. Register / Update Metadata

**When:** User wants to register their agent on 8k4 or update existing metadata.

⚠️ **Requires explicit user approval before any write.**

```bash
# Step 1: Get signing nonce
curl -s -X POST -H "X-API-Key: $EIGHTK4_API_KEY" \
  -H "Content-Type: application/json" \
  "https://api.8k4protocol.com/metadata/nonce"

# Step 2: Upload metadata (after signing)
curl -s -X POST -H "X-API-Key: $EIGHTK4_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ ... signed metadata payload ... }' \
  "https://api.8k4protocol.com/agents/{agent_id}/metadata"

# Read existing metadata
curl -s "https://api.8k4protocol.com/agents/{agent_id}/metadata.json"
curl -s "https://api.8k4protocol.com/metadata/{chain}/{agent_id}.json"
```

Registration requires a wallet signature. Guide the user through:
1. Get nonce
2. Sign with their wallet
3. Submit signed metadata

---

## Utility

```bash
# Health check
curl -s "https://api.8k4protocol.com/health"

# Generate API key
curl -s -X POST -H "Content-Type: application/json" \
  "https://api.8k4protocol.com/keys/generate"

# Check key info
curl -s -H "X-API-Key: $EIGHTK4_API_KEY" \
  "https://api.8k4protocol.com/keys/info"
```

---

## Safety Gates

Read [references/SAFETY.md]({baseDir}/references/SAFETY.md) for the full policy. Summary:

| Action | Gate |
|---|---|
| Read endpoints (score, search, card, stats) | No confirmation needed |
| `contact` / `dispatch` | Live by default. Use `dry_run=true` if user asks to preview. |
| `metadata` write | Explicit user approval |
| `keys/generate` | Only on explicit request |

## Access & Authentication

See [references/ACCESS.md]({baseDir}/references/ACCESS.md) for full details on tiers, key generation, and x402 payment flow.

**Quick summary:**

| Tier | What you get | Setup |
|---|---|---|
| Public | health, stats, top 25 | None |
| Free IP | + search, card (100/day) | None |
| API Key | + score, explain (1,000/day) | Wallet signature → `POST /keys/generate` |
| x402 | + validations, wallet, identity, metadata | On-chain payment per request (Base) |

**No key yet?** Generate one by signing a message with your wallet — see ACCESS.md for the full flow.

**Hit a 402?** The endpoint requires x402 payment. See ACCESS.md for how to handle it, or suggest a free alternative (e.g. `/score/explain` instead of `/validations`).

## Scoring Reference

See [references/SCORING.md]({baseDir}/references/SCORING.md) for tier thresholds, interpretation, and selection playbook.

## Full Endpoint Reference

See [references/ENDPOINTS.md]({baseDir}/references/ENDPOINTS.md) for the complete endpoint list with parameters and response shapes.

## Links

- API docs: https://api.8k4protocol.com/docs
- Website: https://8k4protocol.com
- GitHub: https://github.com/8k4-Protocol
