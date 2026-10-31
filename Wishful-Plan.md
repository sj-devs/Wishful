# Wishful – Project Plan

> Native mobile app in Swift (iOS first, Android later) for wish posts that other people can fulfill.
> Team: Stefan Matteo Kauflin + Joel Felix · Last updated: 2026-10-31

---

## 1. Requirements (Summary)

- **Account & profile:** Sign-up/login. The delivery address is stored **encrypted** and is never publicly visible. Only city/canton and country are public.
- **Wish posts:** Users enter a title (e.g. "PS5 controller") and a product link. The system automatically detects price, image and shop name.
- **Feeds:** "For You" (all/recommended posts) and "Friends" (only tagged friends/family), switchable like on TikTok.
- **Relationships:** Users can tag profiles as friends or family (mutually confirmed).
- **Gifting:** Other users buy a wish **without ever seeing the recipient's address**. Wishful orders the product automatically and has it shipped directly to the recipient.
- **Business model:** Commission paid on top by the gifter (see section 2).
- **Distribution:** Apple App Store first (native Swift app), Google Play Store later.

---

## 2. Core Decision: How Does Gifting Work?

### Options Overview

| Option | Flow | Pros | Cons |
|---|---|---|---|
| **A – Cash gift** | Gifter pays → money goes to the recipient, earmarked → recipient orders themselves | Technically simplest | Fraud risk (fake wishes), essentially "sending money", anti-money-laundering concerns (AMLA) |
| **B – Wishful orders manually** | Gifter pays Wishful → a team member places the order by hand | Real goods, no technical risk | Doesn't scale, lots of manual work |
| **C – Reveal address on purchase** | Recipient releases their address per purchase | Simple | Breaks the privacy promise → **not recommended** |
| **D – Automated ordering (dropshipping model)** (chosen) | Gifter pays in the app → **the system opens the product link, places the order and ships to the recipient's stored address** | Matches the idea exactly, address stays fully hidden, commission per order, no money transfer to private individuals (no AMLA issue) | Technically demanding; legally you are the buyer/reseller (returns, warranty, VAT) |

**Decision: Option D.** It is implemented **in stages**, because fully automated ordering from *any* shop is not realistic at the start.

### Option D in Detail

#### Flow
1. The gifter taps "Fulfill wish".
2. The system **re-checks the price live** via the link (it may have changed since posting) and calculates the commission.
3. The gifter sees the total (item + shipping + commission) and pays.
4. Only **after the payment is confirmed** does the system place the order with the shop. The delivery address is the recipient's encrypted address, decrypted only at that moment and server-side only.
5. The recipient gets a push notification and an email; the gifter gets an order confirmation.
6. If the order fails (sold out, significant price increase, shop doesn't ship to the region), the gifter **automatically receives a full refund**.

#### Commission Model

| Item price | Commission (paid by the gifter) |
|---|---|
| below CHF 100 | **CHF 5 flat** |
| CHF 100 and above | **5% of the item price** |

Examples:

| Item | Commission | Total for the gifter (excl. shipping) |
|---|---|---|
| CHF 25 | CHF 5 | CHF 30 |
| CHF 99 | CHF 5 | CHF 104 |
| CHF 100 | CHF 5 | CHF 105 |
| CHF 300 | CHF 15 | CHF 315 |

> At exactly CHF 100 both rules yield CHF 5, so there is no price jump at the threshold.
>
> **Note on payment provider fees:** Fees (roughly 1.3–3% + a fixed amount, depending on the provider) come out of your commission. On a CHF 25 item you keep roughly CHF 4 of the CHF 5. Calculate this for each provider and decide whether shipping costs are subject to commission (recommendation: commission on the item price only).

#### How Does "the System" Order? (Stages)

| Stage | Method | When |
|---|---|---|
| **1 – Concierge MVP** | The system prepares everything (link, quantity, variant, address, amount) and creates a job in the admin dashboard. **You click "Buy" in the shop yourselves**, then enter the tracking number. | Launch, first users. You learn the processes this way. |
| **2 – Shop APIs & partner programs** | Direct ordering via official interfaces (e.g. Shopify-based shops, partner/affiliate APIs of large retailers, universal checkout services like Rye; check availability in Switzerland) | Once order volume justifies it |
| **3 – Browser automation** | Headless browser (e.g. Playwright) orders from selected top shops | Selectively only. Fragile, often violates shop terms, captchas and bot protection |

#### What Option D Additionally Requires
- **Company payment method for shop orders** (e.g. corporate credit card or a virtual card per order). The gifter's money has always arrived before you order, so no pre-financing is needed.
- **Customer accounts with shops** (company account per shop) or guest checkout
- **Returns & warranty:** define a process (recipient contacts Wishful, you handle it with the shop)
- **VAT:** The full item price may count as your revenue, not just the commission. Swiss VAT registration is mandatory from **CHF 100,000 annual revenue**, which you could reach quickly. Clarify with an accountant whether you act as an **intermediary** (commission = revenue) or a **reseller**.
- **International:** Orders from shops outside Switzerland incur customs duties/import VAT. For the MVP, **Swiss shops only** or shops that ship to Switzerland.
- **Chargebacks:** Gifters reverse payments after the goods are ordered → fraud detection, limits for new accounts

---

## 3. Tech Stack (Recommendation)

| Area | Choice | Rationale |
|---|---|---|
| App (iOS) | **Swift + SwiftUI** (native), Xcode, Swift Package Manager | Best performance and Apple integration (Apple Pay, Keychain, push, Sign in with Apple). **Requires a Mac with Xcode.** |
| App (Android) | **Open** (see below) | Swift doesn't run directly on Android |
| Backend code | Supabase Edge Functions (TypeScript/Deno) or server-side Swift (Vapor) | Server only; the app stays Swift |
| Backend | **Supabase** (Postgres, Auth, Storage, Edge Functions), region **EU (Frankfurt) or Zurich** | Row Level Security, fast start, data stays in Europe |
| Captcha | **Cloudflare Turnstile** or **hCaptcha** | Natively supported by Supabase Auth, more privacy-friendly than Google reCAPTCHA |
| Link scraper | Separate, isolated service (Cloud Run / Fly.io) | Isolated from the backend due to SSRF risk (see section 6) |
| Order service | Separate service with a job queue (stage 1: admin dashboard) | Reliable, retryable, traceable orders |
| Payments | **Open**. Candidates: Stripe, Datatrans, Payrexx, Wallee, Adyen | Must support TWINT, cards, Apple Pay and Google Pay. No card data on your side. |
| Email | Transactional email service (e.g. Resend, Postmark, Brevo), EU region preferred | Automated notifications and order confirmations |
| Push | Apple Push Notification service (APNs), later Firebase Cloud Messaging for Android | – |
| Moderation | Reporting, blocking, admin dashboard | Required for user-generated content (Apple Guideline 1.2) |

### Swift and Android

Swift apps only run on Apple devices. There are three paths to Android:

| Path | Description | Effort |
|---|---|---|
| **1 – iOS first** (recommended) | Build and launch the iOS app in Swift first, Android only after a successful beta | Lowest, focus on one product |
| **2 – Skip** | Framework that transpiles Swift/SwiftUI code into a native Android app (Kotlin/Jetpack Compose). Single Swift codebase. | Medium. Still young, not all libraries supported; check current license and pricing. |
| **3 – Separate Android app** | Second app in Kotlin + Jetpack Compose, same backend | High, everything built and maintained twice |

**Recommendation:** Path 1. All business logic (prices, commission, orders, permissions) lives in the backend, not the app. That keeps a later Android app (path 2 or 3) very feasible.

**Prerequisite:** iOS development with Swift requires **macOS with Xcode**. Anyone working on the app code needs a Mac. Without one, the only options are a rented cloud Mac (e.g. MacinCloud) or cloud builds (Xcode Cloud, GitHub Actions with macOS runners), which is cumbersome for daily development.

> Note: Physical goods may be sold without Apple's In-App Purchase (and without the 30% fee) (Guideline 3.1.5). With option D an actual physical product is sold, so this fits well.

---

## 4. Phases

> Until phase 6 you test the app in the iOS Simulator and on your own iPhones. A **free Apple ID** in Xcode is enough for that (the app must be reinstalled every 7 days). A paid developer account is only required for TestFlight and the App Store.

### Phase 0 – Decisions & Setup (1–2 weeks) · Complexity: low
- [ ] Set minimum age (recommendation: **18+**)
- [ ] Clarify who has a Mac with Xcode (required for Swift development)
- [ ] GitHub repo, CI (GitHub Actions with macOS runner), Xcode project (SwiftUI, iOS 17+), Supabase project in the EU region
- [ ] Define code style: SwiftLint, SwiftFormat, architecture (e.g. MVVM with `@Observable`), tests with Swift Testing / XCTest
- [ ] Wireframes in Figma

### Phase 1 – Auth, Profile, Address (2–3 weeks) · Complexity: medium
- [ ] Login via email, Sign in with Apple (mandatory once Google login exists) and Google, **with captcha on sign-up, login and password reset** (Turnstile/hCaptcha, verified server-side in Supabase Auth)
- [ ] Profile: name, picture, bio, public location (canton/city and country, user-selectable)
- [ ] Address in a separate, encrypted table (see section 6)
- [ ] Account deletion and data export (required by the FADP and Apple)

### Phase 2 – Posts & Link Detection (3 weeks) · Complexity: high
- [ ] Create post (title, link, variant such as size or color, optional note) with preview
- [ ] Scraper tries in order: schema.org/JSON-LD `Product` → Open Graph tags → shop-specific parsers for top shops (Galaxus, Digitec, Zalando, ...)
- [ ] Price and image cached with timestamp, prices refreshed periodically
- [ ] Fallback: enter price and image manually if detection fails
- [ ] Check whether the shop is **supported** (ships to Switzerland, orderable). Unsupported shops are flagged on the post.
- [ ] Post status: open → paid → ordered → shipped → delivered

### Phase 3 – Feed & Friends (3 weeks) · Complexity: medium
- [ ] Friend and family requests with mutual confirmation
- [ ] Friends feed, chronological
- [ ] "For You" feed in the MVP with simple ranking (recency, region, price range). Real profiling later and with transparency (see section 7)
- [ ] Visibility per post: public or friends only

### Phase 4 – Gifting, Payment & Ordering (4–6 weeks) · Complexity: high
- [ ] **Evaluate and choose a payment provider** (still open whether Stripe or an alternative). Criteria: TWINT, fees, fraud protection, Swiss company yes/no, payouts, integration effort
- [ ] Checkout: re-check price live, calculate commission (below CHF 100 → CHF 5, from CHF 100 → 5%), display total transparently
- [ ] "You've received a gift!" notification via push **and an automated email to the recipient's registered email address**, plus order and payment confirmation email to the gifter. Optional anonymous gifting (then the gifter's name appears in no email).
- [ ] Order service stage 1 (concierge): after payment, job in the admin dashboard, record tracking number, status updates via push and email
- [ ] Automatic refund if the order fails
- [ ] Returns and warranty process, chargeback handling

### Phase 5 – Security, Moderation, Legal (2–3 weeks, partly in parallel) · Complexity: medium
- [ ] Reporting, blocking, admin tool, word filter
- [ ] **Found a company (GmbH / LLC)** (liability, business account, company card for orders)
- [ ] Privacy policy, terms of service (incl. commission, refunds, returns), legal notice (imprint), data protection impact assessment (DPIA)
- [ ] Clarify VAT with an accountant (intermediary vs. reseller)
- [ ] External security review / penetration test before launch

### Phase 6 – Store Account, Beta & iOS Launch (3–4 weeks)
- [ ] **Apple Developer account (USD 99/year)**, ideally under the **company**. A company account requires a D-U-N-S number (free, processing can take a few days).
- [ ] Set up signing, provisioning and push certificates (APNs)
- [ ] TestFlight beta (internal and external)
- [ ] App Store listing, age rating, App Review

### Phase 7 – Android (after a successful iOS launch)
- [ ] Decide: Skip (Swift → Android) or a separate Kotlin app
- [ ] **Google Play account (USD 25 one-time)**. **Note:** New personal Play accounts need **at least 12 testers for 14 days** in closed testing before they can publish (does not apply to organization accounts).
- [ ] Build the Android app, push via Firebase Cloud Messaging, Google Pay
- [ ] Play Store listing and review

### Later – Expansion
- [ ] Order service stage 2 (shop APIs, partner programs)
- [ ] Order service stage 3 (browser automation for top shops)
- [ ] International shops (customs, import VAT)

**Estimated total duration:** approx. 5–7 months alongside school or work.

---

## 5. Division of Work

*Proposal, adjust to your strengths.*

| Area | **Stefan** (App & Product) | **Joel** (Backend & Security) |
|---|---|---|
| Phase 0 | Wireframes, design system, Xcode project, SwiftLint | Supabase, CI, repo rules, secrets management |
| Phase 1 | Login, profile and onboarding screens, captcha widget | Auth configuration, captcha verification, DB schema, RLS, address encryption |
| Phase 2 | Create post, link preview, post detail view | Scraper service, shop parsers, price refresh job, shop support check |
| Phase 3 | Feed UI with tab switch, friends list | Feed queries, ranking, friendship logic |
| Phase 4 | Checkout flow, commission display, status views, email templates | Payment provider integration, webhooks, order service, admin dashboard, email delivery |
| Phase 5 | Report and block UI, terms of service copy | Admin tool, rate limiting, logging, pentest fixes |
| Phase 6 | App Store listing, screenshots, organize TestFlight testers | Apple account, signing, APNs, CI builds |
| Phase 7 | Android UI (Skip or Kotlin, depending on decision) | Firebase Cloud Messaging, Play account, backend adjustments |
| Legal | Draft privacy policy and terms of service | Record of processing activities, DPIA, data processing agreements (DPAs) |

**Shared:**
- **Payment provider evaluation** (phase 4, step 1)
- Handling concierge orders in operation (stage 1)
- Company formation and VAT clarification
- Every pull request is reviewed by the other person (branch protection on `main`)
- Weekly planning (e.g. GitHub Projects)
- One person is the official **data protection contact** (proposal: Joel)

---

## 6. Security Measures

### Addresses (highest protection level)
- Separate `addresses` table with no direct read access from the app. Even the owner only sees the address masked after saving.
- **Field-level encryption** (pgsodium/Supabase Vault or KMS), so a DB dump alone is not enough
- Decryption **only by the order service at the moment of ordering**, and **every access is recorded in an audit log**. In concierge mode (stage 1) only the team sees the address, never the gifter.
- Users choose their public location themselves. For small villages, show only the canton, since the village could otherwise identify the person.

### Payment Data
- **No card data on your side.** Everything runs through the hosted payment forms/SDKs of the chosen provider (minimal PCI scope, SAQ-A).
- Verify payment provider webhooks by signature, process payments idempotently to prevent double charges
- **Place orders only after confirmed payment** (webhook), never based solely on a message from the app
- Company card for shop orders: set limits, use virtual cards per order where possible, credentials only in a secret store

### Order Service
- Shop logins and card data only in a secret store (e.g. Supabase Vault), never in code
- Every order with a unique ID, state machine and audit log
- Plausibility check before each order: Is the price the same as at checkout? Is the address in Switzerland?

### Login & Accounts
- **Captcha** on sign-up, login and password reset (protection against bots, fake accounts and credential stuffing)
- Optional 2FA or passkeys; store sessions and tokens securely in the **iOS Keychain**; keep App Transport Security enabled (HTTPS only)
- Rate limiting (login, post creation, scraper, checkout)

### Link Scraper (often underestimated)
- **SSRF protection:** block internal IPs (127.x, 10.x, 172.16–31.x, 192.168.x, 169.254.x, IPv6 equivalents), checked **after** DNS resolution
- `https` only, timeout, maximum response size, redirect limit, no JavaScript execution
- Runs isolated, without access to DB secrets
- Re-encode images server-side (strips EXIF data, prevents malware)
- Check links against phishing and malware lists (e.g. Google Safe Browsing)

### Emails
- Set up SPF, DKIM and DMARC for the sender domain (so emails don't land in spam and can't be spoofed)
- No addresses or sensitive data in emails, only status and a link into the app

### General
- **Row Level Security** on all tables, with dedicated tests
- Never commit secrets; Dependabot for Swift packages and backend dependencies; no API secrets in the app (only the public Supabase key, everything else server-side)
- Backups and an incident response plan for data breaches (who does what?)
- Abuse prevention: minimum age, reporting, **no direct messages** between strangers (for now), limits for new accounts

---

## 7. Swiss Data Protection Law (revFADP, in force since 2023-09-01)

| Obligation | Article | Implementation in Wishful |
|---|---|---|
| Principles: proportionality, purpose limitation, transparency | Art. 6 | Collect only necessary data, use the address exclusively for delivery |
| **Privacy by design and by default** | Art. 7 | Privacy-friendly defaults, coarse location, encrypted address |
| Data security | Art. 8 | Measures from section 6 |
| Processors | Art. 9 | Data processing agreements (DPAs) with Supabase, payment provider, email service, captcha provider, hosting, push service |
| Record of processing activities | Art. 12 | Only mandatory for companies with < 250 employees if high risk; **keep one anyway** |
| Cross-border disclosure | Art. 16 | EU hosting is unproblematic; for US services check certification under the Swiss-US Data Privacy Framework |
| **Duty to inform / privacy policy** | Art. 19 | Linked in the app and store: which data, purpose, recipients (**including: address is passed to the respective shop for delivery**), countries, rights |
| Data protection impact assessment (DPIA) | Art. 22 | **Recommended** due to addresses, payments and feed profiling |
| Reporting data breaches to the FDPIC | Art. 24 | "As soon as possible" if high risk; define the process in advance |
| Right of access | Art. 25 | Data export in the app, response within 30 days |
| Data portability | Art. 28 | Export as JSON |
| Deletion | Art. 32 | Account deletion with actual deletion (backups with retention period; order and accounting records must be **retained for 10 years**, Swiss Code of Obligations Art. 958f) |

> **Important:** The revFADP fines **individuals** (i.e. you personally, not the company) up to **CHF 250,000**. Take these obligations seriously and **found a company**.

### Further Legal Points
- **GDPR:** Applies additionally once you target users in the EU. You may then need an EU representative (Art. 27 GDPR).
- **Legal notice and terms of service:** Mandatory for online offerings (Swiss Unfair Competition Act, Art. 3 para. 1 lit. s). The commission must be **clearly shown before payment** (Swiss Price Disclosure Ordinance).
- **Shop terms:** Many shops prohibit automated orders (bots) or resale. Check per shop before stage 3; prefer partnerships.
- **VAT:** Registration mandatory from CHF 100,000 revenue. Whether the item price or only the commission counts depends on the model (intermediary vs. reseller); clarify with an accountant.
- **Anti-Money Laundering Act (AMLA):** With option D the gifter pays for a product and does not send money to private individuals, which greatly reduces this issue. Still clarify with the payment provider.
- **Affiliate links:** If used, label them as advertising.
- Budget for brief **legal advice** on the DPIA, terms of service and VAT (startup programs, e.g. via universities or incubators)
- **International expansion:** Each new market brings its own data protection, consumer protection and tax rules; review before launching there.

---

## 8. Risks

| Level | Risk | Mitigation |
|---|---|---|
| **HIGH** | **Minors receiving gifts from strangers** (grooming risk) | 18+ in the MVP, no direct messages, reporting, moderation |
| **HIGH** | **Automated ordering fails** (bot protection, shop changes, sold out) | Staged rollout (concierge first), automatic refunds, shop allowlist |
| **HIGH** | Fraud (stolen cards, chargebacks after ordering) | Payment provider fraud detection, 3-D Secure, limits for new accounts |
| **HIGH** | Price changes between posting and purchase | Live price check at checkout, cancel and refund on deviation |
| **MEDIUM** | Commission doesn't cover costs (fees, returns) | Calculate per payment provider, commission on item price only, review model after beta |
| **MEDIUM** | VAT obligation earlier than expected | Clarify early with an accountant |
| **MEDIUM** | App Store rejection (moderation, payment flow) | Review Guidelines 1.2, 3.1.5 and 5.1.1 before building |
| **MEDIUM** | Address data breach | Encryption, audit log, pentest |
| **LOW** | Copyright on product images | Use as preview with source link only, no permanent reuse |

---

## 9. Open Questions

1. **Which payment provider?** Stripe, Datatrans, Payrexx, Wallee or Adyen (comparison in phase 4)
2. Minimum age 18+, agreed?
3. Who can do what? (Experience with Swift/SwiftUI or backend?) → adjust division of work
4. **Who has a Mac with Xcode?** Swift development is barely possible without one.
5. **Android:** iOS first (recommended), Skip, or a separate Kotlin app?
6. Start with Swiss shops only, agreed?
7. Who pays shipping? (Proposal: the gifter, without commission on it)
8. What happens with returns, or if the recipient doesn't want the gift?
9. Do we legally act as an **intermediary** or a **reseller**? (Impact on VAT and liability)
