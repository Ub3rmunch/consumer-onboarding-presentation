# Consumer Onboarding Flowchart

```
                         ┌─────────────────────┐
                         │    Registration      │
                         │  Country, acct type, │
                         │  optional intent     │
                         └──────────┬──────────┘
                                   ...
                         ┌──────────────────────┐
                         │  Finish Registration  │
                         └──────────┬───────────┘
                                    │
                              ┌─────┴─────┐
                              │ Account   │
                              │  type?    │
                              └─────┬─────┘
                         Business   │   Personal
                      ┌─────────────┘─────────────┐
                      ▼                           ▼
              ┌──────────────┐            ┌──────────────┐
              │  Business    │            │   Profile    │
              │  Onboarding  │            └──────┬───────┘
              │  (separate)  │                   │
              └──────────────┘                   ▼
                                    ┌────────────────────┐
                                    │ Verification       │
                                    │ needed?            │
                                    │ *skip if not EU    │
                                    └─────────┬──────────┘
                                     No       │     Yes
                                   ┌──────────┘──────────┐
                                   │                     ▼
                                   │          ┌──────────────────┐
                                   │          │  Verification    │
                                   │          └────────┬─────────┘
                                   │                   │
                                   └────────┬──────────┘
                                            ▼
                              ┌───────────────────────┐
                              │  Has implicit intent?  │
                              │  (from entry point)    │
                              └───────────┬───────────┘
                          Yes             │            No
                       ┌──────────────────┘──────────────────┐
                       │                                     ▼
                       │                          ┌───────────────────┐
                       │                          │   Intent Picker   │
                       │                          │  (Send or Unified │
                       │                          │   Onboarding)     │
                       │                          └─────────┬─────────┘
                       │                                    │
                       └──────────────┬─────────────────────┘
                                      │
                    ┌─────────────────┘─────────────────┐
                    │ Send                              │ Unified Onboarding
                    ▼                                   ▼
          ┌──────────────────┐              ┌──────────────────┐
          │  Transfer Flow   │              │  Requirements    │
          └──────────────────┘              │  *skip: UK CA    │
                                            └────────┬─────────┘
                                                     │
                                                     ▼
                                          ┌──────────────────┐
                                          │ Balance Consent  │
                                          │ *US only         │
                                          └────────┬─────────┘
                                                   │
                                          ┌────────────────────┐
                                          │ Open Balances &    │
                                          │ Order Bank Details │
                                          └────────┬───────────┘
                                                   │
                                             ┌─────┴─────┐
                                             │  Assets   │
                                             │ eligible? │
                                             │ (most EU, │
                                             │  UK, US,  │
                                             │  AU, BR,  │
                                             │  SG, etc) │
                                             └─────┬─────┘
                                      Yes          │         No
                                   ┌───────────────┘───────────────┐
                                   ▼                               ▼
                          ┌──────────────┐              ┌──────────────────┐
                          │    Assets    │              │ Currency         │
                          └──────┬───────┘              │ Selection        │
                                 │                      │ *skip: India     │
                           ┌─────┴──────┐               └────────┬─────────┘
                           │ Completed  │                        │
                           │ or skipped?│                        │
                           └─────┬──────┘                        │
                    Completed    │    Skipped                     │
                  ┌──────────────┘──────────┐                    │
                  │                         ▼                    │
                  │              ┌──────────────────┐            │
                  │              │ Currency         │            │
                  │              │ Selection        │            │
                  │              │ *skip: India     │            │
                  │              └────────┬─────────┘            │
                  │                       │                      │
                  └───────────┬───────────┘──────────────────────┘
                              ▼
                  ┌──────────────────────┐
                  │ Discoverability      │
                  │ Settings             │
                  │ *skip: India         │
                  └──────────┬───────────┘
                             │
                       ┌─────┴─────┐
                       │   Card    │
                       │ eligible? │
                       └─────┬─────┘
                  Yes        │        No
               ┌─────────────┘─────────────┐
               ▼                           │
      ┌──────────────┐                     │
      │  Card Intro  │                     │
      │ *skip: India,│                     │
      │  UK CA       │                     │
      └──────┬───────┘                     │
             ▼                             │
      ┌──────────────┐                     │
      │Card Selection│                     │
      └──────┬───────┘                     │
             │                             │
       ┌─────┴──────┐                      │
       │  Chose or  │                      │
       │  skipped?  │                      │
       └─────┬──────┘                      │
   Chose     │     Skipped                 │
  ┌──────────┘──────────┐                  │
  ▼                     ▼                  ▼
┌───────────────┐  ┌──────────────────┐  ┌──────────────────┐
│ Card Order    │  │ Bank Details     │  │ Bank Details     │
│ Flow          │  │ Flow             │  │ Flow             │
└───────┬───────┘  └────────┬─────────┘  └────────┬─────────┘
        │                   │                      │
        └───────────────────┴──────────────────────┘
                            │
                            ▼
                  ┌──────────────────┐
                  │    COMPLETE      │
                  └──────────────────┘
```

## Implicit Intent Behavior

| Scenario | Intent Assigned | Picker Shown? |
|----------|----------------|---------------|
| Normal registration | None | Yes — user chooses Send or Unified Onboarding |
| Brazil (BR) | Unified Onboarding | Skipped — auto-assigned UO |
| Send-only country (Tier 5) | Send | Skipped — goes to Transfer Flow |
| From homepage with implicit intent | Honored | Skipped |
| From intro calculator | Honored | Skipped |
| UK Current Account (GB) | None | Yes — CA-specific messaging (UO or Send) |

## Balance & Bank Details Opening

Balances are opened and bank details are ordered after Balance Consent (US) or Currency Selection (everyone else). If balance consent is still pending, balance opening is blocked until accepted.

## UK Current Account (GB) — Differences

Assume `current-account-onboarding-uk` enabled for all GB users.

- **Requirements**: skipped entirely
- **Card Intro**: skipped
- **Intent Picker**: shows Current Account-specific messaging
- **Currency Selection**: NOT skipped (still shown)
- Everything else follows the standard flow

Effective path: `Profile → Verify → Intent Picker → Currency Sel. → Discoverability → Card Sel. → Card Order / Bank Details → Done`

## Country-Specific Notes

- **US**: Balance Consent step shown (MCA Disclosure acceptance)
- **India**: Skips Currency Selection, Card Intro, Discoverability. Still eligible for card and bank details.
- **Indonesia**: Hardcoded balance block. Send only.
- **Philippines**: Card blocked at onboarding level despite availability API.
- **Tier 4 countries** (AR, ZA, IL, KR, etc.): No card — go straight to Bank Details after Discoverability.
