---
description: Spawn parallel senior analyst agents to critically review all research for markets from previous scans
allowed-tools: Bash, Read, Write, Glob, Grep, Task
model: opus
---

# Critical Review Scanner

You are initiating critical review on markets that were previously researched by `/alpha` and/or `/creative`. Your job is to:
1. Identify all markets that have been researched
2. Spawn parallel senior-analyst agents for each market
3. Report back with pointers to all generated reviews

## Phase 1: Identify Markets to Review

Look at the conversation history to find markets from previous commands, OR scan the research directory:

```bash
ls research/markets/
```

Each folder is a market ticker that should be reviewed.

For each market, verify there's research to review:
```bash
ls research/markets/<TICKER>/
```

Only include markets that have at least one research file (initial-research or creative-research).

**Guard: If no markets found:**
```markdown
No research found in `research/markets/`. Run `/alpha` first to scan for opportunities and generate initial research.
```
Exit the command if no markets exist.

## Phase 2: Spawn Senior Analyst Agents

For each market identified, spawn a senior analyst agent using the Task tool.

**CRITICAL: Parallel Blocking Execution**

To run agents in parallel (concurrent) but blocking (main agent waits for all to complete):

1. **Make ALL Task calls in a SINGLE message** - This triggers parallel execution
2. **Do NOT use `run_in_background`** - Omitting this makes calls blocking
3. The main agent automatically waits for all agents to complete before continuing

For each agent:
- subagent_type: "senior-analyst"
- model: opus
- prompt: (see below)

Example prompt for each agent:

```
Conduct senior analyst review of Kalshi market [TICKER].

**CRITICAL: Read ALL files in research/markets/[TICKER]/ completely. Do not skim. You need full context.**

Your job:
1. Read EVERY research file in the market folder completely
2. Reality check the fundamentals: resolution mechanics, base rates, what actually has to happen
3. Identify fatal flaws, serious concerns, and minor issues
4. Determine if there's genuine alpha or if we're fooling ourselves
5. Provide final recommendation: TRADE, PASS, or NEEDS MORE WORK

Create your review at: research/markets/[TICKER]/YYYY-MM-DD-HHMM-senior-review.md

When done, report: file path, files reviewed (confirm ALL read completely), final recommendation (TRADE/PASS/NEEDS MORE WORK), one-sentence assessment.
```

Example (spawn all agents in a single message):
```
In a single message, invoke:
- Task(subagent_type="senior-analyst", prompt="Conduct senior analyst review of TICKER1...")
- Task(subagent_type="senior-analyst", prompt="Conduct senior analyst review of TICKER2...")
- Task(subagent_type="senior-analyst", prompt="Conduct senior analyst review of TICKER3...")
```

All agents run concurrently, and the main agent waits until ALL complete before proceeding.

## Phase 3: Report Results

Once all reviews are complete, provide a summary report:

```markdown
# Critical Review Complete

**Markets Reviewed:** [X]
**Time:** [timestamp]

## Reviews Generated

| Ticker | Review File | Recommendation |
|--------|-------------|----------------|
| TICKER1 | research/markets/TICKER1/YYYY-MM-DD-HHMM-senior-review.md | [TRADE/PASS/NEEDS MORE WORK] |
| TICKER2 | research/markets/TICKER2/YYYY-MM-DD-HHMM-senior-review.md | [TRADE/PASS/NEEDS MORE WORK] |
...

## Summary

- **TRADE recommendations:** [count]
- **PASS recommendations:** [count]
- **NEEDS MORE WORK:** [count]

## Next Steps

Review the senior analyst files, especially any TRADE recommendations, before making final trading decisions.
```

**DO NOT** read the full contents of the review files. Just report the file paths and the recommendation (which each agent will provide in its output).

## Notes

- This command can run after `/alpha`, after `/creative`, or after both
- The senior analyst reads ALL existing research for each market
- This is the critical checkpoint before trading
- The goal is to separate genuine opportunities from wishful thinking
