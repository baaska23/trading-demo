# Trading App — Requirements (reference: MYMOT by MOT Capital)

_Compiled 2026-10-02 from public sources (App Store, motcapital.com, broker review sites). Items marked **(unverified)** come from third-party review sites and may be outdated._

---

## 1. What MYMOT actually is

| | |
|---|---|
| Company | MOT Capital LLC (formerly Motforex / MOTFX), Minister Tower, Ulaanbaatar |
| Business | Forex & CFD broker (leveraged trading) — **not** a Mongolian Stock Exchange (MSE) stock app |
| App | MYMOT — iOS (`id6667097735`, iOS 15+, ~155 MB, 18+) and Android (`com.mot.mobile`) |
| Current version | 1.7.4 ("new bonus event, performance, bug fixes"); first release ~2024–2025 |
| Rating | 4.4★ on App Store |
| Trading engine | **MetaTrader 5** (and cTrader **(unverified)**) — MYMOT is a *client portal / wallet* around MT5 accounts, plus its own charts and trading UI |
| Other channels | Web trader at `my.motcapital.com`, MT5 desktop/mobile apps |
| Licence claim | Special authorization under Mongolian FRC (Санхүүгийн зохицуулах хороо) decision #265, licence #301/124. Third-party sites (WikiFX etc.) list it as unregulated |
| Risk disclosure | "84.01% of leveraged trading participants lose money" |

> **Key insight:** MOT doesn't run its own exchange. Trades are executed on an MT5 server (bought/licensed from MetaQuotes or a white-label provider). The app's real job is onboarding, KYC, money in/out, account management, and a nicer trading/chart UI on top of MT5.

---

## 2. Products / instruments
- **Forex** — major, minor, and exotic pairs
- **Metals & energy** — Gold (XAUUSD), Silver, WTI, Brent, Natural Gas
- **Indices** — NAS100, US500, US30, US2000, JPN225
- **Crypto CFDs** — BTC, ETH, SOL, BNB, LTC
- **Share CFDs**
- About 200+ instruments in total

## 3. Account types **(unverified, may have changed)**
| Account | Min deposit | Spread | Commission | Leverage |
|---|---|---|---|---|
| Standard | $25 | from 0.8 pip | none | up to 1:1000 |
| Pro | $25 | from 0.0 pip (EURUSD avg 0.1) | $3.50/side per lot | up to 1:1000 |
| Ultra+ | $100 | — | — | up to 1:3000 |
| Swap-Free / Zero Spread | $100 | — | — | — |
| Demo | free | virtual funds | — | — |

Users can hold **multiple trading accounts** (e.g. MT5 Pro + Swap-Free) and move money between them.

---

## 4. Functional requirements (feature list to replicate)

### 4.1 Onboarding & auth
- [ ] Register with phone/email + OTP
- [ ] Login, Face ID / fingerprint, PIN
- [ ] Forgot password
- [ ] 2FA for withdrawals and sensitive actions
- [ ] Language: Mongolian (primary) + English

### 4.2 KYC (identity verification)
- [ ] Upload ID (иргэний үнэмлэх / passport): front, back, and selfie (the app collects photos)
- [ ] Proof of address (optional tier)
- [ ] Verification status: pending, approved, rejected (with reason)
- [ ] Features locked until verified (deposit/withdraw/live account)
- [ ] Questionnaire: risk disclosure, trading experience, 18+ confirmation

### 4.3 Trading accounts
- [ ] Open a live or demo account and choose the type (Standard/Pro/…)
- [ ] Choose leverage and base currency (USD)
- [ ] Show MT5 login, server, and password, with password reset
- [ ] Account dashboard: balance, equity, margin, free margin, margin level, P/L

### 4.4 Wallet / funding
- [ ] **Deposit:** local bank (Khan, Golomt, TDB, State Bank…), **QPay / SocialPay** (expected for MN), Visa/Mastercard, crypto (USDT), SWIFT
- [ ] Automatic MNT → USD conversion shown with the rate
- [ ] **Withdraw:** to a verified bank account, crypto, or SWIFT. Status tracking (target: minutes to 24 h)
- [ ] **Internal transfer** between the user's own accounts (instant)
- [ ] Transaction history with filters
- [ ] Partner/referral code on deposit

### 4.5 Market & trading
- [ ] Watchlist with live bid/ask quotes and % change
- [ ] Instrument search, grouped by category
- [ ] Interactive charts: candles/line, timeframes M1–MN, indicators (MA, EMA, RSI, MACD, Bollinger…), drawing tools, landscape mode
- [ ] Order ticket: market buy/sell, pending (limit/stop), volume (lots), SL/TP, margin preview
- [ ] Open positions: live P/L, modify SL/TP, partial or full close
- [ ] Pending orders: modify or cancel
- [ ] Trade history and account deals
- [ ] Price alerts and push notifications (fills, margin call, stop-out)

### 4.6 Copy trading
- [ ] Strategy providers list: return %, drawdown, followers, risk score
- [ ] Follow/unfollow with allocation and risk settings
- [ ] Provider side: become a provider, set a performance fee

### 4.7 Partner / IB programme
- [ ] Referral link/code
- [ ] Referred-clients dashboard, volume, and commission earned
- [ ] Commission withdrawal

### 4.8 Promotions & loyalty
- [ ] Deposit bonus (e.g. 100%, up to $2,500), with terms and progress tracking
- [ ] VIP tiers: cashback, priority support
- [ ] Demo trading competitions ("Demo Masters")

### 4.9 Content
- [ ] Financial news feed
- [ ] Economic calendar
- [ ] AI-assisted market analysis (MOT advertises this)
- [ ] Education: lessons for beginners

### 4.10 Support & profile
- [ ] In-app chat or help centre, phone, email
- [ ] Profile, security settings, devices, notification preferences
- [ ] Legal docs: terms, risk disclosure, privacy, AML policy

---

## 5. Non-functional requirements
- **Latency:** quotes streamed by WebSocket at under 200 ms. Order round-trip in milliseconds (handled by MT5)
- **Security:** TLS, certificate pinning, encrypted local storage, jailbreak/root detection, session timeout, audit log
- **Reliability:** reconnect logic and offline state. Never show stale prices as live
- **Compliance:** AML/KYC per Mongolian law, data retention, 18+ only, risk warnings on every trading screen
- **Store policy:** Apple and Google require a licensed entity for real-money CFD/forex apps. The developer account must belong to the licensed company
- **Platforms:** iOS 15+, Android 8+, and possibly web (Flutter covers all three)

---

## 6. System architecture (how it would fit together)

```
Flutter app (iOS/Android/Web)
        │  REST + WebSocket
        ▼
Backend API (auth, KYC, wallet, accounts, copy trading, IB, CMS)
   ├── MT5 Manager / Web API  ← account creation, balance ops, positions, orders
   ├── MT5 price feed         ← streaming quotes to app
   ├── Payment gateways       ← QPay, bank APIs, card PSP, crypto
   ├── KYC provider           ← ID/selfie verification
   ├── Postgres + Redis
   └── Push (FCM/APNs), SMS/OTP, email
Back-office admin (KYC review, withdrawals approval, bonuses, reports)
```

**Build vs buy:**
| Component | Option |
|---|---|
| Trading server | MT5 white-label (MetaQuotes or a reseller), cTrader, or a ready-made demo/paper engine |
| Liquidity | Liquidity provider / prime-of-prime bridge (only for live trading) |
| Charts | TradingView Charting Library (free licence on application), or `syncfusion_flutter_charts` / `k_chart` / `candlesticks` |
| CRM / client portal | Off-the-shelf forex CRMs exist. A custom backend gives full control |

---

## 7. Legal / business prerequisites (for a real-money launch)
1. FRC licence for leveraged FX/CFD activity in Mongolia (or partner with a licensed broker as an IB/white-label)
2. A registered LLC, AML officer, and audited capital requirements
3. Agreements with an MT5 provider, a liquidity provider, and payment providers (QPay merchant, bank)
4. Apple/Google developer accounts in the company's name

---

## 8. Suggested MVP scope for this Flutter project
**Phase 1 (UI demo, mock data):** auth screens, watchlist, chart, order ticket, positions, wallet screens, profile. Mongolian/English localisation.
**Phase 2 (demo trading):** backend with simulated price feed (e.g. free FX/crypto feeds such as Binance WebSocket), paper-trading engine, real auth.
**Phase 3 (live):** MT5 integration, KYC, payments, compliance, back-office.

---

## 9. Open questions
- Goal: a learning/portfolio clone, a demo-trading app, or a real brokerage?
- Asset class: forex/CFD like MOT, or MSE stocks (another product category: Trade.mn, MN Stocks)?
- Do you or your company already have an FRC licence or a broker partner?
- Is there an existing backend/MT5 server, or do we mock everything?
- Design: copy MYMOT's look, or create your own brand?

## Sources
- App Store: https://apps.apple.com/mn/app/mymot/id6667097735
- Google Play: https://play.google.com/store/apps/details?id=com.mot.mobile
- MOT Capital: https://www.motcapital.com/ · https://www.motcapital.com/en/metatrader
- App metadata: https://mwm.ai/apps/mymot/6667097735
- Reviews: https://www.wikifx.com/en/dealer/1330199456.html · https://brokertribunal.com/en/brokers/mot-capital
