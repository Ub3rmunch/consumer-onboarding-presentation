# Consumer Onboarding Flow

---

## Slide 1: Title

**Consumer Onboarding Flow**
_A complete guide for new team members_

- Team: Consumer Onboarding (Personal Onboarding)
- Presenter: ___
- Date: ___

> [VISUAL: Figma SOT cover page — "Personal Onboarding - Source of Truth" by Olli, Q2 2026]

---

# PART 1 — PRODUCT

_What problem we're solving, what the user sees, and how it differs across regions_

---

## Slide 2: What Is the Consumer Onboarding Flow?

The Consumer Onboarding Flow is the guided journey a new **personal** customer goes through after registration to set up their Wise account and adopt their first product.

Think of it as the bridge between "I just signed up" and "I'm actively using Wise."

**What it does:**
- Guides customers through a personalized series of steps based on who they are and where they are
- Lets them set up their multi-currency account, order a card, get bank details, or start sending money
- Adapts dynamically — customers in different countries see different steps in a different order

**Onboarding is "completed" when:**
- The customer adopted at least one product (card, bank details, balance, send), OR
- The customer explicitly skipped all options

**What it is NOT:**
- The registration flow — that happens before onboarding begins
- The business onboarding flow — that's a separate system for business accounts
- A single fixed screen sequence — it's highly dynamic per user

---

## Slide 3: When Does It Start and End in the UX?

### The User Journey

```
Sign up → Registration screens → [ONBOARDING STARTS HERE] → Product adoption → Dashboard
                                        ↑                           ↑
                                   First call to                Flow ends when user
                                   onboarding service           adopts a product or
                                                                skips everything
```

### Start
Immediately after registration completes. The app asks the onboarding service "what should this user do first?" and shows the appropriate step.

### End
When either:
- The customer completed at least one product setup (ordered card, got bank details, sent money, etc.)
- The customer explicitly skipped all remaining options
- There are no more steps to show

### Entry Points — How Users Enter the Flow
| Entry Point | Backend Value | When | Behavior |
|---|---|---|---|
| Registration | `REGISTRATION` | Just signed up | Full flow from the beginning |
| My Account | `ACCOUNT` | Returning user, didn't finish | Resumes from where they left off |
| Deep link | `DEEPLINK` | e.g., card tab, promotion | May skip intent, jump to a specific step |
| Content Management | `CONTENT_MANAGEMENT_WISE` | CMS-driven CTA | Routes to onboarding with CMS context |
| Receive launchpad | `LAUNCHPAD_RECEIVE` | Tapped "Receive money" | Receive-specific flow (no card steps) |
| Spend launchpad | `LAUNCHPAD_SPEND` | Tapped "Spend" on launchpad | Spend-specific flow |
| Young Explorer turning 18 | `YOUNG_EXPLORER_TURNING_18` | User aged into adult account | Transitions from Young Explorer to full onboarding |

> [VISUAL: Simple timeline — Registration → Onboarding Flow Starts → Steps → First Product Adopted → Dashboard]

---

## Slide 4: Flow Resuming — "You Haven't Finished Setting Up"

Most users don't complete onboarding in a single session. When a user drops off mid-flow and returns to the app, we prompt them to resume.

### What the User Sees

**On iOS — My Account alert banner:**
A warning-style inline alert in the My Account screen with a CTA button to resume onboarding.

**On Android — Two prompts:**
1. A survey bottom sheet on the Launchpad (home tab) when re-opening the app — captures feedback about where they dropped off
2. A "Finish onboarding" warning card in Account Management / Settings

**On Web:**
No proactive resume banner. The web relies on backend-driven routing — when a user navigates to `/onboarding` (e.g., from a link or CTA elsewhere), the server checks their flow state and either resumes the flow or redirects to the dashboard if already complete.

### How It Decides Whether to Show the Prompt
1. Call `GET /v1/consumer-onboarding/flow/status` — returns `IN_PROGRESS`, `COMPLETED`, or `INELIGIBLE`
2. If `IN_PROGRESS` → show the resume prompt
3. If `COMPLETED` or `INELIGIBLE` → don't show (and cache that decision)
4. The response also includes `lastStep` (where the user dropped off) and `shouldShowSurvey`

### When the User Taps "Resume"
The flow relaunches with a non-registration entry point (e.g., `ACCOUNT` instead of `REGISTRATION`). COS recalculates the steps from scratch — it doesn't "jump" to a saved position. Instead, it evaluates the user's current state (what they've already completed, what they're still eligible for) and returns whatever comes next.

This means if a user's eligibility changed since they dropped off (e.g., new card availability, compliance requirement), the resumed flow reflects the new reality.

> [VISUAL: Screenshots of iOS My Account alert, Android Launchpad survey, and the resume flow sequence]

---

## Slide 5: Next Best Action — The Evolution of Resume

In Q3 2025, the team evaluated **Resume Journey** (re-enter the onboarding flow) vs. **Next Best Action** (personalized checklist on Launchpad). NBA was chosen for its broader reach — it works for both drop-off users and users who completed onboarding but haven't explored other products.

### What the User Sees
A **"Your next steps" checklist** on the Launchpad (home screen):
- Up to 3 personalized actions with a progress tracker (e.g., 0/3)
- Collapsible, non-dismissible
- Auto-expires after 30 days or when all actions are completed

NBA is a Launchpad feature **owned by the Account Discovery team**, not the Consumer Onboarding team. It effectively replaces the "resume onboarding" banner approach for driving post-registration product adoption.

> [VISUAL: Screenshot of the NBA checklist on Launchpad from Figma — figma.com/design/IW7CDN0qh9BaMIbMR7BekL/Next-best-action]

---

## Slide 6: Eligibility — Who Gets the Onboarding Flow?

Not every registered user enters the consumer onboarding flow.

### Conditions for Entering the Flow
| Condition | Why |
|---|---|
| Personal account | Business accounts have a separate onboarding |
| Not a Young Explorer | Young Explorer has its own dedicated experience |
| Country supports multi-currency accounts | ~80 countries — if a country is send-only, the user goes straight to the transfer flow |
| Not already in a Send flow | If the user's intent is to send money, they skip onboarding |

### What Happens if a User is Ineligible?
- **Send-only countries** (~140 countries: HK, AE, TR, MX, TH, NG, etc.): Users go directly to the send/transfer flow — they never see onboarding
- **Blocked countries** (22 countries: AF, CU, IR, KP, RU, etc.): No Wise products available at all
- **Indonesia**: Specifically blocked from balance products despite being a large market (marketing decision)

> [VISUAL: World map or pie chart showing eligible vs. send-only vs. blocked]

---

## Slide 7: The Intent Picker — "What Do You Want to Do?"

Before diving into product setup, eligible users are asked what they want to do with Wise. This personalizes the rest of the flow.

### Intent Options
| Intent | What Happens Next |
|---|---|
| **Send money** | Goes directly to the transfer flow — exits onboarding |
| **Spend with card** | Goes to card order flow |
| **Receive/Hold money** | Goes to bank details setup |
| **Unified Onboarding** (default) | Full guided flow — the most common path |

### When the Intent Picker is Skipped
| Scenario | Behavior |
|---|---|
| Brazil | Auto-assigned to Unified Onboarding |
| Send-only countries | Auto-assigned to Send |
| User arrived with implicit intent (e.g., from a card promotion) | Intent is pre-set |
| UK Current Account flow | Shows a redesigned picker with "Get a current account" positioning |

> [VISUAL: Screenshot of Intent Picker UI from Figma SOT]

---

## Slide 8: The Five Flows

Based on eligibility and intent, users are routed to one of five distinct paths.

### Flow A — Pre-Intent
_User hasn't picked what they want to do yet._
Quick path: complete profile → verify identity (if needed) → pick intent → then route to one of the flows below.

### Flow B — Intent-Specific
_User picked Send, Spend, or Receive/Hold._
Minimal steps — just what's needed for their chosen intent. No full onboarding.

### Flow C — Standard (Default, Most Common)
_User chose Unified Onboarding or wasn't shown the intent picker._
The full guided experience through profile, verification, legal requirements, currency selection, card, and bank details.

### Flow D — Assets
_User is eligible for savings/investment products._
Same as Standard but includes an Assets step (replaces Currency Selection).

### Flow E — Receive
_User entered from the "Receive money" launchpad._
Focused on getting bank details — no card steps. Two sub-variants depending on assets eligibility.

> [VISUAL: Infographic "Flow Step Sequences" section — flows A–E with shared/unique/conditional steps color-coded]

---

## Slide 9: The Standard Flow — Step by Step

The most common path (~10 steps, each conditionally shown):

| # | Step | What the User Sees | Skipped When |
|---|---|---|---|
| 1 | **Identity** | ID verification setup | Profile already completed |
| 2 | **Profile** | Name, DOB, address, occupation | Profile already completed |
| 3 | **Verification** | KYC / identity verification | Not required by compliance |
| 4 | **Requirements** | Legal disclosures & what's needed | UK Current Account flow |
| 5 | **Currency Selection** | Pick currencies for multi-currency account | India |
| 6 | **Discoverability** | Receive/visibility settings | India |
| 7 | **Card Intro** | One-time card product introduction | India, UK Current Account |
| 8 | **Card Selection** | Choose card type (physical/virtual, Visa/MC) | Card already ordered |
| 9 | **Balance Consent** | MCA disclosure acceptance | Non-US countries |
| 10 | **Card Order / Bank Details** | Complete card order or get bank details | Depends on card choice |

Steps are not all shown to every user — each has conditions that determine whether it appears.

**Note:** Balance Consent position varies by flow. In the Assets flow (D) and Receive flows (E), it appears earlier — before Assets/Currency Selection. See Slide 14 for per-flow sequences.

> [VISUAL: Standard flow from infographic, row C]

---

## Slide 10: Country Product Availability

Products available per country determine which steps a user sees.

| Tier | Products Available | Countries | Example Flow |
|---|---|---|---|
| **Tier 1** — Full Suite | Send + Balance + Card + Bank Details + Assets | 33 (AU, BR, DE, FR, GB, US, SG...) | Standard or Assets flow |
| **Tier 2** — No Assets | Send + Balance + Card + Bank Details | 24 (CA, JP, NZ, BE, MY...) | Standard flow always |
| **Tier 3** — India Only | Send + Balance + Card (no Bank Details, no Assets) | 1 (IN) | Stripped flow: skips Currency, Card Intro, Discoverability |
| **Tier 4** — No Card | Send + Balance + Bank Details | 23 (AR, ZA, IL, KR, SA...) | Standard flow without card steps |
| **Tier 5** — Send Only | Send | ~140 (HK, AE, TR, MX, TH, NG...) | Not eligible for onboarding — goes to transfer flow |
| **Tier 0** — Blocked | None | 22 (AF, BY, CU, IR, KP, RU...) | No access |

> [VISUAL: Infographic "Country Product Availability" table]

---

## Slide 11: Country-Specific Exceptions

Beyond product tiers, some countries have unique onboarding behavior.

| Country | What's Different | Why |
|---|---|---|
| **US** | Extra "Balance Consent" step | MCA Disclosure (legal requirement) must be accepted before opening balances |
| **India** | Skips Currency Selection, Card Intro, Discoverability | Simplified flow — has card + balance but no bank details |
| **Indonesia** | Blocked from balance/card entirely | Marketing decision — despite being a large market, only send is available |
| **Philippines** | Card is blocked despite being in the product catalog | Regional override — balance and bank details only |
| **UK** | "Current Account" onboarding (permanent — rolled out to all eligible GB users) | Redesigned intent picker, skips Requirements and Card Intro, includes unboxing experience. Eligibility: profile = PERSONAL, country = GB, role ≠ YOUNG_EXPLORER |
| **Brazil** | Intent Picker is skipped | Users are auto-routed to Unified Onboarding |

> [VISUAL: Infographic "Country-Specific Exceptions" panel]

---

## Slide 12: Regional Flow Differences

### EU — Verification at Onboarding (CDD)
Since Feb 2026, EU residents go through **identity verification during onboarding** rather than waiting until their first transfer. This collects ID document with liveness check, account purpose, annual volume, and use case countries.

### UK — Current Account Onboarding
A permanent variant (originally feature-flagged, now fully rolled out) that positions Wise as a current account. Eligibility is deterministic: `CurrentAccountEligibilityService` checks profile type = PERSONAL, country = GB, role ≠ YOUNG_EXPLORER. Redesigned intent picker shows "Get a current account" with dynamic interest rates. After completing the flow, users get an "unboxing" experience revealing their GBP account details (sort code, account number).

Effective path: `Profile → Verify → Intent Picker → Currency Sel. → Discoverability → Card Sel. → Card Order / Bank Details → Done`

### Japan — eKYC
Uses electronic KYC (card-based "Tap to Verify" or manual entry) with identity-aware verification APIs.

### Brazil
Intent Picker is entirely skipped — users are auto-routed to Unified Onboarding.

### US/Canada
No KYC at zero — customers can send money under certain thresholds without triggering verification.

> [VISUAL: Map or table showing EU/UK/JP/BR/US with their distinctive flow characteristics]

---

## Slide 13: Flow Routing Decision Tree

How users get routed to one of the five flows:

```
User opens app
       ↓
   Eligible for onboarding?  ─── no ──→ [Send-only flow or blocked]
       ↓ yes
   Intent Picker eligible?   ─── no ──→ Standard (C) or Assets (D) flow directly
       ↓ yes
   Intent already chosen?    ─── no ──→ Flow A: Pre-Intent
       ↓ yes                            (Profile → Verify → Intent Picker)
   Chose UNIFIED?
   ├── yes ──→ Assets eligible?
   │           ├── yes ──→ Came from Receive? ─── yes ──→ Flow E (with Assets)
   │           │                               └── no ──→ Flow D: Assets
   │           └── no ──→ Came from Receive?  ─── yes ──→ Flow E (no Assets)
   │                                           └── no ──→ Flow C: Standard
   └── no ──→ Flow B: Intent-Specific
               (Send → Transfer | Spend → Card Order | Receive → Bank Details)
```

> [VISUAL: Full "Flow Routing Decision Tree" from infographic]

---

## Slide 14: Step Sequences by Flow

### Flow A — Pre-Intent
| Profile already done | No profile yet |
|---|---|
| Verify → Intent Picker | Identity → Profile → Verify |

### Flow B — Intent-Specific
| SEND | SPEND | RECEIVE/HOLD |
|---|---|---|
| Transfer Flow | Card Order Flow | Bank Details → Discoverability |

### Flow C — Standard
`Identity → Profile → Verify → Requirements → Currency Sel. → Discoverability → Card Intro → Card Sel. → [Balance Consent if US] → Card Order → Bank Details`

### Flow D — Assets
`Identity → Profile → Verify → Requirements → [Balance Consent if US] → Assets → Discoverability → Card Intro → Card Sel. → Card Order → Bank Details`

### Flow E — Receive
| With Assets | Without Assets |
|---|---|
| Identity → Profile → Verify → Reqs → [Bal. Consent US] → Assets → Discoverability → Bank Details | Identity → Profile → Verify → Reqs → Currency Sel. → [Bal. Consent US] → Discoverability → Bank Details |

> [VISUAL: Flow Step Sequences from infographic with color-coded shared/unique/conditional steps]

---

## Slide 15: Timeline — How the Flow Evolved

_The service was created in Sep 2019 (intent picker only). The Onboarding Flow API was designed in 2022 to replace client-hardcoded logic. The current architecture was established in 2023._

| When | What Changed |
|---|---|
| **Early 2023** | Unified Onboarding experiment launches. Flow endpoint goes live. Currency selection, card selection, requirements steps added. Rolled out to Android, then iOS. |
| **Mid 2023** | Tracking plan established. Discoverability settings step added. Intent picker redesigned (v4) for Unified Onboarding. |
| **Jan 2024** | V2 Flow API proposed (instructions-based, not implemented). |
| **Feb 2024** | Assets flow integration — savings/investment product added as a step. |
| **Mar 2024** | Profile moved upfront in the flow (was previously embedded mid-flow). |
| **Mid 2024** | Business onboarding added to the same service for maintenance efficiency. |
| **Q3–Q4 2025** | Home currency unboxing feature. Identity onboarding flow step. |
| **Q1 2026** | UK Current Account onboarding — redesigned intent picker, unboxing experience. Experiment fully rolled out, flag cleaned up. |
| **Feb 2026** | EU CDD goes live — verification moved to onboarding for EU. Top-up removal experiment. |
| **Q2 2026** | eKYC integration (Japan). Intent picker before registration (new logged-out experience). |

> [VISUAL: Horizontal timeline with key milestones marked]

---

## Slide 16: Tracking & Metrics

### What We Track
Two event prefixes depending on flow position:
- **`Consumer Onboarding Flow - {Step} - {Action}`** — events after the intent picker
- **`Consumer Get Started Flow - {Step} - {Action}`** — events before the intent picker (profile at onboarding)

| Event Pattern | Example |
|---|---|
| Flow lifecycle | `Flow - Started` / `Flow - Finished` |
| Step lifecycle | `Flow - Requirements - Started` / `Flow - Requirements - Finished` |
| Interactions | `Flow - Currency Selection - Searched`, `Flow - Card Selection - Changed Cards` |

### Tracking Conventions
- **Sticky properties:** Once a property (e.g., registration country) is known, it is attached to all subsequent events in the flow
- **Every word capitalized** in event names
- Events are sent to both **Mixpanel** (user behavior) and **Kibana** (structured logs) in parallel

### Key Properties Attached to Events
| Property | Example |
|---|---|
| Context (what launched the flow) | Intent Picker, Deep Link, Card Tab, My Account |
| Registration Country | GBR (ISO-3) |
| Selected Currencies | ["GBP", "EGP"] |
| Selected Card Program | VISA_DEBIT_CONSUMER_GB_1 |
| Did Skip Card | true / false |
| Result | Closed, Error, Exited, Ineligible |

### Data Model
`rpt_product.consumer_onboarding_flow` in Snowflake — user-level table with timestamps for every macro step from registration to first product adoption. Covers both Send and MCA funnels.

### KPIs (HEART Framework)
| Dimension | Metric |
|---|---|
| **Happiness** | Customer Effort Score (CES), CS contacts in first 30 days |
| **Adoption** | Onboarding flow conversion rate, % completing first job in 30 days |
| **Task Success** | Time to complete onboarding, error rate during onboarding |

---

## Slide 17: The Challenge — Drop-off

**65% of registered customers drop off before their first transaction.**

- ~29% verify their identity but never transact
- ~20% drop off at the top-up screen (experiment running to remove this requirement)
- The flow currently has 15 backend steps (via `OnboardingStepId` enum) — each is a potential drop-off point

> [VISUAL: Funnel diagram showing drop-off percentages at each macro step]

---

# PART 2 — TECH

_How it's built, how to change it, and where things live_

---

## Slide 18: Architecture Overview — Backend (COS)

**Consumer Onboarding Service (COS)** is a backend orchestrator — a state machine that tells clients what step to show next. All service-to-service calls go through **Envoy sidecar proxies** (`SERVICE_NAME.envoy.tw.ee:10101`).

```
Client app → GET /v1/consumer-onboarding/flow?currentStep=X&entryPoint=Y&intent=Z
                   ↓
         ┌─────────────────────────────────────┐
         │  1. Build OnboardingState           │ ← calls 14 external services via Envoy
         │  2. OnboardingStepsFactory          │ ← determines step sequence (flow A–E)
         │  3. OnboardingStepDeciderService    │ ← filters to next step(s)
         │  4. Return GetOnboardingResponse    │ ← nextSteps[] + requirements + status
         └─────────────────────────────────────┘
```

### The Interaction Loop
1. Client calls `GET /v1/consumer-onboarding/flow` (with optional `currentStep`)
2. COS responds with `nextSteps: ["PROFILE", "CURRENCY_SELECTION", ...]` + `status` + `requirements`
3. Client renders the first (or only) step
4. User completes step → state updates flow back via **Kafka events** or **direct service calls**
5. Client calls COS again with updated `currentStep` → COS returns new `nextSteps`
6. Repeat until `status: COMPLETED` or `nextSteps` is empty

### Key Backend Classes
| Class | Role |
|---|---|
| `OnboardingState` | Record holding all eligibility flags, completion flags, settings, user choices |
| `OnboardingStepsFactory` | Builds the ordered list of steps given state — 8 branches covering the A–E flow model plus sub-variants |
| `OnboardingStepDeciderService` | Filters steps based on current step, platform, and show allowance rules |
| `OnboardingStep` interface | Each step implements `showAllowance()` (ALWAYS / ONCE / NEVER) and optional `fallbackStepId()` |
| `OnboardingStepId` enum | All 15 step identifiers: ASSETS_INTEREST_FLOW, BALANCES_CONSENT, BANK_DETAILS_FLOW, CARD_ORDER_FLOW, CARD_INTRO, CARD_SELECTION, CURRENCY_SELECTION, DISCOVERABILITY_SETTINGS, IDENTITY_ONBOARDING_FLOW, INELIGIBLE, INTENT_PICKER, PROFILE, REQUIREMENTS, SEND_FLOW, VERIFICATION_FLOW |
| `Onboarding` entity | Persisted domain entity with all flow state |

### Step Show Allowance
- **ALWAYS**: Re-shown on every flow fetch (e.g., card order — user can resume)
- **ONCE**: Shown once, then marked done (e.g., intent picker)
- **NEVER**: Conditionally hidden

> [VISUAL: Architecture diagram from Confluence — High Level Diagram]

---

## Slide 19: Event-Driven State Updates (Kafka)

COS is a **stateless orchestrator** — it doesn't store step completion state itself. Instead, it learns about the user's progress by calling external services and consuming **Kafka events** from those services.

### Why This Matters
When a user orders a card, that order is processed by `twcard-order`, not COS. COS only finds out when either:
1. It calls the card order service on the next `/flow` request, or
2. It consumes a Kafka event published by the card order service

This is why **clients need retry logic** — the user completes a step, the client calls COS, but COS may not yet know the step is done.

### Kafka Events COS Consumes

| Topic | Source Service | What COS Learns |
|---|---|---|
| `twcard.order.events.orderStatusChanged` | Card Order | Card was ordered/delivered/failed |
| `depositAccount.AccountCreated` | Deposit Account | Bank details are ready |
| `terms.consent.created` | Terms Service | US balance consent accepted |
| `TransferService.transferStateChange` | Transfer Service | Send flow completed |
| `ProfileService.profileUpdated` | Profile Service | Profile was completed |
| `User.Created` | User Service | New user registered (creates onboarding record) |
| `AccountPlans.out.PlanOrderStateChanged` | Account Plans | Plan order state changed |
| `ViralityService.referralStateChanged` | Virality | Referral bonus applied |
| `ViralityService.freeTransferDiscountUsed` | Virality | Free transfer used |
| `PublicApiAuthorization.authorizationEvent` | Public API | API key authorization events |

### Kafka Events COS Produces

| Topic | When |
|---|---|
| `consumerOnboarding.intentPicked` | User selects an intent in the intent picker |
| `Onboarding.out.BusinessOnboardingStateChanged` | Business onboarding state transitions |

### Resilience
- Kafka consumers use `DefaultErrorHandler` with fixed backoff retry (default 500ms)
- Auto-commit disabled — messages are only committed after successful processing
- HTTP calls to external services use Spring `@Retryable` (3 attempts, 3s backoff) with graceful fallback to "ineligible" on exhaustion
- Eligibility data is cached using Caffeine (in-memory) with configurable TTL per cache type

---

## Slide 20: How Clients Talk to COS

All three clients follow the same interaction loop. COS is a **stateless orchestrator** — it doesn't push steps to clients. Clients drive the loop by polling.

```
┌──────────┐                              ┌──────────┐
│  Client  │                              │   COS    │
└────┬─────┘                              └────┬─────┘
     │  GET /flow?entryPoint=REGISTRATION      │
     │ ───────────────────────────────────────→ │  ← builds OnboardingState, picks flow A–E,
     │                                         │    filters to next step(s)
     │  { nextSteps: ["PROFILE"], status: ... } │
     │ ←─────────────────────────────────────── │
     │                                         │
     │  [render PROFILE screen]                │
     │  [user fills in name, DOB, address]     │
     │  [profile saved via Profile Service]    │
     │                                         │
     │  GET /flow?currentStep=PROFILE          │
     │ ───────────────────────────────────────→ │  ← re-evaluates from scratch
     │                                         │
     │  { nextSteps: ["VERIFICATION"], ... }    │
     │ ←─────────────────────────────────────── │
     │                                         │
     │  ... repeat until nextSteps=[] or       │
     │      status=COMPLETED ...               │
```

### The Retry Problem
Some steps complete asynchronously (e.g., card order via Kafka). The client calls COS but the backend hasn't processed the event yet, so COS returns the **same step again**. All clients handle this with a retry loop — poll up to 5 times with backoff until `nextSteps` changes.

Note: client-side polling retry is separate from server-side retry. The backend `ServiceCallExecutor` uses `@Retryable` (3 attempts, 3s fixed backoff) for calls to external services. `OnboardingFetcherService` uses 3 attempts with 500ms backoff.

### How Many Steps Does COS Return?
COS parses the `User-Agent` header to decide:

| Client | Steps per Response | Why |
|---|---|---|
| **Web** | All remaining steps at once | SPA — renders them in sequence via hash routing |
| **iOS** | One step at a time | UIKit push/pop — one screen per navigation event |
| **Android** | One step at a time | Conductor — one Fragment per state transition |
| **Any client + assets-eligible user** | One step at a time | Web can't handle batch + assets together |

---

## Slide 21: Web — Architecture

**Repo:** `consumer-onboarding-web` | **Stack:** Next.js 15, React 18, TypeScript

```
┌─ Server ──────────────────────────────────────────────────┐
│  pages/onboarding.tsx (getServerSideProps)                 │
│  • First COS call happens here (SSR)                      │
│  • If status=COMPLETED → redirect to dashboard            │
│  • Maps URL origin param → entryPoint                     │
└───────────────────────────┬───────────────────────────────┘
                            │ passes initial flow data as props
┌─ Browser ─────────────────▼───────────────────────────────┐
│                                                           │
│  ┌─ OnboardingFlow ─────────────────────────────────────┐ │
│  │  Wrapper: sets up tracking context, error boundaries  │ │
│  │                                                       │ │
│  │  ┌─ CustomFlow (core orchestrator) ────────────────┐ │ │
│  │  │  State machine: loading → ready → redirect       │ │ │
│  │  │                  → complete                      │ │ │
│  │  │                                                  │ │ │
│  │  │  On step complete:                               │ │ │
│  │  │   1. Call COS with updated currentStep           │ │ │
│  │  │   2. If same step → waitForNewSteps()             │ │ │
│  │  │      (5 retries, linear backoff: 4s, 8s, 12s,   │ │ │
│  │  │       16s, 20s — up to 60s total wait)           │ │ │
│  │  │   3. If new step → update hash route             │ │ │
│  │  │   4. If subflow (card/bank) → URL redirect       │ │ │
│  │  └──────────────────────────────────────────────────┘ │ │
│  │                                                       │ │
│  │  ┌─ Step Registry (constants.ts) ──────────────────┐ │ │
│  │  │  { "PROFILE": ProfileStep,                      │ │ │
│  │  │    "CARD_SELECTION": CardSelectionStep,          │ │ │
│  │  │    "CURRENCY_SELECTION": CurrencyStep, ... }    │ │ │
│  │  └──────────────────────────────────────────────────┘ │ │
│  └───────────────────────────────────────────────────────┘ │
│                                                           │
│  ┌─ Shared packages ────────────────────────────────────┐ │
│  │  consumer-onboarding-api    → typed API client       │ │
│  │  consumer-onboarding-types  → shared TS interfaces   │ │
│  │  consumer-onboarding-utils  → HTTP helpers           │ │
│  └──────────────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────────────┘
```

**Web-specific quirks:**
- Navigation is hash-based (`/#profile`, `/#card-selection`) via React Router v5 `HashRouter` with `hashType="noslash"` — all steps live on one page
- Subflows (card order, bank details) use **full URL redirects** via `window.location.assign()` — the user leaves the SPA and returns after completion
- State is managed with `useRef` (non-reactive store) + `useState` (reactive flow status) + a custom `CustomFlowContext` (React Context) for cross-component state sharing. No Redux
- Gets all remaining steps in one COS call — renders them sequentially
- Web defines **18 step keys** — more than the backend's 15, because it splits some backend steps into sub-steps (e.g., `CURRENCY_SELECTION_INTRO`, `CURRENCY_SELECTION_SUCCESS`, `PROFILE_NAME`, `PROFILE_AGE`, `PROFILE_ADDRESS`). This is a concrete example of the client-side duplication from Slide 24

---

## Slide 22: Android — Architecture

**Repo:** `transferwise-android` → `feature/unified-onboarding/` | **Stack:** Kotlin, Compose, Hilt

```
┌─ Entry ──────────────────────────────────────────────────┐
│  UnifiedOnboardingActivity                                │
│  • Launched from 11 different places in the app            │
│  •   (UnifiedOnboardingSource enum: REGISTRATION,         │
│  •    INTENT_PICKER, CARD_TAB, SETTINGS, DEEPLINK,        │
│  •    PAYMENTS, ASSETS, CONTENT_MANAGEMENT_WISE,           │
│  •    LAUNCHPAD_RECEIVE, LAUNCHPAD_SPEND, DEBUG)           │
│  • Registers StepAdapters for each step type              │
└───────────────────────────┬──────────────────────────────┘
                            │
┌─ Flow Control ────────────▼──────────────────────────────┐
│  UnifiedOnboardingFlowViewModel                           │
│  • Calls COS, receives single nextStep                    │
│  • Maps step string → FlowSteps sealed class              │
│  • Dispatches to Conductor for rendering                  │
│                                                           │
│  ┌─ Conductor ─────────────────────────────────────────┐ │
│  │  Custom Fragment state machine (not Jetpack Nav)     │ │
│  │  • Each step is a Fragment or StepAdapter            │ │
│  │  • Manages Fragment lifecycle + transitions           │ │
│  │  • On step complete → ViewModel calls COS again      │ │
│  └─────────────────────────────────────────────────────┘ │
│                                                           │
│  Subflows: card order, bank details, send, verification   │
│  → launched as separate Activities via HandoffActivity     │
│  → result returned via Activity result callback            │
└──────────────────────────────────────────────────────────┘

┌─ Module Structure (Clean Architecture) ──────────────────┐
│                                                           │
│  presentation  → Activity, ViewModel, UI (public API)     │
│       ↓                                                   │
│  core          → FlowSteps sealed class, domain models    │
│       ↓                                                   │
│  core-impl     → Retrofit API service, interactor (retry),│
│                  repository                                │
│       ↓                                                   │
│  android-common → Navigator, shared utilities              │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

**Android-specific quirks:**
- Uses a custom **Conductor** library (in-house Fragment state machine with `StepAdapter` pattern), not Jetpack Navigation
- Subflows launch as separate Activities via `UnifiedOnboardingFlowHandoffActivity` — the onboarding Activity pauses and resumes on return
- `FlowSteps` is a sealed class with 14 concrete steps + `Unknown` — unknown steps are retried up to 5 times via `retryOn()` in the interactor, then fail with error
- Gets one step at a time from COS
- Also has a separate `WiseUnder18Feature` + `WiseUnder18FlowActivity` for users under 18 — a distinct onboarding variant outside the main A–E flow model

---

## Slide 23: iOS — Architecture

**Repo:** `transferwise-ios` → `Wise/OnboardingKit/ConsumerOnboarding/` | **Stack:** Swift, UIKit + SwiftUI

```
┌─ Entry ──────────────────────────────────────────────────┐
│  ConsumerOnboardingFlow (Neptune Flow<Result>)            │
│  • Conforms to Wise's Flow framework — has start/cancel   │
│  • Entry points: .registration, .account, .deeplink,      │
│    .launchpad, .launchpadReceive, .launchpadSpend,         │
│    .contentManagement                                      │
└───────────────────────────┬──────────────────────────────┘
                            │
┌─ Flow Control ────────────▼──────────────────────────────┐
│  StepCoordinatorImpl                                      │
│  • Holds immutable FlowState (updated via keypath lenses) │
│  • Calls COS, receives single nextStep                    │
│  • Maps step → StepFactory → UIViewController or SwiftUI  │
│  • On step complete → updates state, calls COS again      │
│                                                           │
│  ┌─ Router ────────────────────────────────────────────┐ │
│  │  UINavigationController-based                        │ │
│  │  • push/pop for step transitions                     │ │
│  │  • modal presentation for some sub-flows             │ │
│  └─────────────────────────────────────────────────────┘ │
│                                                           │
│  Subflows: card, bank details, verification, profile,     │
│  transfer, assets                                         │
│  → injected as factory closures (protocol-based)          │
│  → coordinator starts sub-flow, awaits result callback    │
└──────────────────────────────────────────────────────────┘

┌─ Step Rendering ─────────────────────────────────────────┐
│                                                           │
│  StepFactory.mapStep("PROFILE") → ProfileViewController   │
│  StepFactory.mapStep("CARD_SELECTION") → CardSelView      │
│  ...                                                      │
│                                                           │
│  Steps are a mix of UIKit ViewControllers and SwiftUI     │
│  views wrapped in UIHostingController                     │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

**iOS-specific quirks:**
- Built on **Neptune Flow Framework** — Wise's internal abstraction for multi-step flows with typed results (`Flow<ConsumerOnboardingFlowResult>`)
- Navigation is UIKit (`UINavigationController`) even though step UIs are increasingly SwiftUI
- State is immutable — updates happen via **keypath lenses** (`state.updating(\.result, .orderedCard)` returns a new copy), not mutable properties
- Sub-flows are injected as `Injectable*FlowFactory` closures (e.g., `InjectableOrderCardFlowFactory`, `InjectableBankDetailsFlowFactory`) — the coordinator doesn't know the concrete types
- `FeatureService` is injected via `@Dependency` but currently **unused** in the flow factory — all feature flag decisions happen server-side
- Gets one step at a time from COS

---

## Slide 24: Cross-Platform Comparison

### Architectural Layers (same concept, different implementation)

| Responsibility | Web | Android | iOS |
|---|---|---|---|
| **Entry point** | `pages/onboarding.tsx` (SSR) | `UnifiedOnboardingActivity` | `ConsumerOnboardingFlow` |
| **Orchestrator** | `CustomFlow` component | `FlowViewModel` + Conductor | `StepCoordinatorImpl` |
| **Step registry** | Object map in `constants.ts` | `FlowSteps` sealed class | `StepFactory` switch |
| **Navigation** | Hash routing (`/#step`) | Custom Conductor (Fragments) | UINavigationController push/pop |
| **Subflow launch** | URL redirect (leaves SPA) | `HandoffActivity` (new Activity) | Injected factory → coordinator |
| **Subflow return** | Page reload + re-fetch | Activity result callback | Flow result callback |
| **State management** | `useRef` + `useState` + Context | ViewModel + Repository | Immutable state + keypath lens |
| **Retry on same step** | `waitForNewSteps()` 5x, linear backoff (4s–20s) | `retryOn()` 5x in interactor | `callService()` 5 consecutive, then `failedToFetchValidStep` error |

### What Every Client Duplicates
All three clients independently implement the same logic because COS returns data, not instructions:

| Duplicated Logic | What It Means |
|---|---|
| **Step registry** | Every new step requires a client release on all 3 platforms |
| **Exit conditions** | Each client has its own "when to leave the flow" — subtle inconsistencies |
| **Retry/polling** | All implement ~5x retry for async processing lag |
| **Subflow invocation** | Each client decides how to launch card order, bank details, verification |
| **Tracking** | Each client composes and fires its own Mixpanel events independently |
| **Feature flag checks** | Scattered inline checks, not always in sync across platforms |

This duplication is the core motivation behind the V2 API RFC (Slide 28) — move this logic server-side so clients just execute instructions.

---

## Slide 25: Flow Resuming — Implementation

How each platform detects and handles users returning to finish onboarding.

### Backend: Status Endpoint

`GET /v1/consumer-onboarding/flow/status` — lightweight check that returns:

| Field | Values | Purpose |
|---|---|---|
| `status` | `IN_PROGRESS` / `COMPLETED` / `INELIGIBLE` | Should the client prompt to resume? |
| `lastStep` | e.g., `CURRENCY_SELECTION` | Where the user dropped off |
| `shouldShowSurvey` | `true` / `false` | Should the client show a feedback survey? |

`OnboardingFlowStatusService` resolves status by checking the persisted `Onboarding` entity and re-evaluating eligibility. The `lastStep` comes from `OnboardingLastStepService`, which records the most recent step returned to any client.

When the user resumes, the client calls the main `GET /v1/consumer-onboarding/flow` with a non-`REGISTRATION` entry point (e.g., `ACCOUNT`, `DEEPLINK`). COS treats this as a fresh evaluation — it rebuilds `OnboardingState`, runs `OnboardingStepsFactory`, and returns whatever step comes next given the user's current state.

### iOS: `ConsumerOnboardingFlowChecker`

```
MyAccountPresenter
   → ConsumerOnboardingFlowChecker.checkIfShouldShowFlow()
      → Is profile PERSONAL? ──── no → return (don't show)
      → Was result cached in last 7 days? (UserDefaults) ──── yes → use cached
      → GET /v1/consumer-onboarding/flow/status
         → IN_PROGRESS → show alert banner in My Account
         → COMPLETED / INELIGIBLE → cache "don't show", hide banner
   → User taps "Resume" → ConsumerOnboardingFlowRouter
      → launches flow with entryPoint: .account
```

The 7-day `UserDefaults` cache avoids hitting the status endpoint on every My Account visit. Cache is keyed per profile and invalidated when the flow completes.

### Android: Status Interactor + Survey + Settings Card

**Survey (Launchpad):**
```
LaunchpadViewModel
   → GetConsumerOnboardingFlowStatusInteractorImpl
      → GET /v1/consumer-onboarding/flow/status (FetchType.Fresh — no cache)
      → IN_PROGRESS + shouldShowSurvey → show survey bottom sheet
```

**Settings card (Account Management):**
```
AccountManagementItemGenerator
   → checks flow status from shared repository
   → IN_PROGRESS → adds "Finish setting up your account" warning card
   → User taps → UnifiedOnboardingNavigator
      → launches UnifiedOnboardingActivity with UnifiedOnboardingSource mapping
```

`UnifiedOnboardingSource` enum (11 values: REGISTRATION, INTENT_PICKER, CARD_TAB, SETTINGS, DEEPLINK, PAYMENTS, ASSETS, CONTENT_MANAGEMENT_WISE, LAUNCHPAD_RECEIVE, LAUNCHPAD_SPEND, DEBUG) maps to `EntryPointRequestParameter` when calling COS.

### Web: SSR Origin Mapping

Web has no proactive resume banner. Instead, `pages/onboarding.tsx` handles resume via server-side rendering:

```
getServerSideProps
   → GET /v1/consumer-onboarding/flow (NOT /flow/status)
   → If status COMPLETED → redirect to /
   → If steps remain → render flow, pass origin as entryPoint
```

Origin mapping in SSR:
| Origin | Entry Point |
|---|---|
| `INTENT_PICKER` / `MCAHP` / `STANDARD` | `REGISTRATION` |
| `MY_ACCOUNT` | `ACCOUNT` |
| `CONTENT_MANAGEMENT_WISE` | `CONTENT_MANAGEMENT_WISE` |
| `LAUNCHPAD_RECEIVE` | `LAUNCHPAD_RECEIVE` |
| `LAUNCHPAD_SPEND` | `LAUNCHPAD_SPEND` |
| `ASSETS_INTEREST_FLOW` | `LAUNCHPAD_RECEIVE` (GB + flag) or undefined |
| `YOUNG_EXPLORER_TURNING_18` | `YOUNG_EXPLORER_TURNING_18` |
| Any other / unknown | `undefined` (falls back to `REGISTRATION`) |

Web does NOT use the `/flow/status` endpoint — it relies entirely on the main `/flow` endpoint for both initial and resume loads.

### Platform Comparison — Resume Behavior

| | iOS | Android | Web |
|---|---|---|---|
| **Proactive prompt** | My Account alert banner | Launchpad survey + Settings card | None |
| **Status check** | `/flow/status` (cached 7 days) | `/flow/status` (no cache) | `/flow` directly (SSR) |
| **Entry point** | `.account` | `UnifiedOnboardingSource` mapping | Origin param in URL |
| **Cache** | UserDefaults, 7-day TTL | No cache (fresh every time) | No cache (SSR per request) |
| **Survey** | Not shown proactively | Bottom sheet on Launchpad | Not implemented |

---

## Slide 26: Relevant Endpoints

### Public V1 (Primary — used by all clients)
| Method | Path | Purpose |
|---|---|---|
| `GET` | `/v1/consumer-onboarding/flow` | Main flow endpoint — returns next steps + requirements + status |
| `GET` | `/v1/consumer-onboarding/flow/status` | Flow completion status |
| `GET` | `/v1/consumer-onboarding/eligibility` | Product eligibility |
| `GET` | `/v1/consumer-onboarding/availability` | Product availability by country |
| `GET` | `/v1/consumer-onboarding/intent-picker` | Intent picker UI data |
| `GET` | `/v1/consumer-onboarding/currencies` | Suggested balance currencies |
| `POST` | `/v1/consumer-onboarding/currencies` | Save selected currencies |

### Public V2 / V3
| Method | Path | Purpose |
|---|---|---|
| `POST` | `/v2/consumer-onboarding/intent` | Save customer intent |
| `GET` | `/v2/consumer-onboarding/intent-picker` | Intent picker v2 (grid + highlights for UK CA) |
| `GET` | `/v2/consumer-onboarding/card-types` | Card selection options v2 |
| `POST` | `/v2/consumer-onboarding/card-types` | Select card program |
| `GET` | `/v2/consumer-onboarding/eligibility` | Eligibility v2 (deprecated — use v3) |
| `GET` | `/v3/consumer-onboarding/eligibility` | Eligibility v3 (latest) |
| `GET` | `/v3/consumer-onboarding/availability` | Availability v3 (deprecated) |

### Internal (service-to-service)
| Method | Path | Purpose |
|---|---|---|
| `GET` | `/users/{userId}/consumer-onboarding/flow/status` | Flow status lookup |
| `PUT` | `/users/{userId}/product-adoption` | Track feature adoption |
| `GET` | `/profiles/{profileId}/unboxing` | Unboxing eligibility + content |
| `GET` | `/users/{userId}/intent` | Customer intent data |

---

## Slide 27: External Service Dependencies

COS integrates with 14 upstream services via Envoy to compute eligibility and state.

| Service | Client | What It Decides |
|---|---|---|
| **Balance Service** | `BalanceServiceClientImpl` | Balance/MCA eligibility per country |
| **Card Order Service** (gRPC internally, REST via Envoy) | `CardOrderClientImpl` | Card availability, orders, order status |
| **Card Program Service** | `CardProgramClientImpl` | Card program options per country (separate from Card Order) |
| **Card Management** | `TwCardManagementClientImpl` | Card management operations |
| **Deposit Account Service** | `DepositAccountServiceClientImpl` | Bank details currencies per country |
| **Hold BFF / Assets** | `AssetsClientImpl` → `hold-bff.envoy.tw.ee` | Assets/investment eligibility per user. Hold BFF is a BFF aggregating balance, assets, and MCA data |
| **Mitigation Requirements** | `MitigationRequirementsClientImpl` | Whether verification/CDD is **needed** (`shouldCompleteVerification`) |
| **Mitigator** | `MitigatorClientImpl` | The actual verification **flow state** — transforms requirements into UI. Used for EU CDD identity verification |
| **Terms Service** | `TermsClientImpl` | US MCA disclosure consent status |
| **NPS Service** | `NpsServiceClientImpl` | Net Promoter Score / feedback collection |
| **Feature Service** | via Wise Feature Service SDK | Feature flags (discoverability, CDD, unboxing, etc.) |
| **Profile Identifier** | `ProfileIdentifierClientImpl` | Profile lookups |
| **Route Availability** | `RouteAvailabilityClientImpl` | Available currencies and routes (not the same as the `availability` service — Route Availability handles currency routing, while `availability` handles product availability by country) |
| **Bank Details Order** | `BankDetailsOrderClientImpl` | Bank account order management |

### Failure Behavior (COS is a T1 service)
| Dependency Fails | COS Behavior |
|---|---|
| Balance/Card service | User appears ineligible for those products |
| Hold BFF (Assets) | User not eligible for assets |
| Mitigation Requirements timeout | Skip verification step |
| Terms service missing | Require consent (safe default) |

> [VISUAL: Network diagram — COS at center with arrows to each dependency]

---

## Slide 28: Attempts at Flow V2 API

### V1 Pain Points
- Clients hardcode exit logic, retry logic, step invocation, tracking
- Can't add new steps without mobile app releases
- Tracking inconsistent across 3 platforms
- No explicit way to exit the flow
- Ineligibility handled as a step, not an error

### The V2 Proposal (RFC, Jan 2024)
An instructions-based API was proposed where the server returns declarative **instructions** (show a view, fire a tracking event, exit the flow, wait for async processing) instead of raw flow state. This would move step registry, retry logic, exit conditions, and tracking composition server-side — eliminating the client duplication described in Slide 24.

**Status:** 38 revisions through Feb 2024, never implemented. A companion client-side V2 RFC also exists. The V1 flow API remains the only flow endpoint. V2/V3 controllers exist in the codebase, but for **non-flow endpoints only**: eligibility (V2, V3), availability (V2, V3), intent picker (V2), card selection (V2), and customer intent (V2).

---

## Slide 29: Observability

### Backend O11y (COS)
COS uses a custom **MonitorEvent** pattern — a thin abstraction over Micrometer + Slf4j.

```
Feature code → MonitorEvent → Monitor (facade) → Micrometer counter + Structured log
```

| Component | Role |
|---|---|
| `MonitorEvent` | Interface: name + properties. One implementation per scenario. |
| `MonitorPropertyMapper` | Reusable mapper: domain model → properties (e.g., ClientInfo, OnboardingState) |
| `MeterMonitor` | Facade that sends events to both Micrometer and structured Slf4j |

### Key Events Monitored
| Event | What It Tracks |
|---|---|
| `ConsumerOnboardingFlowRequestEvent` | Every `/flow` call |
| `ConsumerOnboardingFlowStepEvent` | Which step was returned |
| `IntentPickedMonitorEvent` | Intent picker choices |
| `CardOrderMonitorEvent` / `BankDetailOrderMonitorEvent` | Product order lifecycle |
| `OnboardingRequirementFulfillmentEvent` | Requirements completion |

### Frontend O11y
- **Mixpanel** for user behavior analytics (all platforms)
- **Rollbar** for error capture (web)
- **Kibana** for structured logs (same events as metrics, queryable)
- **Prometheus/Grafana** for service metrics and alerting

---

## Slide 30: Feature Flags & Adoption Tracking

### Feature Flags
COS uses Wise Feature Service for toggling behavior. Key flags:

| Flag | Purpose | Status |
|---|---|---|
| `CDD_EU_V2` | EU CDD verification at onboarding | Active |
| `account-unboxing` / `account-structure` | Home currency unboxing experience | Active (`account-structure` has a TODO to remove after full release) |
| `sportsbar_sponsorship_2026` | Suppresses survey during sponsorship campaign | Active |
| `business-onboarding-free-tier-eea` | Free tier eligibility for EEA business onboarding | Active |
| `intent-picker-receive-phl` | Philippines receive intent experiment | Active |

**Recently removed:** `current-account-onboarding-uk` — cleaned up after successful rollout. UK Current Account eligibility is now permanent via `CurrentAccountEligibilityService`.

Flags are also used on clients (e.g., `launchpad_collapsible_items` on Android, `first-job-adoption-v1` on Web, `phpReceiveEnabled` on Web).

### Feature Adoption Tracking
Separate from flags — tracks which products users actually adopted:

| Product | Tracked States |
|---|---|
| Card | SHOWN → COMPLETED / SKIPPED |
| Bank Details | SHOWN → COMPLETED / SKIPPED |
| Assets | SHOWN → COMPLETED / SKIPPED |
| Send | SHOWN → COMPLETED / SKIPPED |

Stored in a `feature_adoption` table. Consumed by internal APIs and analytics.

---

## Slide 31: Tech Debt & Known Challenges

### Business Onboarding in COS
Business onboarding was added in late 2024 and shares the same service. Package-level separation exists (`com.transferwise.onboarding.consumer` vs `.business`), but deeper layers are coupled: shared Flyway migrations, shared connection pool, shared Spring context startup, and shared Kafka processing. A failure in business code can impact consumer onboarding.

### V1 API Limitations
- All clients hardcode: exit logic, retry logic, step invocation, tracking composition
- Adding a new step requires mobile app releases on all platforms
- No explicit exit mechanism — clients guess when to leave
- Tracking is inconsistent across 3 platforms despite a shared tracking plan
- User-Agent parsing determines platform behavior instead of explicit capabilities

### Other
- 15 backend steps (clients may define more — web has 18 step keys including sub-steps) — each is a potential drop-off point
- Ineligibility is modeled as a "step" rather than an error state
- Batch step return (web gets all steps) complicates per-step observability

---

## Slide 32: Deployment & Operations

### Deployment Pipeline
COS uses **Continuous Deployment via Spinnaker** with automated canary analysis.
- Every merge to master auto-rolls to production (no manual deploy step)
- Spinnaker config: `spinnaker-config/services/consumer-onboarding-service/config.libsonnet`
- Canary analysis gates the rollout — if error rates spike, Spinnaker rolls back automatically

### Service Maturity
COS is a **T1 (Gold maturity)** service. Maturity can drop below Gold if:
- Rollback rate exceeds 1%
- Canary analysis coverage drops below 90%
Recovery is typically time-based (wait for the next clean deploy cycle), not an immediate manual action.

### Dependency Updates
**Code Update Service** bot opens PRs for dependency bumps (from `wise-java-platforms` BOM) every Monday and Friday. These are routine — review the changelog, check CI passes, merge.

### Team-Owned Services
The Consumer Onboarding team monitors 5 services for Gold maturity:

| Service | What It Does |
|---|---|
| `consumer-onboarding-service` | Backend orchestrator (this service) |
| `web-onboarding` | Web frontend |
| `availability` | Product availability API |
| `public-profile-service` | Public profile data |
| `tracking-collector` | Analytics event collection |

### Testing Reality
- **Backend:** JUnit + WireMock integration tests. Parallel test execution with per-fork DB isolation (since Q1 2026)
- **Web:** Cypress E2E tests against staging
- **Android/iOS:** Primarily **manual testing in production** — staging limitations make full-flow testing unreliable (test cards don't work, KYC requires manual backoffice approval)
- No automated cross-platform E2E test suite

### Scheduled Jobs
COS runs one **batch notification job** (business onboarding): sends emails to business users without profiles, 2x/day. Uses leader election (`ServiceLeaderSelector`) so only one instance executes. Kill switch: `business-onboarding-profile-pending-notification` feature flag.

### Post-Merge Monitoring
- **Grafana** JVM HTTP dashboard for latency/error rate
- **Rollbar** for error aggregation
- **Kibana** for structured log queries
- **Spinnaker** pipeline view for rollback if needed

---

## Slide 33: Key Code Locations

### Backend (`consumer-onboarding-service`)
| File | Purpose |
|---|---|
| `consumer/rest/v1/ConsumerOnboardingFlowController.java` | Main flow endpoint |
| `consumer/service/onboarding/OnboardingStepsFactory.java` | Step sequence builder (flow A–E logic) |
| `consumer/service/onboarding/OnboardingStepDeciderService.java` | Next step(s) + platform filtering |
| `consumer/domain/onboarding/step/OnboardingStepId.java` | Step enum (15 steps) |
| `consumer/domain/onboarding/step/OnboardingState.java` | State record |
| `consumer/service/ConsumerOnboardingFlowEligibilityService.java` | Flow-level eligibility |
| `consumer/service/onboarding/regional/RegionalOnboardingFlowService.java` | Country-specific overrides |
| `consumer/platform/observability/MeterMonitor.java` | O11y facade |

### Web (`consumer-onboarding-web`)
| File | Purpose |
|---|---|
| `apps/web-onboarding/pages/onboarding.tsx` | SSR entry point |
| `apps/web-onboarding/flows/CustomFlow/CustomFlow.tsx` | Core orchestrator |
| `apps/web-onboarding/flows/OnboardingFlow/constants.ts` | Step registry |
| `packages/consumer-onboarding-api/.../fetchOnboardingFlow.ts` | API client |

### Android (`transferwise-android`)
| File | Purpose |
|---|---|
| `feature/unified-onboarding/.../UnifiedOnboardingActivity.kt` | Entry point |
| `feature/unified-onboarding/.../UnifiedOnboardingFlowViewModel.kt` | Flow VM |
| `feature/unified-onboarding/.../UnifiedOnboardingResultStepMapper.kt` | Step registry |
| `feature/unified-onboarding/.../FlowSteps.kt` | Step definitions |

### iOS (`transferwise-ios`)
| File | Purpose |
|---|---|
| `.../ConsumerOnboarding/Flow/ConsumerOnboardingFlow.swift` | Main flow |
| `.../Flow/ConsumerOnboardingFlowStepCoordinatorImpl` | Step sequencing |
| `.../Flow/ConsumerOnboardingFlowStepFactory.swift` | Step registry |
| `.../Flow/Service/ConsumerOnboardingFlowService.swift` | API client |

---

## Slide 34: Quick Reference — Key Links

| Resource | Location |
|---|---|
| **SOT Figma** | figma.com/design/8QhoOlv45zPvZkPzHUgcXC/SOT-Consumer-Onboarding-Flows |
| **Backend Repo** | github.com/transferwise/consumer-onboarding-service |
| **Web Repo** | github.com/transferwise/consumer-onboarding-web |
| **Architecture Overview** | Confluence: Consumer Onboarding - Architecture Overview |
| **V2 RFC** | Confluence: RFC: Consumer Onboarding Flow V2 API |
| **Tracking Plan** | Confluence: Consumer Onboarding Flow Tracking - Tracking Plan |
| **O11y Implementation** | Confluence: Consumer Onboarding Service Observability Implementation |
| **On-call Runbook** | Confluence: Consumer Onboarding – On-call Rota and Runbook |
| **Engineer Onboarding** | Confluence: Consumer Onboarding - Engineer Onboarding Guide |
| **Snowflake Data** | `rpt_product.consumer_onboarding_flow` |
| **Slack** | `#consumer-onboarding-private`, `#consumer-onboarding-public` |

---

## Slide 35: Q&A

Questions?

---

_Last updated: April 2026_
_Data sources: COS codebase, consumer-onboarding-web, transferwise-android, transferwise-ios, Confluence (C20 space), Figma SOT, availability API, git history_
