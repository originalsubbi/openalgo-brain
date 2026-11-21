# OpenAlgo Brain

**An AI auto-trading layer on top of [OpenAlgo](https://github.com/marketcalls/openalgo) — self-hosted, paper-first, with risk limits you own.**

[![License: AGPL v3](https://img.shields.io/badge/License-AGPL%20v3-blue.svg)](License.md)
[![Python 3.12+](https://img.shields.io/badge/Python-3.12+-blue.svg)](https://www.python.org/)
[![React 19](https://img.shields.io/badge/React-19-61dafb.svg)](https://react.dev/)
[![OpenAlgo](https://img.shields.io/badge/OpenAlgo-v2.0.3.5-purple.svg)](https://github.com/marketcalls/openalgo)

OpenAlgo is a free, open-source, self-hosted algorithmic trading platform: one Flask + React instance, one broker session, one real-time market-data feed. **Brain** adds a decision layer on top of it. Several strategy engines vote on each setup, a risk gate decides whether the trade is allowed, and an executor places it, in paper mode first and live only after the paper record earns it.

> **Everything runs on your own machine.** No SaaS component, no data leaves your server, and a fresh copy ships with no credentials, databases or trade history.

---

## Contents

- [What's in the box](#whats-in-the-box)
- [How the Brain works](#how-the-brain-works)
- [Quick start](#quick-start)
- [First run checklist](#first-run-checklist)
- [Configuration](#configuration)
- [Brain API](#brain-api)
- [Project layout](#project-layout)
- [Testing](#testing)
- [Documentation](#documentation)
- [Risk disclaimer](#risk-disclaimer)
- [License and credits](#license-and-credits)

---

## What's in the box

### The Brain

| Capability | What it does |
|---|---|
| **Fusion signal engine** | Many engines vote LONG / SHORT per underlying (trend, ORB, VWAP, volatility breakout, OI/PCR, chart patterns, statistical, ML, swing, positional, option selling). The fusion layer turns the votes into a probability, an expected R and a confidence score. |
| **Auto-trade switch** | Turn automated entries on or off with one action. |
| **Kill switch** | Instantly blocks new entries and flattens open positions. Always visible in the dashboard header. |
| **Human-owned risk limits** | Sliders for max drawdown, daily loss, open positions, lots, exposure, consecutive-loss halt, cooldown and entry window. Changes apply live, with no restart. |
| **Machine-owned tune knobs** | Stop and target multipliers, probability gates, IV regime and expiry selection, adjusted by a self-tuner and visible (and overridable) in the Tuning tab. |
| **PPA and Cheap-Expiry sidecars** | Premium Price Action entries and low-premium option scalping. Each has its own sliders, auto-tune, candidate journal and backtest. |
| **Paper to live graduation** | Live trading is off by default. It is judged on the paper record using a lower confidence bound on expectancy (minimum trades, profit factor, drawdown), not on a lucky streak. |
| **Learning loop** | Signals are labelled after they mature, calibration is tracked, and models are retrained and promoted by the learner. |

### The OpenAlgo platform underneath

| Surface | Route | Purpose |
|---|---|---|
| Unified Broker API | `/api/v1/` | REST + WebSocket for TradingView, Amibroker, Excel, Python and 38+ brokers |
| **AI Brain** | `/brain` | Dashboard: Overview, Trading, Tuning, History, Settings |
| Python Strategy Host | `/python` | In-browser editor; scripts scheduled on IST times, run as isolated subprocesses |
| Flow | `/flow` | No-code node-graph strategy builder with webhook triggers |
| AI Agent | `/agent` | LLM chat that reads your market data and proposes orders for your approval |
| Strategy Module and RMS | `/strategy` | Multi-leg options strategies with risk management |
| Options and Portfolio Suite | `/tools` | Option chain, Greeks, OI tracker, Max Pain, vol surface, GEX, IV smile, backtesters, sector rotation and more |
| Charting and Scalping Terminals | `/trading`, `/scalping` | Chart trading and keyboard-driven options scalping |

Also included: the API Analyzer / Sandbox (₹1 Crore paper capital with real market data), Historify market-data store (DuckDB), Telegram alerts, MCP servers for Claude Desktop / Cursor / Windsurf, 2FA, rate limiting and CSRF/CSP/CORS protection.

---

## How the Brain works

```
 market data ──► engines ──► fusion ──► gate ──► risk gate ──► executor ──► journal
 (Historify +    (votes)     (prob,     (liquidity,  (daily loss,   (paper via     (signals,
  broker feed)               exp. R,    IV, spreads, drawdown,      Analyzer, or    positions,
                             conf.)     data age)    cooldown)      live broker)    events)
                                                          ▲                            │
                                         human sliders ───┘      learner / self-tuner ◄─┘
```

1. **Scan.** On each cycle the orchestrator pulls bars and option-chain data for every enabled underlying.
2. **Vote and fuse.** Each engine casts a vote. Fusion scores the setup.
3. **Gate.** Hard checks: entry window, premium range, spread, open interest, data freshness, margin.
4. **Risk gate.** Your limits: daily loss, drawdown halt, consecutive-loss halt, position and lot caps, cooldown.
5. **Execute.** The executor plans entry, stop and target levels and places the order. Paper mode goes through the Analyzer sandbox, live mode through your broker.
6. **Learn.** Trades are journalled and labelled. The learner and self-tuner use the labelled record to adjust the machine-owned knobs. They never touch the human-owned limits.

**Safe by default.** `BRAIN_LIVE_ENABLED` is `false` out of the box, and BANKNIFTY is excluded by default (`BRAIN_EXCLUDED_UNDERLYINGS`). Start in paper mode and let it build a record before you consider going live.

---

## Quick start

### Requirements

| | |
|---|---|
| Python | 3.12 or newer |
| `uv` | Python package manager (`pip install uv`) |
| RAM / disk / CPU | 2 GB (or 0.5 GB + 2 GB swap) / 1 GB / 1 vCPU |
| Node.js | 22+, **only** if you want to change the frontend. The built UI is included. |

### Install and run

```bash
# 1. Unpack the archive (either format)
unzip openalgo_with_brain.zip          # or: tar -xzf openalgo_with_brain.tar.gz
cd openalgo

# 2. Install uv and start the app (dependencies install automatically)
pip install uv
uv run app.py
```

On Windows you can also use `start.bat` (`setup.bat` for first-time setup).

On the first start the app creates `.env` from `.sample.env`, generates unique `APP_KEY`, `API_KEY_PEPPER` and `FERNET_SALT` secrets, and creates the databases under `db/`.

Open **http://127.0.0.1:5000**.

### Docker

```bash
./docker-build.sh        # Windows: docker-build.bat
```

See [`docs/docker/README.md`](docs/docker/README.md) and [`DOCKER_README.md`](DOCKER_README.md).

---

## First run checklist

1. **Create the administrator account** at `http://127.0.0.1:5000` (this is a single-user system).
2. **Add your broker.** Select Broker → Configure, or put the keys in `.env`. See `.sample.env` for the variables. `VALID_BROKERS` lists the supported brokers.
3. **Log in to the broker** each trading day. Broker sessions expire daily.
4. **Switch on Analyzer Mode** so everything runs on paper at first.
5. **Backfill history.** Use Historify to download daily and intraday bars for the underlyings you want the Brain to trade. The learner and backtests need them.
6. **Open `/brain`.**
   - *Settings*: pick your underlyings.
   - *Tuning*: set the human-owned limits (daily loss, drawdown halt, max positions, lots, entry window).
   - *Overview*: turn the Brain on, then turn **Auto-trade** on.
7. **Let it paper trade.** Watch the History tab (trades, calibration, events). Live trading only unlocks after enough paper trades with positive expectancy.
8. **Going live is a deliberate step.** Set `BRAIN_LIVE_ENABLED=true`, and make sure your broker's static-IP whitelisting is set up if required for API orders.

---

## Configuration

Copy-and-edit settings live in `.env`. The ones that matter most for the Brain:

| Variable | Purpose | Default |
|---|---|---|
| `BRAIN_DATABASE_URL` | Brain's own SQLite database | `sqlite:///db/brain.db` |
| `BRAIN_CAPITAL` | Base capital for paper trading and sizing | see `services/brain/config.py` |
| `BRAIN_LIVE_ENABLED` | Allow live orders at all | `false` |
| `BRAIN_RISK_PER_TRADE_PCT` | Risk per trade | |
| `BRAIN_MAX_DAILY_LOSS_PCT` | Daily loss limit | |
| `BRAIN_DRAWDOWN_HALT_PCT` | Halt after this fraction of the equity peak is lost (`0.03` = 3%, `0` disables) | |
| `BRAIN_CONSECUTIVE_LOSS_HALT` | Halt after this many losing trades in a row (`0` disables) | |
| `BRAIN_MAX_OPEN_POSITIONS`, `BRAIN_MAX_LOTS`, `BRAIN_MAX_EXPOSURE_PCT` | Size caps | |
| `BRAIN_ENTRY_START`, `BRAIN_ENTRY_CUTOFF`, `BRAIN_SQUAREOFF` | Entry window and square-off time (IST) | |
| `BRAIN_EXCLUDED_UNDERLYINGS` | Underlyings the Brain must never trade | `BANKNIFTY` |
| `BRAIN_LIVE_MIN_TRADES`, `BRAIN_LIVE_MIN_EXPECTANCY_R`, `BRAIN_LIVE_MIN_PROFIT_FACTOR`, `BRAIN_LIVE_MAX_DRAWDOWN_R` | Paper-to-live graduation bar | 150 / 0.05 / 1.15 / 8.0 |

The full list, with defaults, is in [`services/brain/config.py`](services/brain/config.py). Human-owned limits changed from the Tuning tab are written back to `.env` and applied live; machine-owned tune knobs are stored in the Brain database.

**Ports:** app `5000`, WebSocket proxy `8765`, ZeroMQ `5555`.

**Never commit or share `.env`.** It holds your broker keys and the secrets that encrypt your broker tokens. Keep it together with `db/` when you move or restore an install.

---

## Brain API

All endpoints require a signed-in session.

| Method | Endpoint | Description |
|---|---|---|
| GET | `/brain/api/status` | Enabled, auto-trade, kill switch, P&L, positions |
| POST | `/brain/api/control` | Enable / disable, auto-trade, kill switch, scan, learn |
| POST | `/brain/api/limits` | Update human-owned risk limits |
| POST | `/brain/api/params` | Update tunable engine parameters |
| GET / POST | `/brain/api/tune-knobs` | Read or write machine-owned knobs |
| GET | `/brain/api/knob-specs` | Slider ranges, steps and defaults |
| POST | `/brain/api/capital` | Set paper-trading base capital |
| GET | `/brain/api/signals`, `/signals/<id>` | Signal journal and detail |
| GET | `/brain/api/positions`, `/positions/live` | Open and closed positions |
| POST | `/brain/api/positions/<id>/close` | Close one position |
| GET | `/brain/api/performance`, `/calibration`, `/trade_report` | Results and probability calibration |
| GET | `/brain/api/events` | Event log (filter by `level`, `kind`, `q`) |
| GET / POST / DELETE | `/brain/api/underlyings` | Manage the underlyings list |
| GET / POST | `/brain/api/ppa-config`, `/ppa-stats`, `/cheap-stats` | PPA and Cheap-Expiry sidecars |
| POST | `/brain/api/bootstrap` | Seed history for underlyings |

Full reference: [`blueprints/brain.py`](blueprints/brain.py).

---

## Project layout

```
app.py                    Flask application entry point
blueprints/brain.py       Brain REST API and dashboard route
services/brain/           Brain core
  engines/                  strategy engines that cast votes
  fusion.py, gate.py        signal fusion, entry gate
  risk_gate.py              human-owned limits
  executor.py               order planning and placement
  orchestrator.py           the scan loop
  learner.py, labeling.py   learning and outcome labelling
  premium_pa.py             PPA sidecar
  premiumCheapExp.py        Cheap-Expiry sidecar
  backtest.py, ppa_backtest.py, premium_backtest.py
database/brain_db.py      Brain persistence
frontend/                 React 19 + TypeScript + Vite UI (prebuilt in frontend/dist)
broker/                   38+ broker plugins
docs/                     All documentation (start at docs/INDEX.md)
test/                     Test suite
```

---

## Testing

```bash
uv run pytest test/test_brain_*.py
```

Useful extras:

```bash
uv run pytest test/test_brain_fresh_install.py          # clean-install behaviour
uv run python -m services.brain.ppa_backtest --days 30  # replay PPA on its candidate journal
uv run python scripts/ppa_candidate_report.py           # PPA candidate report
```

Frontend (only if you change it):

```bash
cd frontend && npm ci && npm run dev      # dev server
cd frontend && npm run build              # rebuild frontend/dist
```

---

## Documentation

- [`docs/INDEX.md`](docs/INDEX.md): map of all docs
- [`docs/ppa_candidates.md`](docs/ppa_candidates.md): PPA candidate journal
- [`docs/BRAIN_KNOB_WIRING.md`](docs/BRAIN_KNOB_WIRING.md): how each slider reaches the code
- [`docs/AI_BRAIN_CAPABILITY_ROADMAP.md`](docs/AI_BRAIN_CAPABILITY_ROADMAP.md): roadmap
- [`docs/CHANGELOG.md`](docs/CHANGELOG.md): what changed
- [`INSTALL.md`](INSTALL.md): installation entry points
- Upstream docs: [docs.openalgo.in](https://docs.openalgo.in)

---

## Risk disclaimer

> **This software is for educational purposes only. Do not risk money you are afraid to lose. Use it at your own risk.** The authors and all affiliates assume no responsibility for your trading results.

Auto-trading can lose capital quickly. Test in Analyzer Mode first, understand the risk limits and the kill switch before enabling live execution, and remember that past performance does not guarantee future results. Models learn from limited history and can be wrong.

---

## License and credits

Released under the **AGPL v3.0** license. See [`License.md`](License.md).

Brain is built on [OpenAlgo](https://github.com/marketcalls/openalgo) by marketcalls. Community: [Discord](https://www.openalgo.in/discord) · [X / Twitter](https://twitter.com/openalgoHQ) · [YouTube](https://www.youtube.com/@openalgo).
