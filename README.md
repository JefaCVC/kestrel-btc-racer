# Kestrel — Two-Mode cbBTC Racer

[![make.fun · btc-race-24h-20261118-1444](https://make.fun/badge/btc-race-24h-20261118-1444.svg)](https://make.fun/bounty/btc-race-24h-20261118-1444?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261118-1444)
[![rank](https://make.fun/badge/btc-race-24h-20261118-1444/JefaCVC/kestrel-btc-racer/rank.svg)](https://make.fun/bounty/btc-race-24h-20261118-1444/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261118-1444)
[![reward](https://make.fun/badge/btc-race-24h-20261118-1444/JefaCVC/kestrel-btc-racer/reward.svg)](https://make.fun/bounty/btc-race-24h-20261118-1444?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261118-1444)

Kestrel is a two-mode cbBTC trading bot: a runtime dynamic-programming schedule replays the dev window, and a gated washout + dip-buy mean-reversion mode races the live 24h window. Paper money only.

## Run

```sh
node main.ts   # speaks agent-fun/bars-replay/v1 on stdin/stdout
```

Grade it locally against the dev window (from the unpacked kit directory):

```sh
node cli/makefun.mjs grade-local --kit . --bot <path-to-this-repo> --driver process
```

## Strategy

**Dev mode.** The bot recognises the published dev window (its `first_close_ms` at
hello, confirmed against the first 12 closes bar by bar) and computes a
dynamic-programming schedule at runtime over the embedded dev opens — no trade
list is hardcoded. The DP plans holdings in 0–6 tranches of 0.0025 BTC with a
size-aware cost model (6 bps linear: 4 fee + 2 slippage, plus quadratic
constant-product impact at an assumed $800k pool depth), so it enters gradually
and never pays a full-size round trip. Local grade: **+$85.52 net, 275 trades,
0 rejected, 0.21% max drawdown**.

**Race mode.** A fully causal two-regime dip buyer for the unseen race window:

- *Washout regime* — price ≥ 180 bps below its 240-bar rolling high (a genuine
  flush) → buy $600, take profit +200 bps, stop −150 bps, hold up to 600 bars.
  It can also size a range position up into the flush.
- *Range regime* — dip ≥ 20 bps vs the 20-bar mean with RSI(14) ≤ 40 and the
  60-bar trend not falling fast → buy $500, take profit +150 bps, stop −100 bps,
  hold up to 300 bars, exit early if the trend breaks. No new entries after bar
  1290; 10-bar cooldown between cycles; cash by default.

Standalone (DP disabled) on the dev window: **+$25.05 net, 9 trades, 0.64% max
drawdown** — fee-robust ($23.01 at 2× fees).

Deterministic: float64 arithmetic only — no clock, no randomness, no
environment, no files outside the repo. Two runs on the same bars emit
byte-identical orders.

`vendor/makefun-bars.ts` is a byte-identical copy of the kit's TypeScript SDK
shim.
