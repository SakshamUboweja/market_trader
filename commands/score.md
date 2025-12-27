---
description: Score all researched markets and return top opportunities ranked by quality
allowed-tools: Bash, Read, Write, Glob, Grep, Task
model: opus
---

# Market Scoring & Ranking

You are scoring all researched markets to identify the top opportunities. Your job is to:
1. Spawn parallel judge agents to score each market
2. Collect all scores via bash
3. Rank and return the top 10 opportunities
4. Advise the user to take top markets to a fresh session

## Phase 1: Identify Markets to Score

List all markets with research:

```bash
ls research/markets/
```

Each folder is a market ticker that needs scoring.

**Guard: If no markets found:**
```markdown
No research found in `research/markets/`. Run `/alpha` first to scan for opportunities and generate initial research.
```
Exit the command if no markets exist.

## Phase 2: Spawn Judge Agents

For each market, spawn a judge agent using the Task tool.

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
Score Kalshi market [TICKER].

**CRITICAL: Read ALL files in research/markets/[TICKER]/ completely. No skimming.**

Your job:
1. Read every research file for this market completely
2. Score the opportunity from 0-100 based on:
   - Edge Quality (0-40): How clear and realistic is the edge?
   - Research Quality (0-30): How thorough and credible?
   - Actionability (0-30): Is this tradeable? Did senior analyst recommend TRADE?

3. Write score to: research/markets/[TICKER]/score.txt

**FORMAT IS CRITICAL:**
<SCORE>|<ONE_LINE_RATIONALE>

Example: `87|Strong edge on Fed decision, solid research, senior analyst recommends TRADE`

When done, confirm: files read, score assigned, score file path.
```

Example (spawn all agents in a single message):
```
In a single message, invoke:
- Task(subagent_type="judge", prompt="Score Kalshi market TICKER1...")
- Task(subagent_type="judge", prompt="Score Kalshi market TICKER2...")
- Task(subagent_type="judge", prompt="Score Kalshi market TICKER3...")
```

All agents run concurrently, and the main agent waits until ALL complete before proceeding.

## Phase 3: Collect and Rank Scores

Once all judges are done, run this bash script to collect and rank:

```bash
echo "=== MARKET RANKINGS ===" && \
errors="" && \
for dir in research/markets/*/; do
  ticker=$(basename "$dir")
  if [ -f "${dir}score.txt" ]; then
    score_line=$(cat "${dir}score.txt")
    score=$(echo "$score_line" | cut -d'|' -f1)
    # Validate score is a number 0-100
    if ! [[ "$score" =~ ^[0-9]+$ ]] || [ "$score" -lt 0 ] || [ "$score" -gt 100 ]; then
      errors="${errors}INVALID: ${ticker} - malformed score '${score}'\n"
      continue
    fi
    rationale=$(echo "$score_line" | cut -d'|' -f2-)
    if [ -z "$rationale" ]; then
      errors="${errors}WARNING: ${ticker} - missing rationale\n"
    fi
    echo "${score}|${ticker}|${rationale}"
  else
    errors="${errors}MISSING: ${ticker} - no score.txt file\n"
  fi
done | sort -t'|' -k1 -rn
# Report any errors
if [ -n "$errors" ]; then
  echo ""
  echo "=== SCORING ERRORS ==="
  echo -e "$errors"
fi
```

This outputs all markets sorted by score (highest first), plus reports any validation errors.

## Phase 4: Report Top Opportunities

Parse the sorted output and present the top 10:

```markdown
# Market Rankings Complete

**Total Markets Scored:** [X]
**Time:** [timestamp]

## Top 10 Opportunities

| Rank | Score | Ticker | Rationale |
|------|-------|--------|-----------|
| 1 | 87 | TICKER1 | Strong edge on Fed decision... |
| 2 | 82 | TICKER2 | Good creative research found... |
| 3 | 78 | TICKER3 | ... |
...

## Research Locations

For the top 10, here are the full research folders:

1. **TICKER1** (Score: 87): `research/markets/TICKER1/`
2. **TICKER2** (Score: 82): `research/markets/TICKER2/`
...

## Next Steps

**IMPORTANT:** These top opportunities should be reviewed in a fresh session with full context.

To proceed:
1. Clear this session (the research is saved to disk)
2. Start a new session
3. Ask Claude to review these specific markets:

```
Review the top trading opportunities:
- TICKER1 (research/markets/TICKER1/)
- TICKER2 (research/markets/TICKER2/)
- TICKER3 (research/markets/TICKER3/)
...

Read all research files for each and provide final trade recommendations.
```

This ensures each market gets full attention without context limitations.

## Full Rankings

[Include full sorted list for reference]
```

## Notes

- Score files are simple `SCORE|RATIONALE` format for easy bash parsing
- Judge agents read ALL research (initial, creative, senior review) before scoring
- Top 10 is a guideline - user can adjust based on their risk appetite
- Fresh session recommendation is critical - don't try to deeply analyze 10 markets in this context
