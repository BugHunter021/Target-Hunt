# 🇯🇵 Octopus Energy Japan — Pentest Analysis

> **Domain:** octopusenergy.co.jp  
> **JS files:** `/root/target/Octopus/jp/js/`  
> **GraphQL schemas:** `/root/target/Octopus/jp/graphql/`

---

## GraphQL Endpoints

| # | Endpoint | Status | Note |
|---|----------|--------|------|
| 1 | `https://api.oejp-kraken.energy/v1/graphql/` | ✅ 200 | Kraken Direct — no WAF |
| 2 | `https://octopusenergy.co.jp/api/kraken-graphql` | ✅ 200 | Vercel proxy (same schema) |
| 3 | `https://api.backend.octopusenergy.co.jp/v1/graphql/` | ✅ 200 | Backend API (different schema, 875K) |

### Schema Stats

| Schema | Size | Mutations |
|--------|------|-----------|
| Kraken JP | 5.4M | **352** |
| Vercel Proxy | 5.4M | 352 (same) |
| Backend | 875K | **66** |

---

## REST API Endpoints (from JS)

| Endpoint | Method | HTTP Test | Note |
|----------|--------|-----------|------|
| `/api/onboarding/create-affiliate-session` | POST | 500 | Needs valid params |
| `/api/onboarding/create-quote` | POST | 400 | Needs valid params |
| `/api/onboarding/update-account-consents` | POST | 400 | Needs valid params |
| `/api/onboarding/validate-email` | POST | 500 | Needs valid params |
| `/api/billing/gmo-token` | **GET** | **✅ 200** | Returns a live GMO token! |
| `/api/billing/gmo-script` | **GET** | **✅ 200** | Returns CryptoJS+RSA lib |
| `/api/kraken-graphql` | POST | 400 | Needs valid GraphQL query |

### 🔴 GMO Billing — `/api/billing/gmo-token`

- **GET request** — no auth, no params needed
- Returns: `{"generateCreditCardToken":{"creditCardToken":"token_XXXXXXXXXX"}}`
- This is a **live GMO Payment Gateway token** for card tokenization
- The token format is `token_` + 26 alphanumeric chars

### 🟡 GMO Billing — `/api/billing/gmo-script`

- **GET request** — no auth, no params needed
- Returns a full client-side encryption library (CryptoJS + JSEncrypt RSA)
- Used for encrypting credit card details before sending to GMO
- Contains a **hardcoded RSA public key** for payment encryption

---

## 🔴 Kraken JP — 109 Mutations with `accountNumber`

### 🔴 Account Takeover / Billing

| Mutation | Input Fields | Impact |
|----------|-------------|--------|
| `updateAccountBillingEmail` | `accountNumber, billingEmail` | 🔴 Change billing email |
| `updateAccountBillingName` | `accountNumber, billingName, billingSubName` | 🟠 Change account name |
| `updateAccountConsentData` | `accountNumber, consentData` | 🟡 Modify consent |
| `updateCommsDeliveryPreference` | `accountNumber, commsDeliveryPreference` | 🟡 Comms prefs |

### 🔴 Payment / Financial

| Mutation | Input Fields | Impact |
|----------|-------------|--------|
| `updateCreditCardDetails` | `accountNumber, token, paymentDay` | 🔴 Change saved card! |
| `createPaymentMethod` | `accountNumber, payment, paymentDay` | 🔴 Add payment method |
| `createKonbiniPaymentMethod` | `accountNumber, paymentDay, ledgerNumber` | 🟠 Add konbini payment |
| `createKonbiniPayment` | `accountNumber, amount, description` | 🟠 Create payment |
| `updateBankAccountDetails` | `accountNumber, bankCode, branchCode, ...` | 🔴 Change bank account |
| `collectDeposit` | `accountNumber, depositKey, idempotencyKey` | 🟠 Collect deposit |
| `collectPayment` | `accountNumber, amount, ledgerNumber, ...` | 🟠 Collect payment |
| `refundPayment` | `accountNumber, paymentId, amount, ...` | 🔴 Refund |
| `cancelPayment` | `accountNumber, paymentId, reason` | 🟠 Cancel payment |
| `amendPayment` | `accountNumber, paymentId, amount, date` | 🔴 Modify payments |
| `createPaymentActionIntent` | `accountNumber, targetType, targetIdentifier` | 🟠 Payment intent |
| `getEmbeddedSecretForNewPaymentInstruction` | `accountNumber, instructionType, ledgerNumber` | 🟡 Get secret |
| `storePaymentInstruction` | `accountNumber, instructionType, ledgerNumber, ...` | 🟠 Store payment |

### 🔴 Referral / Loyalty

| Mutation | Input Fields | Impact |
|----------|-------------|--------|
| `payoutReferralForAccount` | `accountNumber, referralId, note` | 🔴 Payout referral! |
| `redeemReferralClaimCode` | `accountNumber, code` | 🟠 Redeem referral |
| `addSignupReferralOnAccount` | `accountNumber, schemeCode` | 🟡 Add signup referral |
| `createReferral` | `accountNumber, reference` | 🟡 Create referral |
| `awardLoyaltyPoints` | `accountNumber, points, reasonCode, note` | 🔴 Award points |
| `deductLoyaltyPoints` | `accountNumber, accountUserId, points, reasonCode` | 🔴 Deduct points |
| `transferLoyaltyPointsBetweenUsers` | `accountNumber, sendingUserId, receivingUserId, points` | 🔴 Transfer points |
| `enrollAccountInLoyaltyProgram` | `accountUserId, accountNumber` | 🟠 Enroll in loyalty |
| `unenrollAccountFromLoyaltyProgram` | `accountNumber` | 🟡 Unenroll |

### 🔴 Contract / Agreement

| Mutation | Input Fields | Impact |
|----------|-------------|--------|
| `createAgreement` | `accountNumber, supplyPointExternalIdentifier, productCode, ...` | 🔴 Create contract |
| `revokeAgreement` | `accountNumber, agreementId, reason` | 🔴 Revoke contract |
| `updateAgreementPeriod` | `agreementId, accountNumber, validFrom, validTo` | 🟠 Modify contract |
| `extendAgreementPeriod` | `agreementId, accountNumber, validTo` | 🟠 Extend contract |
| `createAgreementRollover` | `accountNumber, currentAgreementId, ...` | 🟡 Rollover |
| `withdrawAccount` | `accountNumber` | 🔴 **Withdraw account!** |

### 🔴 Move / Switch

| Mutation | Input Fields | Impact |
|----------|-------------|--------|
| `initiateMoveOut` | `accountNumber, propertyId, moveOutDate` | 🔴 Initiate move out! |
| `initiateProductSwitch` | `accountNumber, quotedProductId, switchDate` | 🟠 Switch product |
| `revertProductSwitch` | `accountNumber, spin` | 🟡 Revert switch |
| `initiateAmperageChange` | `accountNumber, requestedAmperageValue, ...` | 🟠 Change amperage |
| `requestAmperageChange` | `accountNumber, spin, requestedAmperageValue, ...` | 🟠 Request amperage change |

### 🟡 Campaign / Demand Response

| Mutation | Input Fields | Impact |
|----------|-------------|--------|
| `addCampaignToAccount` | `accountNumber, campaign, startDate, expiryDate` | 🟡 Add campaign |
| `removeCampaignFromAccount` | `accountNumber, campaignName, expireAt` | 🟡 Remove campaign |
| `registerDemandResponseCampaignParticipant` | `accountNumber, campaignName` | 🟡 DR participant |
| `enrollOntoFanClub` (Backend) | — | 🟡 Fan club |

### 🔴 Dunning / Collections

| Mutation | Input Fields | Impact |
|----------|-------------|--------|
| `startCollectionProcess` | `accountNumber, ledgerNumber, ...` | 🔴 Start collections |
| `pauseDunning` | `accountNumber, pathName, startDate, stopDate` | 🟠 Pause dunning |
| `withdrawDunning` | `accountNumber, pathName, note` | 🟠 Withdraw dunning |
| `commenceDcaProceeding` | `accountNumber, ledgerNumber, agency, ...` | 🔴 Start DCA |
| `closeDcaProceeding` | `accountNumber, ledgerNumber, ...` | 🟠 Close DCA |

### 🟠 Lead / Opportunity

| Mutation | Input Fields |
|----------|-------------|
| `createOpportunityAndLead` | `accountNumber, funnelCode, leadContactDetails, ...` |
| `createLead` | `leadContacts, accountNumber, ...` |
| `updateLeadDetails` | `leadNumber, extraDetails, accountNumber` |

### 🟡 Customer Support

| Mutation | Input Fields |
|----------|-------------|
| `createComplaint` | `accountNumber, type, subtype, summary, ...` |
| `createCustomerFeedback` | `accountNumber, feedbackSource, ...` |
| `createInkInboundMessage` | `accountNumber, channel, messageId, ...` |
| `escalateInkConversation` | `accountNumber, conversationRelayId` |

### 🟡 Account / Property

| Mutation | Input Fields |
|----------|-------------|
| `addProperty` | `accountNumber, address` |
| `createSimpleService` | `accountNumber, productCode, validFrom, validTo, params` |
| `sendFamilySwitchInOffer` | `accountNumber` |
| `enrollment` | `accountNumber, bankDetails, ...` (many fields) |
| `joinSupplierAcceptTermsAndConditions` | `accountNumber, joinSupplierProcessNumber` |
| `verifyIdentity` | `accountNumber, postcode, fullName, firstLineOfAddress` |
| `suggestFixedPaymentAmount` | `accountNumber` |
| `createAccountNote` | `accountNumber, note, isPinned, unpinAt` |
| `postCredit` | `accountNumber, netAmount, taxAmount, displayNote, reason` |
| `createAccountCharge` | `accountNumber, grossAmount, metadata, note, reason` |
| `setPaymentPreference` | `accountNumber, paymentMethod, ledgerNumbers` |
| `stopAutomatedPayments` | `accountNumber, ledgerNumbers, fromDate` |
| `setUpDirectDebitInstruction` | `accountNumber, ledgerNumber, bankDetails, ...` |
| `purchaseVoucher` | `accountNumber, voucherValueInCents, ...` |

---

## 🔴 Backend Schema — 66 Mutations

### Heat Pump / Energy

| Mutation | Description |
|----------|-------------|
| `heatPumpRequestProvisioningClaimCertificate` | Heat pump certificate |
| `heatPumpSetHushMode` | Set hush mode |
| `heatPumpSetZoneMode` | Set zone mode |
| `heatPumpSetZoneSchedules` | Set zone schedules |
| `createHeatPumpQuote` | Create heat pump quote |
| `acceptHeatPumpQuote` | Accept heat pump quote |
| `initiateHeatPumpDepositPayment` | Initiate deposit payment |

### Saving Sessions / Fan Club

| Mutation | Description |
|----------|-------------|
| `joinSavingSessionsCampaign` | Join saving sessions |
| `joinSavingSessionsEvent` | Join saving session event |
| `enrollOntoFanClub` | Enroll in fan club |
| `spinWheelOfFortune` | 🎰 **Spin the wheel!** |

### OctoCharger / EV

| Mutation | Description |
|----------|-------------|
| `updateOctoChargerOCPPCredentials` | Update charger credentials |
| `onboardChargePoint` | Onboard charge point |
| `deauthenticateChargePoint` | Deauth charge point |
| `chargePointSetSchedule` | Set charging schedule |
| `chargePointSetBoostChargingState` | Set boost charging |
| `chargePointStartBoostCharge` | Start boost charge |
| `chargePointUpgradeFirmware` | 🔴 **Upgrade firmware!** |
| `chargePointSetChargeCableAutoLockState` | Auto-lock cable |
| `chargePointUnlockChargeCable` | Unlock cable |

### Solar / Charity

| Mutation | Description |
|----------|-------------|
| `calculateSolarEstimate` | Solar estimate |
| `submitSolarInterest` | Solar interest |
| `optInToCharityDonation` | Charity opt-in |
| `optOutOfCharityDonation` | Charity opt-out |
| `updateCharityDonationAmount` | **Change donation amount!** |

### Octoplus Rewards

| Mutation | Description |
|----------|-------------|
| `scratchOctoplusScratchcard` | **Scratch card!** |
| `rejectOctoplusScratchcard` | Reject scratch card |
| `claimOctoplusReward` | Claim reward |
| `claimShoptopusReward` | Claim Shoptopus reward |
| `enrollSchemeMember` / `unenrollSchemeMember` | Scheme membership |

---

## 🟢 Interesting Findings

### GMO Payment Token (LIVE!)
```
GET https://octopusenergy.co.jp/api/billing/gmo-token
→ {"generateCreditCardToken":{"creditCardToken":"token_NVYzJMOcayYImhJCsSh2naZtzW"}}
```
Returns a **live GMO payment token** with no auth. This token could be used to initiate card tokenization flows.

### GMO Encryption Library
```
GET https://octopusenergy.co.jp/api/billing/gmo-script
```
Returns a full CryptoJS + RSA encryption library with **hardcoded public key** for encrypting credit card data.

### Account Page Structure
- `/account/A-F7E9DA7E/` — Account dashboard  
- `/account/A-F7E9DA7E/cancel` — Cancel page  
- `/account/A-F7E9DA7E/request-amperage-change-on-site` — Amperage change

### Friend Referral
- `/friend/gray-mitten-277/` — Friend referral page

### Account Number Format
- `A-XXXXXXXX` (same as FR)

---

## 🎯 Prioritized BOLA Testing List

### Priority 1 — 🔴 Critical (Financial / ATO)
1. `updateCreditCardDetails` — Change saved card on another account
2. `updateBankAccountDetails` — Change bank account details
3. `payoutReferralForAccount` — Payout referral to attacker
4. `postCredit` — Post credit to account
5. `refundPayment` — Refund payment
6. `collectPayment` — Collect payment
7. `initiateMoveOut` — Start move out process
8. `withdrawAccount` — Close/withdraw account
9. `createAgreement` / `revokeAgreement` — Contract manipulation

### Priority 2 — 🟠 Medium (Data manipulation)
10. `updateAccountBillingEmail` — Change billing email
11. `updateAccountBillingName` — Change billing name
12. `awardLoyaltyPoints` / `deductLoyaltyPoints` — Point manipulation
13. `transferLoyaltyPointsBetweenUsers` — Transfer points
14. `createAccountCharge` — Create charges
15. `amendPayment` — Modify payments

### Priority 3 — 🟡 Low (Information)
16. `createAccountNote` — Add notes
17. `createComplaint` — File complaints
18. `suggestFixedPaymentAmount` — Suggest payment amount
19. `addCampaignToAccount` — Campaign manipulation

---

## Testing Notes

- Kraken direct (`api.oejp-kraken.energy`) has **no Vercel WAF** — can test from anywhere
- Vercel proxy (`octopusenergy.co.jp/api/kraken-graphql`) works but may have WAF
- Backend API (`api.backend.octopusenergy.co.jp`) has different schema — 66 mutations
- Account number format: `A-XXXXXXXX`
- GMO payment endpoints are **fully accessible without auth**
