---
description: Spawn parallel creative research agents for events from the previous /alpha scan
allowed-tools: Bash, Read, Write, Glob, Grep, Task
model: opus
---

# Creative Research Scanner

You are initiating creative/unconventional research on events that were previously researched by `/alpha`. Your job is to:
1. Identify all events that were researched in the previous `/alpha` run
2. Spawn parallel creative-researcher agents for each event
3. Report back with pointers to all generated research

## Phase 1: Identify Events from /alpha

Look at the conversation history to find the output from the previous `/alpha` command. The `/alpha` output lists all research files created, like:

```
research/events/KXCPI-25DEC/YYYY-MM-DD-HHMM-initial-research.md
research/events/KXFED-26JAN/YYYY-MM-DD-HHMM-initial-research.md
...
```

Extract all the event tickers from these file paths.

Alternatively, if the conversation doesn't have clear /alpha output, scan the research/events/ directory:

```bash
ls research/events/
```

For each event folder, that's an event to research.

**Guard: If no events found:**
```markdown
No research found in `research/events/`. Run `/alpha` first to scan for opportunities and generate initial research.
```
Exit the command if no events exist.

## Phase 2: Spawn Creative Research Agents

For each event identified, spawn a `creative-researcher` subagent using the Task tool.

**CRITICAL: Parallel Blocking Execution**

To run agents in parallel (concurrent) but blocking (main agent waits for all to complete):

1. **Make ALL Task calls in a SINGLE message** - This triggers parallel execution
2. **Do NOT use `run_in_background`** - Omitting this makes calls blocking
3. The main agent automatically waits for all agents to complete before continuing

For each agent:
- subagent_type: "creative-researcher"
- prompt: "Conduct creative/unconventional research on Kalshi event [EVENT_TICKER]. The event research folder is at research/events/[EVENT_TICKER]/. Read the existing initial research first, then explore non-obvious angles, unusual correlations, contrarian perspectives, and obscure data sources. Create a file named research/events/[EVENT_TICKER]/YYYY-MM-DD-HHMM-creative-research.md with your findings."
- model: opus

Example (spawn all agents in a single message):
```
In a single message, invoke:
- Task(subagent_type="creative-researcher", prompt="Conduct creative research on event KXCPI-25DEC...")
- Task(subagent_type="creative-researcher", prompt="Conduct creative research on event KXFED-26JAN...")
- Task(subagent_type="creative-researcher", prompt="Conduct creative research on event KXGDP-26JAN30...")
```

All agents run concurrently, and the main agent waits until ALL complete before proceeding.

## Phase 3: Report Results

Once all creative research is complete, provide a summary report:

```markdown
# Creative Research Complete

**Events Researched:** [X]
**Time:** [timestamp]

## Research Generated

| Event | Creative Research File |
|-------|------------------------|
| KXCPI-25DEC | research/events/KXCPI-25DEC/YYYY-MM-DD-HHMM-creative-research.md |
| KXFED-26JAN | research/events/KXFED-26JAN/YYYY-MM-DD-HHMM-creative-research.md |
...

## Next Steps

1. Run `/critique` to get senior analyst review (incorporates creative findings)
2. Run `/score` to rank opportunities
3. Run `/finalize` to get trade recommendations
```

**DO NOT** read or summarize the research contents. Just provide pointers to the files.

## Notes

- This command is designed to run AFTER `/alpha`
- Each event gets unconventional research that complements the initial research
- Creative research is about finding what others miss, not repeating mainstream analysis
- If no events found from /alpha, explain and suggest running /alpha first
