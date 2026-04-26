# RFC Review Rubric — Full-Stack

Full-stack RFC review assesses whether an AI agent or engineer can implement
both frontend and backend changes from a single RFC — without asking the
author clarifying questions about any layer.

This rubric exists because full-stack RFCs have unique failure modes that
neither the frontend nor backend rubric catches alone:
- Frontend/backend contract mismatch (API shape assumed but not agreed)
- Cross-layer rollout gaps (new frontend + old backend during deploy)
- Scattered ownership (who owns the integration point?)
- End-to-end flow gaps (data enters frontend, passes through API, lands in DB — but the full path is never shown in one place)

**This rubric does NOT duplicate the frontend and backend rubrics.** It
references them for detailed criteria. What it adds is: clear category
ownership, merge rules for shared categories, cross-stack deep-dives,
and a unified weight/cap system.

## How to use this rubric

1. Read `rubric-frontend.md` for detailed criteria on frontend-sourced categories.
2. Read `rubric-backend.md` for detailed criteria on backend-sourced categories.
3. Use THIS file for: which categories to score, how to score merged categories,
   weight table, score caps, and full-stack-specific deep-dives.

## Full-Stack sub-type calibration

- `new-feature`: new user-facing feature spanning UI + API + data; deepest coverage everywhere
- `enhancement`: modifying existing feature across layers; migration, contract versioning, and cross-layer compat are critical
- `tech-improvement`: cross-cutting optimization or refactor (e.g., API redesign with frontend migration); rollout and backward compat are critical

---

## The 18 Scored Categories

### Category Map

| # | Code | Category Name | Source | Merge Rule |
|---|---|---|---|---|
| 1 | **PRT** | PRD Traceability | Both | Single score — cite frontend + backend evidence |
| 2 | **TDC** | Technical Decision Completeness | Both | Single score — cite frontend + backend decisions |
| 3 | **CNT** | Contract Specificity | Frontend | Frontend criteria only — component props, state shapes, event payloads |
| 4 | **SCB** | Scope Boundaries | Frontend | Frontend criteria only — which files/components in scope |
| 5 | **DEP** | Dependency Explicitness | Frontend | Frontend criteria — but also check backend endpoint availability |
| 6 | **NFS** | Non-Functional Specificity | Frontend | Frontend criteria — performance, a11y, browser support |
| 7 | **TPS** | Test Plan Specificity | Frontend | Frontend criteria — but expand to include integration tests that cross the API boundary |
| 8 | **DMS** | Data Model & Schema Specificity | Backend | Backend criteria only — DDL, indexes, migrations |
| 9 | **ACV** | API Contract & Versioning | Backend | Backend criteria — but also check frontend consumes what backend produces |
| 10 | **DIC** | Data Integrity & Consistency | Backend | Backend criteria only — transactions, idempotency, ordering |
| 11 | **FMC** | Failure Mode Coverage | Both | Single score — combine: frontend UI failure states + backend retry/DLQ/circuit breaker. See merge rules below. |
| 12 | **CSS** | Concurrency & Scaling Specificity | Backend | Backend criteria only — QPS, connection pools, rate limits |
| 13 | **SAS** | Security & Authorization Specificity | Backend | Backend criteria — but also check frontend input sanitization and auth token handling |
| 14 | **ROL** | Rollout & Rollback Plan | Both | Single score — merge frontend ROL + backend MRP. See merge rules below. |
| 15 | **OBS** | Observability Definition | Both | Single score — combine: frontend analytics/error tracking + backend metrics/logs/traces. See merge rules below. |
| 16 | **SBC** | Service Boundary & Coupling | Backend | Backend criteria only — which service, sync vs async, coupling |
| 17 | **CPA** | Consistency & Pattern Alignment | Both | Single score — cite frontend patterns + backend patterns separately, then combine. See merge rules below. |
| 18 | **CDG** | Compliance & Data Governance | Backend | Conditional — backend criteria, triggered by PII/regulated data |

**Total: 18 categories** (CDG conditional, so 17 or 18 depending on triggers).

### Advisory section (not scored)

- **RCS — Resource & Cost Notes (Backend-sourced):** keep as advisory commentary only.
- Capacity and cost notes (pod/CPU/memory sizing, DB load projections, storage growth, cloud bill impact) can be included in recommendations, but must not affect category scores, score caps, or readiness verdict.

---

## Merge Rules for Shared Categories

These rules eliminate ambiguity. For each merged category, the reviewer
scores ONCE and cites evidence from both layers.

### PRT — PRD Traceability (merged)

Score once. The traceability matrix must cover both layers:
- Forward: PRD story → which frontend component AND which backend endpoint/service implements it
- Reverse: every frontend component and every backend service → which PRD requirement drives it
- **Full-stack-specific check:** are there PRD requirements where the frontend and backend RFC sections contradict each other on scope or behavior?

Use criteria from `rubric-frontend.md` PRT for frontend evidence.
Use criteria from `rubric-backend.md` PRT for backend evidence.
Final score = judgment across both, not average.

### TDC — Technical Decision Completeness (merged)

Score once. Decisions span both layers:
- Frontend decisions: which framework, state management, component architecture
- Backend decisions: which DB, which queue, which service boundary
- **Full-stack-specific check:** are there decisions where the frontend assumes one thing and the backend specifies another? (e.g., frontend assumes REST but backend specifies gRPC; frontend assumes pagination by offset but backend implements cursor-based)

Use criteria from `rubric-frontend.md` TDC for frontend decisions.
Use criteria from `rubric-backend.md` TDC for backend decisions.
Final score = whichever layer is weaker drags the score down. One layer at 9.0 and the other at 4.0 does not average to 6.5 — it scores closer to 5.0 because the agent will hit the weak layer.

### FMC — Failure Mode Coverage (merged)

Score once. Failure modes span the full request path:
- **Frontend evidence** (from `rubric-frontend.md` FMC): UI states on API failure (400/401/403/404/429/500/timeout/offline), race conditions from rapid clicks, stale cache handling, third-party script failures
- **Backend evidence** (from `rubric-backend.md` FMC): per-call timeout/retry/circuit breaker/DLQ, poison message handling, partial infrastructure failure, graceful degradation
- **Full-stack-specific check:** does the frontend error handling match what the backend actually returns? If the backend returns `{ "error": "INSUFFICIENT_BALANCE", "code": 422 }`, does the frontend know to handle that specific shape and code? Or does it only handle generic 4xx?

Final score = weakest link. If frontend handles errors well but backend doesn't define what errors it returns, the score is low.

### ROL — Rollout & Rollback Plan (merged from frontend ROL + backend MRP)

Score once. This is the most critical merge because phased deploys are where full-stack RFCs fail:
- **Frontend evidence** (from `rubric-frontend.md` ROL): feature flag, rollout stages, cache invalidation, backward compat with saved user state
- **Backend evidence** (from `rubric-backend.md` MRP): zero-downtime migration, dual-write period, backfill, consumer notification, rollback trigger
- **Full-stack-specific checks (mandatory):**
  1. **Deploy order specified?** Which ships first — frontend or backend? Why? What happens if only one is deployed?
  2. **Cross-layer compatibility during rollout?** Does old frontend work with new backend? Does new frontend work with old backend? For how long?
  3. **Coordinated rollback?** If backend rolls back, does frontend need to roll back too? What's the sequence?
  4. **Feature flag coordination?** Is it one flag or two? If two, what's the coupling? Can they be toggled independently?

Final score cannot exceed 6.0 if deploy order and cross-layer compatibility are not specified.

### OBS — Observability Definition (merged)

Score once. Observability must cover both layers:
- **Frontend evidence** (from `rubric-frontend.md` OBS): analytics events, error monitoring, Core Web Vitals, performance dashboards
- **Backend evidence** (from `rubric-backend.md` OBS): RED/USE metrics, structured logs, trace spans, alerts, SLOs
- **Full-stack-specific check:** are there distributed traces that connect frontend request → API call → backend processing → response? Can on-call follow a user-reported issue from the frontend error to the backend root cause?

Final score = judgment across both. Backend observability without frontend analytics (or vice versa) caps at 7.0.

### CPA — Consistency & Pattern Alignment (merged)

Score once. Pattern alignment must be verified per layer:
- **Frontend evidence** (from `rubric-frontend.md` CPA): state management, component, styling, error handling patterns
- **Backend evidence** (from `rubric-backend.md` CPA): repository, service layer, error handling, queue, configuration patterns
- **Full-stack-specific check:** do the frontend and backend follow consistent conventions with each other? (e.g., naming: if backend uses snake_case in API responses, does frontend transform to camelCase consistently? Or is it ad-hoc?)

Final score = average of both layers, unless one layer silently introduces a new pattern (then that layer drags the score down).

---

## Full-Stack-Specific Deep-Dives

These are IN ADDITION to the frontend and backend deep-dives from their respective rubrics. They cover gaps that only exist when both layers are in play.

### Cross-Layer Contract Verification

For every API endpoint the frontend consumes:

| Endpoint | Backend Response Schema | Frontend Expected Schema | Match? | Gaps |
|---|---|---|---|---|
| `<method path>` | `<fields/types from backend spec>` | `<fields/types frontend expects>` | `Yes / No / Partial` | `<mismatched fields, missing error codes, type differences>` |

**What to check:**
- Field name casing (snake_case vs camelCase)
- Nullability assumptions (backend says nullable, frontend doesn't handle null)
- Error response shape (backend returns `{ error, code }`, frontend expects `{ message, status }`)
- Pagination format (offset vs cursor, different page size defaults)
- Auth token format and refresh flow

If any mismatch found → flag as blocker.

### Cross-Layer Rollout Compatibility Matrix

| Scenario | Frontend | Backend | Works? | Notes |
|---|---|---|---|---|
| Pre-deploy | Old | Old | Yes | Baseline |
| Backend first | Old | New | `<Yes/No/Partial>` | `<what breaks or degrades>` |
| Frontend first | New | Old | `<Yes/No/Partial>` | `<what breaks or degrades>` |
| Both deployed | New | New | Yes | Target state |
| Backend rollback | New | Old (rolled back) | `<Yes/No/Partial>` | `<what breaks>` |
| Frontend rollback | Old (rolled back) | New | `<Yes/No/Partial>` | `<what breaks>` |

If any "No" in the matrix → deploy order must be specified and the incompatible scenario must be avoided by the rollout plan.

### End-to-End Data Flow

For each major user action in the RFC, trace the full path:

```
User action → Frontend component → API call (method, path) → Backend handler → Service layer → Database write/read → Response → Frontend state update → UI render
```

**What to check:**
- Is the full path documented, or do you have to piece it together from scattered sections?
- Are there gaps where one layer's output doesn't match the next layer's expected input?
- Are there side effects (events, notifications, audit logs) that aren't shown in the flow?
- Could an agent implement each step in sequence without asking which step comes next?

If the full path is not traceable from the RFC alone → flag for TDC and PRT.

---

## Relative weighting by sub-type

| Category | New Feature | Enhancement | Tech Improvement |
|---|:---:|:---:|:---:|
| PRT — PRD Traceability | High | High | Medium |
| TDC — Technical Decisions | **Highest** | **Highest** | High |
| CNT — Contract Specificity (FE) | **Highest** | High | Medium |
| SCB — Scope Boundaries (FE) | High | **Highest** | Medium |
| DEP — Dependencies (FE) | High | Medium | Low |
| NFS — Non-Functional (FE) | High | Medium | High |
| TPS — Test Plan (FE) | High | High | Medium |
| DMS — Data Model & Schema (BE) | **Highest** | High | Medium |
| ACV — API Contract (BE) | **Highest** | **Highest** | Medium |
| DIC — Data Integrity (BE) | **Highest** | **Highest** | Medium |
| FMC — Failure Modes (merged) | **Highest** | **Highest** | **Highest** |
| CSS — Concurrency & Scaling (BE) | High | Medium | **Highest** |
| SAS — Security (BE) | **Highest** | Medium | High |
| ROL — Rollout (merged) | High | **Highest** | **Highest** |
| OBS — Observability (merged) | High | High | **Highest** |
| SBC — Service Boundary (BE) | High | High | Medium |
| CPA — Pattern Alignment (merged) | High | **Highest** | Medium |
| CDG — Compliance (BE, conditional) | High (when triggered) | Medium | Medium |

## Score caps (non-negotiable gates)

Apply before finalizing. Strictest cap wins. Full-stack inherits caps from
both rubrics PLUS additional cross-layer caps.

**Inherited from backend rubric:**
- `PRT < 5.0` → overall max **6.0**
- `TDC < 5.0` → overall max **6.0**
- `DMS < 5.0` → overall max **6.0**
- `DIC < 5.0` → overall max **6.5**
- `SAS < 5.0` → overall max **6.5**
- `FMC < 4.0` → overall max **7.0**
- `CDG < 5.0` (when triggered) → overall max **6.0**

**Inherited from frontend rubric:**
- `CNT < 5.0` → overall max **6.5**
- `NFS < 4.0` (for performance sub-type) → overall max **6.0**

**Full-stack-specific caps:**
- `ACV < 5.0` → overall max **6.0** — if the API contract is weak, both layers are building against sand
- `ROL < 5.0` → overall max **6.0** — cross-layer rollout without a plan is a guaranteed incident
- Cross-layer contract mismatch found → overall max **6.5** regardless of other scores
- Deploy order not specified → ROL capped at **6.0**
- Any 4 categories below `5.0` → overall max **6.0**
- To earn `9.0+`: all categories `>= 7.5`; `ACV`, `DIC`, `FMC`, `CNT` all `>= 8.5`; no dangling decisions; no cross-layer mismatches
- To earn `10.0`: every decision resolved, all categories `>= 9.0`, zero vague words, full end-to-end flow documented

## Rating bands

| Score | Rating | Agentic Verdict |
|------:|--------|-----------------|
| 8.5–10.0 | **Agentic-Ready** | **PROCEED** — agent can execute both layers |
| 7.0–8.4 | **Strong** | **PROCEED with notes** — minor gaps, address inline |
| 5.5–6.9 | **Needs Work** | **HOLD** — revise noted categories, re-review |
| 3.5–5.4 | **Incomplete** | **BLOCKED** — significant rework needed |
| 0.0–3.4 | **Insufficient** | **BLOCKED** — not a real RFC yet |

## Overall score

After scoring categories, decide the overall score using judgment, not arithmetic.

The overall score answers:

> If Claude Code were given this RFC, could it implement BOTH the frontend and backend — correct schema, correct API handlers, correct UI states, correct error handling, matching patterns in both codebases — without asking the author a question?

Full-stack RFCs are harder to get right than single-layer RFCs. The same overall score represents a higher bar.

## Confidence rules

- `High`: all sections present across both layers; contracts match; flows are traceable
- `Medium`: most sections present but 1–2 cross-layer gaps (e.g., API contract mismatch on one endpoint)
- `Low`: multiple layers thin; contracts don't match; no end-to-end flow

Low-confidence assessments must avoid top-end scores.

## Required evidence style

Same as frontend and backend rubrics: cite specific RFC sections, quote directly, name explicit absences.

For full-stack-specific issues, cite BOTH the frontend and backend sections that conflict or fail to connect. Example: "Backend §4.2 returns `{ user_id, created_at }` but Frontend §3.1 expects `{ userId, createdAt }` — no transformation layer specified."
