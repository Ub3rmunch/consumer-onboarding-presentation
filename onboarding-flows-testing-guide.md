# Consumer Onboarding Flows — Testing Guide

This document maps the major onboarding paths through the consumer onboarding service.
Use it to understand which steps a user sees based on their country, product eligibility, and chosen intent.

> **Data source:** Product availability from [availability service](https://github.com/transferwise/availability/) (live production API, March 2028). Flow logic from `OnboardingStepsFactory`, `OnboardingStepDeciderService`, and per-step `showAllowance()` conditions.

---

## Top-Level Flow Decision

```
┌─────────────────────────────────────┐
│           User opens app            │
│         (signup exists)             │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│    Base eligibility check           │
│  ┌─ signup after 2023-01-26       │
│  ├─ profile type = PERSONAL       │
│  ├─ profile role ≠ YOUNG_EXPLORER │
│  ├─ platform supported            │
│  ├─ no SEND intent selected       │
│  └─ balance eligible (by country) │
└──────────────┬──────────────────────┘
               │
          ┌────┴────┐
          │eligible?│
          └────┬────┘
         no    │    yes
      ┌────────┘────────┐
      ▼                 ▼
 [INELIGIBLE]   ┌──────────────────┐
                │Intent picker     │
                │eligible?         │
                │(Android ≥ 8.74, │
                │ iOS ≥ 12551,    │
                │ Web: always)     │
                └───────┬──────────┘
               no       │      yes
           ┌────────────┘──────────┐
           ▼                       ▼
   ┌───────────────┐    ┌─────────────────────┐
   │ STANDARD or   │    │ Intent already      │
   │ ASSETS flow   │    │ chosen?             │
   │ (see below)   │    └────────┬────────────┘
   └───────────────┘        no   │   yes
                        ┌────────┘────────┐
                        ▼                 ▼
               ┌────────────────┐  ┌──────────────────┐
               │ PRE-INTENT     │  │ intent =         │
               │ FLOW           │  │ UNIFIED?         │
               │ (get profile   │  └────┬─────────────┘
               │  + pick intent)│  yes  │  no
               └────────────────┘  ┌────┘────────┐
                                   ▼             ▼
                           ┌─────────────┐ ┌────────────┐
                           │ STANDARD or │ │ INTENT-    │
                           │ ASSETS flow │ │ SPECIFIC   │
                           │ (see below) │ │ FLOW       │
                           └─────────────┘ └────────────┘
```

---

## Flow A — Pre-Intent (no intent chosen yet)

Shown when user is eligible for intent picker but hasn't chosen yet.

```
Profile already completed?
├─ YES → [VERIFICATION] → [INTENT_PICKER]
└─ NO  → [IDENTITY] → [PROFILE] → [VERIFICATION]
```

After intent is picked, one of the flows below kicks in.

---

## Flow B — Intent-Specific (non-UNIFIED intent chosen)

After choosing SEND, SPEND, RECEIVE, HOLD, or CURRENCYACCOUNT:

```
Chosen intent
├─ SEND     → [SEND_FLOW]
├─ SPEND    → [CARD_ORDER_FLOW]
└─ RECEIVE / HOLD / CURRENCYACCOUNT
            → [BANK_DETAILS_FLOW] → [DISCOVERABILITY_SETTINGS]
```

---

## Flow C — Standard Flow (default, no assets)

The most common path. Used when user chose UNIFIED_ONBOARDING or is not eligible for intent picker, and is NOT eligible for assets.

```
[IDENTITY]
    │  (only if profile not completed AND identity flow required by MIR)
    ▼
[PROFILE]
    │  (only if profile not completed)
    ▼
[VERIFICATION]
    │  (only if MIR says so, platform ≥ Android 8.131 / iOS 8.124)
    ▼
[REQUIREMENTS]
    │  (legal disclosures, skipped for UK current account flow)
    ▼
[CURRENCY_SELECTION]
    │  (pick currencies for multi-currency account)
    │  ⚠️  SKIPPED for India
    ▼
[DISCOVERABILITY_SETTINGS]
    │  (receive/visibility settings)
    │  ⚠️  SKIPPED for India
    ▼
[CARD_INTRO]
    │  (card product intro, one-time)
    │  ⚠️  SKIPPED for India, UK current account
    ▼
[CARD_SELECTION]
    │  (choose physical/virtual card)
    │  (only if card-eligible and not ordered)
    ▼
[BALANCE_CONSENT]
    │  (MCA disclosure acceptance)
    │  ⚠️  US ONLY — skipped for all other countries
    ▼
[CARD_ORDER_FLOW]
    │  (complete card order)
    │  (only if card not skipped, card program chosen)
    ▼
[BANK_DETAILS_FLOW]
    │  (link bank account)
    │  (only if skipped card OR not card-eligible,
    │   AND has currencies with bank details support)
    ▼
   DONE
```

---

## Flow D — Assets Flow (assets-eligible users)

For users eligible for the assets/investment product. Currency selection is replaced by the assets step.

```
[IDENTITY]
    ▼
[PROFILE]
    ▼
[VERIFICATION]
    ▼
[REQUIREMENTS]
    ▼
[BALANCE_CONSENT]          ←── note: earlier than in standard flow
    │  ⚠️  US ONLY
    ▼
[ASSETS]                   ←── replaces CURRENCY_SELECTION
    │  (if user becomes ineligible mid-flow → falls back to CURRENCY_SELECTION)
    ▼
[DISCOVERABILITY_SETTINGS]
    ▼
[CARD_INTRO]
    ▼
[CARD_SELECTION]
    ▼
[CARD_ORDER_FLOW]
    ▼
[BANK_DETAILS_FLOW]
    ▼
   DONE
```

---

## Flow E — Explicit Receive (LAUNCHPAD_RECEIVE entry point)

When user enters from the receive launchpad. No card steps. Two sub-variants:

### E1: Receive + Assets eligible
```
[IDENTITY] → [PROFILE] → [VERIFICATION] → [REQUIREMENTS] →
[BALANCE_CONSENT (US)] → [ASSETS] → [DISCOVERABILITY_SETTINGS] →
[BANK_DETAILS_FLOW] → DONE
```

### E2: Receive + No assets
```
[IDENTITY] → [PROFILE] → [VERIFICATION] → [REQUIREMENTS] →
[CURRENCY_SELECTION] → [BALANCE_CONSENT (US)] →
[DISCOVERABILITY_SETTINGS] → [BANK_DETAILS_FLOW] → DONE
```

---

## Country Product Availability Tiers

Based on production data from the availability API. This determines which steps are reachable.

### Tier 1 — All Products (Send + Balance + Card + Bank Details + Assets)

**33 countries:** AU, AT, BG, BR, HR, CY, CZ, DK, EE, FI, FR, DE, GR, HU, IE, IT, LV, LT, LU, MT, NL, NO, PL, PT, RO, SG, SK, SI, ES, SE, CH, GB, US

These users get the **full flow** (Standard or Assets, depending on individual assets eligibility).

### Tier 2 — 4 Products (Send + Balance + Card + Bank Details, no Assets)

**24 countries:** AD, BE, CA, KY, FK, GF, GI, GP, GG, IS, IM, JP, JE, LI, MY, MQ, YT, MC, NZ, PH, RE, BL, MF, PM, SX

These users always get the **Standard flow** (never Assets). Note: PH has card from availability API but onboarding service blocks card via `RegionalOnboardingFlowService`.

### Tier 3 — 3 Products (Balance + Card, no Bank Details, no Assets)

**India only:** IN

India gets a special variant: skips Currency Selection, Card Intro, and Discoverability Settings. Flow:
```
[IDENTITY] → [PROFILE] → [VERIFICATION] → [REQUIREMENTS] →
[CARD_SELECTION] → [CARD_ORDER_FLOW] → DONE
```

### Tier 4 — 3 Products (Send + Balance + Bank Details, no Card, no Assets)

**23 countries:** AR, BH, CL, CN, CO, CR, GE, IL, KR, KW, MO, ME, MK, PE, PR, QA, SA, ZA, TW, UY, GU, BV

No card steps. Flow effectively becomes:
```
[IDENTITY] → [PROFILE] → [VERIFICATION] → [REQUIREMENTS] →
[CURRENCY_SELECTION] → [BALANCE_CONSENT (US territories)] →
[DISCOVERABILITY_SETTINGS] → [BANK_DETAILS_FLOW] → DONE
```

### Tier 5 — Send Only (~140 countries)

Including: HK, AE, TR, MX, TH, ID, VN, KH, BD, KE, NG, PK, etc.

Not eligible for the consumer onboarding flow at all (no balance = no MCA eligibility). Users go through the **send transfer flow** directly.

**Special case — Indonesia (ID):** Hardcoded block at balance eligibility level (`COUNTRIES_BLOCKED_BY_MARKETING`), even though send is available.

### Tier 0 — Completely Blocked (22 countries)

AF, BY, BI, CF, TD, CG, CD, CU, ER, IR, IQ, KP, LY, MM, RU, SO, SS, SD, SY, VE, VI, YE

No products available.

---

## Country-Specific Special Cases

| Country | Special Behavior |
|---------|-----------------|
| **US** | Extra `BALANCE_CONSENT` step (MCA Disclosure). Requires accepting `US_MCA_DISCLOSURE` terms. |
| **India (IN)** | Skips: Currency Selection, Card Intro, Discoverability Settings. Has card + balance but NO bank details. |
| **Indonesia (ID)** | Hardcoded block for balance + card at `BalanceEligibilityService` and `MultiCurrencyAccountEligibilityService`. Send only. |
| **Philippines (PH)** | Card blocked by `RegionalOnboardingFlowService.eligibleForCard()` despite availability API showing card. Gets balance + bank details + send. |
| **GB (UK)** | May enter UK Current Account flow (feature-flagged: `current-account-onboarding-uk`). When active: skips Requirements and Card Intro. |
| **Brazil (BR)** | Full product availability (Tier 1). If intent picker eligible, likely routes to UNIFIED_ONBOARDING. No explicit COF concept in codebase — standard flow applies. |

---

## Step-by-Step Eligibility Conditions

Each step has a `showAllowance()` that gates whether it actually appears:

| Step | Shown When | Frequency |
|------|-----------|-----------|
| **IDENTITY** | Profile not completed AND MIR requires identity flow (currently disabled) | ALWAYS |
| **PROFILE** | Profile not completed | ALWAYS |
| **VERIFICATION** | Profile exists AND platform supported (Android ≥ 8.131, iOS ≥ 8.124) AND MIR returns true | ONCE |
| **INTENT_PICKER** | Intent picker eligible AND no intent chosen | ONCE |
| **REQUIREMENTS** | Not UK current account AND (assets eligible OR no explicit receive decisions) | ONCE |
| **CURRENCY_SELECTION** | No currencies chosen yet AND country ≠ India | ALWAYS |
| **BALANCE_CONSENT** | Country = US AND MCA disclosure not accepted | ALWAYS |
| **ASSETS** | Assets eligible (from hold-bff API) AND not completed. Fallback → CURRENCY_SELECTION | ALWAYS |
| **DISCOVERABILITY_SETTINGS** | Balance eligible/opened AND feature enabled AND no explicit decisions AND country ≠ India | ONCE |
| **CARD_INTRO** | Card eligible AND order not done AND country ≠ India AND not UK current account | ONCE |
| **CARD_SELECTION** | Card eligible AND order not done | ONCE |
| **CARD_ORDER_FLOW** | Card eligible AND not skipped AND has current step AND order not done AND card program chosen | ALWAYS |
| **BANK_DETAILS_FLOW** | (Not card eligible OR skipped card) AND has currencies with bank details AND order not done | ALWAYS |
| **SEND_FLOW** | Intent picker eligible AND intent = SEND | ONCE |
| **INELIGIBLE** | Not eligible for flow | ALWAYS |

---

## External Services That Determine Eligibility

| Service | What It Decides | Failure Behavior |
|---------|----------------|------------------|
| **Balance Service** (`balance.envoy.tw.ee`) | Balance/MCA eligibility per country | Not eligible → can't enter flow |
| **Card Order Service** (`twcard-order.envoy.tw.ee`, gRPC) | Card program availability per country | Empty → no card steps |
| **Deposit Account Service** (`deposit-account.envoy.tw.ee`) | Bank details currencies per country | Empty → no bank details steps |
| **Hold BFF** (`hold-bff.envoy.tw.ee`) | Assets/investment eligibility per user | Error → not eligible |
| **Mitigation Requirements** (`mitigation-requirements.envoy.tw.ee`) | Whether verification is needed | Timeout → skip verification |
| **Terms Service** (`terms.envoy.tw.ee`) | US MCA disclosure consent status | Missing → require consent |
| **Feature Service** | UK current account flag, discoverability flag | Disabled → feature off |

---

## Recommended Test Matrix

### By Flow Type
| # | Test Case | Country | Setup | Expected Flow |
|---|-----------|---------|-------|---------------|
| 1 | Standard full flow | DE, FR, NL | New user, no assets | C: Identity → Profile → Verification → Requirements → Currency Selection → Discoverability → Card Intro → Card Selection → Balance Consent (skip) → Card Order → Bank Details |
| 2 | Assets flow | GB, US, AU | New user, assets eligible | D: Identity → Profile → Verification → Requirements → Balance Consent (US only) → Assets → Discoverability → Card Intro → Card Selection → Card Order → Bank Details |
| 3 | US balance consent | US | New user | Standard/Assets flow with Balance Consent shown (MCA disclosure) |
| 4 | India flow | IN | New user | Stripped: Identity → Profile → Verification → Requirements → Card Selection → Card Order |
| 5 | No-card country | AR, ZA, IL | New user | Standard without card: ends at Bank Details after Currency Selection |
| 6 | Philippines | PH | New user | Card blocked by regional service despite availability API. Balance + Bank Details only |
| 7 | Send-only country | NG, TH, MX | New user | Not eligible for consumer onboarding. Send flow only |
| 8 | Indonesia | ID | New user | Hardcoded block. Send only despite being large market |

### By Intent Picker
| # | Test Case | Setup | Expected Flow |
|---|-----------|-------|---------------|
| 9 | Intent → SEND | Eligible user, picks SEND | A → B: Send Flow |
| 10 | Intent → SPEND | Eligible user, picks SPEND | A → B: Card Order Flow |
| 11 | Intent → RECEIVE | Eligible user, picks RECEIVE | A → B: Bank Details → Discoverability |
| 12 | Intent → UNIFIED | Eligible user, picks UNIFIED | A → C/D: Standard or Assets flow |
| 13 | No intent picker (old app) | Android < 8.74 | Skips intent picker, goes to Standard/Assets |

### By Entry Point
| # | Test Case | Setup | Expected Flow |
|---|-----------|-------|---------------|
| 14 | Receive launchpad + assets | LAUNCHPAD_RECEIVE, assets eligible | E1: No card steps, ends at Bank Details |
| 15 | Receive launchpad, no assets | LAUNCHPAD_RECEIVE, no assets | E2: Currency Selection → Bank Details |

### Edge Cases
| # | Test Case | Setup | Expected Flow |
|---|-----------|-------|---------------|
| 16 | UK current account | GB, feature flag ON | Skips Requirements and Card Intro |
| 17 | Skip card | Any card-eligible country, user skips | Goes to Bank Details after currency selection |
| 18 | Assets fallback | User becomes assets-ineligible mid-flow | Falls back to Currency Selection |
| 19 | Resume flow | User exits mid-flow, returns | Resumes from last persisted step |
| 20 | Old iOS (< 11273) | iOS < 11273 | Not eligible for consumer onboarding flow |

---

## Platform Behavior Differences

| Platform | Behavior |
|----------|----------|
| **Android < 8.79** | Returns single step at a time, Card Intro filtered out |
| **iOS** (all versions) | Returns single step at a time |
| **Assets-eligible users** | Returns single step at a time (web can't handle batch + assets) |
| **Web / other** | Returns all remaining steps at once |
