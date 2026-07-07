# FinTrack(Name pending) — Personal Finance & Growth Tracker

> Track your budget, savings, and investments in one place — and know exactly what it takes to hit your financial goals.

FinTrack(Name Pending) is a full-stack personal finance app that goes beyond simple expense tracking. It combines **budgeting**, **savings**, and **investment tracking** with a built-in **goal forecasting engine** that tells you not just where you stand, but what to do next — how much to save, whether you're on pace, and what happens if your habits change.

---

## ✨ Features

- **Budgeting** — categorize income and expenses, set monthly limits, get alerted before you overspend
- **Savings** — track savings "buckets" for different goals (emergency fund, vacation, etc.) with automatic contribution tracking
- **Investing** — monitor portfolio performance and asset allocation across accounts
- **Goal Forecasting** — set a target amount and date; FinTrack calculates the exact contribution needed and tells you if you're ahead, on track, or behind
- **What-If Simulator** — adjust contribution amount, growth rate, or target date and see your projected balance update live
- **Net Worth Dashboard** — a single view of assets, liabilities, and trend over time
- **Web + Mobile** — full feature parity across platforms
- **Admin Analytics** — a private, real-time usage dashboard (active users, retention, growth) separate from the user-facing app

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Backend / API | Python (Django / Flask REST API) |
| Web Frontend | JavaScript, HTML5, CSS |
| Mobile App | Java (Android) |
| Database | PostgreSQL |
| Auth | JWT-based session auth, 2FA |
| Hosting | TBD (Heroku / Render / AWS) |

> Stack is still evolving as the project grows — this table will be updated as decisions are finalized.

---

## 📐 Architecture Overview

```
┌─────────────┐     ┌─────────────┐
│  Web Client │     │Mobile Client│
│ (JS/HTML/CSS)│    │   (Java)    │
└──────┬──────┘     └──────┬──────┘
       │                   │
       └─────────┬─────────┘
                  │
           ┌──────▼──────┐
           │  Python API  │
           │ (Django/Flask)│
           └──────┬──────┘
                  │
           ┌──────▼──────┐
           │ PostgreSQL   │
           │  Database    │
           └─────────────┘
```

---

## 🎯 Goal Forecasting (Core Logic)

FinTrack's forecasting engine calculates:

1. **Required contribution** — the monthly amount needed to reach a goal by its target date, accounting for compounding growth
2. **On-track status** — compares actual progress to expected progress at any point in time
3. **Projected balance** — future value of current savings + planned contributions under adjustable growth assumptions

See [`/docs/forecasting.md`](./docs/forecasting.md) for the full formulas and logic.

---

## 🚀 Getting Started

```bash
# Clone the repo
git clone https://github.com/<your-username>/fintrack.git
cd fintrack

# Backend setup
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python manage.py migrate
python manage.py runserver

# Frontend setup
cd ../frontend
npm install
npm start
```

Environment variables needed (create a `.env` file):
```
DATABASE_URL=postgresql://user:password@localhost:5432/fintrack
SECRET_KEY=your-secret-key
```

---

## 📁 Project Structure

```
fintrack/
├── backend/          # Python API (Django/Flask)
│   ├── models/
│   ├── views/
│   └── forecasting/  # goal calculation logic
├── frontend/          # Web app (JS/HTML/CSS)
├── mobile/            # Android app (Java)
├── docs/              # Design docs, forecasting math, ERD
└── README.md
```

---

## 🗺️ Roadmap

- [x] Project planning & feature checklist
- [ ] Budgeting module (MVP)
- [ ] Savings module
- [ ] Goal forecasting engine
- [ ] Investment tracking
- [ ] Web app
- [ ] Mobile app
- [ ] Admin analytics dashboard
- [ ] Bank account linking (Plaid integration)

---

## 📄 License

This project is currently unlicensed / for personal & educational use. Update this section if you decide to open-source it.

---

## 🙋 About

Built as a personal project to learn full-stack development and apply it to a real, useful tool — a finance tracker that actually tells you what to do, not just what you spent.
