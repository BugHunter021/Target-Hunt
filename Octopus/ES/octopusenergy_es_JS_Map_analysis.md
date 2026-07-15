# Octopus Energy Spain (ES) — Source Code Analysis Report

> **Method:** `sourcemapper` (github.com/denandz/sourcemapper) on 144 `.js.map` files
> **Source tree reconstructed:** `~/target/es/map/reconstructed/` (3752 files, 34MB)
> **App source at:** `webpack:/_N_E/src/` (Next.js + React + TypeScript)
> **Date:** 2026-07-11

---

## 1. Reconstruction Summary

- Tool: `sourcemapper -dir ./octopusenergy.es -output ./reconstructed`
- 144 `.map` files processed → **3752 source files** extracted (34MB)
- ⚠️ `sourcemapper` panicked on 1 file (`index out of range` bug in v2026) but already
  extracted 3752 files before crashing — coverage is essentially complete.
- App source located at: `webpack:/_N_E/src/`
  - `src/client`, `src/shared`, `src/pages`, `src/entities`, `src/features`, `src/legacy`,
    `src/lib`, `src/routes`, `src/constants`
- Packages: `packages/ab-testing`, `packages/utils`

---

## 2. REST API Endpoints

| # | Endpoint | Source File | Check |
|---|----------|-------------|-------------|
| 1 | `POST /api/auth/login` | `routes/api.ts` | `No need ❌` |
| 2 | `POST /api/auth/logout` | `routes/api.ts` | `No need ❌` |
| 3 | `GET  /api/auth/session` | `routes/api.ts` | `No need ❌` |
| 4 | `POST /api/battery-lead` | `routes/api.ts` | `y-zoha form ❌` |
| 5 | `POST /api/check-contract-sign-status` | `routes/api.ts` | `y-zoha form ❌` |
| 6 | `POST /api/check-sign-status` | `routes/api.ts` | `y-zoha form ❌` |
| 7 | `POST /api/comparator/calculate` | `routes/api.ts` | `Y-calc ❌` |
| 8 | `GET  /api/comparator/reference-rates` | `routes/api.ts` | `no need ❌` | 
| 9 | `POST /api/contract-embedded-url` *(real: `/api/contract-ln-url`)* | `routes/api.ts` |  `y-zoha form ❌` |
| 10 | `POST /api/create-energy-account` | `routes/api.ts` |  `Y-Valnerable ✅` |
| 11 | `POST /api/ev-charger-lead` | `routes/api.ts` | `Y-not important ❌` | 
| 12 | `POST /api/fraud-meter-point-checks` | `routes/api.ts` | `Y-not important ❌` | 
| 13 | `POST /api/graphql/kraken` | `routes/api.ts` | `need ⚠️` | 
| 14 | `POST /api/graphql/oe` | `routes/api.ts` | `need ⚠️` | 
| 15 | `POST /api/initiate-product-switch` | `routes/api.ts` | `Y-not exist ❌` | 
| 16 | `POST /api/onboarding/check` | `routes/api.ts` | `Y-not important - 1 month ♻️` | 
| 17 | `POST /api/onboarding/submit-form` | `routes/api.ts` | `No need ❌` |
| 18 | `POST /api/retail-lead` | `routes/api.ts` | `y-zoha form ❌` |
| 19 | `POST /api/solar-lead` | `routes/api.ts` | `y-zoha form ❌` |
| 20 | `POST /api/validation` | `routes/api.ts` | `Y-not important ❌` | 
| 21 | `GET  /api/queries/productsWithoutEstimates` | `entities/product` | `Y-not important ❌` | 
| 22 | `GET  /api/content-delivery/v2/stories/retrieve-a-single-story` | `lib/storyblok` | `No need ❌` |
| 23 | `POST /api/v1/requests` | `routes/external.ts` | `need ⚠️` | 

### Detailed Endpoint Descriptions

**Auth**
- `POST /api/auth/login` — User login (email/password) → returns token
- `POST /api/auth/logout` — Logout / clear session
- `GET  /api/auth/session` — Check current session status

**Onboarding / Account Creation**
- `POST /api/create-energy-account` — 🔴 Create new energy account (mass-assignment BOLA candidate, mirrors `.fr`)
- `POST /api/onboarding/check` — Validate onboarding step (address/CUPS validity)
- `POST /api/onboarding/submit-form` — 🔴 Submit onboarding form (store-interests-style BOLA candidate)

**Contracts / Signing**
- `POST /api/contract-embedded-url` *(real name: `/api/contract-ln-url`)* — 🔴 Get embedded Zoho Sign URL for contract
- `POST /api/check-contract-sign-status` — Check if contract is signed
- `POST /api/check-sign-status` — Check general sign status (e.g. Power Contract)

**Lead Capture**
- `POST /api/battery-lead` — Solar battery lead
- `POST /api/ev-charger-lead` — EV charger lead
- `POST /api/solar-lead` — Solar panel lead
- `POST /api/retail-lead` — Retail lead

**Tariff / Product Switch**
- `POST /api/initiate-product-switch` — Initiate product/tariff switch
- `POST /api/comparator/calculate` — Calculate tariff comparison
- `GET  /api/comparator/reference-rates` — Get reference rates for comparator

**Payment / Validation**
- `POST /api/validation` — Validate form fields (email, NIF, IBAN, etc.)
- `POST /api/fraud-meter-point-checks` — 🟡 Check supply point against fraud (PII/enumeration risk)

**GraphQL Proxies**
- `POST /api/graphql/kraken` — Kraken GraphQL proxy (`api.oees-kraken.energy`)
- `POST /api/graphql/oe` — OE-Backend GraphQL proxy

**Content / Pricing**
- `GET /api/queries/productsWithoutEstimates` — List products not needing consumption estimate
- `GET /api/content-delivery/v2/stories/retrieve-a-single-story` — Read a Storyblok CMS story *(out of scope)*

**External System**
- `POST /api/v1/requests` — 🟡 Proxy to Zoho Sign (`https://ln.zoho.eu/api/v1/requests`)

---

## 3. GraphQL Endpoints

- **Kraken (primary):** `/api/graphql/kraken` → backend `api.oees-kraken.energy`
- **OE backend:** `/api/graphql/oe`
- Kraken domains referenced in source:
  - `https://support.oees-kraken.energy/client-app/oe-backend-esp-finance/`
  - `https://support.oees-kraken.energy/referral-schemes`

---

## 4. GraphQL Queries (operation names from `src/`)

```
AccountCreditsQuery, AccountOverviewPage, AccountProperties, AccountReferrals,
AccountsListing, Agreement, AvailableProductSwitchDatesQuery, Bill, Bills,
BillsOverviewPage, CampaignBannerAccount, ChangePowerAccount, ChargePointVariants,
DomesticJoiningRewardScheme, ElectricVehicles, ExploreConsumptionAccount,
LinkedSupplyPointAccounts, LowerPower, MyTariffAccount, OnboardingReferralCurrentlyAllowed,
PaymentDetailsAccount, PersonalDetailsDashboardPage, ProductsWithoutEstimates,
ReferralProgramPage, RetailFinanceInstalments, TariffComparisonQuery,
TariffSwitchEligibility, UpdateCommunicationPreferencesPage, UpdateContactDetailsPage,
UpdatePaymentDetailsAccount, Viewer, ViewerAccount, ViewerProperty,
getAccountLedgers, getDevices, pvpcPrices
```

**BOLA candidates (queries taking `accountNumber`):**
`CampaignBannerAccount($accountNumber!)`, `AccountProperties`, `ViewerAccount`,
`ViewerProperty`, `AccountReferrals`, `PaymentDetailsAccount`,
`PersonalDetailsDashboardPage`, `UpdatePaymentDetailsAccount`, `Bills`, `Bill`,
`AccountCreditsQuery`, `AccountOverviewPage`, `MyTariffAccount`,
`ExploreConsumptionAccount`, `getAccountLedgers`, `LinkedSupplyPointAccounts`

---

## 5. GraphQL Mutations (operation names from `src/`)

```
RequestResetPassword, UpdateAccountUser, UpdateConsents, UpdatePassword,
UpdatePaymentDetails, uploadBillFile
```

> Full Kraken mutation list (`addCampaignToAccount`, `enrollment`, `createAccountNote`,
> etc.) is server-side — same schema as FR/IT/DE/NZ. Client only wires the above.

---

## 6. Campaign-Related Code

- `src/features/campaigns/components/CampaignBanner.tsx`
  - `CampaignBannerAccountQuery` takes `accountNumber: String!`
  - queries `account(accountNumber)` → referral level info
  - **BOLA pattern:** same as other countries — pass any `accountNumber`
- `src/features/campaigns/constants/campaigns.ts` — campaign constants/slugs
- `src/features/campaigns/hooks/useCampaignBannerValues.ts`
- `src/features/campaigns/hooks/useCampaignRewards.ts`
- Referral-schemes backend: `https://support.oees-kraken.energy/referral-schemes`
- `CampaignBanner` uses `useCurrentCampaign(code)` → reads campaign from URL/context

> No client-side `addCampaignToAccount` call found in ES source — but the Kraken
> mutation exists server-side. Needs server test.

---

## 7. Contract Embedded URL — Deep Dive

**Endpoint (real build name):** `POST /api/contract-ln-url`
*(alias in older source map: `contract-embedded-url`)*

**Context found:**
- External route: `ZOHO_SIGN_BASE_URL: "https://ln.zoho.eu/api/v1/requests"` (`routes/external.ts`)
- `ln` = "Luz y Naturaleza" (Octopus Spain brand)
- Components: `ContractItemTariff.tsx`, `ContractItem.tsx` (dashboard → My Tariff → Contract)
- When user clicks "Sign contract", frontend fetches an **embedded Zoho Sign URL**

**Likely request body:**
```json
{ "accountNumber": "A-XXXX", "contractId": "12345" }
```
**Likely response:**
```json
{ "url": "https://sign.zoho.eu/embed/..." }
```

**BOLA hypothesis:** if server doesn't verify the `contractId`/`accountNumber` belongs
to the authenticated user, an attacker could:
1. Obtain the signing URL for **another customer's contract**
2. Potentially open/sign that contract via the embedded Zoho link

**Live test results (no auth):**
| Endpoint | Body | Response | Result |
|----------|------|----------|--------|
| `POST /api/contract-ln-url` | `{accountNumber, contractId}` | `405 Method not allowed` + `FUNCTION_INVOCATION_FAILED` | function exists, POST rejected |
| `POST /api/contract-embedded-url` | `{accountNumber, contractId}` | `400 {"error":"Invalid price"}` | 🟡 ALIVE, parses body, no 401/403 |
| `GET /api/contract-ln-url` | query | HTML 404 page | — |

> `contract-embedded-url` responds **without 401/403** — suggests no auth gate, only
> validation. Requires authenticated session to capture the exact request shape.

---

## 8. NEXT_PUBLIC Env Vars (leaked in client bundle)

```
NEXT_PUBLIC_FEEDBACK_API_ENDPOINT
NEXT_PUBLIC_SOLAR_SIMULATOR_MAP
NEXT_PUBLIC_ALLOW_MUTATIONS_IN_DEV
NEXT_PUBLIC_DATADOG_CLIENT_TOKEN_OEES_CONSUMER_SITE
NEXT_PUBLIC_ENVIRONMENT
NEXT_PUBLIC_GA4_ID
NEXT_PUBLIC_GTM_ENABLED
NEXT_PUBLIC_GTM_ID
NEXT_PUBLIC_OEES_STRIPE_PUBLISHABLE_KEY
NEXT_PUBLIC_SENTRY_DSN
NEXT_PUBLIC_STORYBLOK_ACCESS_TOKEN   ← STORYBLOK TOKEN (out of scope per user)
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY
```

---

## 9. BOLA / Security Candidates (cross-country patterns)

| Priority | Endpoint / Query | Why |
|----------|------------------|-----|
| 🔴 HIGH | `POST /api/create-energy-account` | Mirrors FR create-account mass-assignment BOLA |
| 🔴 HIGH | `POST /api/onboarding/submit-form` | Mirrors FR/NZ store-interests pattern |
| 🔴 HIGH | `query CampaignBannerAccount(accountNumber)` | `accountNumber` in arg → IDOR/BOLA |
| 🔴 HIGH | `query ViewerAccount / ViewerProperty(accountNumber)` | `accountNumber` in arg |
| 🔴 HIGH | `POST /api/contract-embedded-url` (live, no 401) | Embedded sign URL fetch |
| 🟡 MED | `POST /api/fraud-meter-point-checks` | May take meter point → PII/enumeration |
| 🟡 MED | `POST /api/v1/requests` | Zoho proxy — if unauthenticated, dangerous |
| 🟡 MED | Kraken mutation `addCampaignToAccount` | Confirmed in other countries; test on ES |
| 🟢 LOW | `POST /api/validation` | Input validation, low impact |
| 🟢 LOW | `/api/comparator/*` | Public rate calculation |

---

## 10. Notes / Next Steps

- All endpoints live under Next.js (App + Pages router) — same stack as IT/DE/NZ.
- Kraken auth model expected: `KT-CT-1111` / `1112` / `4123` (same as other countries).
- To test: need valid ES login (email/password) to obtain Kraken JWT, then test
  `accountNumber` substitution on queries above.
- Source files available for deep review at:
  `~/target/es/map/reconstructed/webpack:/_N_E/src/`
- Specifically interesting files:
  - `src/routes/api.ts` — all REST paths
  - `src/features/campaigns/*` — campaign logic
  - `src/features/onboarding/*` — account creation flow
  - `src/legacy/components/pages/dashboard/My-tariff/ContractItemTariff/*` — contract signing
  - `src/entities/*/hooks/*.ts` — GraphQL query definitions

---

*End of report*
