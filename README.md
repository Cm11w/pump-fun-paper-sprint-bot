[![make.fun](https://make.fun/badge/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b.svg)](https://make.fun/bounty/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b)
[![rank](https://make.fun/badge/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b/Cm11w/pump-fun-paper-sprint-bot/rank.svg)](https://make.fun/bounty/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b)
[![reward](https://make.fun/badge/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b/Cm11w/pump-fun-paper-sprint-bot/reward.svg)](https://make.fun/bounty/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b)

# Pump.fun Paper Sprint — Young-Pumper Pullback Scalper

A deterministic paper-trading bot for the [Pump.fun Paper Sprint: Any Bot Can Win](https://make.fun/bounty/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b) bounty on make.fun. It trades 10 SOL of paper money on a replay of recorded Solana pump.fun bonding-curve activity. Nothing is traded for real: the house replay scorer runs this bot over 6 hours of recorded pump.fun activity and scores what it would have made. Highest net PnL wins the pot.

## Strategy: buy the pullback of a young proven pumper, bank the bounce, never hold the dump

Measured on the kit's own 6-hour dev dataset (1,222 coins) before writing this bot:

- Holding any coin for 600 slots loses money in every slice of the data (median forward return −2% to −30%): this market mean-reverts hard.
- But coins that just pumped bounce again first: after a >50% rise in 120 slots the median further high within 600 slots is +26%, and 18% double again.
- Young coins (<600 slots) bounce best; older coins barely bounce at all.

So this bot does one thing: buy the pullback of a young coin that already proved it can pump, then sell the whole position at +15%, cut it at −5%, or dump it after 300 slots if it never moves.

**Entry (max 5 open positions, max 0.10 SOL of cost per coin):**
- **PULLBACK** — coin aged ≤300 slots whose peak reached ≥2× its first-seen price, price now ≥15% off that peak, recovered ≥5% from the trough after the peak, with non-negative net buy flow and ≥2 buys in the last 60 slots, curve real SOL between 0.3 and 100.
- **Second buy only** — one add-on (up to 0.10 SOL total cost per coin) while the coin is still rising ≥10% over 120 slots on positive buy flow, confirming the bounce instead of rolling over.
- Every buy is capped at 1% of the curve's real SOL reserve (the harness cap) and at 0.05 SOL, so entries build over batches as the coin grows instead of one rejected oversized order. A coin that already exited is never re-entered.

**Exits (checked every batch):** take-profit +15% (sell all — the bounce is short), stop-loss −5% (sell all), trailing stop (arms at +25% peak, exits on a 10% fall from the peak), stale exit (held >300 slots and still under +10%).

**Guardrails:** the run must end with ≥20 coins traded in round trips of ≥0.01 SOL and max drawdown ≤50%. Offline simulation of this exact logic on the dev window trades ~150 coins with ~1% max drawdown. Positions stay tiny (≤0.5 SOL deployed at once), so a bad stretch cannot breach the drawdown guardrail the way a hold-and-hope bot does (the kit's pilot momentum bot loses −9.5 SOL with a 96% drawdown on the same data).

**Determinism:** no clock, no environment, no randomness; iteration only over insertion-ordered maps fed by the batch stream, so the two grading executions produce byte-identical action logs.

## Files

- `main.ts` — entrypoint: the whole strategy in one file (the grader runs `node /bot/main.ts`).
- `vendor/makefun-bot.ts` — the kit's `sdk/ts/makefun-bot.ts` shim, vendored byte for byte (it owns the `agent-fun/replay/v2` wire protocol).
- `LICENSE` — MIT.
