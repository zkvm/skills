# Matrixport Skills

Agent skills for the [Matrixport](https://www.matrixport.com) platform. Compatible with Claude Code, GitHub Copilot, Cursor, and any tool that supports the [Agent Skills open standard](https://agentskills.io).

## Available Skills

| Skill | Description |
|-------|-------------|
| [matrixport/us-equity](skills/matrixport/us-equity/) | Trade US equities on Matrixport — place orders, query order status, check balance and positions |

## Installation

### Claude Code — via `npx skills add`

Install globally (available across all projects):
```bash
npx skills add matrixport/skills --skill skills/matrixport/us-equity -g
```

Install for a specific project only:
```bash
npx skills add matrixport/skills --skill skills/matrixport/us-equity
```

### OpenClaw — via `clawhub`

> Coming soon — the skill will be published to the [ClawHub registry](https://clawhub.ai) shortly.

```bash
clawhub install matrixport-us-equity
```

### Manual installation

```bash
git clone https://github.com/matrixport/skills.git
cp -r skills/matrixport/skills/skills/matrixport/us-equity ~/.claude/skills/us-equity
```

## Usage

Once installed, the skill activates automatically when you ask Claude to trade on Matrixport. You need a Matrixport API key and secret key — store them in `~/.matrixport/credentials` (API key on line 1, secret key on line 2).

Examples of prompts that trigger the skill:
- *"Buy 10 shares of AAPL on Matrixport"*
- *"What's my account balance?"*
- *"Show my current positions"*
- *"Did my order fill?"*

## License

MIT
