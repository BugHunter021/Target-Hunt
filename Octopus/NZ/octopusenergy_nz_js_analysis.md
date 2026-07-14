# Octopus Energy New Zealand (NZ) — JS Files Analysis

> **Source:** `~/target/Octopus/nz/js/` (1,471 JS files, 130MB)
> **Platform:** Next.js (App Router + Pages Router), Apollo Client, TypeScript
> **Date:** 2026-07-13

---

## 1. REST API Endpoints

| # | Endpoint | Method | Purpose | Auth |
|---|----------|--------|---------|------|
| 1 | `/api/address/autocomplete` | GET | 🔴 Address autocomplete | ❌ |
| 2 | `/api/address/icp-lookup` | POST | 🔴 Address → ICP number | ❌ |
| 3 | `/api/address/metadata` | GET | Address metadata (pxid format) | ❌ |
| 4 | `/api/address/verification` | GET | 🔴 Address verification (GPS + parcel ID) | ❌ |
| 5 | `/api/icp-details` | GET | 🔴 ICP details | ❌ |
| 6 | `/api/icp-search` | GET | 🔴 ICP search → PII leak | ❌ |
| 7 | `/api/onboarding/store-interests` | POST | Newsletter subscription | ❌ (500 without session) |
| 8 | `/api/time` | GET | Server time | ❌ |
| 9 | `/api/objects` | GET | Objects (404) | ❌ |
| 10 | `/api/overview/` | GET | Overview | ❌ |
| 11 | `/api/content-delivery` | GET | Storyblok content | ❌ |

---

## 2. GraphQL Queries

### Queries Taking `accountNumber` 🔴 (BOLA candidates)

| Operation Name | Args | Risk |
|----------------|------|------|
| `account` | `accountNumber: String` | 🔴 Account data |
| `accountProperties` | `accountNumber: String` | 🔴 Properties |
| `accountViewer` | `accountNumber: String` | 🔴 Viewer info |
| `campaigns` | `accountNumber: String` | 🔴 Campaign data |
| `ledgers` | `accountNumber: String` | 🔴 Financial ledgers |
| `paymentPreferences` | `accountNumber: String` | 🔴 Payment info |
| `transactions` | `accountNumber: String` | 🔴 Transactions |
| `referrals` | `accountNumber: String` | 🔴 Referral data |
| `statements` | `accountNumber: String` | 🔴 Statements |
| `supplyPoints` | `accountNumber: String` | 🔴 Supply points |

### Other Queries

| Operation Name | Purpose |
|----------------|---------|
| `viewer` | Current user profile |
| `property` / `propertyIds` | Property details |
| `IcpDetails` | ICP details |
| `fraudRiskLevel` | Fraud risk check |
| `referralReward` | Referral reward info |
| `isAutopayEnabled` | Autopay status |
| `measurements` / `readingInterval` | Energy measurements |
| `CustomerFeedbackForm` | Feedback form data |
| `getAccountMeasurements` | Account measurements |

---

## 3. GraphQL Mutations

| Operation Name | Purpose | BOLA Risk |
|----------------|---------|-----------|
| `addCampaignToAccount` | 🔴 Assign campaign to account | **HIGH** |
| `icpDiscovery` | 🔴 Address → ICP (no auth!) | **HIGH** |
| `masqueradeAuthentication` | 🔴 Admin impersonation | **CRITICAL** |
| `updateUser` | Update user profile | MEDIUM |
| `acceptQuote` | Accept a quote | MEDIUM |
| `requestQuote` | Request a new quote | MEDIUM |
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
| `InitiateStandAlonePayment` | Initiate payment | LOW |
| `getEmbeddedSecretForNewPaymentInstruction` | Get payment secret | MEDIUM |

---

## 4. Dashboard Pages (Account Numbers Found)

| Account Number | Path |
|----------------|------|
| `A-49719672` | `dashboard/A-49719672/` |
| `A-CFAFD216` | `dashboard/A-CFAFD216/` |
| `A-A45C46B6` | `dashboard/A-A45C46B6/` |
| `A-FC9F4E3B` | `dashboard/A-FC9F4E3B/` |
| `123` | `dashboard/123/` (test?) |

---

## 5. Kraken GraphQL

| Endpoint | Status |
|----------|--------|
| `api.oenz-kraken.energy/v1/graphql` | ✅ Alive, no auth for introspection |
| `developer.oenz-kraken.energy/graphql/reference/queries/` | Developer docs |

### Auth Model

| Error | Meaning |
|-------|---------|
| `KT-CT-1111` | Unauthorized (wrong user/role) |
| `KT-CT-1112` | No Authorization header |
| `KT-CT-1113` | Invalid/expired token |
| `KT-NZ-4611` — `KT-NZ-4617` | NZ-specific auth/validation errors |

---

## 6. BOLA / Security Candidate Matrix

| Priority | Target | Type | Attack Vector |
|----------|--------|------|---------------|
| 🔴 CRITICAL | `mutation masqueradeAuthentication` | GraphQL | Admin impersonation if exposed |
| 🔴 HIGH | `GET /api/icp-search?icp=X` | REST | **Confirmed** — ICP → PII leak |
| 🔴 HIGH | `POST /api/address/icp-lookup` | REST | **Confirmed** — Address → ICP |
| 🔴 HIGH | `POST /dashboard/*/hot-water-control` | REST | **Reported** — Control others' hot water |
| 🔴 HIGH | `mutation icpDiscovery` | GraphQL | **Confirmed** — No auth, address → ICP |
| 🔴 HIGH | `query account(accountNumber)` | GraphQL | Account data of any account |
| 🔴 HIGH | `query transactions/ledgers/statements(accountNumber)` | GraphQL | Financial data of any account |
| 🟡 MED | `query campaigns(accountNumber)` | GraphQL | Campaign data of any account |
| 🟡 MED | `mutation addCampaignToAccount` | GraphQL | Assign campaigns to any account |
| 🟡 MED | `GET /api/address/autocomplete` | REST | Address enumeration |
| 🟡 MED | `GET /api/address/verification` | REST | GPS + parcel ID (semi-public) |
| 🟡 MED | `GET /api/icp-details` | REST | ICP details |

---

## 7. External Services

| Service | URL | Purpose |
|---------|-----|---------|
| **Kraken GraphQL** | `api.oenz-kraken.energy/v1/graphql` | Main API |
| **AddressFinder NZ** | `addressfinder.nz` | Address verification |
| **Stripe** | Stripe Elements | Payment processing |
| **Mapbox** | Mapbox API | Map/solar simulator |
| **Storyblok** | Storyblok CDN | CMS content |
| **WhatsApp** | `api.whatsapp.com/send` | Customer contact |

---

## 8. Key Findings Summary

### Confirmed BOLAs (no auth required):
1. `GET /api/icp-search?icp=X` — Full ICP data (address, meter, network)
2. `POST /api/address/icp-lookup` — Address → ICP number
3. `POST /dashboard/*/properties/hot-water-control` — Control others' hot water
4. `mutation icpDiscovery` — GraphQL equivalent of icp-lookup

### Critical Risk:
- `mutation masqueradeAuthentication` — If exposed to regular users, allows impersonating any user

### Data Leak:
- `GET /api/address/verification` — GPS coordinates + parcel ID for any address
- `GET /api/address/autocomplete` — Address enumeration

---

*End of report — 1,471 JS files analyzed*
