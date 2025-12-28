---
description: Spawn parallel senior analyst agents to critically review all research for events from previous scans
allowed-tools: Bash, Read, Write, Glob, Grep, Task
model: opus
---

# Critical Review Scanner

You are initiating critical review on events that were previously researched by `/alpha` and/or `/creative`. Your job is to:
1. Identify all events that have been researched
2. Spawn parallel senior-analyst agents for each event
3. Report back with pointers to all generated reviews

## Phase 1: Identify Events to Review

Look at the conversation history to find events from previous commands, OR scan the research directory:

```bash
ls research/events/
```

Each folder is an event ticker that should be reviewed.

For each event, verify there's research to review:
```bash
ls research/events/<EVENT_TICKER>/
```

Only include events that have at least one research file (initial-research or creative-research).

**Guard: If no events found:**
```markdown
No research found in `research/events/`. Run `/alpha` first to scan for opportunities and generate initial research.
```
Exit the command if no events exist.

## Phase 2: Spawn Senior Analyst Agents

For each event identified, spawn a senior analyst agent using the Task tool.

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
Conduct senior analyst review of Kalshi event [EVENT_TICKER].

**CRITICAL: Read ALL files in research/events/[EVENT_TICKER]/ completely. Do not skim. You need full context.**

Your job:
1. Read EVERY research file in the event folder completely
2. Reality check the fundamentals: resolution mechanics, base rates, what actually has to happen
3. Verify the bracket recommendation makes sense (the initial research recommends a specific bracket)
4. Identify fatal flaws, serious concerns, and minor issues
5. Determine if there's genuine alpha or if we're fooling ourselves
6. Provide final recommendation: TRADE [SPECIFIC_TICKER], PASS, or NEEDS MORE WORK

Create your review at: research/events/[EVENT_TICKER]/YYYY-MM-DD-HHMM-senior-review.md

When done, report: file path, files reviewed (confirm ALL read completely), final recommendation (TRADE [ticker]/PASS/NEEDS MORE WORK), one-sentence assessment.
```

Example (spawn all agents in a single message):
```
In a single message, invoke:
- Task(subagent_type="senior-analyst", prompt="Conduct senior analyst review of event KXCPI-25DEC...")
- Task(subagent_type="senior-analyst", prompt="Conduct senior analyst review of event KXFED-26JAN...")
- Task(subagent_type="senior-analyst", prompt="Conduct senior analyst review of event KXGDP-26JAN30...")
```

All agents run concurrently, and the main agent waits until ALL complete before proceeding.

## Phase 3: Report Results

Once all reviews are complete, provide a summary report:

```markdown
# Critical Review Complete

**Events Reviewed:** [X]
**Time:** [timestamp]

## Reviews Generated

| Event | Review File | Recommendation |
|-------|-------------|----------------|
| KXCPI-25DEC | research/events/KXCPI-25DEC/YYYY-MM-DD-HHMM-senior-review.md | TRADE KXCPI-25DEC-T0.3 YES |
| KXFED-26JAN | research/events/KXFED-26JAN/YYYY-MM-DD-HHMM-senior-review.md | PASS |
| KXGDP-26JAN30 | research/events/KXGDP-26JAN30/YYYY-MM-DD-HHMM-senior-review.md | NEEDS MORE WORK |
...

## Summary

- **TRADE recommendations:** [count] (with specific tickers)
- **PASS recommendations:** [count]
- **NEEDS MORE WORK:** [count]

## Next Steps

1. Run `/score` to rank the TRADE recommendations
2. Run `/finalize` to get final trade recommendations and execute
```

**DO NOT** read the full contents of the review files. Just report the file paths and the recommendation (which each agent will provide in its output).

## Notes

- This command can run after `/alpha`, after `/creative`, or after both
- The senior analyst reads ALL existing research for each event
- **TRADE recommendations must include a specific bracket ticker** (e.g., "TRADE KXCPI-25DEC-T0.3 YES")
- This is the critical checkpoint before trading
- The goal is to separate genuine opportunities from wishful thinking
