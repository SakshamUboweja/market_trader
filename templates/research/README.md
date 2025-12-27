# Research Catalog

This directory contains market research organized by ticker.

## Structure

```
research/
├── README.md                   # This file
└── markets/
    └── <TICKER>/               # One folder per market
        ├── YYYY-MM-DD-initial-research.md
        ├── YYYY-MM-DD-creative-research.md
        ├── YYYY-MM-DD-senior-review.md
        ├── score.txt           # Numeric score (0-100)
        └── thesis.md           # Position thesis (created when trade executed)
```

## File Types

### Initial Research
Created by `/kalshi-trader:alpha` via `market-researcher` agent.
- Market details and resolution rules
- Case for YES and NO
- Key uncertainties
- Data sources to monitor

### Creative Research
Created by `/kalshi-trader:creative` via `creative-researcher` agent.
- Unconventional angles
- Contrarian perspectives
- Obscure data sources
- Cross-domain correlations

### Senior Review
Created by `/kalshi-trader:critique` via `senior-analyst` agent.
- Critical assessment of research quality
- Identified flaws or gaps
- Final recommendation: TRADE / PASS / NEEDS MORE WORK

### Score
Created by `/kalshi-trader:score` via `judge` agent.
- Simple numeric score 0-100
- Used for ranking opportunities

### Thesis
Created by `/kalshi-trader:finalize` when a trade is executed.
- Position details (direction, size, entry price)
- Core thesis and edge explanation
- Exit criteria (take profit, stop loss)
- Monitoring plan

**The thesis is essential for `/kalshi-trader:positions` to properly manage positions.**

## Best Practices

1. **Don't delete research files** - They provide context for position management
2. **Keep thesis updated** - If your view changes, update the thesis
3. **Review before resolution** - Check if thesis held up for learning
