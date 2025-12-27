---
description: Spawn parallel creative research agents for markets from the previous /alpha scan
allowed-tools: Bash, Read, Write, Glob, Grep, Task
model: opus
---

# Creative Research Scanner

You are initiating creative/unconventional research on markets that were previously researched by `/alpha`. Your job is to:
1. Identify all markets that were researched in the previous `/alpha` run
2. Spawn parallel creative-researcher agents for each market
3. Report back with pointers to all generated research

## Phase 1: Identify Markets from /alpha

Look at the conversation history to find the output from the previous `/alpha` command. The `/alpha` output lists all research files created, like:

```
research/markets/TICKER1/YYYY-MM-DD-HHMM-initial-research.md
research/markets/TICKER2/YYYY-MM-DD-HHMM-initial-research.md
...
```

Extract all the tickers from these file paths.

Alternatively, if the conversation doesn't have clear /alpha output, scan the research/markets/ directory:

```bash
ls research/markets/
```

For each market folder, that's a ticker to research.

**Guard: If no markets found:**
```markdown
No research found in `research/markets/`. Run `/alpha` first to scan for opportunities and generate initial research.
```
Exit the command if no markets exist.

## Phase 2: Spawn Creative Research Agents

For each ticker identified, spawn a `creative-researcher` subagent using the Task tool.

**CRITICAL: Parallel Blocking Execution**

To run agents in parallel (concurrent) but blocking (main agent waits for all to complete):

1. **Make ALL Task calls in a SINGLE message** - This triggers parallel execution
2. **Do NOT use `run_in_background`** - Omitting this makes calls blocking
3. The main agent automatically waits for all agents to complete before continuing

For each agent:
- subagent_type: "creative-researcher"
- prompt: "Conduct creative/unconventional research on Kalshi market [TICKER]. The market research folder is at research/markets/[TICKER]/. Read the existing initial research first, then explore non-obvious angles, unusual correlations, contrarian perspectives, and obscure data sources. Create a file named research/markets/[TICKER]/YYYY-MM-DD-HHMM-creative-research.md with your findings."
- model: opus

Example (spawn all agents in a single message):
```
In a single message, invoke:
- Task(subagent_type="creative-researcher", prompt="Conduct creative research on TICKER1...")
- Task(subagent_type="creative-researcher", prompt="Conduct creative research on TICKER2...")
- Task(subagent_type="creative-researcher", prompt="Conduct creative research on TICKER3...")
```

All agents run concurrently, and the main agent waits until ALL complete before proceeding.

## Phase 3: Report Results

Once all creative research is complete, provide a summary report:

```markdown
# Creative Research Complete

**Markets Researched:** [X]
**Time:** [timestamp]

## Research Generated

| Ticker | Creative Research File |
|--------|------------------------|
| TICKER1 | research/markets/TICKER1/YYYY-MM-DD-HHMM-creative-research.md |
| TICKER2 | research/markets/TICKER2/YYYY-MM-DD-HHMM-creative-research.md |
...

## Next Steps

Review the creative research files alongside the initial research to identify non-obvious alpha opportunities.
```

**DO NOT** read or summarize the research contents. Just provide pointers to the files.

## Notes

- This command is designed to run AFTER `/alpha`
- Each market gets unconventional research that complements the initial research
- Creative research is about finding what others miss, not repeating mainstream analysis
- If no markets found from /alpha, explain and suggest running /alpha first
