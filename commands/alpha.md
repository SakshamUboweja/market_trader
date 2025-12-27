---
description: Scan for alpha opportunities and spawn parallel research agents
allowed-tools: Bash, Read, Write, Glob, Grep, Task, WebSearch
model: opus
---

# Alpha Opportunity Scanner

You are initiating an alpha opportunity scan on Kalshi markets. Your job is to:
1. Find markets matching the user's criteria
2. Identify potential opportunities (low filter)
3. Spawn parallel subagents to research each opportunity deeply
4. Report back with pointers to all generated research

## User Request

$ARGUMENTS

If no arguments provided, scan for markets closing in the next 1-4 weeks.

## Phase 1: Market Discovery

Use the kalshi CLI to find markets. Choose the appropriate approach based on user's request:

### By Keyword (if user specified a topic)
```bash
kalshi find "<keyword>"
```

### By Time Range (default: 1-4 weeks out)
```bash
# Calculate dates
kalshi markets --closes-after <today> --closes-before <4-weeks-from-now> --limit 100
```

### By Series (if user specified category)
```bash
kalshi series | grep -i <category>
kalshi markets --series <SERIES_TICKER>
```

### All Markets
```bash
kalshi markets --limit 100
```

## Phase 2: Initial Filtering

Review the markets found and identify candidates worth researching. Apply LOW filter criteria:

**Include markets that:**
- Have sufficient liquidity (volume > $100 or actively traded)
- Are not obviously efficient (not heavily traded by institutions)
- Have a clear resolution mechanism
- Have a realistic chance of alpha from research depth

**Exclude markets that:**
- Are purely sports/entertainment with no research edge
- Have resolution in less than 24 hours (no time for research to matter)
- Are clearly just coin flips with no predictive angle

For each candidate, note:
- Ticker
- Market question
- Current YES price
- Why it might have alpha potential

## Phase 3: Spawn Research Subagents

For each candidate market, spawn a subagent using the Task tool:

```
Use the Task tool with subagent_type "market-researcher" for each ticker.
The prompt should be: "Research Kalshi market [TICKER]: [Market Question]"
```

**CRITICAL: Parallel Blocking Execution**

To run agents in parallel (concurrent) but blocking (main agent waits for all to complete):

1. **Make ALL Task calls in a SINGLE message** - This triggers parallel execution
2. **Do NOT use `run_in_background`** - Omitting this makes calls blocking
3. The main agent automatically waits for all agents to complete before continuing

Example (spawn 5 agents in parallel, all blocking):
```
In a single message, invoke:
- Task(subagent_type="market-researcher", prompt="Research Kalshi market TICKER1: Market question 1")
- Task(subagent_type="market-researcher", prompt="Research Kalshi market TICKER2: Market question 2")
- Task(subagent_type="market-researcher", prompt="Research Kalshi market TICKER3: Market question 3")
- Task(subagent_type="market-researcher", prompt="Research Kalshi market TICKER4: Market question 4")
- Task(subagent_type="market-researcher", prompt="Research Kalshi market TICKER5: Market question 5")
```

All 5 agents run concurrently, and the main agent waits until ALL complete before proceeding to Phase 4.

## Phase 4: Report Results

Once all research is complete, provide a summary report:

```markdown
# Alpha Scan Complete

**Scan Parameters:** [What was searched for]
**Markets Found:** [X]
**Research Conducted:** [Y markets]
**Time:** [timestamp]

## Research Generated

| Ticker | Market | Research File |
|--------|--------|---------------|
| TICKER1 | [question] | research/markets/TICKER1/YYYY-MM-DD-HHMM-initial-research.md |
| TICKER2 | [question] | research/markets/TICKER2/YYYY-MM-DD-HHMM-initial-research.md |
...

## Next Steps

Review the research files above to identify trade opportunities.
Markets with strongest preliminary edge signals: [list if any stood out]
```

**DO NOT** read or summarize the research contents. Just provide pointers to the files.

## Notes

- If the scan finds 0 markets, explain why and suggest adjustments
- If the scan finds >20 markets, consider suggesting a more focused search
- Log any errors from subagents but continue with others
- The goal is breadth of coverage, not depth - subagents handle the depth
