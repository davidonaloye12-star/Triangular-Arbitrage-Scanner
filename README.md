# Triangular Arbitrage Scanner

A Python tool that watches live order books on several crypto exchanges and checks whether a three-step trade loop could make a profit after fees, order-book depth and delay. It runs in **paper-trading mode only**. It places no real orders.

## What is triangular arbitrage? (plain English)

Prices for the same coins are quoted in several pairs. Sometimes going the long way round gives you more than you started with:

1. Start with 1,000 USDT and buy BTC
2. Swap that BTC for ETH
3. Swap that ETH back to USDT

*Illustrative numbers:* if you end with 1,008 USDT, that is +0.8% before fees. With a 0.1% taker fee on each of three trades (about 0.3%), you keep roughly +0.5%. Most of the time the gap disappears once fees and real prices are counted. This project measures how often it doesn't.

## Why I built it

To test one question honestly: **do real, tradeable triangular opportunities exist on these exchanges after fees, depth and latency?** The scanner answers that with logged data before any live trading is considered.

## How it works

- Connects over **WebSocket** (not polling) to each exchange's public order-book feed
- Generates candidate triangles automatically from the exchange's own market list, ranked by liquidity (the lowest-volume leg is the bottleneck score)
- Walks real **order-book depth**, not just the top price, to estimate a realistic fill for a given trade size
- Applies **per-exchange taker fees** on all three legs
- On finding a profitable loop, simulates execution delay and **re-checks** whether it still holds
- Logs every opportunity to CSV with theoretical vs post-latency profit

## Running it

- Requires Python 3 and the `ccxt` library
- Uses public market data only, so no API keys are needed
- Install: `pip install ccxt` and run: `[command]`
- Runs 24/7 as a **systemd service** on a small Ubuntu VPS
- Exchanges covered: `[list]`

## Output

One CSV row per detected opportunity: `[columns, e.g. timestamp, exchange, triangle, trade size, theoretical profit %, post-latency profit %]`

## Problems hit and fixed

- **Rate limiting:** opening too many subscriptions at startup got connections rejected. Fixed by staggering subscriptions and capping tracked triangles per exchange.
- **Out-of-memory crashes:** the process was killed on a 512MB VPS with several exchanges connected. Fixed by moving to 1GB and confirming stable memory under load.
- **False positives from stale data:** added checks that reject opportunities where the three legs' prices weren't fresh or captured close together in time.
- **Fee accuracy:** fees are tracked per exchange rather than as one flat rate, which changes which opportunities are real.

## Status and limits

Running as a live paper-trading experiment. It has **not** shown profitable arbitrage and has not been reviewed for production use. Latency is simulated rather than measured from real order execution. Any move to live trading would need separate risk controls.
