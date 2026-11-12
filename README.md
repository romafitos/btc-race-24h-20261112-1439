# cbBTC 1-minute trading bot — btc-race-24h-20261112-1439

Long-only bot for Coinbase Wrapped BTC (cbBTC/USDC) on 1-minute bars.
Standard library only, deterministic: every decision is a pure function
of the bars received (no network, no clock, no environment reads).

## Strategy

Two independent mechanisms, full size, at most one position:

1. **Trend** — buy all cash when EMA(10) > EMA(60) and the trailing
   30-bar return exceeds +0.30%. Exit on a 0.70% trailing stop from the
   peak close, or when the close falls under EMA(90). After a trend
   exit, wait 240 bars before a new trend entry so chop cannot whipsaw
   the bot.
2. **Dip** — buy all cash when the trailing 60-bar return is below
   −1.20% (a sharp flush) to catch the bounce. Exit at +0.60% from
   entry, at −0.70% from entry, or after 240 bars.

## Local grade (dev window, 1,440 1-min bars)

Measured with a faithful local replay of the kit's bars
(`work/replay.py`: fill at next bar's open, 4 bps fee, 2 bps slippage,
constant-product impact vs the bar's open depth, integer arithmetic):

- Net PnL: **+$7.96** on $1,000 start (gate: > $0) ✓
- Max drawdown: **0.73%** (race bar: ≤ 25%) ✓
- Trades: **4** (2 round trips: trend buy bar 54 → sell bar 253,
  +$4.69; dip buy bar 513 → sell bar 553, +$3.27)
- Deterministic: two full runs produce byte-identical replies ✓

Parameter-neighbourhood check: all 17 one-parameter variations of the
chosen parameters stayed profitable (+$1.02 to +$10.31), so the result
is not a knife-edge fit.

## Layout

- `main.py` — the entry (Python 3.13, stdlib only)
- `vendor/makefun_bars.py` — byte-identical copy of the kit's Python
  bars-replay shim (`sdk/makefun_bars.py`)
- `work/` — kit download, offline backtests/sweeps, replay driver
  (not part of the entry)

## Note

The kit CLI (`grade-local --driver process`) cannot run a Python bot
on a machine without Docker ("the process sandbox runs node22 only"),
so the local grade above comes from the faithful replay driver instead
of the CLI. The entry should be confirmed with the CLI under the
Docker driver before submitting.
