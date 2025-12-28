---
description: Score all researched events and return top opportunities ranked by quality
allowed-tools: Bash, Read, Write, Glob, Grep, Task
model: opus
---

# Event Scoring & Ranking

You are scoring all researched events to identify the top opportunities. Your job is to:
1. Spawn parallel judge agents to score each event
2. Collect all scores via bash
3. Rank and return the top 10 opportunities
4. Advise the user to take top events to a fresh session for /finalize

## Phase 1: Identify Events to Score

List all events with research:

```bash
ls research/events/
```

Each folder is an event ticker that needs scoring.

**Guard: If no events found:**
```markdown
No research found in `research/events/`. Run `/alpha` first to scan for opportunities and generate initial research.
```
Exit the command if no events exist.

## Phase 2: Spawn Judge Agents

For each event, spawn a judge agent using the Task tool.

**CRITICAL: Parallel Blocking Execution**

To run agents in parallel (concurrent) but blocking (main agent waits for all to complete):

1. **Make ALL Task calls in a SINGLE message** - This triggers parallel execution
2. **Do NOT use `run_in_background`** - Omitting this makes calls blocking
3. The main agent automatically waits for all agents to complete before continuing

For each agent:
- subagent_type: "judge"
- model: opus
- prompt: (see below)

Example prompt for each agent:

```
Score Kalshi event [EVENT_TICKER].

**CRITICAL: Read ALL files in research/events/[EVENT_TICKER]/ completely. No skimming.**

Your job:
1. Read every research file for this event completely
2. Score the opportunity from 0-100 based on:
   - Edge Quality (0-40): How clear and realistic is the edge on the recommended bracket?
   - Research Quality (0-30): How thorough and credible?
   - Actionability (0-30): Is this tradeable? Did senior analyst recommend TRADE with a specific bracket?

3. Write score to: research/events/[EVENT_TICKER]/score.txt

**FORMAT IS CRITICAL:**
<SCORE>|<RECOMMENDED_TICKER>|<ONE_LINE_RATIONALE>

Example: `87|KXCPI-25DEC-T0.3|Strong edge on CPI, solid research, senior analyst recommends TRADE YES`

When done, confirm: files read, score assigned, score file path.
```

Example (spawn all agents in a single message):
```
In a single message, invoke:
- Task(subagent_type="judge", prompt="Score Kalshi event KXCPI-25DEC...")
- Task(subagent_type="judge", prompt="Score Kalshi event KXFED-26JAN...")
- Task(subagent_type="judge", prompt="Score Kalshi event KXGDP-26JAN30...")
```

All agents run concurrently, and the main agent waits until ALL complete before proceeding.

## Phase 3: Collect and Rank Scores

Once all judges are done, run this bash script to collect and rank:

```bash
echo "=== EVENT RANKINGS ===" && \
errors="" && \
for dir in research/events/*/; do
  event=$(basename "$dir")
  if [ -f "${dir}score.txt" ]; then
    score_line=$(cat "${dir}score.txt")
    score=$(echo "$score_line" | cut -d'|' -f1)
    # Validate score is a number 0-100
    if ! [[ "$score" =~ ^[0-9]+$ ]] || [ "$score" -lt 0 ] || [ "$score" -gt 100 ]; then
      errors="${errors}INVALID: ${event} - malformed score '${score}'\n"
      continue
    fi
    ticker=$(echo "$score_line" | cut -d'|' -f2)
    rationale=$(echo "$score_line" | cut -d'|' -f3-)
    if [ -z "$rationale" ]; then
      errors="${errors}WARNING: ${event} - missing rationale\n"
    fi
    echo "${score}|${event}|${ticker}|${rationale}"
  else
    errors="${errors}MISSING: ${event} - no score.txt file\n"
  fi
done | sort -t'|' -k1 -rn
# Report any errors
if [ -n "$errors" ]; then
  echo ""
  echo "=== SCORING ERRORS ==="
  echo -e "$errors"
fi
```

This outputs all events sorted by score (highest first), including the recommended ticker for each, plus reports any validation errors.

## Phase 4: Report Top Opportunities

Parse the sorted output and present the top 10:

```markdown
# Event Rankings Complete

**Total Events Scored:** [X]
**Time:** [timestamp]

## Top 10 Opportunities

| Rank | Score | Event | Recommended Ticker | Rationale |
|------|-------|-------|-------------------|-----------|
| 1 | 87 | KXCPI-25DEC | KXCPI-25DEC-T0.3 | Strong edge on CPI, recommends YES |
| 2 | 82 | KXFED-26JAN | KXFED-26JAN-T5.0 | Good research, recommends NO |
| 3 | 78 | KXGDP-26JAN30 | KXGDP-26JAN30-T2.5 | ... |
...

## Research Locations

For the top 10, here are the full research folders:

1. **KXCPI-25DEC** (Score: 87): `research/events/KXCPI-25DEC/`
   - Recommended trade: KXCPI-25DEC-T0.3 YES
2. **KXFED-26JAN** (Score: 82): `research/events/KXFED-26JAN/`
   - Recommended trade: KXFED-26JAN-T5.0 NO
...

## Next Steps

**Run `/finalize` to get final trade recommendations and execute.**

```
/finalize
```

Or specify top events explicitly:
```
/finalize KXCPI-25DEC KXFED-26JAN
```

This reads all research and presents specific trade recommendations with sizing.

## Full Rankings

[Include full sorted list for reference]
```

## Notes

- Score files are `SCORE|TICKER|RATIONALE` format for easy bash parsing
- Judge agents read ALL research (initial, creative, senior review) before scoring
- **Each score includes the specific ticker to trade** (not just the event)
- Top 10 is a guideline - user can adjust based on their risk appetite
- Run `/finalize` next to execute trades
