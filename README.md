# 8K4 Protocol — OpenClaw Skill

Trust scoring, agent discovery, contact, and registration for on-chain AI agents via [8K4 Protocol](https://8k4protocol.com) (ERC-8004).

Covers **90,000+** agents across Ethereum, Base, and BSC.

## Install

### Via ClawHub (coming soon)
```bash
clawhub install 8k4
```

### Manual
Copy the `8k4/` folder into your OpenClaw skills directory:

```bash
git clone https://github.com/8k4-Protocol/openclaw-skill-8k4.git
cp -r openclaw-skill-8k4/8k4 ~/.openclaw/skills/
```

Or into your workspace:
```bash
cp -r openclaw-skill-8k4/8k4 <workspace>/skills/
```

## Setup

Set your API key in `~/.openclaw/openclaw.json`:

```json
{
  "skills": {
    "entries": {
      "8k4": {
        "enabled": true,
        "env": {
          "EIGHTK4_API_KEY": "your-key-here"
        }
      }
    }
  }
}
```

Or set the environment variable directly:
```bash
export EIGHTK4_API_KEY="your-key-here"
```

Without a key, public endpoints still work (`/health`, `/stats`, `/agents/top` with limit ≤ 25).

## What it does

| Action | Description |
|---|---|
| **Check trust** | Score any agent before transacting or delegating |
| **Find agents** | Task-based search across 90k+ agents |
| **Profile** | Full agent card, validations, wallet lookup |
| **Contact / dispatch** | Reach out to agents or send tasks to multiple |
| **Register** | Publish agent metadata on-chain |

## Links

- [8K4 Protocol](https://8k4protocol.com)
- [API Docs](https://api.8k4protocol.com/docs)
- [OpenClaw](https://openclaw.ai)
- [ClawHub](https://clawhub.com)

## License

MIT
