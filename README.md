# BIT Skills

Agent skills for the [BIT](https://www.matrixport.com) platform. Compatible with Claude Code, GitHub Copilot, Cursor, and any tool that supports the [Agent Skills open standard](https://agentskills.io).

## Available Skills

| Skill | Description |
|-------|-------------|
| [bit/us-equity](skills/bit/us-equity/) | Trade US equities on BIT — place, edit and cancel orders, check order status, view balance and positions |

## Installation

### Claude Code — via `npx skills add`

Install globally (available across all projects):
```bash
npx skills add zkvm/skills --skill us-equity -g
```

Install for a specific project only:
```bash
npx skills add zkvm/skills --skill us-equity
```

### OpenClaw — via `clawhub`

> Coming soon — the skill will be published to the [ClawHub registry](https://clawhub.ai) shortly.

```bash
clawhub install bit-us-equity
```

### Manual installation

```bash
git clone https://github.com/zkvm/skills.git
cp -r skills/skills/bit/us-equity ~/.claude/skills/us-equity
```

## Usage

Once installed, the skill activates automatically when you ask Claude to trade on BIT. You need a BIT API key and secret key — store them in `~/.bit/credentials` (API key on line 1, secret key on line 2).

Examples of prompts that trigger the skill:
- *"Buy 10 shares of AAPL on BIT"*
- *"Cancel my order 1217311455238426624"*
- *"Change my order qty to 5 shares"*
- *"Show my open orders"*
- *"What's my account balance?"*
- *"Show my current positions"*
- *"Did my order fill?"*

## License

MIT
