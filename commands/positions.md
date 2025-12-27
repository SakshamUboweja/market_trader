---
description: Review all open positions with parallel subagents. Decides HOLD or SELL for each position.
allowed-tools: Bash, Read, Write, Glob, Grep, Task, WebSearch
model: opus
---

# Positions - Daily Position Review

This command reviews all open positions using parallel subagents. Each position is evaluated against its original thesis to determine whether to HOLD or SELL.

**Run this daily** to manage your portfolio actively.

## Phase 1: Get Current Positions with P&L

First, get portfolio summary with unrealized P&L:

```bash
kalshi summary
```

This shows all positions with:
- Entry price (calculated from fills)
- Current price
- Unrealized P&L (both $ and %)

If there are no positions, inform the user and exit:
> "No open positions to review. Run `/alpha` to find new opportunities or `/finalize` to execute pending trades."

## Phase 2: Verify Position Details

The `summary` command already provides most details. For additional context:

```bash
kalshi market <TICKER>  # Shows your position + current bid/ask
kalshi trades <TICKER> --summary  # Shows recent market activity
```

## Phase 3: Verify Thesis Exists

For each position, check if we have a thesis document:

```bash
ls research/markets/<TICKER>/thesis.md
```

**If thesis is missing:**
- Note this as a problem position
- We cannot properly review a position without knowing why we entered
- Flag for manual review

## Phase 4: Spawn Parallel Position Reviewers

For each position with a thesis, spawn a `position-reviewer` agent.

**CRITICAL: Parallel Blocking Execution**

To run agents in parallel (concurrent) but blocking (main agent waits for all to complete):

1. **Make ALL Task calls in a SINGLE message** - This triggers parallel execution
2. **Do NOT use `run_in_background`** - Omitting this makes calls blocking
3. The main agent automatically waits for all agents to complete before continuing

For each agent:
- subagent_type: "position-reviewer"
- model: opus
- prompt: (see below)

Example prompt for each agent:

```
Review position for <TICKER>.

Position Details:
- Direction: YES/NO
- Contracts: X
- Entry Price: X¢ (from fills)
- Current Price: X¢ (from market)
- Unrealized P&L: +/-$X.XX
- Days held: X

Research folder: research/markets/<TICKER>/
Thesis file: research/markets/<TICKER>/thesis.md

Determine whether to HOLD or SELL this position.
If SELL: document the outcome in learnings.
Return your decision with reasoning.
```

Example (spawn all position reviews in a single message):
```
In a single message, invoke:
- Task(subagent_type="position-reviewer", prompt="Review position for TICKER1...")
- Task(subagent_type="position-reviewer", prompt="Review position for TICKER2...")
- Task(subagent_type="position-reviewer", prompt="Review position for TICKER3...")
```

All agents run concurrently, and the main agent waits until ALL complete before proceeding.

## Phase 5: Collect Results

Once all position-reviewer agents complete (automatic with blocking execution), collect their results.

Each will return:
```
TICKER: <TICKER>
DECISION: HOLD | SELL
OUTCOME: [if SELL: WIN +$X.XX | LOSS -$X.XX | BREAK-EVEN]
REASON: [One sentence]
REVIEW FILE: [path]
LEARNINGS UPDATED: Yes/No
```

## Phase 6: Execute SELL Decisions

For each position with SELL decision, confirm with user before executing.

Present a summary:

```markdown
## Sell Recommendations

| Ticker | Direction | Contracts | Entry | Current | P&L | Reason |
|--------|-----------|-----------|-------|---------|-----|--------|
| XXX | YES | 10 | 40¢ | 55¢ | +$1.50 | Target reached |
| YYY | NO | 5 | 60¢ | 45¢ | -$0.75 | Thesis invalidated |

### Detailed Reasoning

**XXX - SELL (WIN +$1.50)**
[Reason from agent]

**YYY - SELL (LOSS -$0.75)**
[Reason from agent]

---

**Approve sells?** Reply:
- "Approved" - Execute all sells
- "Approved: XXX only" - Execute specific ticker(s)
- "Hold all" - Keep all positions, skip sells
```

### Execute Approved Sells

Use the `close` command to exit positions - it automatically determines side and quantity:

```bash
# Close entire position at market
kalshi close <TICKER> --force

# Or close partial position
kalshi close <TICKER> --qty <CONTRACTS> --force
```

The `close` command:
- Automatically detects whether you hold YES or NO
- Uses current bid price for market orders
- Shows entry price and P&L before executing

**For better prices:** Use limit orders inside the spread:
```bash
kalshi close <TICKER> --price <limit_price> --force
```

## Phase 7: Summary Report

After all decisions are processed, provide a summary:

```markdown
# Daily Position Review - YYYY-MM-DD

## Portfolio Status

| Metric | Value |
|--------|-------|
| Positions Reviewed | X |
| HOLD Decisions | X |
| SELL Decisions | X |
| Sells Executed | X |

## Outcomes

### Positions Held
| Ticker | Direction | P&L | Status | Next Review Trigger |
|--------|-----------|-----|--------|---------------------|
| ... |

### Positions Sold
| Ticker | Direction | Result | P&L | Lesson |
|--------|-----------|--------|-----|--------|
| ... |

## Capital Status

- **Previous Available:** $XX.XX
- **Capital Freed (from sells):** $XX.XX
- **Realized P&L Today:** +/-$XX.XX
- **New Available:** $XX.XX

## Wins & Losses Updated

[If any sells occurred, note that learnings files were updated]

## Positions Needing Attention

[Any positions without thesis files or other issues]

---

*Review completed. Run `/alpha` to find new opportunities for freed capital.*
```

## Edge Cases

### No Thesis File
If a position has no thesis file:
```
WARNING: Position in <TICKER> has no thesis file.
Cannot properly review. Options:
1. Create thesis retroactively: research/markets/<TICKER>/thesis.md
2. Sell immediately (flying blind)
3. Hold for manual review

Recommend creating thesis to enable proper position management.
```

### Stale Position (No Recent Research)
If last research is >30 days old, flag for attention.

### Position Doesn't Match Research
If we hold YES but research recommended NO (or vice versa), flag this discrepancy.

## Output Summary

At the end, confirm:
1. All positions reviewed
2. Sell decisions presented
3. Approved sells executed
4. Capital freed up (if any)
5. Learnings updated (if any sells)
6. Review files created for each position
