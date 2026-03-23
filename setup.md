# Revidata.ai — Setup Guide

## Prerequisites
- Python 3.11+
- Node.js 18+
- A Google Cloud project (see instructions below)
- Anthropic API key
- Stripe account

---

## Step 1 — Python environment

```bash
cd "Analytics Project"
python -m venv .venv
.venv\Scripts\activate          # Windows
pip install -r requirements.txt
```

---

## Step 2 — Frontend dependencies

```bash
cd frontend
npm install
cd ..
```

---

## Step 3 — Environment variables

```bash
copy .env.example .env
```

Edit `.env` and fill in:

1. **SECRET_KEY** — generate with: `python -c "import secrets; print(secrets.token_hex(32))"`
2. **ENCRYPTION_KEY** — generate with: `python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"`
3. **GOOGLE_CLIENT_ID** / **GOOGLE_CLIENT_SECRET** — from your Google Cloud OAuth credentials JSON
4. **ANTHROPIC_API_KEY** — your Claude API key
5. **STRIPE_SECRET_KEY** / **STRIPE_PUBLISHABLE_KEY** — from Stripe dashboard
6. **STRIPE_WEBHOOK_SECRET** — from Stripe CLI or dashboard
7. **STRIPE_PRICE_ID** — the `price_...` ID of your $19.99/month product

---

## Step 4 — Run the backend

```bash
# From the Analytics Project root
.venv\Scripts\activate
uvicorn backend.main:app --reload
```

Backend runs at: http://localhost:8000
API docs: http://localhost:8000/docs

---

## Step 5 — Run the frontend

In a new terminal:
```bash
cd frontend
npm run dev
```

Frontend runs at: http://localhost:5173

---

## Step 6 — Stripe webhooks (local dev)

Install the Stripe CLI: https://stripe.com/docs/stripe-cli

```bash
stripe listen --forward-to localhost:8000/api/billing/webhook
```

Copy the webhook signing secret it prints and add it to `.env` as `STRIPE_WEBHOOK_SECRET`.

---

## First-time setup

1. Open http://localhost:5173
2. Click "Get started" → register an account
3. Go to Settings → Connect GA4 (this opens Google OAuth)
4. Go to Settings → Subscribe ($19.99/month)
5. Go to "Ask AI" and start querying!

---

## Make yourself admin (optional)

```bash
# After registering, open a Python shell:
.venv\Scripts\activate
python -c "
from backend.database import SessionLocal
from backend.models.user import User
db = SessionLocal()
u = db.query(User).filter(User.email == 'your@email.com').first()
u.is_admin = True
db.commit()
print('Done')
"
```

---

## Project structure

```
Analytics Project/
├── backend/          # FastAPI Python backend
│   ├── api/          # Route handlers
│   ├── models/       # SQLAlchemy ORM models
│   ├── schemas/      # Pydantic request/response schemas
│   ├── services/     # Business logic (AI, GA4, Stripe, etc.)
│   ├── prompts/      # Claude AI prompt templates
│   └── utils/        # Encryption, chart helpers
├── frontend/         # React + Vite frontend
│   └── src/
│       ├── api/      # Typed API client
│       ├── components/
│       ├── pages/
│       └── store/    # Zustand auth state
├── data/             # SQLite DB + Parquet snapshots (auto-created)
├── .env              # Your secrets (never commit)
└── requirements.txt
```
