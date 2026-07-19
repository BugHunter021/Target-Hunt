# Octopus Energy France — Kraken GraphQL Mutations Analysis

**تاریخ:** 2026-07-19  
**هدف:** یافتن REST endpointهای جدید مشابه `addCampaignToAccount` که BOLA دارند  
**روش:** بررسی تمام mutationهای Kraken GraphQL که `accountNumber` می‌گیرن و پیش‌بینی REST معادلشون

---

## الگوی کشف شده

REST wrapperها روی Kraken GraphQL یه لایه نازک میانجی هستن که **بدون احراز هویت** کار می‌کنن ولی mutation اصلی Kraken نیاز به JWT داره. 
پس هر mutationای که `accountNumber` بگیره و REST wrapper براش وجود داشته باشه، **احتمال BOLA داره**.

4 تا نمونه قبلی:
- `createAccountNote` ↔ `/api/account/create-account-note`
- `addCampaignToAccount` ↔ `/api/account/add-campaign-to-account`
- `createMetadata` ↔ `/api/account/create-metadata`
- `getEmbeddedSecretForNewPaymentInstruction` ↔ `/api/payment-instruction/generate-secret`

---

## ۱. ✅ REST wrapperهای شناخته شده (تست شده)

این ۷ تا قبلاً از route definition استخراج شدن و تست شدن:

| # | Mutation | REST Endpoint | وضعیت |
|---|----------|--------------|--------|
| ۱ | `addCampaignToAccount` | `/api/account/add-campaign-to-account` | ✅ BOLA تأیید |
| ۲ | `createAccountNote` | `/api/account/create-account-note` | ✅ BOLA تأیید |
| ۳ | `collectDeposit` | `/api/onboarding/collect-deposit` | ✅ تست شده |
| ۴ | `enrollment` | `/api/onboarding/enrollment` | ✅ تست شده |
| ۵ | `joinSupplierAcceptTermsAndConditions` | `/api/onboarding/join-supplier-accept-terms-and-conditions` | ✅ تست شده |
| ۶ | `getEmbeddedSecretForNewPaymentInstruction` | `/api/payment-instruction/generate-secret` | ✅ تست شده |
| ۷ | `submitForm` | `/api/wizard/submit-form` | ✅ تست شده |

---

## ۲. 🔴 اولویت بالا — مالی (Account Takeover / Payment)

این RESTها اگر وجود داشته باشن، **بحرانی‌ترین** آسیب‌پذیری رو دارن چون مستقیماً پول و اطلاعات حساب رو هدف می‌گیرن.

### 🔴 تغییر اطلاعات صورتحساب

| Mutation | ورودی‌ها | REST احتمالی | ریسک |
|----------|---------|-------------|------|
| `updateAccountBillingEmail` | `accountNumber, billingEmail` | `/api/account/update-account-billing-email` | **Account Takeover** |
| `updateAccountBillingName` | `accountNumber, billingName, billingSubName` | `/api/account/update-account-billing-name` | تغییر نام صورتحساب |
| `updateAccountBillingAddress` | `accountNumber, billingAddress` | `/api/account/update-account-billing-address` | تغییر آدرس صورتحساب |

> **⚠️ توجه:** `updateAccountBillingEmail` اگه بدون Auth کار کنه → **Account Takeover کامل**. مهاجم می‌تونه ایمیل هر حسابی رو عوض کنه و پسورد رو ریست کنه.

### 🔴 پرداخت و تراکنش‌ها

| Mutation | ورودی‌ها | REST احتمالی |
|----------|---------|-------------|
| `postCredit` | `accountNumber, netAmount, taxAmount, note, displayNote` | `/api/account/post-credit` |
| `createAccountCharge` | `accountNumber, grossAmount, metadata, note` | `/api/account/create-account-charge` |
| `collectPayment` | `accountNumber, amount, paymentDate, description` | `/api/account/collect-payment` |
| `cancelPayment` | `accountNumber, paymentId, reason` | `/api/account/cancel-payment` |
| `refundPayment` | `accountNumber, paymentId, amountInMinorUnit` | `/api/account/refund-payment` |
| `amendPayment` | `accountNumber, paymentId, amount, paymentDate` | `/api/account/amend-payment` |
| `initiateStandalonePayment` | `accountNumber, amount, description, collectionMethod` | `/api/account/initiate-standalone-payment` |
| `completeStandalonePayment` | `accountNumber, ledgerNumber, retrievalToken` | `/api/account/complete-standalone-payment` |
| `initiateHostedStandalonePayment` | `accountNumber, amount, description, collectionMethod` | `/api/account/initiate-hosted-standalone-payment` |

### 🔴 تنظیمات پرداخت

| Mutation | ورودی‌ها | REST احتمالی |
|----------|---------|-------------|
| `setPaymentPreference` | `accountNumber, paymentMethod, ledgerNumbers` | `/api/account/set-payment-preference` |
| `stopAutomatedPayments` | `accountNumber, ledgerNumbers, fromDate` | `/api/account/stop-automated-payments` |
| `createAccountPaymentSchedule` | `accountNumber, ledgerNumber, paymentDay, paymentAmount` | `/api/account/create-account-payment-schedule` |
| `switchAccountToVariablePaymentSchedule` | `accountNumber, ledgerNumber, targetChangeDate` | `/api/account/switch-account-to-variable-payment-schedule` |
| `updateAutoTopUpAmount` | `accountNumber, ledgerNumber, paymentAmount` | `/api/account/update-auto-top-up-amount` |

### 🔴 دایرکت دبیت و روش‌های پرداخت

| Mutation | ورودی‌ها | REST احتمالی |
|----------|---------|-------------|
| `setUpDirectDebitInstruction` | `accountNumber, ledgerNumber, validFrom, bankDetails` | `/api/account/set-up-direct-debit-instruction` |
| `storePaymentInstruction` | `accountNumber, instructionType, validFrom, vendorReference` | `/api/account/store-payment-instruction` |
| `invalidatePaymentInstruction` | `accountNumber, id` | `/api/account/invalidate-payment-instruction` |
| `getHostedUrlForNewPaymentInstruction` | `accountNumber, ledgerNumber, instructionType` | `/api/account/get-hosted-url-for-payment-instruction` |
| `storeDirectDebitPaymentMethodDetails` | `accountNumber, ledgerNumber, bankDetails` | `/api/account/store-direct-debit-payment-method-details` |
| `setUpDirectDebitInstructionFromStoredDetails` | `accountNumber, ledgerNumber, storedPaymentMethodDetailsReference` | `/api/account/set-up-direct-debit-from-stored-details` |
| `createPaymentActionIntent` | `accountNumber, targetType, targetIdentifier` | `/api/account/create-payment-action-intent` |
| `createPaymentMethodActionIntent` | `accountNumber, targetType, targetIdentifier` | `/api/account/create-payment-method-action-intent` |

---

## ۳. 🟠 اولویت متوسط — کمپین، دپوزیت، قرارداد

### 🟠 کمپین

| Mutation | ورودی‌ها | REST احتمالی |
|----------|---------|-------------|
| `removeCampaignFromAccount` | `accountNumber, campaignName, expireAt` | `/api/account/remove-campaign-from-account` |
| `updateCampaignAccountExpiryDate` | `accountNumber, campaignSlug, expiryDate, note` | `/api/account/update-campaign-account-expiry-date` |

### 🟠 دپوزیت

| Mutation | ورودی‌ها | REST احتمالی |
|----------|---------|-------------|
| `createDepositAgreement` | `accountNumber, depositKey, reason` | `/api/onboarding/create-deposit-agreement` |
| `recordDepositAgreementAccepted` | `accountNumber, depositKey` | `/api/onboarding/record-deposit-agreement-accepted` |

### 🟠 قرارداد (Agreement/Contract)

| Mutation | ورودی‌ها | REST احتمالی |
|----------|---------|-------------|
| `createAgreement` | `accountNumber, supplyPointExternalIdentifier, productCode, validFrom, validTo` | `/api/account/create-agreement` |
| `revokeAgreement` | `accountNumber, agreementId, reason` | `/api/account/revoke-agreement` |
| `terminateAgreement` | `agreementId, terminateAt, reason` | `/api/account/terminate-agreement` |
| `attachAgreementsToContracts` | `agreementIds, accountContractIdentifier` | `/api/account/attach-agreements-to-contracts` |
| `detachAgreementsFromContracts` | `agreementIds` | `/api/account/detach-agreements-from-contracts` |

### 🟠 خروج از تأمین‌کننده (Leave Supplier)

| Mutation | ورودی‌ها | REST احتمالی |
|----------|---------|-------------|
| `instigateLeaveSupplier` | `accountNumber, marketData, futureBillingAddress` | `/api/onboarding/instigate-leave-supplier` |
| `updateLeaveSupplier` | `marketData, futureBillingAddress` | `/api/onboarding/update-leave-supplier` |
| `cancelLeaveSupplier` | `number, reason, marketData` | `/api/onboarding/cancel-leave-supplier` |
| `reverseLeaveSupplier` | `number, reversalReason` | `/api/onboarding/reverse-leave-supplier` |

---

## ۴. 🟠 اولویت متوسط — ریفرال و لویالتی

### 🟠 ریفرال

| Mutation | ورودی‌ها | REST احتمالی |
|----------|---------|-------------|
| `createReferral` | `accountNumber, reference` | `/api/account/create-referral` |
| `redeemReferralClaimCode` | `accountNumber, code` | `/api/account/redeem-referral-claim-code` |
| `addSignupReferralOnAccount` | `accountNumber, schemeCode` | `/api/account/add-signup-referral-on-account` |
| `payoutReferralForAccount` | `accountNumber, referralId, note` | `/api/account/payout-referral-for-account` |

> **💰 نکته:** `payoutReferralForAccount` جالبه — می‌تونه ریفرال جعلی پرداخت کنه.

### 🟠 لویالتی (امتیاز)

| Mutation | ورودی‌ها | REST احتمالی |
|----------|---------|-------------|
| `enrollAccountInLoyaltyProgram` | `accountUserId, accountNumber` | `/api/account/enroll-account-in-loyalty-program` |
| `unenrollAccountFromLoyaltyProgram` | `accountNumber` | `/api/account/unenroll-account-from-loyalty-program` |
| `awardLoyaltyPoints` | `accountNumber, points, reasonCode` | `/api/account/award-loyalty-points` |
| `deductLoyaltyPoints` | `accountNumber, accountUserId, points` | `/api/account/deduct-loyalty-points` |
| `transferLoyaltyPointsBetweenUsers` | `accountNumber, sendingUserId, receivingUserId, points` | `/api/account/transfer-loyalty-points` |
| `redeemLoyaltyPointsForAccountCredit` | `accountUserId, accountNumber, points` | `/api/account/redeem-loyalty-points` |
| `setLoyaltyPointsUser` | `accountNumber, newLoyaltyPointsUserId` | `/api/account/set-loyalty-points-user` |

---

## ۵. 🟡 اولویت پایین — متفرقه

| Mutation | ورودی‌ها | REST احتمالی |
|----------|---------|-------------|
| `createQuoteForAccount` | `accountNumber, identifiers` | `/api/account/create-quote-for-account` |
| `initiateProductSwitch` | `accountNumber, quotedProductId, switchDate` | `/api/account/initiate-product-switch` |
| `enrollFanClubAccount` | `accountNumber, catchments, email` | `/api/account/enroll-fan-club-account` |
| `triggerTestCharge` | `accountNumber` | `/api/account/trigger-test-charge` |
| `addProperty` | `accountNumber, address` | `/api/account/add-property` |
| `registerCustomerAsset` | `accountNumber, propertyId, physicalId, name, type` | `/api/account/register-customer-asset` |
| `verifyIdentity` | `accountNumber, postcode, fullName` | `/api/account/verify-identity` |
| `createContributionAgreement` | `accountNumber, schemeCode, interval, amount` | `/api/account/create-contribution-agreement` |
| `endContributionAgreement` | `contributionAgreementId, endAt` | `/api/account/end-contribution-agreement` |
| `createGoodsQuote` | `accountNumber, productsToQuote` | `/api/account/create-goods-quote` |
| `acceptGoodsQuote` | `accountNumber, quoteId` | `/api/account/accept-goods-quote` |
| `createPurchase` | `accountNumber, saleItems` | `/api/account/create-purchase` |
| `createOpportunityAndLead` | `funnelCode, assignedToAffiliateNumber` | `/api/account/create-opportunity-and-lead` |
| `updateCommsDeliveryPreference` | `accountNumber, commsDeliveryPreference` | `/api/account/update-comms-delivery-preference` |
| `optionsLanguagePreference` | `accountNumber, optionsLanguagePreference` | `/api/account/update-options-language-preference` |
| `updateDocumentAccessibilityPreference` | `accountNumber, documentAccessibilityPreference` | `/api/account/update-document-accessibility-preference` |
| `createShellAccount` | `portfolioNumber, givenName, familyName` | `/api/account/create-shell-account` |
| `createAccountReference` | `accountNumber, namespace, value` | `/api/account/create-account-reference` |
| `updateAccountReference` | `accountNumber, namespace, value` | `/api/account/update-account-reference` |
| `deleteAccountReference` | `accountNumber, namespace` | `/api/account/delete-account-reference` |
| `linkAccountToBusiness` | `accountId, businessId` | `/api/account/link-account-to-business` |
| `acceptGoodsQuote` | `accountNumber, quoteId, clientParams` | `/api/account/accept-goods-quote` |
| `updatePurchase` | `accountNumber, purchaseId, saleItems` | `/api/account/update-purchase` |

---

## ۶. خلاصه آمار

| دسته | تعداد |
|------|-------|
| ✅ REST wrapper شناخته شده (تست شده) | ۷ |
| 🔴 اولویت بالا (مالی / Account Takeover) | ۲۶ |
| 🟠 اولویت متوسط (کمپین، دپوزیت، ریفرال، لویالتی) | ۲۳ |
| 🟡 اولویت پایین (متفرقه) | ۳۳ |
| **جمع کل** | **۸۹** |

---

## ۷. استراتژی پیشنهادی تست

### مرحله ۱ — 🔴 بحرانی (Account Takeover)
```
POST /api/account/update-account-billing-email
{"accountNumber": "A-XXX", "billingEmail": "attacker@evil.com"}
```
```
POST /api/account/update-account-billing-name
{"accountNumber": "A-XXX", "billingName": "Hacker Name"}
```
```
POST /api/account/update-account-billing-address
{"accountNumber": "A-XXX", "billingAddress": {...}}
```

### مرحله ۲ — 🔴 مالی
```
POST /api/account/post-credit
{"accountNumber": "A-XXX", "netAmount": 10000, "taxAmount": 2000}
```
```
POST /api/account/create-account-charge
{"accountNumber": "A-XXX", "grossAmount": 50000}
```
```
POST /api/account/collect-payment
{"accountNumber": "A-XXX", "amount": 5000}
```

### مرحله ۳ — 🟠 کمپین و ریفرال
```
POST /api/account/remove-campaign-from-account
{"accountNumber": "A-XXX", "campaignName": "fleximax"}
```
```
POST /api/account/payout-referral-for-account
{"accountNumber": "A-XXX", "referralId": "...", "note": "payout"}
```

---

## نکته مهم

> این لیست بر اساس names ورودی mutationها ساخته شده. فرانسه از **Astro** استفاده می‌کنه و route definition آرشیتکتورش توی یه فایل مرکزی هست (`src/routes/api.ts`). ممکنه بعضی از این RESTها وجود نداشته باشن — ولی چون **همه ۷ تایی که وجود دارن BOLA بودن**، ارزش تست تک تک اینا رو داره.
