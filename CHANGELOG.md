# Changelog

All notable changes to this project will be documented in this file.

## [1.0.0] - 2025-12-27

### Added
- Initial public release
- **Commands:**
  - `/kalshi-trader:alpha` - Scan for alpha opportunities, spawn parallel research agents
  - `/kalshi-trader:creative` - Run creative/unconventional research on discovered markets
  - `/kalshi-trader:critique` - Senior analyst critical review of all research
  - `/kalshi-trader:score` - Score and rank all researched markets
  - `/kalshi-trader:finalize` - Final investment recommendations with trade execution
  - `/kalshi-trader:positions` - Daily position review with HOLD/SELL recommendations
- **Agents:**
  - `market-researcher` - Deep-dive initial research on market fundamentals
  - `creative-researcher` - Unconventional angles, contrarian views, obscure sources
  - `senior-analyst` - Critical review, identifies flawed reasoning
  - `judge` - Scores markets 0-100 for ranking
  - `position-reviewer` - Reviews positions against thesis, recommends HOLD/SELL
- Template files for setting up research, state, and learnings directories
