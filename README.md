# Kalshi BTC Bot

**Work in progress** — actively being developed and refined.

Automated trading bot for Kalshi BTC hourly price prediction markets. Uses a log-normal pricing model calibrated to live Binance volatility to find mispriced contracts, then executes trades when edge clears fee-adjusted EV filters.

---

## How It Works

1. **Pulls** live BTC spot price and volatility from Binance
2. **Scans** open Kalshi BTC hourly markets for contracts near the current price
3. **Models** the probability of BTC being above/below each strike at expiry using a log-normal distribution scaled to the contract's time to expiry
4. **Filters** signals through a multi-layer quality check (spread, depth, edge, EV)
5. **Executes** the top-ranked trades by fee-adjusted expected value
6. **Calibrates** edge shrinkage over time via live performance tracking

Two trade modes:
- **Scheduler mode** — scans on a fixed interval, trades highest-EV signals
- **Tape mode** — follows orderbook flow for faster reaction to price moves

---

## Architecture

```
kalshi-btc-bot/
├── main.py                  # Entry point
├── hft_latency_arb.py       # Latency arbitrage module
├── config/
│   └── settings.py          # All parameters via .env
└── src/
    ├── strategy.py          # Signal generation and multi-filter scan logic
    ├── pricing_model.py     # Log-normal BTC probability model
    ├── scheduler.py         # Main trading loop (scheduler mode)
    ├── tape_follower.py     # Orderbook tape following (tape mode)
    ├── order_manager.py     # Order placement and lifecycle
    ├── position_manager.py  # Open position tracking
    ├── risk_manager.py      # Pre-trade risk checks
    ├── edge_tracker.py      # Live edge shrinkage calibration
    ├── portfolio_tracker.py # P&L tracking and reporting
    ├── binance_client.py    # Binance spot price and vol feed
    ├── kalshi_client.py     # Kalshi REST API client
    └── logger_setup.py      # Structured logging
```

---

## Signal Filters

Every candidate contract passes through six sequential filters before generating a trade signal:

1. **Strike distance** — skips strikes more than N sigma from spot (scales with √time)
2. **Spread filter** — rejects wide bid/ask spreads (illiquid markets)
3. **Depth filter** — requires minimum orderbook depth on both sides
4. **Model prob bounds** — skips near-certain outcomes (2%–98%) to avoid fake edge
5. **Edge threshold** — minimum edge after shrinkage calibration
6. **Fee-adjusted EV** — final check; signal must clear minimum EV per contract net of maker fees on both entry and exit

Signals are ranked by EV per contract, and only the top N are traded per tick.

---

## Stack

- **Python 3.11+**
- **Kalshi API** — prediction market execution
- **Binance API** — BTC spot price and realized volatility
- Paper trading mode supported — full simulation without live orders

---

## Setup

```bash
cp .env.example .env
# Fill in Kalshi and Binance API credentials
python main.py
```

> Production mode requires explicit confirmation — bot warns and waits 5 seconds before placing live orders.
