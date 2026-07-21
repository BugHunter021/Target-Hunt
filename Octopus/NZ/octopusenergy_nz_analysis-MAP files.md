# Octopus Energy New Zealand (NZ) — Full Source Code Analysis

> **Method:** `sourcemapper` on 427 `.js.map` files
> **Source tree:** `~/target/Octopus/nz/map/sourcemaps/reconstructed/` (5,710 files, 64MB)
> **Date:** 2026-07-13

---

## 1. Reconstruction Summary

- **427 `.map` files** extracted → **5,710 source files** (64MB)
- 5 broken webpack runtime files (84KB total, negligible) excluded
- **App source at:** `webpack:/_N_E/src/` (Next.js App Router + Pages Router, TypeScript)
- Architecture: **GraphQL via Apollo Client** (different from other countries using raw GraphQL)

---

## 2. REST API Endpoints

| # | Endpoint | Method | Purpose | Check |
|---|----------|--------|---------|---------|
| 1 | `/api/address/autocomplete` | GET | 🔴 Address autocomplete — no auth | Checked ❌
| 2 | `/api/address/icp-lookup` | POST | 🔴 Address → ICP lookup — no auth | Checked ❌
| 3 | `/api/address/metadata` | GET | Address metadata (needs pxid format) | Checked ❌
| 4 | `/api/address/verification` | POST | Address verification | Checked ❌
| 5 | `/api/icp-details` | GET | 🔴 ICP details (confirmed BOLA) | Checked ❌
| 6 | `/api/icp-search` | GET | 🔴 ICP search by number — **PII leak** | Checked ❌
| 7 | `/api/nz/address/verification/` | POST | AddressFinder NZ verification | Checked ❌
| 8 | `/api/onboarding/store-interests` | POST | Store onboarding interests | Checked ❌
| 9 | `/api/payment_intents/create` | POST | Create Stripe payment intent | Not Check 🟡
| 10 | `/api/time` | GET | Server time | Checked ❌
| 11 | `/api/globals.html` | GET | External HTML globals | Not Check 🟡
| 12 | `/api/content-delivery/v2/links/retrieve-multiple-links` | GET | Storyblok links | Not Check 🟡

---

## 3. GraphQL Endpoint

| Endpoint | Backend | Status |
|----------|---------|--------|
| `/api/graphql/kraken` | `api.oenz-kraken.energy/v1/graphql` | ✅ Alive, no auth for introspection |
| Apollo Client | `NEXT_PUBLIC_KRAKEN_GRAPHQL_URI` | Client configured via env var |

**Developer Docs:** `https://developer.oenz-kraken.energy/graphql/reference/queries/`

---

## 4. GraphQL Queries

### Queries Taking `accountNumber` 🔴 (BOLA candidates)

| Operation Name | Purpose |
|----------------|---------|
| `account(accountNumber: String!)` | Account details |
| `accountProperties(accountNumber: String!)` | Account properties |
| `accountViewer(accountNumber: String!)` | Account viewer info |
| `campaigns(accountNumber: String!)` | Account campaigns |
| `ledgers(accountNumber: String!)` | Account ledgers |
| `paymentPreferences(accountNumber: String!)` | Payment preferences |
| `transactions(accountNumber: String!)` | Account transactions |
| `referrals(accountNumber: String!)` | Referral data |
| `statements(accountNumber: String!)` | Account statements |
| `measurements` | Energy measurements |
| `readingInterval` | Reading interval data |

### Other Queries

| Operation Name | Purpose |
|----------------|---------|
| `viewer` | Current user profile |
| `property` / `propertyIds` | Property details |
| `fraudRiskLevel` | Fraud risk for identifier |
| `IcpDetails` | ICP details (supply point) |
| `referralReward` | Referral reward info |
| `isAutopayEnabled` | Autopay check |
| `CustomerFeedbackForm` | Customer feedback form |

---

## 5. GraphQL Mutations

| Operation Name | Purpose | BOLA Risk |
|----------------|---------|-----------|
| `addCampaignToAccount` | 🔴 Assign campaign to any account | **HIGH** |
| `updateUser` | Update user profile | MEDIUM |
| `acceptQuote` | Accept a quote | MEDIUM |
| `requestQuote` | Request a new quote | MEDIUM |
| `icpDiscovery` | Discover ICP by address | MEDIUM |
| `setUpDirectDebitInstruction` | Set up direct debit | LOW |
| `storePaymentInstruction` | Store payment method | LOW |
| `setPaymentPreference` | Set payment preference | LOW |
| `verifyBankDetails` | Verify bank account | LOW |
| `requestUsageData` | Request usage data download | LOW |
| `UpdateBillingAddress` | Update billing address | MEDIUM |
| `UserLogin` | User login | LOW |
| `resetUserPassword` | Reset password | LOW |
| `requestPasswordReset` | Request password reset | LOW |
| `SubmitCustomerFeedback` | Submit feedback | LOW |
| `masqueradeAuthentication` | 🔴 **Masquerade login** (admin impersonation?) | **CRITICAL** |

---

## 6. BOLA Candidates (Confirmed + Potential)

| # | Endpoint / Query | Type | Auth | Status |
|---|-----------------|------|------|--------|
| 1 | `POST /dashboard/*/properties/hot-water-control` | REST | ❌ | ✅ **Reported** |
| 2 | `GET /api/icp-search?icp=XXX` | REST | ❌ | ✅ **PII leak** |
| 3 | `POST /api/address/icp-lookup` | REST | ❌ | ✅ **ICP discovery** |
| 4 | `GET /api/address/autocomplete` | REST | ❌ | 🟡 Address enumeraton |
| 5 | `query account(accountNumber)` | GraphQL | ✅ Bearer | 🟡 Needs auth test |
| 6 | `query campaigns(accountNumber)` | GraphQL | ✅ Bearer | 🟡 Needs auth test |
| 7 | `mutation addCampaignToAccount` | GraphQL | ✅ Bearer | 🟡 Needs auth test |
| 8 | `mutation masqueradeAuthentication` | GraphQL | admin | 🔴 **Critical** |

---

## 7. Pages with `[accountNumber]` in URL

**App Router (modern):** `/dashboard/[accountNumber]/`
- `page-content.tsx` — Dashboard overview
- `pay/page-content.tsx` — Payments
- `payments/page-content.tsx` — Payment methods
- `payments/choose-payment-method/page-content.tsx` — Choose payment
- `transactions/page-content.tsx` — Transactions
- `properties/[propertyId]/page-content.tsx` — Property detail
- `properties/[propertyId]/hot-water-control/page-content.tsx` — 🔴 **Hot water control**
- `components/Navigation.tsx` — Dashboard nav
- `components/SectionDevices.tsx` — Devices section

**Pages Router (legacy):** `/account/[accountNumber]/`
- `properties.tsx` — Properties list
- `properties/[propertyId].tsx` — Property detail
- `payments/add-new-card.tsx` — Add credit card
- `payments/add-new-direct-debit.tsx` — Add direct debit
- `transactions.tsx` — Transaction list

> Each page passes `accountNumber` from the URL to its GraphQL queries.

---

## 8. Environment Variables

### Client-Side (`NEXT_PUBLIC_*`)

| Variable | Risk | Notes |
|----------|------|-------|
| `NEXT_PUBLIC_KRAKEN_GRAPHQL_URI` | 🟡 | GraphQL endpoint URL |
| `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | 🟢 | Stripe publishable key |
| `NEXT_PUBLIC_MAPBOX_TOKEN` | 🟡 | Mapbox access token |
| `NEXT_PUBLIC_STORYBLOK_PUBLIC_ACCESS_TOKEN` | 🟢 | Storyblok CDN token |
| `NEXT_PUBLIC_MERCHANT_AUTH_CODE` | 🟡 | Direct debit auth code |
| `NEXT_PUBLIC_MERCHANT_BANK_REFERENCE` | 🟡 | Bank reference |
| `NEXT_PUBLIC_MERCHANT_COUNTRY_CODE` | 🟢 | Country code |
| `NEXT_PUBLIC_MERCHANT_NAME` | 🟢 | Merchant name |
| `NEXT_PUBLIC_GOOGLE_ANALYTICS_` | 🟢 | GA tracking |
| `NEXT_PUBLIC_GOOGLE_TAG_MANAGER_ID` | 🟢 | GTM ID |
| `NEXT_PUBLIC_GOOGLE_UNIVERSAL_ANALYTICS_ID` | 🟢 | UA ID |
| `NEXT_PUBLIC_FEATURE_*` | 🟢 | Feature flags (17 flags) |
| `NEXT_PUBLIC_ENVIRONMENT` / `NEXT_PUBLIC_ENV` | 🟢 | Environment |

---

## 9. Error Codes (NZ-specific)

| Code | Description |
|------|-------------|
| `KT-NZ-3910` | NZ-specific error |
| `KT-NZ-4410` | NZ-specific error |
| `KT-NZ-4502` | NZ-specific error |
| `KT-NZ-4611` — `KT-NZ-4617` | 🔴 **Auth/validation errors** (BOLA-related) |
| `KT-NZ-5800` — `KT-NZ-5815` | NZ-specific errors |
| `KT-NZ-6410` — `KT-NZ-6432` | NZ-specific errors |

Also includes generic Kraken codes `KT-CT-1111` (Unauthorized), `KT-CT-1112` (no auth header), `KT-CT-1113` (invalid token).

---

## 10. External Services / Domains

| Service | URL | Purpose |
|---------|-----|---------|
| **Kraken GraphQL** | `api.oenz-kraken.energy/v1/graphql` | Main GraphQL API |
| **Kraken Developer** | `developer.oenz-kraken.energy` | API documentation |
| **AddressFinder NZ** | `addressfinder.nz/api/nz/address/verification/` | Address verification |
| **Stripe** | Stripe Elements | Payment processing |
| **Mapbox** | Mapbox API | Map/solar simulator |
| **Storyblok** | Storyblok CDN | CMS content |
| **WhatsApp** | `api.whatsapp.com/send` | Customer contact |

---

## 11. BOLA Priority Matrix

| Priority | Target | Attack Vector |
|----------|--------|---------------|
| 🔴 CRITICAL | `mutation masqueradeAuthentication` | Admin impersonation if exposed to regular users |
| 🔴 HIGH | `GET /api/icp-search?icp=X` | **Confirmed** — ICP search → PII leak |
| 🔴 HIGH | `POST /api/address/icp-lookup` | **Confirmed** — Address → ICP mapping |
| 🔴 HIGH | `POST /dashboard/*/hot-water-control` | **Reported** — Already filed by user |
| 🔴 HIGH | `query account(accountNumber)` | Account data of any account |
| 🟡 MED | `query campaigns(accountNumber)` | Campaign data of any account |
| 🟡 MED | `mutation addCampaignToAccount` | Assign campaigns to any account |
| 🟡 MED | `GET /api/address/autocomplete` | Address enumeration |
| 🟡 MED | `query transactions / ledgers / statements(accountNumber)` | Financial data of any account |
| 🟡 MED | `NEXT_PUBLIC_MERCHANT_AUTH_CODE` + `_BANK_REFERENCE` | Direct debit config |

---

*End of report — 5,710 source files extracted from 427 .js.map files*
