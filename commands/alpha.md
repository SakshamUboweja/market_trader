---
description: Scan for alpha opportunities and spawn parallel research agents
allowed-tools: Bash, Read, Write, Glob, Grep, Task, WebSearch
model: opus
---

# Alpha Opportunity Scanner

You are initiating an alpha opportunity scan on Kalshi markets. Your job is to:
1. Find markets in target series (where we have research edge)
2. Deduplicate to unique EVENTS (not individual brackets)
3. Spawn parallel subagents to research each EVENT deeply
4. Report back with pointers to all generated research

## User Request

$ARGUMENTS

If no arguments provided, scan economics series by default.

## Phase 1: Market Discovery

### Default: Economics Series (High Edge)

If no arguments or `$ARGUMENTS` is empty, scan these series:

```bash
# Get all markets from our target economics series
for series in KXCPI KXFED KXFEDDECISION KXGDP KXPCE KXPCECORE KXPAYROLLS KXINFL KXGOVSHUT; do
  kalshi markets --series $series --limit 100 --json 2>/dev/null
done | python3 -c "
import json, sys

all_markets = []
for line in sys.stdin:
    line = line.strip()
    if line:
        try:
            data = json.loads(line)
            markets = data.get('markets', data) if isinstance(data, dict) else data
            all_markets.extend(markets)
        except:
            pass

# Deduplicate to events
events = {}
for m in all_markets:
    evt = m.get('event_ticker', m['ticker'])
    if evt not in events:
        events[evt] = {
            'event_ticker': evt,
            'title': m.get('title', '')[:60],
            'close_time': m.get('close_time', ''),
            'brackets': []
        }
    events[evt]['brackets'].append({
        'ticker': m['ticker'],
        'subtitle': m.get('yes_sub_title', m.get('subtitle', ''))[:30],
        'yes_bid': m.get('yes_bid', 0),
        'volume': m.get('volume', 0)
    })

print(f'Found {len(all_markets)} markets across {len(events)} unique events')
print()
for evt, info in sorted(events.items(), key=lambda x: x[1]['close_time'])[:30]:
    print(f\"{evt}|{info['title']}|{len(info['brackets'])} brackets\")
    for b in info['brackets'][:3]:
        print(f\"  - {b['ticker']}: {b['subtitle']} @ {b['yes_bid']}c (vol: {b['volume']})\")
    if len(info['brackets']) > 3:
        print(f\"  ... and {len(info['brackets'])-3} more brackets\")
"
```

### By Keyword

If user specifies a keyword like `/alpha fed` or `/alpha cpi`:

```bash
kalshi find "<keyword>" --limit 100 --json
```

Then deduplicate to events using the same logic.

### By Category Alias

Support these aliases:
- `/alpha econ` or `/alpha economics` → Economics series
- `/alpha politics` → KXGOVSHUT, KXSCOTUS, KXTARIFF, KXIMPEACH series
- `/alpha weather` → KXTEMP, KXRAIN, KXSNOW, KXHURRICANE series
- `/alpha all` → All markets (old behavior, not recommended)

## Phase 2: Initial Filtering

Review the events found and identify candidates worth researching. Apply LOW filter criteria:

**Include events that:**
- Have sufficient liquidity (total volume > $1000 across brackets)
- Are not obviously efficient (not heavily traded by institutions)
- Have a clear resolution mechanism
- Have a realistic chance of alpha from research depth
- Close within reasonable timeframe (2-8 weeks preferred)

**Exclude events that:**
- Are purely sports/entertainment with no research edge
- Have resolution in less than 48 hours (no time for research)
- Are clearly just coin flips with no predictive angle
- Already have research (check `research/events/<EVENT>/` folder exists)

**Idempotency check:**
```bash
# Skip events that already have research
ls research/events/ 2>/dev/null | while read evt; do
  echo "SKIP: $evt (research exists)"
done
```

For each candidate event, note:
- Event ticker
- Event title
- Number of brackets
- Current prices across brackets
- Why it might have alpha potential

## Phase 3: Spawn Event Research Subagents

For each candidate event, spawn a subagent using the Task tool:

```
Use the Task tool with subagent_type "event-researcher" for each event.
```

**CRITICAL: Parallel Blocking Execution**

To run agents in parallel (concurrent) but blocking (main agent waits for all to complete):

1. **Make ALL Task calls in a SINGLE message** - This triggers parallel execution
2. **Do NOT use `run_in_background`** - Omitting this makes calls blocking
3. The main agent automatically waits for all agents to complete before continuing

**Prompt format for each event:**

```
Research Kalshi event [EVENT_TICKER]: [Event Title]

Brackets:
- [TICKER1] | [Subtitle] | YES: [X]¢ | Vol: [X]
- [TICKER2] | [Subtitle] | YES: [X]¢ | Vol: [X]
- [TICKER3] | [Subtitle] | YES: [X]¢ | Vol: [X]
...

Research the underlying event, analyze all brackets, and recommend the optimal trade.
```

Example (spawn 5 agents in parallel, all blocking):
```
In a single message, invoke:
- Task(subagent_type="event-researcher", prompt="Research Kalshi event KXCPI-25DEC: CPI for December 2025\n\nBrackets:\n- KXCPI-25DEC-T0.4 | Above 0.4% | YES: 5¢ | Vol: 1K\n- KXCPI-25DEC-T0.3 | Above 0.3% | YES: 15¢ | Vol: 40K\n...")
- Task(subagent_type="event-researcher", prompt="Research Kalshi event KXFED-26JAN: Fed Rate Decision January 2026\n\nBrackets:\n...")
...
```

All agents run concurrently, and the main agent waits until ALL complete before proceeding to Phase 4.

## Phase 4: Report Results

Once all research is complete, provide a summary report:

```markdown
# Alpha Scan Complete

**Scan Parameters:** [What was searched for - e.g., "Economics series (default)"]
**Markets Found:** [X] across [Y] unique events
**Events Researched:** [Z] (skipped [N] with existing research)
**Time:** [timestamp]

## Research Generated

| Event | Title | Brackets | Research File | Recommended Trade |
|-------|-------|----------|---------------|-------------------|
| KXCPI-25DEC | CPI Dec 2025 | 6 | research/events/KXCPI-25DEC/YYYY-MM-DD-HHMM-initial-research.md | KXCPI-25DEC-T0.3 YES +12% edge |
| KXFED-26JAN | Fed Jan 2026 | 8 | research/events/KXFED-26JAN/YYYY-MM-DD-HHMM-initial-research.md | KXFED-26JAN-T5.0 NO +8% edge |
...

## Events Skipped

[List any events that were skipped and why - e.g., "already researched", "too illiquid"]

## Next Steps

1. Run `/creative` to explore unconventional angles on these events
2. Run `/critique` to get senior analyst review
3. Run `/score` to rank opportunities
4. Run `/finalize` to get trade recommendations

Markets with strongest preliminary edge signals: [list top 2-3 from agent outputs]
```

**DO NOT** read or summarize the full research contents. Just provide:
- File paths
- Recommended trade (ticker + direction + edge estimate) from each agent's output

## Notes

- **Event-level, not market-level** - Each event is researched once, covering all its brackets
- **Recommended trade is specific** - Always a specific ticker, not just "this event has edge"
- If the scan finds 0 events, explain why and suggest adjustments
- If the scan finds >15 events, consider suggesting a more focused search
- Log any errors from subagents but continue with others
- The goal is breadth of coverage at event level, depth within each event
