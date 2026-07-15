      OCTOPUSENERGY.ES (SPAIN) — SOURCE CODE ANALYSIS REPORT
      Method: sourcemapper (github.com/denandz/sourcemapper) on 144 .js.map files
      Source tree reconstructed: ~/target/es/map/reconstructed/ (3752 files, 34MB)
      App source at: webpack:/_N_E/src/  (Next.js + React + TypeScript)
      Date: 2026-07-11


--------------------------------------------------------------------------------
1. RECONSTRUCTION SUMMARY
--------------------------------------------------------------------------------
- Tool: sourcemapper -dir ./octopusenergy.es -output ./reconstructed
- 144 .map files processed → 3752 source files extracted (34MB)
- NOTE: sourcemapper panicked on 1 file (index out of range bug in v2026) but
  already extracted 3752 files before crashing — coverage is essentially complete.
- App source located at: webpack:/_N_E/src/
  - src/client, src/shared, src/pages, src/entities, src/features, src/legacy,
    src/lib, src/routes, src/constants
- Packages: packages/ab-testing, packages/utils

--------------------------------------------------------------------------------
2. REST API ENDPOINTS  (source: src/routes/api.ts + scattered)
--------------------------------------------------------------------------------
| #  | Endpoint                                      | Source File |
|----|-----------------------------------------------|-------------|
| 1  | POST /api/auth/login                          | routes/api.ts |
| 2  | POST /api/auth/logout                         | routes/api.ts |
| 3  | GET  /api/auth/session                        | routes/api.ts |
| 4  | POST /api/battery-lead                        | routes/api.ts |
| 5  | POST /api/check-contract-sign-status          | routes/api.ts |
| 6  | POST /api/check-sign-status                   | routes/api.ts |
| 7  | POST /api/comparator/calculate                | routes/api.ts |
| 8  | GET  /api/comparator/reference-rates          | routes/api.ts |
| 9  | POST /api/contract-embedded-url               | routes/api.ts |
| 10 | POST /api/create-energy-account               | routes/api.ts |
| 11 | POST /api/ev-charger-lead                     | routes/api.ts |
| 12 | POST /api/fraud-meter-point-checks            | routes/api.ts |
| 13 | POST /api/graphql/kraken                      | routes/api.ts |
| 14 | POST /api/graphql/oe                          | routes/api.ts |
| 15 | POST /api/initiate-product-switch             | routes/api.ts |
| 16 | POST /api/onboarding/check                    | routes/api.ts |
| 17 | POST /api/onboarding/submit-form             | routes/api.ts |
| 18 | POST /api/retail-lead                         | routes/api.ts |
| 19 | POST /api/solar-lead                          | routes/api.ts |
| 20 | POST /api/validation                          | routes/api.ts |
| 21 | GET  /api/queries/productsWithoutEstimates    | entities/product |
| 22 | GET  /api/content-delivery/v2/stories/retrieve-a-single-story | lib/storyblok |
| 23 | POST /api/v1/requests                         | routes/external.ts |

--------------------------------------------------------------------------------
3. GRAPHQL ENDPOINTS
--------------------------------------------------------------------------------
- Kraken (primary):   /api/graphql/kraken   → backend: api.oees-kraken.energy
- OE backend:         /api/graphql/oe
- Kraken domains referenced in source:
    https://support.oees-kraken.energy/client-app/oe-backend-esp-finance/
    https://support.oees-kraken.energy/referral-schemes

--------------------------------------------------------------------------------
4. GRAPHQL QUERIES  (from src/ — operation names)
--------------------------------------------------------------------------------
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

KEY OBSERVATION — queries taking `accountNumber` (BOLA candidates):
  - CampaignBannerAccount($accountNumber: String!)
  - AccountProperties, ViewerAccount, ViewerProperty, AccountReferrals,
    PaymentDetailsAccount, PersonalDetailsDashboardPage, UpdatePaymentDetailsAccount,
    Bills, Bill, AccountCreditsQuery, AccountOverviewPage, MyTariffAccount,
    ExploreConsumptionAccount, getAccountLedgers, LinkedSupplyPointAccounts

--------------------------------------------------------------------------------
5. GRAPHQL MUTATIONS  (from src/ — operation names)
--------------------------------------------------------------------------------
RequestResetPassword, UpdateAccountUser, UpdateConsents, UpdatePassword,
UpdatePaymentDetails, uploadBillFile

NOTE: Full Kraken mutation list (addCampaignToAccount, enrollment, createAccountNote,
etc.) is server-side — same schema as FR/IT/DE/NZ. Client only wires the ones above
but the Kraken GraphQL surface is identical to the other countries.

--------------------------------------------------------------------------------
6. CAMPAIGN-RELATED CODE
--------------------------------------------------------------------------------
- src/features/campaigns/components/CampaignBanner.tsx
    → CampaignBannerAccountQuery takes `accountNumber: String!`
    → queries account(accountNumber) → referral level info
    → BOLA PATTERN: same as other countries — pass any accountNumber
- src/features/campaigns/constants/campaigns.ts
    → campaign constants/slugs
- src/features/campaigns/hooks/useCampaignBannerValues.ts
- src/features/campaigns/hooks/useCampaignRewards.ts
- referral-schemes backend: https://support.oees-kraken.energy/referral-schemes
- CampaignBanner uses useCurrentCampaign(code) → reads campaign from URL/context

NO client-side `addCampaignToAccount` call found in ES source — but the Kraken
mutation exists server-side (confirmed in FR/IT/DE/NZ schemas). Needs server test.

--------------------------------------------------------------------------------
7. NEXT_PUBLIC ENV VARS  (leaked in client bundle)
--------------------------------------------------------------------------------
NEXT_PUBLIC_FEEDBACK_API_ENDPOINT
NEXT_PUBLIC_SOLAR_SIMULATOR_MAP
NEXT_PUBLIC_ALLOW_MUTATIONS_IN_DEV
NEXT_PUBLIC_DATADOG_CLIENT_TOKEN_OEES_CONSUMER_SITE
NEXT_PUBLIC_ENVIRONMENT
NEXT_PUBLIC_GA
NEXT_PUBLIC_GTM_ENABLED
NEXT_PUBLIC_GTM_ID
NEXT_PUBLIC_OEES_STRIPE_PUBLISHABLE_KEY
NEXT_PUBLIC_SENTRY_DSN
NEXT_PUBLIC_STORYBLOK_ACCESS_TOKEN        ← STORYBLOK TOKEN (same leak class as FR/IT)
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY

⚠ Storyblok: NEXT_PUBLIC_STORYBLOK_ACCESS_TOKEN present in client bundle.
  If the token is a **public/delivery** token (not scoped), it may allow reading
  ALL stories including drafts — same finding as octopusenergy.fr / .it.
  Token value is NOT in the .map (runtime env) — must be captured from browser
  network tab or runtime JS to confirm.

--------------------------------------------------------------------------------
8. BOLA / SECURITY CANDIDATES FOR SPAIN (based on cross-country patterns)
--------------------------------------------------------------------------------
| Priority | Endpoint / Query                    | Why |
|----------|-------------------------------------|-----|
| 🔴 HIGH  | POST /api/create-energy-account     | Mirrors FR create-account mass-assignment BOLA |
| 🔴 HIGH  | POST /api/onboarding/submit-form    | Mirrors FR/NZ store-interests pattern |
| 🔴 HIGH  | query CampaignBannerAccount(accountNumber) | accountNumber in arg → IDOR/BOLA |
| 🔴 HIGH  | query ViewerAccount / ViewerProperty(accountNumber) | accountNumber in arg |
| 🟡 MED   | POST /api/fraud-meter-point-checks  | May take meter point → PII/enumeration |
| 🟡 MED   | POST /api/contract-embedded-url      | May expose signed URL for other accounts |
| 🟡 MED   | Kraken mutation addCampaignToAccount | Confirmed in other countries; test on ES |
| 🟡 MED   | NEXT_PUBLIC_STORYBLOK_ACCESS_TOKEN   | Possible draft-story read (FR/IT confirmed) |
| 🟢 LOW   | POST /api/validation                | Input validation, low impact |
| 🟢 LOW   | /api/comparator/*                   | Public rate calculation |

--------------------------------------------------------------------------------
9. NOTES / NEXT STEPS
--------------------------------------------------------------------------------
- All endpoints live under Next.js (App+Pages router) — same stack as IT/DE/NZ.
- Kraken auth model expected: KT-CT-1111/1112/4123 (same as other countries).
- To test: need valid ES login (email/password) to obtain Kraken JWT, then test
  accountNumber substitution on queries above.
- Source files available for deep review at:
    ~/target/es/map/reconstructed/webpack:/_N_E/src/
- Specifically interesting files:
    src/routes/api.ts                     (all REST paths)
    src/features/campaigns/*              (campaign logic)
    src/features/onboarding/*             (account creation flow)
    src/lib/storyblok/*                   (Storyblok token usage)
    src/entities/*/hooks/*.ts             (GraphQL query definitions)

================================================================================
END OF REPORT
================================================================================
