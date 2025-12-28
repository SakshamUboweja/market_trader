# Research Catalog

This directory contains research organized at two levels:
- **Events** (`events/`) - Research on underlying events (e.g., "December CPI release")
- **Markets** (`markets/`) - Position thesis files for specific tickers traded

## Structure

```
research/
├── README.md                   # This file
├── events/                     # EVENT-level research
│   └── <EVENT_TICKER>/         # One folder per event (e.g., KXCPI-25DEC)
│       ├── YYYY-MM-DD-HHMM-initial-research.md
│       ├── YYYY-MM-DD-HHMM-creative-research.md
│       ├── YYYY-MM-DD-HHMM-senior-review.md
│       └── score.txt           # Score with recommended bracket
│
└── markets/                    # TICKER-level position management
    └── <TICKER>/               # One folder per traded position
        └── thesis.md           # Position thesis (references event research)
```

## Event vs Market

**Events** are the underlying questions (e.g., "What will December CPI be?").
**Markets** are specific bracket contracts (e.g., "CPI > 0.3%").

One event has many markets (brackets). Research is done at the EVENT level to avoid duplication.

## File Types

### Initial Research (Event Level)
Created by `/alpha` via `event-researcher` agent.
- Event details and resolution rules
- Bracket analysis with prices
- **Recommended trade** (specific bracket + direction)
- Key uncertainties
- Data sources to monitor

### Creative Research (Event Level)
Created by `/creative` via `creative-researcher` agent.
- Unconventional angles
- Contrarian perspectives
- Obscure data sources
- Cross-domain correlations

### Senior Review (Event Level)
Created by `/critique` via `senior-analyst` agent.
- Critical assessment of research quality
- Bracket recommendation review
- Final recommendation: TRADE [TICKER] / PASS / NEEDS MORE WORK

### Score (Event Level)
Created by `/score` via `judge` agent.
- Format: `SCORE|TICKER|RATIONALE`
- Used for ranking opportunities
- Includes specific bracket to trade

### Thesis (Market Level)
Created by `/finalize` when a trade is executed.
- **Research Reference** - Links to event research folder
- Position details (direction, size, entry price)
- Core thesis and edge explanation
- Exit criteria (take profit, stop loss)
- Monitoring plan

**The thesis is essential for `/positions` to properly manage positions.**

## Workflow

1. `/alpha` → Creates `events/<EVENT>/initial-research.md`
2. `/creative` → Adds `events/<EVENT>/creative-research.md`
3. `/critique` → Adds `events/<EVENT>/senior-review.md`
4. `/score` → Adds `events/<EVENT>/score.txt`
5. `/finalize` → Creates `markets/<TICKER>/thesis.md` with event reference
6. `/positions` → Reads thesis, follows event reference, evaluates position

## Best Practices

1. **Don't delete research files** - They provide context for position management
2. **Keep thesis updated** - If your view changes, update the thesis
3. **Review before resolution** - Check if thesis held up for learning
4. **One event, many brackets** - Research the event once, trade specific brackets
