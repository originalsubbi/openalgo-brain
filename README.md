# OpenAlgo Brain — AI Auto-Trading Platform

[![License: AGPL v3](https://img.shields.io/badge/License-AGPL%20v3-blue.svg)](https://www.gnu.org/licenses/agpl-3.0)
[![Python 3.12+](https://img.shields.io/badge/Python-3.12+-blue.svg)](https://www.python.org/)
[![React 19](https://img.shields.io/badge/React-19-61dafb.svg)](https://react.dev/)
[![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-Live-228BE6.svg)](https://originalsubbi.github.io/openalgo-brain/)

OpenAlgo is a free, open-source, self-hosted **algorithmic trading platform**.  
The **Brain** extension adds AI-driven auto-trading: signal generation, auto-trade on/off, kill switch, risk limits, paper & live modes, and model learning — all running on your own server.

## 🧠 What is OpenAlgo?

OpenAlgo is not just a broker bridge — it's a complete self-hosted trading environment built on **Python Flask + React 19**. It combines four trading surfaces in one instance, sharing a single broker session and real-time market-data feed:

| Surface | Route | Purpose |
|---------|-------|---------|
| **Unified Broker API** | `/api/v1/` | Connect TradingView, Amibroker, Excel, Python, and 38+ brokers via REST + WebSocket |
| **Python Strategy Host** | `/python` | In-browser CodeMirror editor; schedule strategies on IST times with process isolation |
| **Flow: No-Code Builder** | `/flow` | Drag-and-drop node editor for strategies; webhook triggers for TradingView |
| **AI Agent** | `/agent` | LLM-powered chat that reads your market data, draws charts, and places orders with your approval |

## 🚀 The Brain — AI Auto-Trading

The Brain is an AI layer added to OpenAlgo that automates trading decisions:

- **Fusion Signal Engine**: Multiple strategy engines vote on each setup (LONG/SHORT), producing probability, expected R, and confidence scores
- **Auto-Trade Toggle**: Enable or disable automated execution with one action
- **Kill Switch**: Instantly blocks new entries and flattens open positions
- **Human-Owned Risk Limits**: Sliders for max drawdown, daily loss, positions, lots, exposure, cooldown — written to `.env` and applied live, no restart required
- **Paper & Live Modes**: Test risk-free in Analyzer Mode (₹1 Crore sandbox capital) or go live with real capital
- **Tunable Parameters**: Machine-owned knobs (stop/target multipliers, probability gates, IV regime, expiry selection) adjusted by a nightly self-tuner
- **PPA & Cheap-Expiry Sidecars**: Premium Price Action entries and low-premium option scalping, each with independent sliders and auto-tune

## 📦 Key Features

- **38+ Broker Plugins**: Zerodha, Upstox, Dhan, Fyers, AngelOne, Delta Exchange, Shoonya, and many more
- **Real-Time WebSocket Feed**: Unified ZMQ market-data bus with LTP, quotes, and depth
- **API Analyzer Mode**: Backtest with ₹1 Crore sandbox capital and real market data
- **Options & Portfolio Suite**: 20+ analytical tools (option chain, Greeks, OI tracker, Max Pain, Vol Surface, GEX, IV Smile, portfolio backtester, SIP backtester, sector rotation)
- **Enterprise Security**: Argon2 hashing, Fernet encryption, 2FA (TOTP), rate limiting, CSP/CORS/CSRF, IP bans, zero data collection
- **Telegram Bot & MCP**: Real-time alerts and natural-language trading via Claude Desktop, Cursor, Windsurf, or ChatGPT

## 🛠 Installation

### Requirements
- **RAM**: 2 GB (or 0.5 GB + 2 GB swap)
- **Disk**: 1 GB
- **CPU**: 1 vCPU
- **Python**: 3.12 or newer
- **Node.js**: 22+ (frontend development only)

### Quick Start

```bash
# Clone the repository
git clone https://github.com/marketcalls/openalgo.git
cd openalgo

# Install uv package manager
pip install uv

# Run the application
uv run app.py
```

Open `http://127.0.0.1:5000` in your browser, sign in, add your broker keys in **Broker Configuration**, then enable the Brain.

For detailed installation, see the [official documentation](https://docs.openalgo.in/installation-guidelines/getting-started).

## 🎯 Using the Brain

Once OpenAlgo is running, the Brain is accessible through its dedicated dashboard. Core capabilities:

| Action | Endpoint | Description |
|--------|----------|-------------|
| Status | `GET /brain/api/status` | Live status: enabled, auto-trade, kill switch, P&L, positions |
| Control | `POST /brain/api/control` | Enable, disable, auto-trade on/off, kill switch, scan, learn |
| Risk Limits | `POST /brain/api/limits` | Update human-owned risk sliders (drawdown, positions, lots, etc.) |
| Parameters | `POST /brain/api/params` | Update tunable engine parameters |
| Capital | `POST /brain/api/capital` | Set paper-trading base capital |

See [`blueprints/brain.py`](https://github.com/marketcalls/openalgo/blob/main/blueprints/brain.py) for the full API reference.

## 🏗 Architecture

- **Backend**: Flask 3, SQLAlchemy 2.0, Flask-SocketIO, ZeroMQ, Agno, LiteLLM
- **Frontend**: React 19, TypeScript, Vite, Tailwind CSS 4, shadcn/ui, TanStack Query, Zustand
- **Databases**: SQLite (main, logs, latency, health, sandbox, Brain), DuckDB (Historify market data)
- **Ports**: App `5000`, WebSocket proxy `8765`, ZeroMQ `5555`
- **Brain DB**: Separate SQLite file at `db/brain.db` for journaling signals, positions, models, and events

## ⚠️ Risk Disclaimer

> **This software is for educational purposes only.**  
> Do not risk money which you are afraid to lose.  
> **Use the software at your own risk.** The authors and all affiliates assume no responsibility for your trading results.

Always test your strategies in **Analyzer Mode** (paper trading) before deploying with real money. Past performance does not guarantee future results. Trading involves substantial risk of loss. Auto-trading can lose capital quickly — understand the risk limits and kill switch before enabling live execution.

## 📄 License

OpenAlgo is released under the **AGPL V3.0 License**. See [`License.md`](https://github.com/marketcalls/openalgo/blob/main/License.md) for details.

## 🔗 Links

- **Documentation**: [docs.openalgo.in](https://docs.openalgo.in)
- **GitHub Repository**: [marketcalls/openalgo](https://github.com/marketcalls/openalgo)
- **Discord**: [Join the community](https://www.openalgo.in/discord)
- **Twitter/X**: [@openalgoHQ](https://twitter.com/openalgoHQ)
- **YouTube**: [@openalgo](https://www.youtube.com/@openalgo)

---

*Built with by traders, for traders. Making algorithmic trading accessible to everyone.*
