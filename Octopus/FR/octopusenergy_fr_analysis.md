# Octopus Energy France (FR) — JS File Analysis

> **Source:** `~/target/octo/fr` (874 JS files, 112MB)
> **Platform:** Astro (static pages) + Kraken GraphQL
> **Status:** ✅ Analysis complete

---

## 1. API Endpoints (REST) — 37 Endpoints

### Auth
| Endpoint | Method |
|----------|--------|
| `/api/auth/login` | POST |
| `/api/auth/logout` | POST |
| `/api/auth/session` | GET |
| `/api/auth/sso/[partnerSlug]/callback` | GET |
| `/api/auth/sso/[partnerSlug]/logout` | GET |
| `/api/auth/update-org-token` | POST |

### Account
| Endpoint | Method | BOLA |
|----------|--------|------|
| `/api/account/add-campaign-to-account` | POST | 🔴 **Confirmed** |
| `/api/account/create-account` | POST | 🟡 Potential |
| `/api/account/create-account-note` | POST | 🟡 Potential |
| `/api/account/create-metadata` | POST | 🔴 **Confirmed** |

### GraphQL
| Endpoint | Backend |
|----------|---------|
| `/api/graphql/kraken` | Kraken API |
| `/api/graphql/oe-backend` | OE Backend |

### Meelo (BNPL / Open Banking)
| Endpoint | Method |
|----------|--------|
| `/api/meelo/journeys/[journeyId]/generate-link` | POST |
| `/api/meelo/journeys/[journeyId]/last-task` | GET |
| `/api/meelo/webhook` | POST |

### Onboarding
| Endpoint | Method | BOLA |
|----------|--------|------|
| `/api/onboarding/check-meter-point-eligibility` | POST | 🟡 Potential |
| `/api/onboarding/collect-deposit` | POST | 🟡 Potential |
| `/api/onboarding/compute-energy-landscape` | POST | 🟡 Potential |
| `/api/onboarding/eligibility-check-quote` | POST | 🟡 Potential |
| `/api/onboarding/enrollment` | POST | 🟡 **High Risk** |
| `/api/onboarding/fraudster-detection` | POST | 🟡 Potential |
| `/api/onboarding/join-supplier-accept-terms-and-conditions` | POST | 🟡 Potential |
| `/api/onboarding/select-products` | POST | 🟡 Potential |
| `/api/onboarding/send-quote` | POST | 🟡 Potential |
| `/api/onboarding/update-user-details-deposit-type` | POST | 🔴 **Confirmed** |

### Payment
| Endpoint | Method |
|----------|--------|
| `/api/payment-instruction/generate-secret` | POST |

### Content / CMS
| Endpoint | Method |
|----------|--------|
| `/api/content-delivery` | GET |
| `/api/exit-preview` | GET |
| `/api/preview` | GET |
| `/api/revalidate` | POST |
| `/api/storyblok/exit-preview` | GET |
| `/api/storyblok/preview` | GET |
| `/api/storyblok/revalidate` | POST |

### Other
| Endpoint | Method |
|----------|--------|
| `/api/wizard/submit-form` | POST |
| `/api/invalidate-pre-signed-token` | POST |
| `/api/v2/` | - |
| `/api/vercel/flags` | GET |

---

## 2. GraphQL Mutations

| Mutation | Type |
|----------|------|
| `AddAccountCampaign` | 🔴 Campaign assignment |
| `AddUserToAccount` | 🔴 User management |
| `ChangeMonthlyPayment` | 💰 Payment |
| `EnrollEcoSession` | 🌿 Eco program |
| `ExtractDataFromInvoice` | 📄 Invoice OCR |
| `ExtractPdlFromInvoice` | 📄 PDL extraction |
| `InitiateMeeloOpenBanking` | 💳 BNPL |
| `InitiateProductSwitch` | 🔄 Tariff switch |
| `InvalidatePreSignedToken` | 🔐 Token |
| `ObtainKrakenToken` | 🔐 **Auth token** |
| `RedeemGiftCard` | 🎁 Gift card |
| `RequestPasswordReset` | 🔐 Password |
| `requestQuote` | 💰 Quote |
| `ResetPassword` | 🔐 Password |
| `SelfReading` | 📊 Meter reading |
| `SetUpDirectDebitInstruction` | 💰 Direct debit |
| `SpinWheelOfFortune` | 🎡 Gamification |
| `StorePaymentInstruction` | 💰 Payment |
| `UpdateAccountUserConsents` | ✅ Consents |
| `UpdatePassword` | 🔐 Password |
| `instigateLeaveSupplier` | 🚪 Leave supplier |
| `initiateStandalonePayment` | 💰 One-off payment |

---

## 3. GraphQL Queries

| Query | Purpose |
|-------|---------|
| `Viewer` | Current user profile |
| `BillList` | Account bills |
| `DashboardLayoutContainer` | Dashboard data |
| `FanClubEligibility` | Campaign eligibility |
| `GetConsumptionForecast` | Energy forecast |
| `MeterPointNumberByAdress` | 🔴 **Address → PDL** |
| `ReferralList` | Referral data |
| `LeaveSupplierProcesses` | Supplier exit |
| `isPasswordResetTokenValid` | Token validation |
| `paymentSchedule` | Payment schedule |

---

## 4. Confirmed BOLAs

| # | Endpoint | Details |
|---|----------|---------|
| 1 | `POST /api/onboarding/update-user-details-deposit-type` | Change deposit type for any account |
| 2 | `POST /api/account/create-metadata` | Create metadata for any account |
| 3 | `POST /api/account/add-campaign-to-account` | Assign any campaign to any account |

---

## 5. Error Codes

### KT-CT-* (Cross-team)
`10315`, `1111`, `1112`, `1120`, `1124`, `1129`, `1130`, `1134`, `1135`, `1138`, `1142`, `1143`, `1199`, `3948`, `7713`, `7718`, `7899`

### KT-FR-* (France-specific)
`3910`, `4113`, `4513`, `4514`, `4515`, `4516`, `4517`, `4518`, `4527`, `8926`

---

## 6. Partners Discovered

| Partner | URL | Service |
|---------|-----|---------|
| **Carrefour** | `carrefour.octopusenergy.fr` | White-label energy |
| **Darty** | `darty.octopusenergy.fr` | White-label energy |
| **Ford** | Ford credit partnership | EV-related |
| **Meelo** | BNPL / open banking | Payment service |

---

## 7. Environment Variables (Client-Side)

- `NEXT_PUBLIC_BASE_API_URL`
- `NEXT_PUBLIC_CARREFOUR_BASE_URL`
- `NEXT_PUBLIC_DARTY_BASE_URL`
- `NEXT_PUBLIC_DATADOG_APPLICATION_ID`
- `NEXT_PUBLIC_STORYBLOK_ACCESS_TOKEN_FR`
- `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`
- `NEXT_PUBLIC_DIDOMI_API_KEY`
- `NEXT_PUBLIC_VERCEL_ENV`
- `NEXT_PUBLIC_GIT_COMMIT_SHA`
- `NEXT_PUBLIC_PLAYWRIGHT_MODE`
- `NEXT_PUBLIC_FF_UNPUBLISHED_STORIES`

---

*Generated from 874 JS files — 37 API endpoints, 28 mutations, 33 queries*
