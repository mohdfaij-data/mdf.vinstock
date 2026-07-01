<div align="center">

# 📈 VinStock

**A stock research platform for Indian (NSE) equities — real-time fundamentals, a rule-based screener, watchlists, and score-driven insights.**

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=flat&logo=python&logoColor=white)](https://www.python.org/)
[![Flask](https://img.shields.io/badge/Flask-2.x-000000?style=flat&logo=flask&logoColor=white)](https://flask.palletsprojects.com/)
[![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-ORM-D71F00?style=flat)](https://www.sqlalchemy.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Live Demo](https://img.shields.io/badge/demo-live-brightgreen)](https://vin-stock-mocha.vercel.app)
[![YouTube](https://img.shields.io/badge/YouTube-Watch_Demo-FF0000?style=flat&logo=youtube&logoColor=white)](https://youtu.be/YOUR_VIDEO_ID)

[Live Demo](https://vin-stock-mocha.vercel.app) · [Watch Demo Video](https://youtu.be/YOUR_VIDEO_ID) · [Report Bug](../../issues) · [Request Feature](../../issues)

</div>

---

## Table of Contents

- [Overview](#overview)
- [Demo Video](#demo-video)
- [Screenshots](#screenshots)
- [Features](#features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Populating the Screener Cache](#populating-the-screener-cache)
- [What's Real vs. Approximated](#whats-real-vs-approximated)
- [Database Migration (SQLite → PostgreSQL)](#database-migration-sqlite--postgresql)
- [Deployment](#deployment)
- [Known Limitations](#known-limitations)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

VinStock turns raw NSE market data into structured, decision-ready insight. Instead of scattering P/E ratios, balance sheets, and screeners across five different broker apps, VinStock centralizes them behind a single Flask application with a clean data-access layer, a cached screener engine, and a transparent scoring system — so every number on screen traces back to a real API call, not a guess.

The project is explicitly versioned as **Phase 1**: a working, honestly-documented foundation with clear seams for the analytics and ML work planned in later phases.

---

## Demo Video

<div align="center">

[![Watch the VinStock demo on YouTube](https://img.youtube.com/vi/YOUR_VIDEO_ID/maxresdefault.jpg)]([https://youtu.be/YOUR_VIDEO_ID](https://www.youtube.com/channel/UCHw7w6PaEkZ25kme3Ix62GA))

**[▶ Watch the full walkthrough on YouTube]([https://youtu.be/YOUR_VIDEO_ID](https://www.youtube.com/channel/UCHw7w6PaEkZ25kme3Ix62GA))**

</div>

A short walkthrough covering the screener, fundamental analysis view, VinStock Score, and watchlists — start to finish in a few minutes.

---

## Screenshots

<table>
  <tr>
    <td width="50%">
      <b>Home / Dashboard</b><br/>
      <img src="home-screen.png" alt="VinStock home screen showing stock search and market overview" width="100%"/>
    </td>
    <td width="50%">
      <b>About Section</b><br/>
      <img src="about-section.png" alt="VinStock about section describing the platform and its purpose" width="100%"/>
    </td>
  </tr>
</table>

> Replace the images above by adding your own screenshots to `assets/screenshots/` in the repo (create the folder if it doesn't exist) and committing them — GitHub will render them automatically once pushed.

---

## Features

| Category | Capability |
|---|---|
| **Market Data** | Live price and fundamental data for a curated ~184-stock NSE universe via `yfinance` |
| **Fundamental Analysis** | P/E, P/B, PEG, EPS, ROE, ROCE, Market Cap, Enterprise Value, P/S, Dividend Yield, Debt/Equity, Beta, 52-week high/low |
| **Financial Statements** | Revenue, Net Profit, Operating Profit, EBITDA, Cash Flow, Balance Sheet, Assets, Liabilities, Shareholders' Equity |
| **Screener** | Prebuilt + custom screens evaluated against a refreshed cache table — not live calls per request |
| **VinStock Score** | Composite score summarizing financial strength across multiple fundamental indicators |
| **Company Intelligence** | Sector/industry classification, business summaries, hand-curated peer groups for ~20 major stocks |
| **Watchlists** | Per-user watchlists with full CRUD |
| **Accounts** | Email + password registration and login |
| **Transparency by design** | Screens that are approximated (rather than historically accurate) are labeled in the UI with an explanation of exactly what data is missing — nothing is silently faked |

---

## Architecture

VinStock separates data access, business logic, and presentation into distinct layers so that adding a new data provider, screen, or model later doesn't ripple through the whole codebase.

```
vinstock/
│
├── providers/              Data-source abstraction layer
│   └── yfinance_provider.py    Only provider today. Adding NSE/BSE/FMP/
│                                Alpha Vantage later = new provider class,
│                                zero changes to routes or templates.
│
├── models/                 SQLAlchemy ORM models
│   ├── Users
│   ├── Watchlists / WatchlistStocks
│   ├── Alerts
│   ├── SavedScreen
│   ├── StockUniverse
│   └── StockMetricsCache
│
├── services/                Business logic (no DB/HTTP wiring in routes)
│   ├── auth_service.py         Registration & login
│   ├── watchlist_service.py    Watchlist CRUD
│   ├── stock_universe.py       Curated list of ~184 NSE stocks (data, not logic)
│   ├── collector.py            Pulls each stock through the provider,
│   │                            computes derived metrics + score, writes cache
│   └── screener_engine.py      Evaluates prebuilt/custom screens against cache
│
├── templates/               Jinja2 templates (server-rendered, no SPA build)
├── static/style.css          Single stylesheet, extended across phases
├── app.py                    Application entrypoint
├── Procfile                  Render/Gunicorn process definition
└── requirements.txt
```

**Design principle:** routes never talk to `yfinance` directly. Everything flows `provider → collector → cache table → screener_engine / routes`, which is what makes the screener fast (reads are pure SQL, not live network calls) and makes swapping data vendors a contained change.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Python, Flask |
| ORM / DB | SQLAlchemy, SQLite (Postgres-ready) |
| Templating | Jinja2 |
| Frontend | HTML5, CSS3, vanilla JavaScript |
| Market Data | yfinance API |
| Deployment | Render (Gunicorn) |
| Version Control | Git & GitHub |

---

## Getting Started

### Prerequisites
- Python 3.10+
- pip

### Installation

```bash
git clone https://github.com/mohdfaij-data/VinStock.git
cd VinStock
pip install -r requirements.txt
python app.py
```

Visit **[http://localhost:5000](http://127.0.0.1:5000)**. The SQLite database (`instance/vinstock.db`) and the 184-stock universe are created automatically on first run.

---

## Populating the Screener Cache

Screens filter against a **cache table**, not live data — so the cache needs to be populated before `/screens` will show results:

1. Go to `/admin/refresh`
2. Click **"Run Refresh Now"** — try a small `limit` (e.g. `10`) first. Refreshing all ~184 stocks takes several minutes since each is a live network call.
3. Once complete, `/screens` will return real matches.

This mirrors a production setup where a scheduled job (cron / APScheduler) refreshes the cache every 6–12 hours, and end users only ever read from it.

---

## What's Real vs. Approximated

Every number in VinStock comes from real `yfinance` data — **nothing is fabricated**. Some screens, however, are labeled **"Approximated"** in the UI because their true version requires multi-year historical snapshots not yet stored (planned for Phase 2):

- **"Low on 10-Year Average P/E"** — currently checks *low absolute P/E*, not P/E relative to the stock's own historical average.
- **"Debt Reduction"** — checks *current* low Debt/Equity, not a multi-year decreasing trend.
- **"Growth Without Dilution"** — checks growth rates, not share-count stability over time.

Fields with no available data source at all (Promoter/FII/DII holding, Quick Ratio, Interest Coverage) are marked **"Coming soon"** rather than shown with a guessed value.

---

## Database Migration (SQLite → PostgreSQL)

All models use standard SQLAlchemy types with no SQLite-specific features. Migrating is a one-line change in `app.py`:

```python
app.config["SQLALCHEMY_DATABASE_URI"] = "postgresql://user:pass@host/dbname"
```

Then install the driver:

```bash
pip install psycopg2-binary
echo "psycopg2-binary" >> requirements.txt
```

---

## Deployment

**Render**

1. Push this repo to GitHub.
2. Create a new Web Service on Render and connect the repo.
3. Build command: `pip install -r requirements.txt`
4. Start command: `gunicorn app:app` *(already defined in `Procfile`)*
5. **Important:** Render's free tier uses an ephemeral filesystem — SQLite data will not persist across deploys/restarts. For anything beyond a demo, provision a managed PostgreSQL instance and point `SQLALCHEMY_DATABASE_URI` at it via an environment variable.

---

## Known Limitations

Documented up front rather than discovered the hard way:

- **`yfinance` is unofficial** — Yahoo can change response formats or rate-limit without notice. Mass refresh failures are almost always caused by this.
- **Annual financials** typically return ~4 years of history (not 10).
- **Quarterly financials** typically return ~4–5 quarters.
- **Stock universe (184 symbols)** is hand-curated for Phase 1, not the full NSE listing. Index-membership labels (e.g. "Nifty 50") are best-effort and should be reconciled against nseindia.com's official files before being treated as authoritative.
- **Peer comparison** covers hand-curated peer groups for ~20 major stocks only.
- **No password reset / email verification** yet — auth is email+password only, no email sending is wired up.
- **`SECRET_KEY` is a hardcoded dev value** in `app.py` — replace with an environment variable before any real deployment.

---

## Roadmap

- [ ] Historical metric snapshots to power true trend-based screens (10-yr avg P/E, debt-reduction trend, etc.)
- [ ] Predictive analytics / ML-assisted screening
- [ ] Portfolio analytics (returns, allocation, risk)
- [ ] Broader NSE/BSE universe beyond the curated 184
- [ ] Scheduled background refresh (APScheduler / cron) in production
- [ ] Price/email alerts on watchlist stocks
- [ ] Full NSE index reconciliation for membership labels

---

## Contributing

Contributions, issues, and feature requests are welcome.

1. Fork the repo
2. Create a branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m "Add: your feature"`
4. Push and open a Pull Request

---

## License

Distributed under the MIT License. See `LICENSE` for details.

---

<div align="center">

Built by [Mohd Faij](https://github.com/mohdfaij-data) · If this project is useful, consider ⭐ starring the repo.

</div>
