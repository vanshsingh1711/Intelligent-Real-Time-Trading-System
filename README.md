# Intelligent Real-Time Trading System 

## Overview

An AI-powered, full-stack trading intelligence platform for Indian stock markets (NSE/BSE). It provides real-time trading signals, technical analysis, sentiment monitoring, macro indicator tracking, portfolio management, and risk analysis — all powered by Google Gemini AI and a Python FastAPI backend.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 15 (App Router) |
| Language | TypeScript |
| Styling | Tailwind CSS |
| Auth | Clerk |
| Database | PostgreSQL via NeonDB |
| ORM | Prisma |
| AI | Google Gemini 2.5 Flash Lite |
| Charts | Recharts, Lightweight Charts |
| Python Backend | FastAPI + Uvicorn |
| ML | NumPy (LSTM-like, RL-inspired) |
| UI Components | Radix UI + shadcn/ui |
| Animations | Framer Motion |
| Drag & Drop | dnd-kit |
| Fonts | Poppins, Roboto Mono |

---

## Architecture Diagram

```
Browser (Next.js Client)
        │
        ▼
┌─────────────────────────┐
│   Next.js App Router     │  ← SSR + Client Components
│   /app/page.tsx          │  ← Landing Page
│   /app/(auth)/...        │  ← Sign In / Sign Up (Clerk)
│   /app/(dashboard)/...   │  ← Protected Dashboard Pages
│   /app/api/...           │  ← Next.js API Routes (Route Handlers)
└────────────┬────────────┘
             │
     ┌───────┴────────┐
     │                │
     ▼                ▼
┌─────────┐    ┌──────────────┐
│ Gemini  │    │  NeonDB      │
│ AI API  │    │  (Postgres)  │
│ (LLM)   │    │  via Prisma  │
└─────────┘    └──────────────┘
             │
             ▼
┌──────────────────────┐
│  Python FastAPI      │  ← Optional ML backend (port 8000)
│  /api/technical-analysis
│  /api/predict        │
│  /api/rl-signal      │
│  /api/sentiment      │
└──────────────────────┘
```

---

## Authentication Flow

- Powered by **Clerk** (`@clerk/nextjs`)
- Root layout wraps entire app in `<ClerkProvider>`
- `middleware.ts` protects all routes except: `/`, `/sign-in`, `/sign-up`, `/api/stocks`
- Dashboard routes require authentication via `auth.protect()`
- `UserButton` from Clerk renders in the header for profile/sign-out

---

## Database Schema (Prisma + PostgreSQL/NeonDB)

### Models

| Model | Purpose |
|---|---|
| `User` | Synced with Clerk; owns portfolios, watchlists, signals, alerts, trade history |
| `Stock` | NSE/BSE stock metadata (price, PE, market cap, etc.) |
| `Portfolio` | User's named portfolio container |
| `PortfolioItem` | Individual stock holding (symbol, qty, avg buy price) |
| `Watchlist` | User's named watchlist |
| `WatchlistItem` | Stock in a watchlist (unique per watchlist) |
| `TradingSignal` | BUY/SELL/HOLD signal with confidence, scores, target/stop-loss |
| `NewsArticle` | Article with sentiment score (BULLISH/BEARISH/NEUTRAL) |
| `MacroIndicator` | GDP, inflation, FII flows, VIX, rates etc. |
| `Alert` | Price alert per user (ABOVE/BELOW/PERCENT_CHANGE) |
| `TradeHistory` | Log of BUY/SELL actions with price and qty |

---

## Frontend Structure

```
app/
├── layout.tsx              ← Root layout (ClerkProvider, ThemeProvider, fonts)
├── page.tsx                ← Landing page
├── globals.css             ← Global styles
├── (auth)/
│   ├── sign-in/[...sign-in]/page.tsx
│   └── sign-up/[...sign-up]/page.tsx
└── (dashboard)/
    ├── layout.tsx          ← Dashboard shell (Sidebar + Header)
    ├── dashboard/page.tsx  ← Main dashboard
    ├── stocks/
    │   ├── page.tsx        ← Stock screener
    │   └── [symbol]/page.tsx ← Individual stock detail
    ├── portfolio/page.tsx  ← Portfolio manager
    ├── signals/page.tsx    ← AI trading signals
    ├── sentiment/page.tsx  ← Market sentiment
    ├── macro/page.tsx      ← Macro indicators
    └── risk/page.tsx       ← Risk management

components/
├── layout/
│   ├── header.tsx          ← Top header (search, notifications, user)
│   ├── sidebar.tsx         ← Side nav + AI chat panel
│   └── mobile-nav.tsx      ← Mobile drawer navigation
├── dashboard/
│   ├── market-overview.tsx
│   ├── portfolio-summary.tsx
│   ├── sentiment-widget.tsx
│   ├── top-movers.tsx
│   └── trading-signals-widget.tsx
├── charts/                 ← Chart components
├── signals/                ← Signal card components
├── stocks/                 ← Stock table components
└── ui/                     ← shadcn/ui base components (21 files)
```

---

## Page-by-Page Details

### 1. Landing Page (`/`)
- Public marketing page
- Hero section with typewriter effect ("Ready to Trade Smarter?")
- Live scrolling ticker of mock AI signals (`InfiniteMovingCards`)
- Stats section: 10+ years, 100k+ trades, 100% uptime, 100+ indicators
- Interactive feature grid (6 cards, expandable on click): AI Signals, Technical Analysis, Real-Time Sentiment, Macro Indicators, Risk Management, 50+ Indian Stocks
- Tech stack showcase (Next.js, TypeScript, AI, Prisma, NeonDB, Clerk, FastAPI, Tailwind)
- CTA section with typewriter animation
- Clerk-aware: shows Dashboard/Sign Out if logged in, Sign In/Get Started if not

### 2. Sign In/Sign Up (`/sign-in`, `/sign-up`)
- Handled entirely by Clerk's hosted components
- Redirects to `/dashboard` on success

### 3. Dashboard (`/dashboard`)
- **Polling:** fetches `/api/stocks` and `/api/signals` every **3 seconds** for live data
- **Market Overview Widget:** key indices stats, gainers/losers count
- **Trading Signals Widget:** top 5 AI signals with BUY/SELL/HOLD badges
- **Sentiment Widget:** overall sentiment score gauge
- **Top Movers:** top 5 gainers and losers table
- **Portfolio Summary:** total value, P&L, day P&L
- Skeleton loading states on all sections

### 4. Stock Screener (`/stocks`)
- Fetches all 51 Indian stocks + signals every **3 seconds**
- Search bar (by symbol or name)
- Sector filter dropdown (15 sectors)
- Signal filter (ALL / BUY / SELL / HOLD)
- Signal count badges (BUY/SELL/HOLD totals)
- Full sortable stock table with current price, change, signal, volume

### 5. Individual Stock Detail (`/stocks/[symbol]`)
- Loads OHLCV data, computes all indicators client-side
- Candlestick chart (Lightweight Charts) with SMA20/SMA50 overlays
- RSI chart and MACD chart
- Technical indicator panel: RSI, MACD, Bollinger Bands, SMA, EMA, support/resistance, trend, momentum
- "Analyse with AI" button → calls `/api/analysis` → Gemini full analysis
- Shows AI summary, strengths, risks, recommendation, price target

### 6. Trading Signals (`/signals`)
- Fetches pre-generated signals from `/api/signals`
- Filter by: ALL/BUY/SELL/HOLD and ALL/High(75%+)/Medium/Low confidence
- "Generate New" button triggers `/api/signals` POST to regenerate all signals
- Cards show: symbol, signal badge, confidence bar, technical/sentiment/macro scores, target price, stop-loss, timeframe, reasoning

### 7. Portfolio (`/portfolio`)
- Displays holdings table: symbol, qty, avg buy price, CMP, P&L, value
- Summary cards: Total Value, Total P&L, Today's P&L, Invested Amount
- Sector allocation donut chart (Recharts PieChart)
- Add Stock dialog: symbol + qty + avg price → POST `/api/portfolio`
- Remove stock button → DELETE `/api/portfolio?symbol=XYZ`
- Default pre-loaded with 5 stocks: RELIANCE, TCS, HDFCBANK, INFY, ICICIBANK

### 8. Market Sentiment (`/sentiment`)
- Overall sentiment gauge (progress bar, score 0-100)
- Breakdown: count of Bullish / Neutral / Bearish stocks
- 30-day sentiment trend line chart (Recharts LineChart)
- Stock-wise sentiment table: symbol, badge, score, news count
- Recent News feed: up to 10 articles with source, timestamp, linked symbols

### 9. Macro Indicators (`/macro`)
- Displays 14 real Indian macroeconomic indicators in 4 categories:
  - **MONETARY:** RBI Repo Rate, Reverse Repo, CPI Inflation, WPI Inflation
  - **FISCAL:** GDP Growth, Fiscal Deficit, IIP
  - **TRADE:** Forex Reserves, Trade Deficit, USD/INR
  - **MARKET:** India VIX, FII Flows, DII Flows, 10-yr G-Sec Yield
- Each card shows value, previous value, change, impact badge (HIGH/MEDIUM/LOW), date
- "AI Analysis" button → POST `/api/macro` → Gemini analyzes all indicators
- Shows AI market outlook + sector-wise impact (Banking, IT, Pharma, Auto, FMCG, Energy, Metals, Infrastructure)

### 10. Risk Management (`/risk`)
- Portfolio risk metrics: Beta, Sharpe Ratio, Max Drawdown, Volatility (Annual), VaR 95%, Sortino Ratio, Calmar Ratio
- Risk rating banner (Low/Moderate/High based on Beta)
- Risk Alerts panel (CONCENTRATION, VOLATILITY, DRAWDOWN)
- Top Risk Contributors: per-stock risk with progress bars
- **Position Sizing Calculator:** Enter symbol, entry price, stop-loss, risk amount → calculates recommended quantity, total investment, risk %, target price (1:2 RR), risk:reward ratio

---

## Backend API Routes (Next.js Route Handlers)

### `GET /api/stocks`
Returns all 51 mock Indian stocks with live-simulated prices (seeded RNG, 3-second tick during market hours).

### `GET/POST /api/signals`
- **GET:** Returns cached signals (refreshed every 5 min). Generates signals for top 20 stocks using technical analysis.
- **POST `{ symbol }`:** Generates a fresh Gemini AI signal for a specific stock.
- **POST `{ generateAll: true }`:** Regenerates all signals.

### `GET/POST/DELETE /api/portfolio`
- **GET:** Returns portfolio with live prices, P&L, weights, sector data.
- **POST:** Add/update a position (averages down/up if already held).
- **DELETE `?symbol=XYZ`:** Removes a holding.

### `GET/POST /api/macro`
- **GET:** Returns 14 hardcoded macro indicators.
- **POST:** Calls Gemini to analyze all indicators → returns overall impact + sector breakdown.

### `GET /api/sentiment`
Generates mock news for 15 stocks, scores each article (BULLISH/BEARISH/NEUTRAL), computes stock and overall sentiment scores, and a 30-day trend series.

### `POST /api/analysis`
Full deep-dive analysis for one symbol: stock data + technical indicators + Gemini sentiment from news headlines + Gemini trading signal + Gemini full stock analysis (summary, strengths, risks, recommendation, price target).

### `GET/POST/DELETE /api/watchlist`
In-memory watchlist management. Default: RELIANCE, TCS, HDFCBANK, INFY, ICICIBANK.

### `POST /api/chat`
Gemini-powered chat with conversation history. System prompt: Indian markets trading assistant (RSI, MACD, signals, portfolio help).

---

## Core Libraries (`/lib`)

### `indian-stocks.ts`
- 51 NSE stocks (NIFTY 50 + extras) with base prices and sectors
- `SeededRandom` class: deterministic mock prices (changes every 3s during market hours, frozen after close)
- `getMockStockData(symbol)` → real-feeling live quote
- `generateHistoricalData(symbol, days)` → realistic OHLCV with upward drift
- `getIndianMarketStatus()` → IST-aware market open/close check
- `getTopMovers()`, `searchStocks()`, `getSectors()`

### `technical-analysis.ts`
Pure TypeScript implementations of:
- `calculateSMA(data, period)` — Simple Moving Average
- `calculateEMA(data, period)` — Exponential Moving Average
- `calculateRSI(data, period=14)` — Relative Strength Index
- `calculateMACD(data, fast=12, slow=26, signal=9)` — MACD
- `calculateBollingerBands(data, period=20, mult=2)` — Bollinger Bands
- `calculateSupportResistance(highs, lows)` — Key S/R levels
- `detectTrend(closes)` — UPTREND / DOWNTREND / SIDEWAYS
- `calculateMomentum(closes)` — 10-day price momentum
- `analyzeVolume(volumes)` — Volume vs 20-day average
- `generateTechnicalSignal(indicators)` — Scoring-based BUY/SELL/HOLD
- `calculateAllIndicators(ohlcv[])` — Computes all of the above in one call
- `prepareChartData/prepareRSIData/prepareMACDData` — For chart rendering

### `gemini.ts`
Wrapper around `@google/generative-ai` (model: `gemini-2.5-flash-lite`):
- `analyzeStockSentiment(symbol, headlines[])` → BULLISH/BEARISH/NEUTRAL + score + reasoning + key factors
- `generateTradingSignal(stockData)` → BUY/SELL/HOLD + confidence + reasoning + target + stop-loss + timeframe
- `analyzeMacroImpact(indicators[])` → overall impact + market outlook + per-sector impact
- `getMarketSummary(stocks[])` → 2-3 sentence professional market summary
- `analyzeFullStock(symbol, data)` → summary + strengths + risks + recommendation + price target + time horizon
- All functions have graceful fallbacks if Gemini API key is missing or call fails

### `types.ts`
Full TypeScript interfaces for: `StockQuote`, `OHLCVData`, `StockDetail`, `TechnicalIndicators`, `MACDData`, `BollingerBands`, `VolumeAnalysis`, `TradingSignal`, `SentimentAnalysis`, `StockSentiment`, `NewsArticle`, `MacroIndicator`, `MacroAnalysis`, `Portfolio`, `PortfolioItem`, `RiskMetrics`, `PositionSizing`, `MarketIndex`, `MarketOverview`, `WatchlistItem`, `ApiResponse<T>`, chart data types.

### `utils.ts`
Helper functions: `formatPrice`, `formatChange`, `formatCurrency`, `formatDate`, `formatTimeAgo`, `getChangeColor`, `getSentimentColor`, `generateMockNewsHeadlines`, `cn` (class merger).

---

## Python FastAPI Backend (`/python`)

An optional auxiliary backend (port 8000) providing more advanced ML capabilities.

### `main.py` — FastAPI Application
Endpoints:
- `POST /api/technical-analysis` — Full technical analysis from OHLCV input
- `POST /api/predict` — Price prediction (horizon N days)
- `POST /api/rl-signal` — RL-inspired trading signal
- `POST /api/sentiment` — NLP sentiment from headlines
- `GET /health` — Health check
- `GET /api/market-status` — IST-aware market open/close status

CORS configured for `localhost:3000` and Vercel deployments.

### `ml_models.py`
**SimpleLSTMPredictor:**
- Exponential smoothing + linear regression trend slope
- Predicts N future prices with confidence intervals
- Confidence decreases with horizon length

**RLSignalGenerator:**
- Extracts 7 features: RSI, price/SMA20 ratio, volume ratio, H-L range, 5d momentum, 20d returns, volatility
- Simulated Q-values for BUY/SELL/HOLD actions
- Rule-based policy mimicking RL decision-making
- Returns action, confidence (40-95%), and reasoning

### `technical_analysis.py`
Python-side technical analysis (SMA, EMA, RSI, MACD, Bollinger Bands, ATR, Stochastic, etc.) for the FastAPI endpoints.

### `sentiment_analyzer.py`
Keyword/rule-based sentiment analysis on news headlines. Outputs BULLISH/BEARISH/NEUTRAL with score and key phrases.

---

## Layout Components

### `sidebar.tsx`
- Fixed left sidebar (desktop only, 224px wide)
- Navigation links: Dashboard, Stocks, Portfolio, Signals, Sentiment, Macro, Risk
- Active route highlighting
- **AI Assistant** toggle button at bottom → opens floating chat panel (positioned right of sidebar)
- Chat panel: full conversation with Gemini via `/api/chat`, message history, Enter to send
- **Market Status indicator:** live countdown timer showing time until NSE closes (green pulsing dot when open, red when closed)

### `header.tsx`
- Page title (auto-detected from route)
- Stock search bar → navigates to `/stocks/[SYMBOL]`
- **Notification bell** → dropdown panel with 4 mock notifications (BUY/SELL signals, price alerts, market updates), "Mark all as read" clears the badge
- Clerk `UserButton` (avatar, profile, sign-out)
- Mobile menu button (hamburger)

### `mobile-nav.tsx`
- Sheet/drawer navigation for mobile
- Same nav links as sidebar

---

## Data Flow — Signal Generation

```
User visits /signals
      │
      ▼
GET /api/signals
      │
      ├── Cache hit (< 5 min old)? → Return cached signals
      │
      └── Cache miss → generateMockSignals()
              │
              ├── For each of top 20 stocks:
              │     generateHistoricalData(symbol, 100 days)
              │     calculateAllIndicators(historical)
              │     generateTechnicalSignal(indicators) → BUY/SELL/HOLD + score
              │     getMockStockData(symbol) → current price
              │     Compute: technicalScore + sentimentScore + macroScore
              │     → Weighted overall confidence (40-95%)
              │     → Target price & stop-loss multipliers
              └── Returns array of TradingSignal objects
```

## Data Flow — AI Analysis (Per Stock)

```
User clicks "Analyse with AI" on /stocks/[symbol]
      │
      ▼
POST /api/analysis { symbol }
      │
      ├── getMockStockData(symbol)         ← current quote
      ├── generateHistoricalData(symbol)   ← 100-day OHLCV
      ├── calculateAllIndicators(data)     ← all TA indicators
      ├── generateMockNewsHeadlines()      ← simulated news
      ├── analyzeStockSentiment() [Gemini] ← BULLISH/BEARISH/NEUTRAL
      ├── generateTradingSignal() [Gemini] ← BUY/SELL/HOLD + target
      └── analyzeFullStock() [Gemini]      ← full written analysis
            │
            ▼
      Returns: stock + indicators + sentiment + signal + fullAnalysis + headlines
```

---

## Environment Variables (`.env.local`)

```env
# Clerk Authentication
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_...
CLERK_SECRET_KEY=sk_...
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/dashboard
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/dashboard

# Database
DATABASE_URL=postgresql://...

# Gemini AI
GEMINI_API_KEY=AIza...
```

---

## Key Design Decisions

1. **Mock data with seeded RNG:** Stock prices use a deterministic seeded random number generator tied to a 3-second "market tick" so all API responses show the same price for the same stock at the same moment — no inconsistency between API calls.

2. **Market-hours-aware pricing:** The RNG seed is frozen at the last market close price outside of 9:15 AM – 3:30 PM IST on weekdays.

3. **5-minute signal cache:** Signals are cached server-side for 5 minutes to avoid re-computing 20 stocks' worth of technical analysis on every request.

4. **Gemini fallbacks:** Every Gemini call has a deterministic fallback response based on technical indicators alone, so the UI always renders even without an API key.

5. **In-memory portfolio/watchlist:** Currently stored in module-level variables (reset on server restart). Production would use Prisma + PostgreSQL.

6. **Python backend is optional:** The FastAPI ML backend at port 8000 is an extension. The Next.js app works fully standalone using the TypeScript implementations in `/lib`.

---

## Running the Project

```bash
# Install dependencies
npm install

# Set up environment variables
cp .env.example .env.local
# Fill in CLERK keys, DATABASE_URL, GEMINI_API_KEY

# Push database schema
npm run db:push

# Run development server
npm run dev
# → http://localhost:3000

# (Optional) Run Python backend
cd python
pip install -r requirements.txt
python main.py
# → http://localhost:8000
```

---

## Folder Structure Summary

```
trading-main/
├── app/
│   ├── layout.tsx              # Root layout
│   ├── page.tsx                # Landing page
│   ├── globals.css             # Global CSS + Tailwind
│   ├── (auth)/                 # Clerk auth pages
│   ├── (dashboard)/            # Protected dashboard pages
│   └── api/                    # Next.js API route handlers
├── components/
│   ├── layout/                 # Sidebar, Header, MobileNav
│   ├── dashboard/              # Dashboard widgets
│   ├── charts/                 # Chart components
│   ├── signals/                # Signal card components
│   ├── stocks/                 # Stock table components
│   └── ui/                     # Base UI components (shadcn)
├── lib/
│   ├── gemini.ts               # Gemini AI integration
│   ├── indian-stocks.ts        # Stock data + mock engine
│   ├── technical-analysis.ts   # All TA indicator functions
│   ├── types.ts                # TypeScript interfaces
│   ├── utils.ts                # Helper utilities
│   └── prisma.ts               # Prisma client singleton
├── prisma/
│   └── schema.prisma           # Database schema
├── python/
│   ├── main.py                 # FastAPI app
│   ├── technical_analysis.py   # Python TA
│   ├── ml_models.py            # LSTM + RL models
│   ├── sentiment_analyzer.py   # NLP sentiment
│   └── requirements.txt
├── middleware.ts               # Clerk route protection
├── next.config.ts
├── tailwind.config.ts
└── package.json
```
