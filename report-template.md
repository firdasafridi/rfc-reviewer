# RFC Review: <rfc-title>

## Executive Summary

- **Overall Score:** `<score>/10`
- **Rating:** `<Agentic-Ready | Strong | Needs Work | Incomplete | Insufficient>`
- **RFC Type:** `<frontend | backend | full-stack>`
- **Sub-Type:** `<new-feature | enhancement | performance | tech-improvement>`
- **Assessment Confidence:** `High | Medium | Low`
- **Applied Caps/Gates:** `<none | list caps applied and why>`
- **Implementation Readiness Verdict:** `<PROCEED | PROCEED with notes | HOLD — [list blockers] | BLOCKED — [list critical gaps]>`
- **Report Path:** `<path>`
- **RFC Author:** `<name>` | **Reviewed:** `<date>`

Short paragraph (3–5 sentences) answering:
- Can an AI agent read this RFC and produce correct implementation code without asking the author a question?
- What is the biggest strength and the biggest gap?
- What is the one thing that must change before agentic execution is possible?

---

## Quick Verdict

**Why this RFC can be implemented agentically:** `<1–3 concise bullets>`

**Why this RFC will cause agent guessing or rework:** `<1–3 concise bullets>`

---

## PRD → RFC Traceability Matrix

> Forward: every PRD element should map to an RFC section.
> Reverse: every RFC decision should trace to a PRD requirement.
> Missing mappings are gaps.
>
> **No-PRD RFCs** (tech-debt, performance, infra): replace this matrix with a
> Self-Contained Justification check — see alternate format below.

### Standard format (PRD exists)

| PRD Element | RFC Section | Coverage |
|---|---|---|
| `<story/rule/criterion>` | `<RFC section or "—">` | `Full | Partial — [what's missing] | **Missing**` |
| `<story/rule/criterion>` | `<RFC section or "—">` | `Full | Partial | **Missing**` |
| `<RFC decision with no PRD driver>` | — | `**No PRD driver** (scope creep? unstated need?)` |

**Summary:** X of Y PRD items fully covered, X partial, X missing. X RFC decisions without PRD justification.

### Alternate format (no PRD — self-contained justification)

| Element | Present? | Quality | Assessment |
|---|---|---|---|
| Problem statement | `<yes/no>` | `<specific/vague/absent>` | `<agent knows what's broken?>` |
| Success criteria | `<yes/no>` | `<quantified/qualitative/absent>` | `<agent knows when it's done?>` |
| Scope boundaries | `<yes/no>` | `<explicit/implicit/absent>` | `<agent knows where to stop?>` |
| Non-goals | `<yes/no>` | `<concrete/vague/absent>` | `<agent won't over-build?>` |

**PRT score note:** No PRD linked. Scored on self-contained justification. Max possible: 8.0.

---

## Scorecard

> Use ONE of the three scorecards below based on RFC type.

### Frontend Scorecard (11 categories)

*Use for frontend-only RFCs.*

| Category | Score | Evidence-Based Rationale |
|----------|------:|--------------------------|
| PRT — PRD Traceability | `<score>` | `<cite section/quote or absence>` |
| TDC — Technical Decisions | `<score>` | `<cite section/quote or absence>` |
| CNT — Contract Specificity | `<score>` | `<cite section/quote or absence>` |
| SCB — Scope Boundaries | `<score>` | `<cite section/quote or absence>` |
| DEP — Dependencies | `<score>` | `<cite section/quote or absence>` |
| FMC — Failure Mode Coverage | `<score>` | `<cite section/quote or absence>` |
| NFS — Non-Functional Specificity | `<score>` | `<cite section/quote or absence>` |
| TPS — Test Plan Specificity | `<score>` | `<cite section/quote or absence>` |
| ROL — Rollout & Rollback | `<score>` | `<cite section/quote or absence>` |
| OBS — Observability | `<score>` | `<cite section/quote or absence>` |
| CPA — Pattern Alignment | `<score>` | `<cite section/quote or absence>` |

### Backend Scorecard (12 core + 1 conditional)

*Use for backend-only RFCs.*

| Category | Score | Evidence-Based Rationale |
|----------|------:|--------------------------|
| PRT — PRD Traceability | `<score>` | `<cite section/quote or absence>` |
| TDC — Technical Decisions | `<score>` | `<cite section/quote or absence>` |
| DMS — Data Model & Schema | `<score>` | `<cite section/quote or absence>` |
| ACV — API Contract & Versioning | `<score>` | `<cite section/quote or absence>` |
| DIC — Data Integrity & Consistency | `<score>` | `<cite section/quote or absence>` |
| FMC — Failure Mode & Retry Coverage | `<score>` | `<cite section/quote or absence>` |
| CSS — Concurrency & Scaling | `<score>` | `<cite section/quote or absence>` |
| SAS — Security & Authorization | `<score>` | `<cite section/quote or absence>` |
| MRP — Migration & Rollout Plan | `<score>` | `<cite section/quote or absence>` |
| OBS — Observability Definition | `<score>` | `<cite section/quote or absence>` |
| SBC — Service Boundary & Coupling | `<score>` | `<cite section/quote or absence>` |
| CPA — Pattern Alignment | `<score>` | `<cite section/quote or absence>` |
| CDG — Compliance & Data Governance | `<score or N/A>` | `<cite trigger or "no compliance trigger">` |

### Full-Stack Scorecard (18 categories)

*Use for full-stack RFCs. See `rubric-fullstack.md` for merge rules.*
*Merged categories cite evidence from BOTH layers. Weakest layer drags the score.*

| # | Category | Source | Score | Evidence-Based Rationale |
|---|----------|--------|------:|--------------------------|
| 1 | PRT — PRD Traceability | Merged | `<score>` | `FE: <evidence> · BE: <evidence>` |
| 2 | TDC — Technical Decisions | Merged | `<score>` | `FE: <evidence> · BE: <evidence>` |
| 3 | CNT — Contract Specificity | FE | `<score>` | `<cite>` |
| 4 | SCB — Scope Boundaries | FE | `<score>` | `<cite>` |
| 5 | DEP — Dependencies | FE | `<score>` | `<cite>` |
| 6 | NFS — Non-Functional Specificity | FE | `<score>` | `<cite>` |
| 7 | TPS — Test Plan Specificity | FE | `<score>` | `<cite>` |
| 8 | DMS — Data Model & Schema | BE | `<score>` | `<cite>` |
| 9 | ACV — API Contract & Versioning | BE | `<score>` | `<cite>` |
| 10 | DIC — Data Integrity & Consistency | BE | `<score>` | `<cite>` |
| 11 | FMC — Failure Mode Coverage | Merged | `<score>` | `FE: <UI error states> · BE: <retry/DLQ/circuit breaker>` |
| 12 | CSS — Concurrency & Scaling | BE | `<score>` | `<cite>` |
| 13 | SAS — Security & Authorization | BE | `<score>` | `<cite>` |
| 14 | ROL — Rollout & Rollback | Merged | `<score>` | `FE: <flag/stages> · BE: <migration/rollback> · Cross: <deploy order/compat>` |
| 15 | OBS — Observability | Merged | `<score>` | `FE: <analytics/errors> · BE: <metrics/logs/traces>` |
| 16 | SBC — Service Boundary & Coupling | BE | `<score>` | `<cite>` |
| 17 | CPA — Pattern Alignment | Merged | `<score>` | `FE: <patterns> · BE: <patterns> · Cross: <naming/convention consistency>` |
| 18 | CDG — Compliance & Data Governance | BE | `<score or N/A>` | `<cite trigger or "no compliance trigger">` |

### Resource & Cost Advisory (backend/full-stack only)

> This section is optional and non-blocking. Notes here must not change category scores, score caps, or the readiness verdict unless the RFC is explicitly capacity/cost scoped.

- `<advisory note about compute/DB/storage/cost impact, or "No advisory note">`

---

## Decision Closure Assessment

> Every technical decision assessed for resolution completeness.
> Fix **Dangling** decisions before asking an AI agent to implement.

### Decision Index

| # | Decision | Status | Critical Gaps |
|---|----------|--------|---------------|
| `<#>` | `<decision summary>` | Resolved / Partial / Dangling | `<gap1>, <gap2>` |

**Aggregate:** X of Y decisions Resolved, X Partial, X Dangling

---

### Decision: `<#>` — `<Decision summary>`

**Status:** `Resolved | Partial | Dangling`

---

#### What was decided

`<State the decision as written in the RFC. Quote directly where possible.>`
`<If not explicitly stated, write: "NOT EXPLICITLY STATED — inferred from [section]."`

---

#### Alternatives considered

`<List alternatives from the RFC with their rejection rationale.>`
`<If none, write: "NO ALTERNATIVES DOCUMENTED — agent cannot evaluate if this is the right choice.">`

---

#### Grounding in existing code

`<What specific files, components, services, or endpoints does this reference?>`
`<If none, write: "NOT GROUNDED — agent must search the codebase.">`

---

#### Interface specification

`<Fully specified? Types, fields, nullability, error codes, prop contracts?>`
`<If incomplete, list exactly what is missing.>`

---

#### Failure handling

`<What happens on the unhappy path?>`
`<If absent, write: "NOT SPECIFIED — agent will implement happy path only.">`

---

#### Challenge results

- **Scale:** `<Does this hold at 10x? Assessment.>`
- **Reversibility:** `<How hard to change after implementation?>`
- **Consistency:** `<Conflicts with other decisions?>`
- **Agent implementability:** `<Could an agent implement this without asking? What would it guess?>`

---

#### Gaps and suggestions

**Missing:**
- `<specific gap>`

**Suggested resolution:**
`<Concrete proposal using the RFC's own context.>`

**Open questions for the author:**
- `<specific question>`

---

*Repeat for each major technical decision.*

---

<!--
  FRONTEND DEEP-DIVES (include for frontend and full-stack RFCs)
-->

## UI State Audit

> Every data-driven component must define 5 states: loading, empty, error, partial, success.

| Component | Loading | Empty | Error | Partial | Success | Assessment |
|-----------|:-------:|:-----:|:-----:|:-------:|:-------:|------------|
| `<component>` | `<defined/missing>` | `<defined/missing>` | `<defined/missing>` | `<defined/missing>` | `<defined/missing>` | `<X/5 states>` |

**Summary:** X of Y components have all 5 states defined. X components missing states.

---

## Performance Budget Check

> For performance and new-feature RFCs.

| Metric | Target | Current Baseline | Source | Assessment |
|--------|--------|-----------------|--------|------------|
| `LCP` | `<target>` | `<baseline or "not stated">` | `<how measured>` | `<adequate / missing / vague>` |
| `INP` | `<target>` | `<baseline>` | `<source>` | `<assessment>` |
| `CLS` | `<target>` | `<baseline>` | `<source>` | `<assessment>` |
| `Bundle size delta` | `<budget>` | `<current>` | `<source>` | `<assessment>` |

`<If no performance budget specified, write: "NO PERFORMANCE BUDGET — agent cannot optimize against targets.">`

---

## Accessibility Review

| Aspect | Specified? | Details | Assessment |
|--------|-----------|---------|------------|
| Keyboard navigation flow | `<yes/no>` | `<details>` | `<adequate / incomplete / missing>` |
| Focus management (modal/route) | `<yes/no>` | `<details>` | `<assessment>` |
| ARIA labels | `<yes/no>` | `<details>` | `<assessment>` |
| Heading hierarchy | `<yes/no>` | `<details>` | `<assessment>` |
| Color contrast | `<yes/no>` | `<details>` | `<assessment>` |
| Motion sensitivity | `<yes/no>` | `<details>` | `<assessment>` |
| Screen reader behavior | `<yes/no>` | `<details>` | `<assessment>` |

---

## Pattern Alignment Check

> For enhancement RFCs: every existing pattern touched.

| Pattern | RFC Approach | Assessment |
|---------|-------------|------------|
| `<state management pattern>` | `follows / extends / replaces / creates parallel` | `<assessment>` |
| `<component pattern>` | `<approach>` | `<assessment>` |
| `<styling pattern>` | `<approach>` | `<assessment>` |
| `<analytics events>` | `preserved / modified / dropped` | `<assessment>` |

---

<!--
  BACKEND DEEP-DIVES (include for backend and full-stack RFCs)
-->

## Data Integrity Deep-Dive

> For every write path in the RFC.

| Write Path | Transaction Scope | Partial Failure Behavior | Idempotency Key | Consistency Guarantee | Duplicate Handling |
|---|---|---|---|---|---|
| `<write path>` | `<scope or "NOT SPECIFIED">` | `<behavior or "NOT SPECIFIED">` | `<key or "NONE">` | `<strong/eventual or "NOT SPECIFIED">` | `<mechanism or "NOT SPECIFIED">` |

`<If no write paths analyzed, write: "NO WRITE PATHS DOCUMENTED — agent will guess transaction boundaries.">`

---

## Concurrency Collision Map

> For every shared resource in the RFC.

| # | Shared Resource | Writers | Collision Scenario | Resolution Mechanism | Lock Failure Behavior | Assessment |
|---|---|---|---|---|---|---|
| `<#>` | `<resource>` | `<actor1, actor2>` | `<what happens>` | `<mechanism or "NONE">` | `<behavior or "NOT SPECIFIED">` | `<adequate / incomplete / missing>` |

`<If no collision points identified despite multi-actor system, write: "NO COLLISION POINTS LISTED — the RFC describes [X concurrent actors] but doesn't address what happens when they collide.">`

---

## API Contract Completeness Check

> For every endpoint in the RFC.

| Endpoint | Request Schema | Response Schema | Error Taxonomy | Auth Spec | Idempotency | Example Payloads | Assessment |
|---|---|---|---|---|---|---|---|
| `<method path>` | `<complete/partial/missing>` | `<complete/partial/missing>` | `<complete/partial/missing>` | `<specific/vague/missing>` | `<defined/missing>` | `<yes/no>` | `<X/6 complete>` |

---

## Async Job / Event Consumer Spec

> For any RFC with background workers, cron jobs, or event consumers.
> Skip if no async work in this RFC.

| Job/Consumer | Trigger | Input Shape | Retry Policy | DLQ | Concurrency Limit | Idempotency Key | Timeout | Assessment |
|---|---|---|---|---|---|---|---|---|
| `<job name>` | `<cron/queue/event>` | `<specified/missing>` | `<specified/missing>` | `<named/missing>` | `<specified/missing>` | `<specified/missing>` | `<specified/missing>` | `<X/7 specified>` |

`<If RFC describes async work but no job spec, write: "RFC DESCRIBES [X] ASYNC JOBS BUT NO SPECIFICATION — agent will guess retry, DLQ, and concurrency.">`

---

## Compliance Trigger Check

> Scan the RFC for compliance triggers. If any found, CDG category is active.

| Trigger | Found? | Data Location | Classification | Assessment |
|---|---|---|---|---|
| PII (name, email, phone, address, IP, device ID) | `<yes/no>` | `<table/field or N/A>` | `<classification or N/A>` | `<handled / unhandled>` |
| Payment data (card, bank account) | `<yes/no>` | `<location>` | `<PCI scope?>` | `<assessment>` |
| Health data | `<yes/no>` | `<location>` | `<HIPAA scope?>` | `<assessment>` |
| User content with retention | `<yes/no>` | `<location>` | `<retention policy?>` | `<assessment>` |
| Auth/session data | `<yes/no>` | `<location>` | `<audit requirements?>` | `<assessment>` |
| Cross-border data transfer | `<yes/no>` | `<jurisdiction>` | `<UU PDP / GDPR?>` | `<assessment>` |

**CDG Status:** `Active — score CDG category | N/A — no compliance triggers found`

---

<!--
  FULL-STACK DEEP-DIVES (include for full-stack RFCs only)
-->

## Cross-Layer Contract Verification

> For every API endpoint the frontend consumes — verify the backend produces what the frontend expects.

| Endpoint | Backend Response Schema | Frontend Expected Schema | Match? | Gaps |
|---|---|---|---|---|
| `<method path>` | `<fields/types from BE spec>` | `<fields/types FE expects>` | `Yes / No / Partial` | `<mismatched fields, casing, nullability, error shape>` |

**Checks performed:**
- [ ] Field name casing consistent (snake_case vs camelCase — transformation specified?)
- [ ] Nullability aligned (backend nullable → frontend handles null?)
- [ ] Error response shape matched (backend error format = frontend error handler?)
- [ ] Pagination format aligned (offset vs cursor, page size defaults)
- [ ] Auth token format and refresh flow consistent

**Mismatches found:** `<count>` — `<if any, this caps ROL at 6.5>`

---

## Cross-Layer Rollout Compatibility Matrix

> What works during phased deployment? Every "No" must be addressed in the rollout plan.

| Scenario | Frontend | Backend | Works? | Notes |
|---|---|---|---|---|
| Pre-deploy (baseline) | Old | Old | Yes | Current state |
| Backend first | Old | New | `<Yes / No / Partial>` | `<what breaks or degrades>` |
| Frontend first | New | Old | `<Yes / No / Partial>` | `<what breaks or degrades>` |
| Both deployed (target) | New | New | Yes | Target state |
| Backend rollback | New | Old (rolled back) | `<Yes / No / Partial>` | `<what breaks>` |
| Frontend rollback | Old (rolled back) | New | `<Yes / No / Partial>` | `<what breaks>` |

**Deploy order:** `<Backend first / Frontend first / Simultaneous / NOT SPECIFIED>`
**Incompatible scenarios:** `<count>` — `<if deploy order not specified, ROL capped at 6.0>`

---

## End-to-End Data Flow

> For each major user action, trace the full path from click to database and back.
> If the full path is not traceable from the RFC alone, flag for PRT and TDC.

### Flow: `<user action name>`

```
User clicks [action]
  → Frontend: [component] calls [method] on [store/service]
  → API: [HTTP method] [path] with [key request fields]
  → Backend: [handler] → [service] → [repository]
  → DB: [table] [operation] (INSERT/UPDATE/SELECT)
  → Side effects: [events published, notifications sent, audit logged]
  → Response: [key response fields]
  → Frontend: [store update] → [component re-render] → [UI state]
```

**Gaps in flow:** `<list any steps where one layer's output doesn't match the next layer's input>`

*Repeat for each major user action.*

---

## Agentic Readiness Deep-Dive

### Vague Word Audit

| # | Word/Phrase | Location | Impact | Concrete Replacement |
|---|-----------|----------|--------|---------------------|
| 1 | `<word>` | `<section>` | `<what agent would guess>` | `<specific replacement>` |

**Total vague words in spec sections:** `<count>`

### Dangling Alternatives

| # | Alternatives | Location | Impact |
|---|-------------|----------|--------|
| 1 | `<"X or Y">` | `<section>` | `<what agent would do>` |

**Total dangling alternatives:** `<count>`

### Task Decomposition Assessment

| Chunk | Acceptance Criteria | Assessment |
|-------|-------------------|------------|
| `<chunk>` | `<criteria from RFC>` | `<verifiable / vague / missing>` |

`<If absent, write: "NOT SPECIFIED — agent must invent implementation order.">`

---

<!--
  SHARED SECTIONS (always include)
-->

## Strengths

- `<strength with RFC section citation>`
- `<strength with RFC section citation>`
- `<strength with RFC section citation>`

---

## Biggest Gaps

- `<gap with RFC section citation and impact on agentic execution>`
- `<gap with RFC section citation and impact on agentic execution>`
- `<gap with RFC section citation and impact on agentic execution>`

---

## Priority Actions

> Ordered by impact on implementation readiness. Do these before asking an AI agent to implement.
> For backend/full-stack RFCs, every action touching API or data must state the missing contract fields explicitly (not "add more detail").

1. **`<section or decision #>`** — `<what to add/change and why it unblocks agentic execution>`
2. **`<section or decision #>`** — `<what to add/change>`
3. **`<section or decision #>`** — `<what to add/change>`
4. **`<section or decision #>`** — `<what to add/change>`

---

## Backend Contract Addendum *(backend/full-stack only, if applicable)*

> Include this section whenever the RFC introduces or changes backend endpoints and/or database structures.

### Endpoint Contract Details

| Endpoint | Method/Path | AuthZ | Request Contract | Response Contract | Error Contract | Idempotency/Versioning | Status |
|---|---|---|---|---|---|---|---|
| `<name>` | `<POST /v1/...>` | `<scope/role + ownership rule>` | `<fields, types, required, validation>` | `<fields, types, nullable, examples>` | `<code + status + trigger>` | `<idempotency key + API version rule>` | `Complete / Missing [fields]` |

### Database Changes Details

| Change | Table/Entity | DDL / Shape Diff | Data Migration Plan | Rollback Plan | Compatibility Window | Status |
|---|---|---|---|---|---|---|
| `<new table / alter column / new index>` | `<table>` | `<types, nullability, constraints, indexes>` | `<backfill steps + rate + duration>` | `<revert mechanism + impact on new writes>` | `<old/new schema support period>` | `Complete / Missing [details]` |

`<If backend is in scope but one table is empty, write: "MISSING BACKEND CONTRACT DETAIL — cannot proceed agentically without endpoint contract/database change details.">`

---

## Implementation Readiness Checklist

> A concise gate before handing to an AI agent for implementation.

### Unblocked (agent can proceed)

**All types:**
- [ ] PRD → RFC traceability matrix complete (or self-contained justification if no PRD)
- [ ] All technical decisions resolved with alternatives rejected
- [ ] All failure modes handled per external interaction with error message catalog
- [ ] Configuration contract: env vars, config keys, feature flags with defaults documented
- [ ] Pattern alignment verified *(enhancements)*
- [ ] Rollout plan with feature flag and rollback mechanism
- [ ] Observability metrics and alerts defined
- [ ] Task decomposition with acceptance criteria per chunk
- [ ] Zero vague words in specification sections

**Frontend:**
- [ ] All interfaces / contracts / prop types fully specified
- [ ] All UI states defined (loading, empty, error, partial, success)
- [ ] Performance budget quantified *(if applicable)*
- [ ] Accessibility requirements specified
- [ ] Browser support matrix defined

**Full-stack (in addition to frontend + backend items):**
- [ ] Cross-layer contract verified — backend response matches frontend expectation
- [ ] Deploy order specified (backend first / frontend first / simultaneous)
- [ ] Cross-layer rollout compatibility matrix — no unaddressed "No" scenarios
- [ ] End-to-end data flow documented for each major user action
- [ ] Feature flag coordination defined (one flag or two? independent toggle?)

**Backend:**
- [ ] Schema changes, if applicable, at DDL-level precision (columns, types, constraints, indexes)
- [ ] API contracts, if applicable, with full request/response schemas, error taxonomy, example payloads
- [ ] Transaction boundaries and idempotency keys defined per write path
- [ ] Concurrency collision points listed with resolution mechanisms
- [ ] Security: auth boundaries, input validation, injection surfaces, tenancy isolation
- [ ] Migration plan: zero-downtime, backfill, backward compat window, rollback script
- [ ] Service boundary and coupling documented
- [ ] Compliance handled *(if PII/regulated data touched)*

### Blocked (must fix first)
- [ ] `<specific gap that blocks implementation>`
- [ ] `<specific gap>`

**Verdict:** `Ready to implement | Fix [N] blockers first | Major rewrite needed`

---

## Task Manifest

> The bridge from review to execution. Ordered list of implementation chunks
> with files to create/modify and acceptance criteria per chunk.
> If the RFC specifies task decomposition, verify it here. If not, propose one.

| Order | Chunk | Files to Create/Modify | Acceptance Criteria | Dependencies |
|------:|-------|----------------------|-------------------|--------------|
| 1 | `<chunk name>` | `<file paths>` | `<"done when" — verifiable by running something>` | `None / Chunk #N` |
| 2 | `<chunk name>` | `<file paths>` | `<acceptance criteria>` | `<dependencies>` |
| 3 | `<chunk name>` | `<file paths>` | `<acceptance criteria>` | `<dependencies>` |

`<If RFC has no task decomposition, write: "RFC DOES NOT SPECIFY — proposed manifest above based on reviewer analysis. Author should verify before agent execution.">`

---

## Dangling Decisions Log

> Decisions that remain unresolved and must be closed before implementation.

| # | Decision | Location | Owner | Deadline |
|---|----------|----------|-------|----------|
| 1 | `<decision>` | `<section>` | `<owner or unassigned>` | `<date or unset>` |

---

## Open Questions

> Questions that surfaced during review and must be answered by the author.

| # | Question | Category | Severity |
|---|----------|----------|----------|
| 1 | `<question>` | `<which rubric category>` | `Blocking | Important | Nice-to-have` |

---

## Evidence Notes

> Most important sections and evidence inspected for this assessment.

- `<section name>` — `<what was found and how it affected scoring>`
- `<section name>` — `<what was absent and how it affected scoring>`
