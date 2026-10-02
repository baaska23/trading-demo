# Trading App — Phases (portfolio edition)

_Derived from [REQUIREMENTS.md](REQUIREMENTS.md) on 2026-10-02. Section numbers in brackets (e.g. **[4.5]**) point back to that document._

## Goal

A showcase project for a **Dart/Flutter developer application at MOT Capital**. It has to prove two things:
1. **I understand their business.** MYMOT is a client portal and wallet around MT5 trading accounts: multiple account types, leverage and margin, MNT deposits converted to USD, KYC gating, risk warnings **[1]**.
2. **I can build their mobile app.** That means clean architecture, real-time prices, correct trading maths, tests, MN/EN localisation, and polish.

It is **not** a product launch. Anything that only matters for a real launch (licensing, real payments, store release, back-office, compliance) is cut.

| Phase | What it proves | Status |
|---|---|---|
| **1. MVP — Core trading app** | "I can build a real-time trading app cleanly" | **Must have.** This alone is applicable. |
| **2. Middle — MOT business layer** | "I understand how *their* product works" | **Must have** for this company specifically |
| **3. Extras** | Range: full-stack Dart, security, social features | **Optional.** Pick 1–2 if time allows. |

Rough timebox: Phase 1 ≈ 2–3 weeks, Phase 2 ≈ 1–2 weeks, Phase 3 open-ended. **Apply after Phase 2.** A finished, polished smaller app beats an unfinished big one.

---

## 0. Ground rules

### 0.1 Decisions (already made)
- [x] Live prices from free APIs, simulated money only, no MT5, no real banking APIs
- [x] **No backend needed for Phases 1–2.** The app talks to free public feeds directly, and a local database stands in for the server. Phase 3 can add a Dart backend behind the same interfaces, which is the best demonstration that the architecture works.
- [ ] **Own brand name and look**, with something like "inspired by MYMOT" in the README. Don't use MOT's name, logo or screenshots inside the app.
- [ ] Clean up `pubspec.yaml`:
  - Choose **Riverpod or tanstack_query, not both** (recommendation: Riverpod)
  - Choose **one chart library for candles** (Syncfusion has candles and indicators built in; fl_chart is fine for sparklines)
  - **Remove** firebase_*, freerasp, google_mlkit_face_detection, webview_flutter and envied until a phase actually uses them
  - **Add** a local DB for the ledger and engine state: `drift` (SQLite) is the strongest Dart showcase

### 0.2 Architecture (this is what reviewers read)
Feature-first, with layers inside each feature:
```
lib/
  app/        → bootstrap, router, theme, DI
  core/       → network (dio, ws client), db, errors, l10n, design-system widgets
  features/<feature>/
    domain/       → entities, repository interfaces, use-cases (pure Dart)
    data/         → DTOs, data sources, repository impls (fake_* / local_* / api_*)
    presentation/ → screens, widgets, Riverpod controllers
```
- `presentation` imports `domain` only. `domain` is pure Dart.
- One `Result`/`Failure` type for errors. No exceptions reach the UI.
- **Money and prices are never `double`.** Use decimal or integer minor units, and give each instrument `digits`/`pipSize`. Reviewers at a broker will look for this.
- Quotes, positions and connection state are `Stream`s in the domain, so the UI doesn't know their source.

### 0.3 Process (visible in the repo)
- Git from day one, small PRs into `main`, Conventional Commits
- GitHub Actions: format check, `flutter analyze` with strict lints, tests, and a web build deployed to GitHub Pages
- Definition of Done per feature: layers respected, tests written, loading/empty/error/offline states handled, MN + EN strings, risk warning on trading screens

---

## Phase 1 — MVP: core trading app (must have)

### 1.1 Foundation
- [ ] Git, CI, strict lints, folder structure from 0.2
- [ ] `go_router` with an auth guard and a bottom-nav shell (Markets / Trade / Positions / Wallet / Profile)
- [ ] Design system: tokens, light + dark themes, price cell with an up/down flash, P/L text, skeleton loaders
- [ ] Localisation: **Mongolian primary**, English second **[4.1]**, with `intl` number/currency formats

### 1.2 Market data
- [ ] **Crypto:** Binance public WebSocket, real-time and no API key needed (BTC, ETH, SOL, BNB, LTC) **[2]**
- [ ] **FX / gold / indices:** free real-time FX feeds are rare and limited. Two honest options:
  - a free tier such as Finnhub or Twelve Data (check the current limits), or
  - a random walk anchored to a free daily reference rate (e.g. Frankfurter / ECB), **clearly labelled "Simulated"**
- [ ] Bid/ask built from the mid price plus the account type's spread
- [ ] WS client: heartbeat, auto-reconnect, resubscribe. **Stale prices greyed out and never shown as live [5].**

### 1.3 Features
| Area | Scope | Ref |
|---|---|---|
| Auth (local) | Register, OTP screen with a fixed demo code, login, PIN. Token in secure storage. | 4.1 |
| Markets | Categories, search, watchlist (persisted), live bid/ask/% change | 4.5 |
| Chart | Candles + line, timeframes M1–D1, MA/EMA/RSI, landscape mode. History from the Binance REST API. | 4.5 |
| Order ticket | Market buy/sell, lots, SL/TP, **margin + pip value preview** | 4.5 |
| Engine (local) | Fill at bid/ask, SL/TP triggers, live P/L | 4.5 |
| Positions | Live P/L, modify SL/TP, close | 4.5 |
| Account summary | Balance, equity, margin, free margin, margin level | 4.3 |
| Profile | Language and theme switch, risk disclosure text | 4.10 |

### 1.4 Tests (the strongest signal you can give)
- [ ] Unit tests for all trading maths: P/L, margin, margin level, pip value, SL/TP validation, including JPY, gold and crypto digit edge cases
- [ ] Controller tests with mocked repositories (`mocktail`)
- [ ] Widget tests for the order ticket and position close
- [ ] One integration test covering register → buy → close

### 1.5 Exit
- Runs on Android and iOS, plus a deployed web build
- Live prices tick smoothly and the full trade flow works in both languages
- CI is green

---

## Phase 2 — Middle: MOT business layer (must have for this application)

This is where you show you studied **their** product, not just "a trading app".

| Area | Scope | Why it impresses them | Ref |
|---|---|---|---|
| Account types | Standard / Pro / Ultra+ / Swap-Free with different spread, commission, leverage and minimum deposit. Open multiple accounts and switch between them. Demo vs live. | It's their core product model | 3, 4.3 |
| Margin rules | Margin call and stop-out levels, auto-close at stop-out, warning banner | Shows broker-level domain knowledge | 4.5 |
| Pending orders | Limit/stop orders, modify/cancel, partial close | Expected in any MT5-style UI | 4.5 |
| Wallet ledger | Simulated deposit/withdraw/transfer as **DB records** in a double-entry ledger (details below) | Their app is a wallet first | 4.4 |
| Deposit flow | Method labels (QPay, Khan, Golomt, card, USDT), **MNT input → USD at a shown rate**, simulated confirmation, status | MNT→USD conversion is MN-market-specific | 4.4 |
| Withdraw flow | To a saved payout record, funds held, status timeline, cancel while pending | Shows you think in state machines | 4.4 |
| KYC flow | ID front/back + selfie capture (`image_picker`), status pending/approved/rejected with a reason, **live account and withdrawal locked until approved**. Approval via a hidden debug toggle. | Gating is central to their onboarding | 4.2 |
| Onboarding | Risk disclosure, 18+ confirmation, experience questionnaire | Compliance awareness | 4.2 |
| Security | Biometric unlock (`local_auth`), PIN fallback, session timeout | Fintech baseline | 4.1, 5 |
| Risk warning | Banner on every trading screen | They are required to show one | 5 |
| Transaction history | Filter by type, status, account and date | | 4.4 |

### 2.1 Ledger requirements (keep these, they're what a fintech reviewer checks)
- Double-entry: every movement is entries that sum to zero (wallet, trading accounts, held funds, "simulated bank", fees)
- Entries are immutable. Corrections are reversing entries.
- Each transaction stores type, method label, amount + currency, original MNT amount + rate + rate time, status, and an idempotency key
- Status machines:
  - deposit: `pending → completed | rejected`
  - withdrawal: `requested (held) → completed | rejected (released) | cancelled`
  - transfer: atomic
- Trade P/L and commission settle into the ledger when a position closes
- A test proving balances always equal the sum of entries, including under rapid repeated taps

### 2.2 Presentation (this is the part that gets you the interview)
- [ ] **README:** what the app is, a short "How MYMOT's business works" section (MT5 portal, account types, margin, MNT→USD, KYC), an architecture diagram, key decisions and why, how to run it, and a test summary
- [ ] Screenshots + a 60–90 s demo video/GIF in MN and EN
- [ ] Downloadable APK + web link
- [ ] Short ADRs in `docs/adr/` (state management choice, money representation, ledger design, data-source swapping)
- [ ] Clean commit history with no giant "final" commits

### 2.3 Exit → apply
- Any reviewer can open the web link, register, deposit "MNT", open a Pro account, trade gold or BTC, hit SL/TP and withdraw, in under 2 minutes

---

## Phase 3 — Extras (optional, pick 1–2)

Ranked by value for a **Dart** developer role:

| Extra | Value | Notes |
|---|---|---|
| **Dart backend** (`dart_frog`, `shelf` or `serverpod`) holding auth, engine and ledger, with the app switching from `local_*` to `api_*` repositories | ★★★ | Proves full-stack Dart, and proves the clean architecture by swapping data sources without touching any screen |
| Push notifications (FCM) for fills, SL/TP hits and margin call | ★★ | Needs the backend, or local notifications only |
| Copy-trading screen: provider list, follow with allocation, mocked stats | ★★ | One of their marketed features **[4.6]** |
| Partner/IB dashboard: referral code, referred clients, commission from ledger entries | ★★ | Shows you read their whole business model **[4.7]** |
| Security showcase: `freerasp` root/jailbreak detection, certificate pinning | ★ | One short ADR explaining it is enough |
| Chart drawing tools, more indicators (MACD, Bollinger), MN timeframe | ★ | Polish |
| News feed / economic calendar | ★ | Low signal |

---

## Cut entirely (not needed for the goal)
- Real OTP/SMS provider, real KYC provider, ML face detection
- Real payments, MT5, liquidity, licensing **[7]**
- Back-office web app (a debug menu replaces it: approve KYC, confirm deposit, jump prices)
- Promotions/bonus, VIP tiers, demo competitions, AI analysis, support chat
- App Store / Play Store release, pen test, reconciliation jobs, compliance sign-off, audit logs
- Firebase Crashlytics and analytics

---

## Requirement → phase map

| Req | Phase 1 | Phase 2 | Phase 3 | Cut |
|---|---|---|---|---|
| 4.1 Auth | Local auth, OTP UI, PIN | Biometrics, session timeout | Real auth on backend | SMS OTP, 2FA |
| 4.2 KYC | — | Capture, status, gating, questionnaire | — | Real provider, PoA |
| 4.3 Accounts | Summary | Types, multiple accounts, demo/live | — | MT5 credentials |
| 4.4 Wallet | — | Ledger, deposit/withdraw/transfer, history | Server-side ledger | Real payments |
| 4.5 Trading | Watchlist, chart, market orders, positions | Pending orders, partial close, stop-out | Drawing tools, push | — |
| 4.6 Copy trading | — | — | Optional | — |
| 4.7 IB | — | — | Optional | — |
| 4.8 Promotions | — | — | — | ✔ |
| 4.9 Content | — | — | News/calendar optional | AI analysis |
| 4.10 Profile | Language, theme, risk text | — | — | Support chat |
| 5 Non-functional | Reconnect, stale prices, tests, i18n | Risk banners, secure storage | RASP, pinning | Audit, compliance |
