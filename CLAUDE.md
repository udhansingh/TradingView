# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository contains **TradingView Pine Script v6** indicators/strategies for stock and crypto analysis. There is no build system, test runner, or package manager — all code runs directly on TradingView's platform.

## Files

- **WirezardStrategyTester.pine** (~895 lines) — Multi-indicator scoring strategy that synthesizes RSI, Impulse MACD, VIDYA, Volume, Market Depth, and Sector Strength into composite BUY/SELL signals (score range -100 to +100). Supports dual-mode: overlay indicator or backtestable strategy. Includes webhook/alert JSON output.
- **WirezardWaveRider.pine** (~808 lines) — Elliott Wave-inspired overlay indicator using multi-timeframe (MTF) analysis, VIDYA, ADX, RSI divergence, and pivot detection. Generates tiered BUY/SELL signals (Strong, Moderate, Correction, Exhaustion, Divergence, Wave Top).

## Development Workflow

1. Edit `.pine` files locally
2. Copy/paste into TradingView's Pine Editor to test
3. No CLI build, lint, or test commands exist

## Architecture Notes

### WirezardStrategyTester
Sections flow sequentially:
1. **Inputs** — Grouped by indicator (RSI, MACD, VIDYA, Volume, Market Depth, Sector)
2. **Auto Sector ETF Detection** — Maps ~300 US tickers to SPDR sector ETFs; falls back to SPY with 50% weight
3. **Adaptive Timeframe Scaling** — Auto-adjusts indicator lengths based on chart timeframe (Intraday/Daily/Weekly)
4. **Indicator Calculations** — Six independent sub-indicators (3A–3F), each producing a score component
5. **Composite Scoring Engine** — Weighted sum with core alignment check (RSI+MACD+VIDYA must agree)
6. **Webhook Alerts** — Dual-path: `alert()` fires JSON payload directly; `strategy.entry/close` carries `alert_message` for order-fill events
7. **Dashboard Table** — Real-time scoring breakdown overlay
8. **Strategy Entries/Exits** — Conditional on `i_strategyMode` toggle

### WirezardWaveRider
- **Input tiers**: Simple (7 params) → Intermediate (7) → Advanced (31) for progressive complexity
- **MTF Composite**: Scores from 3 timeframes (2x, 4x, 8x chart TF) weighted and summed (range ±30)
- **Signal hierarchy**: Strong BUY/SELL (tier 1) → Correction BUY / Exhaustion SELL (tier 2) → Divergence / Wave Top (tier 3-4)
- **Tolerance bands**: ±% price zone drawn after signals to account for pivot confirmation lag

## Pine Script Conventions Used

- `strategy()` declaration with `process_orders_on_close=true` and `calc_on_every_tick=true`
- `barstate.isconfirmed` gating for anti-repaint when `i_useConfirmedBars` is enabled
- `request.security()` for MTF data and sector ETF relative strength
- `var` keyword for persistent state across bars
- Input groups (`group=`) for organized Settings panel
