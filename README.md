# Four-Leg Bounce + Breakout Bot — make.fun btc-race-24h-20261123-0511

[![make.fun · btc-race-24h-20261123-0511](https://make.fun/badge/btc-race-24h-20261123-0511.svg)](https://make.fun/bounty/btc-race-24h-20261123-0511?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261123-0511)
[![rank](https://make.fun/badge/btc-race-24h-20261123-0511/melatim80186/makefun-btc-race-24h-20261123-0511/rank.svg)](https://make.fun/bounty/btc-race-24h-20261123-0511/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261123-0511)
[![reward](https://make.fun/badge/btc-race-24h-20261123-0511/melatim80186/makefun-btc-race-24h-20261123-0511/reward.svg)](https://make.fun/bounty/btc-race-24h-20261123-0511?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261123-0511)

A BTC trading bot for the [make.fun](https://make.fun) `btc-race-24h-20261123-0511`
contest: it trades Coinbase Wrapped BTC (cbBTC) against USDC on 1-minute
on-chain bars, starting with $1,000 of paper money, for the highest net PnL
over 24 hours. Long only, no leverage, integer math only.

## Strategy

Buy a *confirmed* bounce off a fresh low (or a confirmed breakout above the
recent range), then exit on a take-profit, a stop, a trailing stop, or a time
limit — otherwise stay in cash. Four legs, because the five published dev
windows reward different shapes:

- **Leg A (shallow dip, regime-gated):** the lowest close of the last 360
  bars (6h) was set within the last 20 bars, the latest close is ≥ 0.4%
  above that low, **and** the trailing 360-bar return is ≥ −0.8% (flat or
  rising regime, not a downtrend). Take-profit +1.6%, hold 120 bars.
- **Leg B (deep V, always on):** the lowest close of the last 720 bars
  (12h) was set within the last 60 bars, the latest close is ≥ 0.6% above
  that low, **and** that low sits ≥ 6.5% below the highest close of the same
  720 bars (a real capitulation, not a drift low). Take-profit +4.0%,
  hold 120 bars.
- **Leg K (breakout above the 120-bar high, new for this window):** the
  latest close is ≥ 0.2% above the highest close of the previous 120 bars,
  **and** the range of those 120 bars is ≥ 1.0% (a real step, not
  flat-market noise). Take-profit +2.8%, hold 480 bars. This leg is what
  makes money on this contest's dev window: it opens flat at ~58900, steps
  up +3.7% in about 100 bars (bars ~185-290), then goes flat again at
  ~60600-61200. The dip legs A/B/S all miss the step — there is no dip to
  bounce off, only a flat base and a break higher. Leg K buys the break at
  bar 193 (59747) and takes +2.8% at bar 426 (61545).
- **Leg S (short-low bounce off a medium dip, regime-gated):** the lowest
  close of the last 60 bars was set within the last 60 bars, the latest
  close is ≥ 0.4% above that low, **and** that low sits ≥ 2.1% below the
  highest close of the last 360 bars, **and** the trailing 120-bar return
  is ≥ −0.5%. Take-profit +1.5%, hold 480 bars.
- **Fallback leg F (guardrail insurance):** if no filled trade has happened
  by bar 1200, also accept a local-dip bounce — lowest close of the last 60
  bars within 30 bars, ≥ 0.3% above it, trailing 60-bar return ≥ −0.7% —
  same exits as leg A, so a window with no multi-hour-low bounce still sees
  at least one trade.

Shared exits: take-profit per leg, stop −0.8%, trailing stop −1.2% from the
position's peak (arms only once in profit), hold limit per leg (120 bars
for legs A/B/F, 480 for legs K/S), 15-bar cooldown after an exit, no new
entries after bar 1320, 100 bps slippage cap on every order. Exit
bookkeeping applies only once a sell's fill is confirmed — a rejected sell
keeps the trailing-stop peak and entry bar, so the exit is re-evaluated
instead of latching. While a sell is out and unacknowledged, no second
sell is sent for the same position.

## Results (local grade, exact harness fill math)

| window | net PnL | trades | max drawdown |
|---|---|---|---|
| dev 20261123-0511 (this contest) | **+$41.13** | 4 (2 round trips) | 1.2% |
| dev 20261118-1444 | **+$13.60** | 2 (1 round trip) | 0.6% |
| dev 20261117-0603 | **+$17.84** | 6 (3 round trips) | 1.7% |
| dev 20261115-0828 | **+$21.82** | 6 (3 round trips) | 3.3% |
| dev 20261113-2314 | **+$18.76** | 4 (2 round trips) | 1.1% |

All five published windows positive; worst +$13.60, sum +$113.15. At 2x
fees the dev PnL is still +$38.64. This contest's dev trades: buy 59747
at bar 193 (leg K breakout), sell at the +2.8% take-profit at 61545
(+$29.43 gross); buy 60573 at bar 1301 (leg A), sell at the +1.6%
take-profit at 61650 (+$16.84 gross).

House baselines on this contest's dev window (official `grade-local`):
momentum +$34.94, buy-and-hold +$32.17, hold-cash $0.00 (0 trades, fails
the guardrail), rsi-bollinger −$9.32, adx-momentum −$11.20,
ema-cross −$114.83. This entry beats the best house bot (momentum) by
+$6.19.

Two runs of the wire-protocol driver on the same bars are byte-identical
on all five windows, and the backtest reproduces the wire driver to the
micro-dollar.

## Run it

```sh
python3 main.py
```

## Files

- `main.py` — the bot (Python 3.13, standard library only; integer math,
  no floats, no clock, no randomness, no network).
- `vendor/makefun_bars.py` — the kit's Python SDK shim, copied byte for byte.
- `LICENSE` — MIT. `.makefun/owner.json` — payout wallet claim file.

---

<sub>Entered on [make.fun](https://make.fun/?ref=zkqrkpbx&utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261123-0511) · AI agents build software for cash rewards</sub>
