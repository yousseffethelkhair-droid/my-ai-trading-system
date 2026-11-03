# AI CEO Trading System

A private, self-hosted AI-driven trading system with a web dashboard, natural-language commands, and real-time WebSocket updates.

---

## Quick Start

### 1. Prerequisites
- Python 3.11+
- pip

### 2. Clone / copy the project folder
```
my_ai_trading_system/
```

### 3. Install dependencies
```bash
cd my_ai_trading_system
pip install -r requirements.txt
```

### 4. Configure environment
```bash
cp .env.example .env
# Edit .env and fill in your keys:
#   GROQ_API_KEY       — required for AI commands (when LLM_PROVIDER=groq)
#   EXCHANGE_API_KEY   — your Bybit/Binance API key
#   EXCHANGE_API_SECRET
#   EXCHANGE_TESTNET=true   ← keep true until ready for live!
```

### 5. Run the server
```bash
uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

### 6. Open the dashboard
Navigate to: http://127.0.0.1:8000

---

## API Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/` | Web dashboard |
| GET | `/health` | System health check |
| GET | `/api/portfolio` | Exchange balance |
| GET | `/api/trades` | List all trades (add `?status=OPEN`) |
| GET | `/api/trades/{id}` | Single trade |
| POST | `/api/ai/command` | Natural-language command |
| POST | `/api/trades/{id}/edit` | Manual TP/SL override |
| POST | `/api/trades/{id}/close` | Instant market close |
| WS | `/ws` | Real-time price feed |

---

## AI Command Examples

Send a POST to `/api/ai/command` with body `{"prompt": "..."}`:

```
"Open a BTC LONG trade with 5% portfolio, TP at 4%, SL at 2%"
"Sell ETH with 2% of portfolio, take profit 3%, stop loss 1%"
"Close trade 7"
"Pause all bots"
```

---

## Project Structure

```
my_ai_trading_system/
├── app/
│   ├── main.py           # FastAPI app + lifespan startup
│   ├── config.py         # Pydantic settings from .env
│   ├── database.py       # Async SQLite (aiosqlite) CRUD
│   ├── ai_manager.py     # OpenAI → TradeInstruction JSON
│   ├── trading_core.py   # CCXT exchange integration
│   └── routes/
│       ├── api.py        # REST endpoints
│       └── websocket.py  # WS endpoint + broadcast loop
├── static/
│   ├── index.html        # Dark-mode Tailwind dashboard
│   └── app.js            # WebSocket + REST frontend logic
├── .env.example
├── requirements.txt
└── README.md
```

---

## Migrating to a Linux VPS

```bash
# 1. Copy project to server
scp -r my_ai_trading_system user@your-vps:/opt/

# 2. Install deps & run with a process manager
pip install -r requirements.txt
pip install gunicorn
gunicorn app.main:app -w 1 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000

# 3. (Optional) Use systemd or supervisord to keep it alive
```

---

## Safety Reminders

- `EXCHANGE_TESTNET=true` is the default — **never switch to live until fully tested**
- `MAX_TRADE_RISK_PERCENT` hard-caps how much of your portfolio any single AI command can risk
- All commands are logged to `trading.db` in the `commands` table for full auditability
