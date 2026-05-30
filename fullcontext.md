# 🤖 Telegram Order Bot — Vynkrova

A lightweight, AI-free food ordering automation bot for Tamil Nadu SMBs (cloud kitchens, tiffin services, bakeries). Customers place orders via Telegram, orders are parsed instantly using regex, logged to Google Sheets, and a confirmation is sent back — all in under 3 seconds.

> **Status:** MVP Complete ✅ | Deployed on: Local (Docker + ngrok) | Next: VPS + WhatsApp Cloud API

---

## 🧱 Stack

| Layer | Tool |
|---|---|
| Messaging | Telegram Bot API |
| Automation | n8n (self-hosted, Docker) |
| Order Parsing | Regex (JavaScript, no AI calls) |
| Storage | Google Sheets |
| Tunnel (dev) | ngrok |

---

## ⚡ Features

- ✅ Receives orders via Telegram chat
- ✅ Parses item names + quantities from natural language (`2 idli 1 dosa`, `masala dosa 3`)
- ✅ Handles menu queries (`menu`, `hi`, `hello`)
- ✅ Generates unique Order IDs (`ORD-XXXXXX`)
- ✅ Sends itemized confirmation with total to customer
- ✅ Logs all orders to Google Sheets with timestamp
- ✅ Fallback help message for unrecognised input

---

## 🗂️ Workflow Architecture

```
Telegram Message
      ↓
Extract Message (Code node)
      ↓
Parse Intent & Items (Regex)
      ↓
Is Order? ──true──→ Build Order → Log to Sheets → Send Confirmation
      │
    false
      ↓
Is Menu Query? ──true──→ Send Menu
      │
    false
      ↓
Send Help Message
```

---

## 🚀 Local Setup

### Prerequisites
- Docker installed
- ngrok account (free tier works)
- Google Cloud project with Sheets API + Drive API enabled
- Telegram bot created via [@BotFather](https://t.me/BotFather)

---

### Step 1 — Start ngrok

```bash
cd ~/Downloads/ngrok-v3-stable-linux-amd64
./ngrok http 5678
```

Copy the `https://xxxx.ngrok-free.app` URL.

---

### Step 2 — Start n8n via Docker

```bash
docker stop n8n && docker rm n8n

docker run -d --name n8n -p 5678:5678 \
  -e WEBHOOK_URL=https://YOUR_NGROK_URL \
  -e N8N_SECURE_COOKIE=false \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n
```

> ⚠️ Replace `YOUR_NGROK_URL` with the URL from Step 1 every session.

---

### Step 3 — Import Workflow

1. Open n8n at `https://YOUR_NGROK_URL` **(not localhost — important for OAuth)**
2. Workflows → Import → upload `telegram_order_bot.json`

---

### Step 4 — Set up Credentials

**Telegram:**
- Credentials → New → Telegram API → paste Bot Token from BotFather

**Google Sheets:**
- Go to [Google Cloud Console](https://console.cloud.google.com) → APIs & Services → Credentials → OAuth 2.0 Client
- Add `https://YOUR_NGROK_URL/rest/oauth2-credential/callback` to Authorized Redirect URIs
- Enable **Google Sheets API** and **Google Drive API**
- Add your Gmail as a test user in OAuth Consent Screen
- In n8n → Credentials → Google Sheets OAuth2 → Sign in with Google

> ⚠️ Always do OAuth from the ngrok URL, never localhost.

---

### Step 5 — Configure Workflow

1. Open `Log to Google Sheets` node → replace `YOUR_GOOGLE_SHEET_ID` with your actual Sheet ID (found in the Sheet URL between `/d/` and `/edit`)
2. Create a Google Sheet named `Orders` with these columns:
```
Order ID | Date | Customer | Chat ID | Items | Total | Status
```

---

### Step 6 — Delete any stuck Telegram webhook

Open in browser:
```
https://api.telegram.org/botYOUR_BOT_TOKEN/deleteWebhook
```

---

### Step 7 — Activate & Test

1. In n8n → click **Publish** to activate the workflow
2. Send `menu` to your Telegram bot
3. Send `2 idli 1 dosa` to place a test order
4. Check Google Sheets for the logged order

---

## 📋 Menu (edit in Parse Intent node)

| Item | Price |
|---|---|
| Idli | ₹15 |
| Dosa | ₹20 |
| Masala Dosa | ₹35 |
| Pongal | ₹18 |
| Vada | ₹12 |
| Sambar Rice | ₹40 |
| Curd Rice | ₹35 |
| Chapati | ₹15 |
| Parota | ₹20 |
| Fried Rice | ₹60 |
| Meals | ₹80 |

To update prices or add items, open the **Parse Intent & Items** node and edit the `MENU` object.

---

## 🐛 Known Issues

- **Masala dosa / dosa overlap** — ordering masala dosa may also count plain dosa separately. Fix pending in regex parser.

---

## 🗺️ Roadmap

- [ ] Fix masala dosa / dosa regex overlap
- [ ] Add Razorpay payment link generation
- [ ] Deploy n8n to VPS (DigitalOcean / Hetzner) — eliminate ngrok dependency
- [ ] Migrate to WhatsApp Cloud API (Meta)
- [ ] Owner notification on new order
- [ ] Order status updates

---

## 📁 Repo Structure

```
├── telegram_order_bot.json   # n8n workflow export
└── README.md
```

---

## 🙌 Built by

**Praneeth** — [praneeth.tech](https://praneeth.tech) | [GitHub](https://github.com/Praneethe358) | Vynkrova
