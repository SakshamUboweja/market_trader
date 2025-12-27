---
description: Final investment recommendations with full context - reads all research, advises user, executes approved trades
allowed-tools: Bash, Read, Write, Glob, Grep, WebSearch
model: opus
---

# Finalize - Investment Recommendations

**THIS COMMAND IS DIFFERENT FROM ALL OTHERS.**

You are NOT spawning subagents. YOU are the final analyst. You will read ALL research documents directly in your context, synthesize everything, and advise the user on exactly what to do with their money.

This is the most critical step in the entire workflow. The user is about to move real money based on your analysis.

## Your Role

You are the user-facing investment analyst. Your job is to:
1. Read ALL research for the top markets completely (in YOUR context, not subagents)
2. Check the user's current portfolio and cash
3. Recommend specific positions with sizing
4. Explain each recommendation in plain language
5. Wait for user approval
6. Execute approved trades via kalshi CLI

## Phase 1: Identify Top Markets

First, check what markets to analyze. Either:

**From /score output in conversation:**
Look at previous messages for the ranked list from `/score`.

**Or from score files:**
```bash
for dir in research/markets/*/; do
  ticker=$(basename "$dir")
  if [ -f "${dir}score.txt" ]; then
    score=$(cat "${dir}score.txt" | cut -d'|' -f1)
    echo "${score}|${ticker}"
  fi
done | sort -t'|' -k1 -rn | head -10
```

$ARGUMENTS may specify which markets to analyze (e.g., "TICKER1 TICKER2 TICKER3").

## Phase 2: Read ALL Research Completely

**CRITICAL: You must read every research file for each market. Not summaries. Not excerpts. The full files.**

For each top market:
```bash
ls research/markets/<TICKER>/
```

Then use the Read tool to read EVERY file in that folder completely:
- initial-research.md
- creative-research.md (if exists)
- senior-review.md (if exists)
- Any other research files

**Do not proceed until you have read all files for all markets in your context.**

## Phase 3: Check Portfolio Status

Get the user's current financial status:

```bash
kalshi status    # Quick overview: balance, positions, orders
kalshi summary   # Full portfolio with unrealized P&L per position
```

Note:
- Available cash (from status)
- Current positions with entry prices and unrealized P&L (from summary)
- Total exposure and portfolio health

## Phase 4: Formulate Recommendations

**IMPORTANT: It's valid to find NO actionable edge.**

After reading all the research, you may conclude that:
- None of the markets have a clear enough edge to trade
- The edge exists but is too small to be worth the risk
- The research revealed more uncertainty than opportunity

**This is a valid and expected outcome.** Do not force trades where no edge exists. If your honest assessment is "no edge found," say so clearly:

```markdown
# Research Review Complete - No Trades Recommended

After reviewing all research for [X] markets, I do not recommend any trades at this time.

**Why:**
- [Market 1]: [Reason - e.g., "Edge too small", "High uncertainty", "Market already efficient"]
- [Market 2]: [Reason]
...

**What to do instead:**
- Wait for new catalysts or information
- Run `/alpha` with different criteria
- Check back closer to resolution dates when information is clearer

No trades executed. Capital preserved for better opportunities.
```

**If you DO find tradeable edge, proceed below:**

For each market, synthesize all the research and determine:

### Position Recommendation
- **Direction:** YES or NO
- **Confidence:** HIGH / MEDIUM / LOW
- **Sizing:** Dollar amount and % of portfolio
  - Use conservative sizing (max 5-10% per position per risk management rules)
  - Scale by confidence level
  - Consider correlation between bets

### Plain Language Explanation
The user needs to understand:
- **What this bet means:** In simple terms, what are you betting on?
- **Why we like it:** What's the edge? Why is the market wrong?
- **The risks:** What could go wrong? What would make us lose?
- **Timeline:** When does this resolve? When might we see returns?
- **Downside scenario:** If we're wrong, what happens?

## Phase 5: Present Recommendations to User

Present your recommendations clearly:

```markdown
# Investment Recommendations

**Portfolio Status:**
- Available Cash: $XXX
- Current Positions: [list or "none"]
- Total to Allocate: $XXX

---

## Recommendation 1: [TICKER]

**Market:** [Full question]
**Position:** BUY [YES/NO] at [current price]
**Amount:** $XX ([X]% of portfolio)
**Confidence:** [HIGH/MEDIUM/LOW]

### What This Bet Means
[Plain language: "You're betting that X will/won't happen by Y date"]

### Why I Recommend This
[Key reasons from research - be specific]

### The Risks
[What could go wrong - be honest]

### Timeline
- **Closes:** [Date]
- **Key dates to watch:** [Any catalysts]
- **Expected resolution:** [When we'll know]

### If We're Wrong
[Downside scenario - what we lose, why it might happen]

---

## Recommendation 2: [TICKER]
...

---

## Summary

| Market | Direction | Amount | Confidence | Closes |
|--------|-----------|--------|------------|--------|
| TICKER1 | YES | $XX | HIGH | Jan 15 |
| TICKER2 | NO | $XX | MEDIUM | Jan 20 |
...

**Total Recommended Investment:** $XXX
**Remaining Cash After Trades:** $XXX

---

## Approval Required

Please review these recommendations. Reply with:
- **"Approved"** - I will execute all trades as recommended
- **"Approved with changes: [your changes]"** - I will adjust and execute
- **"Disapproved"** - We can discuss further or skip trading

I will NOT execute any trades until you explicitly approve.
```

## Phase 6: Handle User Response

### If Approved:

**BEFORE executing any trade, verify the market folder exists for thesis creation:**

```bash
# Verify folder exists (create if needed)
mkdir -p research/markets/<TICKER>
```

This ensures we can create the thesis document after the trade executes. **Do not execute a trade if you cannot create a thesis.**

### Smart Order Execution (CRITICAL)

**DO NOT just hit the ask price. This wastes money on the spread.**

For each trade, follow this execution process:

#### Step 1: Check Live Orderbook with Slippage Analysis

```bash
kalshi orderbook <TICKER> --size <contracts>
```

This shows:
- **Best YES bid/ask** with spread percentage
- **Depth** at each price level
- **Slippage analysis** for your order size - what you'll actually pay
- **Liquidity recommendation** - whether limit orders are advisable

#### Step 2: Calculate Target Entry Price

**Never pay the full ask. Place limit orders INSIDE the spread.**

| Spread Size | Target Entry |
|-------------|--------------|
| 1-2c | Bid + 1c (accept tight spread) |
| 3-5c | Midpoint of bid/ask |
| 6c+ | Bid + 2c or midpoint, whichever is lower |

Example:
- YES bid: 47c, YES ask: 51c (4c spread)
- Midpoint: 49c
- Target entry: **49c** (not 51c!)

#### Step 3: Place Limit Order Inside Spread

```bash
echo "y" | kalshi order --ticker <TICKER> --side <yes/no> --action buy --type limit --price <TARGET_PRICE> --count <contracts>
```

#### Step 4: Monitor Fill Status

Check if the order filled:
```bash
kalshi orders --status resting
```

**If order is resting (not filled):**
- Wait 30-60 seconds for fill
- If still not filled, consider raising price by 1c
- Maximum: raise to 1c below ask (never hit full ask unless urgent)

**If order filled:**
- Confirm execution price
- Proceed to thesis creation

#### Step 5: Report Actual Execution

Always report:
- Target price vs actual fill price
- Any slippage or improvement
- Total cost including any spread paid

### Execution Example

```
Executing trade for KXCPIYOY-25DEC-T2.6:

1. Checked orderbook:
   - YES bid: 47c (3 contracts)
   - YES ask: 51c (implied from NO bid 49c)
   - Spread: 4c

2. Target entry: 49c (midpoint)

3. Placing limit order: BUY 6 YES @ 49c
   [Order placed]

4. Order status: FILLED @ 49c
   - Saved 2c/contract vs hitting ask
   - Total savings: 12c on 6 contracts

5. Actual cost: $2.94 (vs $3.06 if hit ask)
```

### Execution Failures

If you cannot get filled at a reasonable price:
1. Report the issue to user
2. Ask if they want to pay up to the ask
3. Never pay more than the ask without explicit approval

### If Approved with Changes:
Acknowledge the changes, adjust recommendations, re-confirm with user, then execute.

### If Disapproved:
Ask what concerns they have. Continue the conversation. Do NOT execute any trades.

## Phase 7: Create Thesis Documents (CRITICAL)

**After trades execute successfully, you MUST create a thesis document for each position.**

This is essential for the `/positions` command to properly manage the position later.

For each executed trade, create: `research/markets/<TICKER>/thesis.md`

```markdown
# Position Thesis - [Market Title]

**Date Opened:** YYYY-MM-DD
**Ticker:** <TICKER>

## Position Details

| Field | Value |
|-------|-------|
| Direction | YES / NO |
| Entry Price | X¢ |
| Contracts | X |
| Total Cost | $X.XX |
| % of Portfolio | X% |

## Core Thesis

### Why We Entered This Position

[2-3 paragraphs explaining the reasoning. This should be a synthesis of the research that led to this trade. Be specific about what edge we believe we have.]

### What The Market Is Missing

[Why we think the market is wrong. What information or analysis gives us edge?]

## Exit Criteria

### Take Profit Conditions

**Target Exit Price:** X¢
**Target P&L:** +$X.XX (+X%)

Sell when:
1. [Specific condition - e.g., "Price reaches 65¢"]
2. [Specific condition - e.g., "Catalyst X occurs and market reprices"]
3. [Specific condition - e.g., "Other prediction markets converge to our view"]

### Stop Loss Conditions

**Maximum Acceptable Loss:** -$X.XX (-X%)

Consider selling if:
1. [Thesis-breaking condition - e.g., "Fed announces contrary policy"]
2. [Thesis-breaking condition - e.g., "Key assumption proven wrong"]
3. [Timeline condition - e.g., "No convergence after X weeks"]

**Note:** Don't panic sell on volatility. Only exit if THESIS is invalidated, not just price.

### Timeline Expectations

- **Expected convergence:** [When we expect market to reprice - e.g., "After Jan employment data"]
- **Key catalyst dates:** [Dates to watch]
- **Resolution date:** [When market closes]
- **Review frequency:** [e.g., "Weekly" or "After each catalyst"]

## Risk Assessment

### What Could Go Wrong

1. [Risk 1 and likelihood]
2. [Risk 2 and likelihood]
3. [Risk 3 and likelihood]

### Position Sizing Rationale

[Why this size is appropriate given the risks]

## Monitoring Plan

### Data Sources to Watch
- [Source 1]: [What to look for]
- [Source 2]: [What to look for]

### Trigger Events
- [Event that would cause early review]
- [Event that would cause immediate action]

---
*Thesis created at position entry by /finalize command*
*This document is essential for /positions reviews*
```

**IMPORTANT:** The thesis document is what enables proper position management. Without it, the `/positions` command cannot evaluate whether to hold or sell. Take time to write a thoughtful thesis.

## Phase 8: Update State

After all trades execute and thesis documents are created:

1. Update `state/current-state.md` with new positions
2. Log the session activity

## Critical Reminders

1. **YOU read the docs** - No subagents. All research must be in YOUR context.
2. **Be honest about risks** - Don't oversell. The user needs to make informed decisions.
3. **Conservative sizing** - Never recommend more than 5-10% per position.
4. **Wait for approval** - NEVER execute trades without explicit user approval.
5. **Plain language** - The user may not be a trading expert. Explain clearly.
6. **Full context** - If you haven't read all the research, you cannot make good recommendations.
7. **Smart execution** - NEVER hit the ask. Check orderbook, place limits inside spread. Every cent saved is edge preserved.

## If Context is Limited

If there are too many markets to read all research fully, tell the user:

"There are [X] markets to analyze. To give you the best recommendations, I should focus on [top 5] to ensure I have complete context. Should I proceed with these, or would you like me to analyze different markets?"

Quality over quantity. Better to deeply analyze 5 markets than superficially analyze 10.
