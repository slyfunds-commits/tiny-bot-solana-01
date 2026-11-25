[![make.fun](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1.svg)](https://make.fun/bounty/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)
[![rank](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/slyfunds-commits/tiny-bot-solana-01/rank.svg)](https://make.fun/bounty/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)
[![reward](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/slyfunds-commits/tiny-bot-solana-01/reward.svg)](https://make.fun/bounty/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)

# Tiny Bot — selective dip-buying on Solana bars

A tiny paper-trading bot for hourly Solana DEX prices. It beats buy-and-hold
by being selective: it buys only a genuine washout and exits fast, sitting in
cash through the grinding down-moves.

## The engine (Pilot D, measured with the kit's own grader)

- **Entry:** RSI(14) below 30 **and** the close below the lower Bollinger
  band (20 typical prices, 2σ), on a symbol not held.
- **Exit, whole position, re-checked every bar:** RSI above 55, or the
  position worth +5% over its cost, or −4% under it.
- **Stake:** equity / 7 per slot, capped at 25% of equity and by cash on
  hand; sells first, then buys, ≤16 orders per bar, 100 bps slippage cap.
- **Trade-floor fallback:** if fewer than 8 trades have filled by bar 48,
  the entry relaxes (RSI below 40 under the lower band, or a momentum entry:
  12-bar return above +1% with the close above EMA(12)) so the 8-trade
  guardrail still clears on a trend window.
- **Deterministic:** no clock, no env, no unseeded random — pure functions
  of the bars and the account state. Each entry runs twice; the two action
  logs must match byte for byte.

## Why this engine

On the dev window (Oct 29 – Nov 3, 136 hourly bars × 7 assets, a broad down
market) every always-in strategy lost. Only hold-cash ($0) and this engine
(+$151.57, +1.52%, 27 trades, max drawdown 3.78%) finished non-negative —
beating buy-and-hold (−$869.34) by 10.2 points and every house baseline.
Faster, more-trading engines bled fees (fast EMA: −4.52% on 67 trades).

Measured with the kit's own grader (`grade-local --driver process`), this
session — an exact reproduction of the study's Pilot D numbers:

| Gate | Result |
|---|---|
| Net PnL (full dev) | +$151.57 (>$0 and > buy-and-hold −$869.34) |
| Trades | 27 (≥8; target ≥12) |
| Max drawdown | 3.78% (≤25%; target ≤10%) |
| Split-half | +$161.07 / −$9.50 vs buy-and-hold −$640.63 / −$228.71 |
| Last-72-bar proxy | +$2.13 on 14 trades vs buy-and-hold −$221.75 |
| Determinism | l1 reproduced (two executions byte-identical) |
| 2x fees | +$111.15 (still > buy-and-hold) |

The trade-floor fallback did not trigger on dev (12 trades had already
filled by bar 48); it stands as insurance for a trend final window.

## Files

- `main.ts` — the bot (TypeScript, runs as-is under Node 22).
- `vendor/makefun-bars.ts` — byte-identical copy of the kit's
  `agent-fun/bars-replay/v1` shim (stdin/stdout line protocol).
- `LICENSE` — MIT.

## Grade locally

```sh
node <kit>/cli/makefun.mjs grade-local --kit <kit> --bot . --driver process --json
```

## Comp

[Tiny Bot, Easy $2: Beat Buy-and-Hold on Solana](https://make.fun/bounty/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1) — $2, top 3, free entry. Places by net PnL on the final 3-day window (Nov 27–30); a prize needs PnL above buy-and-hold and above the best house baseline, with ≥8 trades and max drawdown ≤25%.
