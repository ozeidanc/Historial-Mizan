# Mizan · Project Context and Conversation History

> **Purpose of this file**  
> This document is the canonical handoff for any new ChatGPT, Claude Code, GitHub Copilot, engineer, reviewer, or future session working on Mizan.  
> It summarizes the project vision, operating rules, architecture, decisions, incidents, deployments, validations, current production state, known debt, and roadmap.  
> Read this file before proposing or changing anything.

---

## 1. Project identity

**Project:** Mizan  
**Owner / operator:** Omar Zeidan  
**Current operational focus:** Crecimiento  
**Current product direction:** Growth-only  
**Current production baseline:**

```text
HEAD   = 7e32a22
TAG    = mizan-growth-core-end-to-end-v1.0
SCHEMA = v26
PORT   = :3000
```

**Latest operational verdict:**

```text
GROWTH_CORE_READY_FOR_REAL_OPERATION = GO
```

Important qualification:

```text
SOFTWARE_READY = GO
REAL_FINANCIAL_BASELINE = pending manual action by Omar
```

The system is technically certified. Omar still needs to reconcile the real cash balance from Wio and seal the real financial baseline.

---

## 2. Product vision

Mizan is not an auto-trading system.

It is a portfolio surveillance, analysis, decision, execution-recording, and track-record system.

Canonical principle:

> **Mizan does not decide the trade. Mizan decides whether the position remains defendable.**

Canonical operating loop:

```text
Mizan evaluates
→ Mizan proposes
→ Omar accepts or declines
→ Omar executes externally in Wio
→ Omar records the real fill
→ Book, cash, NAV, and Track update
```

Daily reevaluation does not imply daily trading.

Only confirmed real movements change positions.

All read endpoints must remain read-only.

---

## 3. Non-negotiable operating rules

### 3.1 Human control

- No automatic decisions.
- No automatic execution links.
- No automatic movements.
- No automatic cash events.
- No automatic trades.
- No auto-trading integration with Wio.

### 3.2 Broker integration boundaries

For the current phase:

- No direct Wio connection.
- No CSV import.
- No PDF import.
- No broker parser.
- No automated reconciliation feed.
- All real operations are entered manually.

### 3.3 Recommendation versus execution

Recommendations remain immutable.

```text
recommended_quantity = quantity proposed by Mizan
executed_quantity    = quantity actually executed in Wio
```

The executed quantity may differ materially from the proposed quantity.

The Book must always use the real executed quantity.

A materially different execution must be classified as discretionary, even if linked causally to a recommendation.

### 3.4 Read-only GET rule

- GET endpoints must not write.
- Staleness may be computed on read.
- Scheduler/startup may only persist allowed operational review metadata.
- Scheduler/startup may not create decisions, executions, movements, or cash events.

---

## 4. Growth-only strategy

The strategic direction is now fixed:

1. Perfect Crecimiento completely.
2. Make Crecimiento the only operational portfolio.
3. Remove Defensiva and Equilibrada from the active product.
4. Preserve Defensiva and Equilibrada history as read-only archive.
5. Remove residual references to Conservadora.
6. Simplify navigation, scheduler, APIs, reporting, and configuration.
7. Perfect:
   - Cockpit;
   - Track Record;
   - checklists;
   - charts;
   - Track Record of La Lente;
   - Campo de Caza;
   - La Lente.
8. Design future portfolios only after Crecimiento is perfect.

Future portfolios must have:

- independent risk profile;
- independent policy;
- independent selector;
- independent sizing;
- independent validation;
- shared proven infrastructure from Crecimiento.

Do not revive the old multiporfolio architecture merely because it already exists.

---

## 5. Crecimiento policy and action semantics

### 5.1 Sealed policy

```text
policy_id   = crecimiento-v2-c0-r1
methodology = 2-c0-r1
effective   = 2026-07-20
```

Do not change policy, methodology, selector, thresholds, universe rules, or target-weight logic during infrastructure work unless Omar explicitly opens a policy phase.

### 5.2 Canonical actions

- `INCORPORAR`: create a new position.
- `AUMENTAR`: buy more of an existing position.
- `REDUCIR`: partial sale; position remains open.
- `ELIMINAR`: sell 100% of the live position; target quantity = 0.
- `MANTENER`: no operation and no action buttons.
- `REVISAR`: decision-only, non-executable.
- `SUSTITUIR`: deferred.

### 5.3 Daily review contract

For operational review date `D`:

```text
review_date      = D
evidence_session = latest fully closed NYSE session before D
```

The daily process must:

1. resolve the operational day;
2. resolve the evidence session;
3. load `crecimiento-v2-c0-r1`;
4. load the full allowed universe;
5. obtain valid PIT evidence;
6. apply eligibility;
7. run selector C0;
8. create a session-specific freeze;
9. persist the canonical selection;
10. compare selection with the live Book;
11. calculate actions;
12. persist the immutable review;
13. expose it through API;
14. show it automatically before market open.

---

## 6. Canonical financial architecture

### 6.1 Book

The Book is derived only from confirmed movements.

The Book is not changed by:

- accepting a recommendation;
- declining a recommendation;
- recording a capital contribution;
- generating a resizing proposal;
- sealing a baseline.

### 6.2 Cash ledger

Canonical cash is append-only.

Current event types:

- `CASH_OPENING_RECONCILIATION`
- `CAPITAL_CONTRIBUTION`
- `BUY_SETTLEMENT`
- `SELL_SETTLEMENT`
- `COMMISSION`
- `CASH_CORRECTION`

Current balance:

```text
available_cash = sum(confirmed cash events)
```

The dashboard must not use an editable cash field as source of truth.

### 6.3 Operational NAV

```text
market_value    = Σ live_quantity_i × frozen_EOD_price_i
operational_NAV = market_value + available_cash
```

Keep separate:

- live indicative value;
- certified EOD value;
- cost basis;
- operational NAV;
- Track Record value.

### 6.4 Capital resizing

```text
target_value_i          = target_weight_i × operational_NAV
target_quantity_i       = target_value_i / frozen_reference_price_i
proposed_trade_quantity = target_quantity_i - current_quantity_i
```

Capital resizing:

- uses the latest current daily review;
- uses the review's selection;
- uses the review's policy and freeze;
- uses certified frozen EOD prices;
- does not run another selector;
- does not replace the daily review;
- becomes `STALE` if Book, cash, review, evidence session, freeze, policy, or contribution state changes.

---

## 7. Chronology of major deliveries

### 7.1 Pre-market operational review baseline

Earlier work established:

- schema v24;
- separation of `review_date` and `evidence_session`;
- startup catch-up;
- periodic scheduler;
- pre-market visibility;
- idempotent current review per portfolio/review date;
- no GET writes.

This became the base for later work.

---

### 7.2 CSRF execution hotfix

**Commit:** `5b2ff21`  
**Tag:** `mizan-growth-execution-csrf-hotfix-v1.0`

#### Incident

Omar tried to record a real execution and received:

```text
CSRF token ausente o inválido
```

Nothing was persisted:

```text
NO_DECISION_PERSISTED = true
NO_EXECUTION_LINK      = true
NO_MOVEMENT            = true
NO_BOOK_CHANGE         = true
```

#### Root cause

The CSRF token was generated per server process.

After restart:

- server token rotated;
- browser retained stale token;
- POST returned 403.

#### Fix

A canonical `secureMizanFetch` client:

- injects CSRF header;
- includes same-origin cookies;
- refreshes token on CSRF 403;
- retries once;
- preserves idempotency key;
- does not retry economic errors;
- does not disable CSRF.

#### Validated behavior

- proposed quantity remains immutable;
- real executed quantity can differ;
- executed quantity of 52 is supported;
- Book uses 52, not the recommendation;
- atomicity;
- idempotency;
- no duplicate movement;
- decline does not touch Book.

#### Directed tests

```text
verify-growth-execution-csrf.mjs
21 PASS
0 FAIL
```

#### Known technical debt

`secureMizanFetch` currently uses a global `window.fetch` monkey-patch.

It works and is tested, but should eventually become an explicit write client.

Do not refactor this until the operational product is stable.

---

### 7.3 Phase 1 · Manual Wio reconciliation and Book

**Commit:** `39d1258`  
**Tag:** `mizan-growth-wio-reconciliation-phase1-v1.0`  
**Schema:** `v25`

#### Goal

Make Mizan's Crecimiento Book exactly match Wio before building NAV and capital resizing.

#### Delivered

##### Clear quantity and amount labels

- Current quantity (shares)
- Recommended quantity (shares)
- Reference price (USD)
- Estimated amount (USD)

This prevents confusion such as:

```text
0.00161 shares
versus
$0.52 estimated amount
```

##### Register real executed movement

Independent form with:

- ticker;
- buy/sell;
- quantity;
- price;
- date/time;
- commission;
- currency;
- broker reference;
- reason;
- optional recommendation link.

##### Discretionary execution

- any positive real quantity;
- recommendation remains immutable;
- material deviation classified as discretionary;
- Book uses executed quantity.

##### Manual Wio comparison

Manual read-only comparison:

- Wio quantity;
- Mizan quantity;
- Wio average cost;
- Mizan average cost;
- cash snapshot;
- snapshot date.

Statuses:

- `MATCH`
- `MISSING_IN_MIZAN`
- `EXCESS_IN_MIZAN`
- `QUANTITY_MISMATCH`
- `COST_BASIS_MISMATCH`

##### Reconciliation adjustments

- append-only;
- no direct Book edits;
- use exact real trades when known;
- use `RECONCILIATION_ADJUSTMENT` only when only aggregate broker state is available.

##### Position reconciliation seal

A position reconciliation checkpoint was added.

Important semantic rule:

- cost basis is not market value;
- cost basis is not NAV;
- Phase 1 seal is not the final financial baseline.

#### Directed tests

```text
verify-wio-reconciliation-phase1.mjs
18 PASS
0 FAIL
```

#### Real user outcome

Omar manually entered enough operations until:

```text
Mizan positions = Wio positions
Mizan average costs = Wio average costs
```

The Crecimiento Book is now reconciled with Wio.

---

### 7.4 Phase 2 · Cash, NAV, capital contribution, resizing

**Commit:** `07880dc`  
**Tag:** `mizan-growth-capital-resizing-phase2-v1.0`  
**Schema:** `v26`

#### Delivered

##### Cash ledger

Reused:

- `portfolio_cash_event`
- `financial-core.mjs`

No duplicate ledger or financial engine.

##### Opening cash reconciliation

```text
CASH_OPENING_RECONCILIATION
```

Represents current liquid cash in Wio after historic trades.

Rules:

- not a new contribution;
- no retroactive settlements;
- no duplicate historical cash impact;
- corrections append-only.

##### Operational NAV

```text
market_value + available_cash
```

Uses frozen EOD prices for sizing.

##### Capital contribution

```text
CAPITAL_CONTRIBUTION
```

- increases cash;
- increases NAV;
- does not change positions;
- does not create trades;
- does not create decisions;
- idempotent by reference.

##### Financial baseline

```text
GROWTH_WIO_FINANCIAL_BASELINE
```

Requires:

- reconciled positions;
- confirmed opening cash.

##### Capital resizing

Separate entities:

- `capital_resizing_proposal`
- `capital_resizing_proposal_line`

Supports:

- AUMENTAR;
- INCORPORAR;
- REDUCIR;
- ELIMINAR;
- MANTENER.

Acceptance/decline is inert.

Real fill updates Book and cash atomically.

Overspend is blocked.

Proposal staleness is enforced.

#### Directed tests

```text
verify-capital-resizing-phase2.mjs
20 PASS
0 FAIL
```

#### Production state after deploy

```text
CASH_BASELINE_REQUIRED
```

The feature was live but economically inert until Omar entered real cash.

---

### 7.5 Certification of daily selection cycle

**Commit:** `663665c`  
**Tag:** `mizan-growth-daily-selection-cycle-v1.0`  
**Schema:** `v26`

#### User concern

For three days, the system showed only:

- AUMENTAR;
- REDUCIR.

No:

- INCORPORAR;
- ELIMINAR.

The concern was that the system might only rebalance incumbents.

#### Audit result

The system reevaluated the full universe:

```text
61 sector-C0 candidates per session
25 selected
```

Session evidence:

| Evidence session | Universe | Selected | Selection change |
|---|---:|---:|---|
| 2026-07-22 | 61 | 25 | CDNS in, PANW out |
| 2026-07-23 | 61 | 25 | no change |
| 2026-07-24 | 61 | 25 | CDNS out, PANW in |

Conclusion:

```text
ZERO_INCORPORATE_ELIMINATE_REASON =
NO_REAL_SELECTION_CHANGE
```

The prior zero entry/exit days were valid.

The next review correctly produced:

```text
ELIMINAR CDNS
INCORPORAR PANW
```

#### Delivered

- full universe reevaluation;
- PIT evidence by session;
- session-specific freeze;
- startup catch-up;
- scheduler;
- restart-safe review generation;
- idempotency;
- `SELECTION_DELTA`;
- UI panel for selection changes;
- capital resizing uses latest review;
- new review makes older resizing stale.

#### Canonical source of truth

```text
freeze + canonical selection + review + Book
```

`SELECTION_DELTA` audit data must not become a second economic source of truth.

#### Directed tests

```text
verify-growth-daily-selection-cycle.mjs
31 PASS
0 FAIL
0 NO_RESULT
```

---

### 7.6 Final end-to-end gate

**Commit:** `7e32a22`  
**Tag:** `mizan-growth-core-end-to-end-v1.0`  
**Schema:** `v26`

#### Critical defect found

Before this hotfix, a capital contribution would have artificially increased Track return.

Example:

```text
Initial NAV          = 1,000
Capital contribution = 5,000
New NAV              = 6,000
Wrong return         ≈ +500%
Correct return       = 0%
```

#### Root cause

Unitization assumed constant units and did not treat external cash flows correctly.

#### Hotfix

In `financial-core.mjs`:

```text
EXTERNAL_CASH_FLOWS = {
  CAPITAL_CONTRIBUTION,
  CASH_CORRECTION
}
```

The unit value is calculated before the external flow, then units are issued or retired at that value.

Result:

- external contribution is return-neutral;
- Track remains continuous;
- commissions still reduce NAV;
- buy/sell settlement is not return by itself.

#### Important reservation

`CASH_CORRECTION` is too generic to always be treated as an external neutral flow.

Future debt:

`CASH_CORRECTION` must require an economic classification, for example:

- opening-state correction;
- external funding correction;
- omitted commission;
- dividend;
- interest;
- broker error;
- accounting correction.

Until then, avoid using `CASH_CORRECTION` casually.

#### Certified

- external flow not counted as return;
- buy/sell settlement not counted as return;
- fees reduce NAV;
- Track continuity preserved;
- CDNS → PANW rotation flow;
- overspend blocked;
- execution atomic;
- execution idempotent;
- trace complete;
- GET read-only;
- no automatic economic writes.

#### Directed tests

```text
verify-growth-core-end-to-end.mjs
23 PASS
0 FAIL
0 NO_RESULT
```

#### Final verdict

```text
GROWTH_CORE_READY_FOR_REAL_OPERATION = GO
```

---

## 8. Current production truth

### 8.1 Software

```text
HEAD   = 7e32a22
TAG    = mizan-growth-core-end-to-end-v1.0
SCHEMA = v26
PORT   = :3000
```

### 8.2 Current portfolio state

- Crecimiento Book reconciled with Wio.
- 25 live positions.
- 83 recorded movements.
- Current market value observed by the gate: approximately `2,771.25 USD` using frozen EOD prices.
- Opening cash not yet reconciled at the moment of the gate.
- Operational NAV not yet certified in production at the moment of the gate.
- Financial baseline not yet sealed in production at the moment of the gate.
- No cash events at the time of the gate.
- No real capital resizing proposals at the time of the gate.

### 8.3 Current review

```text
review_date      = 2026-07-27
evidence_session = 2026-07-24
```

Current selection change:

```text
ELIMINAR CDNS
INCORPORAR PANW
```

CDNS:

- target quantity = 0;
- proposed sale = 100% of live quantity;
- live quantity reported by Code = 0.2323.

PANW:

- not in Book;
- target weight = 4%;
- target quantity > 0;
- pending Omar decision.

---

## 9. Required manual steps for Omar

Before using capital resizing with real money:

### 9.1 Reconcile current cash

In Crecimiento:

```text
Reconciliar efectivo actual
```

Enter only the current liquid cash shown by Wio.

Do not enter:

- total account value;
- market value of positions;
- historic capital transferred;
- total purchase cost.

### 9.2 Verify operational NAV

```text
market_value_EOD + available_cash = operational_NAV
```

Wio may show live values while Mizan uses frozen EOD prices.

Cash must match exactly.

### 9.3 Seal financial baseline

Create:

```text
GROWTH_WIO_FINANCIAL_BASELINE
```

Only after:

- positions match;
- costs match;
- cash matches;
- valuation session is understood;
- NAV identity holds.

### 9.4 Operate CDNS → PANW

If Omar accepts both:

```text
accept CDNS and PANW
→ sell CDNS in Wio
→ record real CDNS fill
→ verify cash increased
→ buy PANW in Wio
→ record real PANW fill
```

Acceptance alone changes nothing.

### 9.5 Future capital contribution

```text
Aportar capital
→ verify cash and NAV
→ Recalcular cartera
→ review CAPITAL_RESIZING
→ accept or decline
→ execute in Wio
→ record real fills
```

---

## 10. Current next phase: Growth-only cleanup

After Omar reconciles cash and seals the financial baseline, the next Code phase is:

```text
MIZAN · DEPURACIÓN GROWTH-ONLY
FASE 1 · ÚNICO PRODUCTO OPERATIVO = CRECIMIENTO
```

Goal:

```text
ACTIVE_OPERATIONAL_PORTFOLIOS = [crecimiento]
```

Expected state:

```text
crecimiento = ACTIVE_OPERATIONAL
defensiva   = ARCHIVED_READ_ONLY
equilibrada = ARCHIVED_READ_ONLY
conservadora = no operational product
```

### Growth-only requirements

#### Scheduler

Only Crecimiento may generate:

- reviews;
- recommendation sessions;
- selection deltas;
- pre-market activity.

#### Startup catch-up

Only Crecimiento.

#### Navigation

Primary operational navigation:

```text
CRECIMIENTO
```

Secondary archive:

- Defensiva;
- Equilibrada.

No operational Conservadora references.

#### Backend write protection

Archived portfolios may allow historical GET.

They must reject economic writes:

- decisions;
- executions;
- movements;
- cash events;
- contributions;
- reconciliation;
- resizing;
- review generation.

Canonical error:

```text
PORTFOLIO_ARCHIVED_READ_ONLY
```

#### Historical preservation

Do not delete:

- reviews;
- recommendations;
- decisions;
- movements;
- Track;
- hashes;
- sessions;
- snapshots;
- foreign-key relationships.

Do not move archived data into Crecimiento.

#### Reporting

Operational reporting:

- Crecimiento only.

Historical reporting:

- archived portfolios clearly separated.

#### Invariant

```text
GROWTH_ECONOMIC_STATE_DELTA = 0
```

No economic data of Crecimiento may change during cleanup.

---

## 11. Known technical debt and risks

### 11.1 Global fetch monkey-patch

`secureMizanFetch` currently patches `window.fetch`.

Future cleanup:

- replace with explicit write client;
- preserve retry and idempotency behavior;
- do not touch before operational stability.

### 11.2 CASH_CORRECTION semantics

Current hotfix treats `CASH_CORRECTION` as external neutral flow.

Risk:

Not every correction is economically external.

Required future change:

Add mandatory classification.

### 11.3 Global suite debt

The global suite still contains pre-existing failures caused by:

- data drift from live user decisions;
- environment/network;
- empty keys;
- flaky GET readonly test.

Rule:

- directed suites for new work must be green;
- global suite must report baseline and net regression;
- never claim global green if pre-existing failures remain.

### 11.4 Reboot-safe infrastructure

A Windows scheduled-task path was prepared earlier:

```text
C:\Users\support\mizan-ops\MizanServer.boot-task.xml
C:\Users\support\mizan-ops\register-and-validate-mizanserver.ps1
C:\Users\support\mizan-ops\run-server-task.cmd
```

It required elevated execution:

```powershell
powershell -ExecutionPolicy Bypass -File C:\Users\support\mizan-ops\register-and-validate-mizanserver.ps1
```

This remains secondary until Growth-only cleanup and core UX are complete.

---

## 12. Repository-level working rules

Any future agent working from GitHub must follow these rules.

### Before changing code

1. Read this file.
2. Confirm current commit, tag, and schema.
3. Audit existing implementation before adding new infrastructure.
4. Reuse existing engines and tables.
5. Avoid parallel implementations.
6. Capture current economic invariants.
7. Back up SQLite.
8. Run integrity check.

### During development

- No production economic writes.
- LAB for simulated fills.
- Production audit read-only.
- No policy changes unless explicitly requested.
- No hidden automatic actions.
- No direct Book edits.
- No direct cash edits.
- Append-only corrections.
- Preserve recommendation immutability.
- Preserve Track continuity.

### Before deployment

- Directed tests: `0 FAIL / 0 NO_RESULT`.
- Global suite: report pass/fail/no-result and net regression.
- Chrome validation.
- Single server instance.
- Git clean.
- DB integrity OK.
- Backup created.
- Rollback documented.

### After deployment

Report:

- commit;
- tag;
- schema;
- files changed;
- migrations;
- production writes;
- directed suite;
- global suite;
- invariants;
- rollback;
- next exact user actions.

---

## 13. Required review format after every Code response

Every Code delivery must be reviewed with:

### A. Technical verdict

- Accepted.
- Accepted with reservations.
- Rejected.

### B. What is truly proven

Separate:

- production evidence;
- LAB evidence;
- assumptions;
- pending manual user state.

### C. Risks and debt

Do not hide caveats behind PASS tables.

### D. Exact next action

Either:

- user action;
- Code order;
- no action.

### E. Full master plan

Never abbreviate the plan.

### F. Mandatory tracking

#### Current operational product

- completed;
- current;
- pending;
- progress.

#### Bright layer / Analyst Decision Layer

- completed;
- current;
- pending;
- progress.

#### General Mizan plan

- completed;
- current;
- pending;
- progress.

---

## 14. Full master roadmap

### Phase A · Crecimiento core

1. Pre-market review.
2. Session-specific evidence.
3. Full universe evaluation.
4. C0 freeze.
5. Selection delta.
6. INCORPORAR.
7. ELIMINAR.
8. AUMENTAR.
9. REDUCIR.
10. Book.
11. Real execution.
12. CSRF.
13. Manual Wio reconciliation.
14. Cash ledger.
15. Operational NAV.
16. Capital contribution.
17. Capital resizing.
18. Staleness.
19. Atomic Book + cash.
20. Track neutral to external flows.
21. End-to-end GO.

**Status:** technically complete.  
**Pending:** real cash reconciliation and real financial baseline by Omar.

### Phase B · Growth-only cleanup

22. Canonical portfolio registry.
23. Crecimiento only active.
24. Defensiva archived.
25. Equilibrada archived.
26. Conservadora residual purge.
27. Scheduler cleanup.
28. Startup cleanup.
29. Navigation cleanup.
30. Reporting cleanup.
31. Backend write blocks.
32. Historical preservation.
33. Dead-code removal.
34. Growth economic invariants.

### Phase C · Cockpit

35. Information hierarchy.
36. Current review.
37. Selection changes.
38. Pending decisions.
39. Pending executions.
40. Book.
41. Cash.
42. NAV.
43. Alerts.
44. System health.
45. Evidence/session state.
46. Remove dead or misleading panels.

### Phase D · Track Record and charts

47. External flows separated.
48. TWR.
49. Money-weighted return.
50. Benchmark.
51. Drawdown.
52. Realized P&L.
53. Unrealized P&L.
54. Attribution.
55. Fees.
56. Cash history.
57. Contribution history.
58. Reconciled charts.
59. Continuous per-portfolio Track.

### Phase E · Checklists

60. Evidence.
61. Eligibility.
62. Policy.
63. Risk.
64. Sizing.
65. Decision.
66. Execution.
67. Follow-up.
68. Verdict.
69. Expiry.

### Phase F · La Lente and Campo de Caza

70. Thesis record.
71. Thesis date.
72. Conviction.
73. Validity.
74. Outcome.
75. Thesis changes.
76. Catalyst.
77. Catalyst source.
78. First-seen date.
79. Last verification.
80. Expiry date.
81. Catalyst state:
    - ACTIVE;
    - CHANGED;
    - FULFILLED;
    - EXPIRED;
    - REFUTED.
82. Campo de Caza freshness.
83. Remove expired catalysts.
84. La Lente Track Record.
85. Full La Lente improvement.

### Phase G · Future portfolios

86. Risk profile.
87. Independent policy.
88. Independent selector.
89. Independent sizing.
90. Independent validation.
91. Backtest.
92. Robustness tests.
93. Deployment on proven Crecimiento infrastructure.

---

## 15. Current tracking

### Current operational product

**Completed**

- automatic daily review;
- full universe;
- session freeze;
- INCORPORAR;
- ELIMINAR;
- real Book;
- manual Wio reconciliation;
- cash ledger;
- operational NAV;
- capital contribution;
- capital resizing;
- atomic execution;
- return-neutral external flows;
- end-to-end GO.

**Current**

- reconcile real Wio cash;
- seal real financial baseline;
- then Growth-only cleanup.

**Pending**

- remove multiporfolio operational residue.

**Progress:** `99%`

### Bright layer / Analyst Decision Layer

**Completed**

- basic evidence structure;
- session freeze;
- selection delta.

**Current**

- paused during Growth-only cleanup.

**Pending**

- La Lente;
- thesis Track;
- Campo de Caza freshness;
- catalyst lifecycle;
- conviction and verdict tracking.

**Progress:** `15%`

### General Mizan plan

**Completed**

- Crecimiento core is coherent and technically operable.

**Current**

- convert all of Mizan into a clean Growth-only product.

**Pending**

- Cockpit;
- Track;
- checklists;
- charts;
- La Lente;
- Campo de Caza;
- new portfolios.

**Progress:** `92%`

---

## 16. Compact prompt for a new AI session

Use this when starting a new conversation:

```text
You are continuing the Mizan project.

Read PROJECT_CONTEXT_MIZAN.md first.

Current production:
HEAD 7e32a22
tag mizan-growth-core-end-to-end-v1.0
schema v26

Crecimiento core is certified GO end-to-end:
daily review, full C0 universe, INCORPORAR/ELIMINAR, Book, manual Wio reconciliation, cash ledger, NAV, contributions, resizing, atomic fills, staleness, and return-neutral external cash flows.

Omar still needs to reconcile the real Wio cash balance and seal the real financial baseline.

After that, the next phase is Growth-only cleanup:
Crecimiento active, Defensiva and Equilibrada archived read-only, Conservadora residual references removed, no economic change to Crecimiento.

After every Code response provide:
1. rigorous review;
2. verdict;
3. exact next order if needed;
4. full master plan;
5. mandatory tracking for:
   - current operational product;
   - Analyst Decision Layer;
   - general Mizan plan.

Do not create new scope or lose the canonical vision.
```

---

## 17. Changelog reference

| Commit | Tag | Purpose |
|---|---|---|
| `5b2ff21` | `mizan-growth-execution-csrf-hotfix-v1.0` | CSRF refresh and real executed quantity |
| `39d1258` | `mizan-growth-wio-reconciliation-phase1-v1.0` | Manual Wio reconciliation and canonical Book |
| `07880dc` | `mizan-growth-capital-resizing-phase2-v1.0` | Cash, NAV, contributions, resizing |
| `663665c` | `mizan-growth-daily-selection-cycle-v1.0` | Full universe, INCORPORAR/ELIMINAR, selection delta |
| `7e32a22` | `mizan-growth-core-end-to-end-v1.0` | Final end-to-end gate and return-neutral external flows |

---

## 18. Final canonical summary

Mizan now has a technically certified Crecimiento core.

The immediate operational sequence is:

```text
Reconcile current Wio cash
→ seal real financial baseline
→ operate real recommendations under human control
→ perform Growth-only cleanup
→ perfect Cockpit
→ perfect Track Record
→ perfect checklists
→ perfect charts
→ perfect La Lente
→ refresh Campo de Caza
→ design future risk portfolios
```

Do not skip directly to new portfolios.

Do not reintroduce automatic broker integration.

Do not change policy during cleanup.

Do not allow external cash flows to contaminate return.

Do not let archived portfolios remain operational through hidden schedulers or backend routes.

Crecimiento is the canonical foundation for everything that follows.
