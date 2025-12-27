# Kalshi Trader Plugin

A Claude Code plugin for AI-assisted research and trading on [Kalshi](https://kalshi.com) prediction markets.

## Philosophy

**We have an edge in research depth, not speed.** This plugin leverages AI's ability to:
- Deeply analyze primary sources (Fed minutes, legislation, economic data)
- Synthesize information across domains
- Identify mispricings in low-volume markets where fewer participants mean more opportunity

## Prerequisites

1. **Claude Code** installed and working
2. **kalshi-cli** installed and configured:
   ```bash
   pip install kalshi-cli
   ```
3. **Kalshi account** with API credentials configured (see kalshi-cli docs)

## Installation

### From GitHub

```bash
/plugin install github:austron24/kalshi-trader-plugin
```

### Local Development

```bash
claude --plugin-dir /path/to/kalshi-trader-plugin
```

## Commands

After installation, the following commands are available:

### `/kalshi-trader:alpha [criteria]`

Scan for alpha opportunities and spawn parallel research agents.

```bash
/kalshi-trader:alpha                    # Default: markets closing 1-4 weeks out
/kalshi-trader:alpha fed               # Markets related to "fed"
/kalshi-trader:alpha --closes-before 2025-02-01
```

**What it does:**
1. Scans markets using kalshi CLI
2. Filters for research-friendly opportunities
3. Spawns parallel `market-researcher` agents
4. Reports paths to all generated research files

### `/kalshi-trader:creative`

Run unconventional research on markets from previous `/alpha` scan.

```bash
/kalshi-trader:creative
```

**What it does:**
1. Identifies markets from previous `/alpha` in conversation
2. Spawns parallel `creative-researcher` agents
3. Explores non-obvious angles, contrarian views, obscure sources

### `/kalshi-trader:critique`

Senior analyst critical review of all research.

```bash
/kalshi-trader:critique
```

**What it does:**
1. Identifies all markets with research
2. Spawns parallel `senior-analyst` agents
3. Critically assesses research quality, identifies flawed reasoning
4. Returns TRADE / PASS / NEEDS MORE WORK recommendations

### `/kalshi-trader:score`

Score and rank all researched markets.

```bash
/kalshi-trader:score
```

**What it does:**
1. Spawns parallel `judge` agents to score each market 0-100
2. Ranks markets by opportunity quality
3. Returns top 10 opportunities

### `/kalshi-trader:finalize [tickers]`

Final investment recommendations with trade execution.

```bash
/kalshi-trader:finalize                 # Analyze top markets from /score
/kalshi-trader:finalize TICKER1 TICKER2 # Analyze specific markets
```

**What it does:**
1. Reads ALL research for specified markets (in main context)
2. Checks portfolio status via kalshi CLI
3. Recommends positions with sizing and plain language explanations
4. Waits for user approval
5. Executes approved trades
6. Creates thesis documents for position management

**Note:** This command does NOT spawn subagents. It reads everything directly for full context.

### `/kalshi-trader:positions`

Daily position review. **Run this daily.**

```bash
/kalshi-trader:positions
```

**What it does:**
1. Gets all open positions via kalshi CLI
2. Spawns parallel `position-reviewer` agents
3. Each reviews position against its thesis
4. Recommends HOLD or SELL for each
5. Presents sell recommendations for approval
6. Executes approved sells
7. Updates learnings with wins/losses

## Agents

The plugin includes these specialized agents:

| Agent | Purpose |
|-------|---------|
| `market-researcher` | Initial deep-dive research on market fundamentals |
| `creative-researcher` | Unconventional angles, contrarian perspectives |
| `senior-analyst` | Critical review, identifies flawed reasoning |
| `judge` | Scores markets 0-100 for ranking |
| `position-reviewer` | Reviews positions, recommends HOLD/SELL |

## Typical Workflow

```
1. /kalshi-trader:alpha              # Find opportunities
2. /kalshi-trader:creative           # Explore unconventional angles
3. /kalshi-trader:critique           # Critical review
4. /kalshi-trader:score              # Rank opportunities
5. /kalshi-trader:finalize           # Get recommendations, execute trades
6. /kalshi-trader:positions          # Daily: review and manage positions
```

## Directory Structure

The plugin creates and uses these directories in your project:

```
your-project/
├── research/
│   └── markets/
│       └── <TICKER>/
│           ├── YYYY-MM-DD-initial-research.md
│           ├── YYYY-MM-DD-creative-research.md
│           ├── YYYY-MM-DD-senior-review.md
│           ├── score.txt
│           └── thesis.md              # Created by /finalize
├── state/                             # Optional
│   ├── current-state.md
│   └── session-log.md
└── learnings/                         # Optional
    ├── strategies-that-worked.md
    └── strategies-that-failed.md
```

## Setup Templates

The `templates/` directory contains starter files for:
- Research folder structure
- State tracking
- Learnings documentation

Copy these to your project to get started:

```bash
cp -r /path/to/kalshi-trader-plugin/templates/* ./
```

## Risk Management

The plugin enforces conservative position sizing:
- Max 5-10% of portfolio per position
- Thesis documents required for position management
- Smart order execution (limit orders inside spread)
- Daily position review against thesis

## License

MIT License - see [LICENSE](LICENSE) for details.

## Disclaimer

This plugin is for informational and educational purposes only. Trading on prediction markets involves risk. Always do your own research and make informed decisions. The AI recommendations are not financial advice.
