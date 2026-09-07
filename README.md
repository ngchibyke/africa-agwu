# BookBuy — Buy Anything. Go Anywhere.

![Status](https://img.shields.io/badge/status-active-brightgreen)
![Built with](https://img.shields.io/badge/built%20with-HTML5%20%7C%20CSS3%20%7C%20JS-orange)
![Payments](https://img.shields.io/badge/payments-Paystack-0BA4DB)
![Data Protection](https://img.shields.io/badge/NDPA%20Act-Observing-2e7d32)
![License](https://img.shields.io/badge/license-Proprietary-red)

**A product of NGCHIBYKE LTD**

> The world's all-in-one e-commerce, travel booking, and transportation super app — powered by AI.

---

## About

BookBuy is a full-stack multi-vendor marketplace and travel super app built for a global audience, with a strong focus on Africa and the Nigerian market. It combines:

- 🛒 Amazon-style global marketplace
- ✈️ Skyscanner-style flight search with price calendar & alerts
- 🏨 Booking.com-style hotel reservations with loyalty rewards
- 🚗 Uber-style transport booking
- 🤖 AI-powered shopping and travel assistant

---

## Pages

All paths below are relative to `frontend/`.

| File | Page |
|------|------|
| `index.html` | Landing Page — full navigation, Explore Everywhere, Price Calendar, BookBuy Gold |
| `product.html` | Product Detail — gallery, variants, reviews |
| `checkout.html` | Checkout — Paystack, 7 payment methods |
| `dashboard.html` | User Dashboard — orders, bookings, tracking, wallet |
| `flights.html` | Flight Search — filters, calendar view, price alerts |
| `hotels.html` | Hotel Search — filters, ratings, booking modal |
| `transport.html` | Transport Booking — taxi, bus, train, ferry, airport transfer |
| `vendor.html` | Vendor Dashboard — products, orders, analytics, payouts |
| `admin.html` | Admin Panel — users, vendors, disputes, commissions |
| `ai-assistant.html` | AI Chat Assistant — trip planning, product search |
| `help.html` | Help Centre — FAQ, contact support |
| `terms.html` | Terms & Conditions |
| `privacy.html` | Privacy Policy |
| `cookies.html` | Cookie Policy + live preference toggles |

---

## Company Information

**NGCHIBYKE LTD**
- **CEO / Founder:** Ngwobia Ofuche
- **Address:** #5 Bike Close, off Sakono Street, opposite AP Plaza, Wuse 2, Abuja FCT, Nigeria
- **Email:** ngchibykefotos@gmail.com
- **Phone:** +234 803 534 6630 | +234 805 267 5004
- **Website:** BookBuy

---

## Tech Stack

- **Frontend:** HTML5, CSS3 (custom, no framework), Vanilla JavaScript
- **Payments:** Paystack (primary), Flutterwave, PayPal, Apple Pay, Google Pay
- **Shipping:** DHL, FedEx, UPS integration
- **AI:** BookBuy AI Assistant (Claude-powered backend)
- **Hosting:** Ready for Netlify / Vercel / AWS deployment

---

## Features vs Competitors

| Feature | Booking.com | Skyscanner | BookBuy |
|---------|------------|------------|---------|
| Hotel booking | ✅ | ❌ | ✅ |
| Flight search | ❌ | ✅ | ✅ |
| Price calendar | ❌ | ✅ | ✅ |
| Explore Everywhere | ❌ | ✅ | ✅ |
| Price alerts | ✅ | ✅ | ✅ |
| Loyalty rewards | ✅ (Genius) | ❌ | ✅ (BookBuy Gold) |
| Global marketplace | ❌ | ❌ | ✅ |
| Transport booking | ❌ | ❌ | ✅ |
| AI trip planner | ❌ | ❌ | ✅ |
| Paystack payments | ❌ | ❌ | ✅ |
| African market focus | Partial | Partial | ✅ |

---

## Getting Started

The static site lives in `frontend/`. To run it locally:

```bash
# Option 1 — Open directly
open frontend/index.html

# Option 2 — Use VS Code Live Server extension
# Right-click frontend/index.html → Open with Live Server

# Option 3 — Python local server
cd frontend
python3 -m http.server 3000
# Then visit http://localhost:3000
```

The `backend/` API (auth, cart, orders, payments) runs separately — see `backend/README.md`.

---

## Deployment

### Netlify (Recommended — free)
1. Go to [netlify.com](https://netlify.com)
2. Drag and drop the `frontend` folder onto the Netlify dashboard
3. Your site is live instantly at a `.netlify.app` URL
4. Add your custom domain (e.g. bookbuy.ng) in Site Settings → Domain Management

### Vercel
```bash
npm i -g vercel
vercel --prod frontend
```

### GitHub Pages
GitHub Pages only serves from the repo root or `/docs`, not `/frontend`, so either:
- Publish `frontend/` as its own repo/branch, or
- Use a GitHub Action (e.g. `actions/deploy-pages`) that uploads `frontend/` as the Pages artifact.

---

## Push to GitHub (Step by Step)

```bash
# 1. Clone or navigate to this folder
cd bookbuy

# 2. The repo is already initialized. Add your GitHub remote:
git remote add origin https://github.com/YOUR_USERNAME/bookbuy.git

# 3. Stage all files
git add .

# 4. Make your first commit
git commit -m "Initial commit: BookBuy v1.0 — Full 14-page super app by NGCHIBYKE LTD"

# 5. Push to GitHub
git push -u origin main
```

---

## License

© 2025 NGCHIBYKE LTD. All rights reserved.
BookBuy is a registered trademark of NGCHIBYKE LTD.
Unauthorized reproduction or distribution is prohibited.
