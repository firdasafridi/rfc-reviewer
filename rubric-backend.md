# RFC Review Rubric — Backend

Backend RFC review assesses whether an AI agent or engineer can implement
the proposed backend changes — without asking the author clarifying questions
about schema shapes, transaction boundaries, retry policies, API contracts,
or concurrency resolution.

Score what is actually present in the RFC. Do not score intent, author
reputation, or "they'll cover this during implementation."

**Scope assumptions:**
- Backend environment is a mix of monolith and microservices — service boundary
  discipline matters, and so does module discipline within a monolith.
- Compliance (UU PDP / GDPR / PII handling) is context-dependent — triggered
  when the RFC touches personal or regulated data, not a fixed checkbox.

## Core question

How likely is an AI coding agent (or a human backend engineer) to:

- implement the correct schema, transactions, and data integrity rules on the first pass
- produce API handlers with complete error taxonomy without guessing status codes
- handle concurrency, retries, and failure modes correctly without inventing policies
- write secure code with proper auth, validation, and tenancy isolation
- produce code that matches existing codebase patterns and service boundaries

## Backend sub-type calibration

Identify which sub-type best describes this RFC:

- `new-feature`: new service, API, or data pipeline; deepest coverage needed across schema, API, security, scaling
- `enhancement`: modifying existing services, APIs, or data models; migration, API versioning, and data integrity are critical
- `tech-improvement`: security hardening, performance optimization, or infrastructure change; trade-offs, observability, and rollback are critical

## The 19 Reviewer Skills

These skills are the lenses applied during review. Each produces evidence for scoring.

| # | Skill | New Feature | Enhancement | Tech Improvement |
|---|---|:---:|:---:|:---:|
| 1 | Systems thinking & dependency mapping | M | **M** | **M** |
| 2 | Requirements & scope interrogation | **M** | M | **M** |
| 3 | Trade-off analysis | M | M | **M** |
| 4 | Non-functional requirements literacy | **M** | M | **M** |
| 5 | Failure mode imagination | **M** | **M** | **M** |
| 6 | Observability thinking | **M** | M | **M** |
| 7 | Cost of change / reversibility | M | **M** | **M** |
| 8 | Abstraction judgment | **M** | M | O |
| 9 | Communication & written critique | M | M | M |
| 10 | Knowing what's missing | **M** | **M** | **M** |
| 11 | Migration & rollout reasoning | O | **M** | **M** |
| 12 | API contract & versioning discipline | **M** | **M** | O |
| 13 | Data integrity & consistency | **M** | **M** | M |
| 14 | Schema & data model critique | **M** | M | O |
| 15 | Concurrency & scaling | **M** | M | **M** |
| 16 | Security & authorization | **M** | M | **M** (when security) |
| 17 | Compliance & data governance | O | O | O |
| 18 | Resource & cost awareness | O | O | O |
| 19 | Performance reasoning | M | M | **M** (when performance) |

**M** = Mandatory for this sub-type
**Bold M** = Particularly high-leverage for this sub-type
**O** = Optional / context-dependent (see activation rules below)

### When "Optional" becomes mandatory

- **Compliance (#17):** mandatory the moment the RFC touches PII, payment data, health records, user-generated content retention, authentication logs, audit trails, or cross-border data flow. In Indonesia, UU PDP triggers apply to any personally identifiable data processing.
- **Migration & rollout (#11) for new features:** optional only if greenfield — no existing data, no existing consumers, no shared infrastructure. The moment there's a shared database, existing clients, or existing deploy pipeline, it becomes mandatory.
- **Abstraction judgment (#8) for tech improvement:** mandatory if the improvement introduces a new caching layer, queue, sharding strategy, or service boundary.
- **API contract (#12) for tech improvement:** mandatory if it changes external timing, response shape, error semantics, or SLA — even subtly.
- **Schema critique (#14) for tech improvement:** mandatory if the improvement adds indexes, changes column types, partitions tables, or touches hot paths in the data model.
- **Security (#16) in non-security tech improvements:** mandatory if performance work touches authorization paths, caching of sensitive data, rate limiting, session handling, or tenant isolation. Performance optimizations often introduce security bugs (e.g., cache keys that leak across tenants).
- **Performance reasoning (#19) in non-performance RFCs:** mandatory for any new feature at moderate-to-high scale, or any enhancement to a hot path.

### The always-mandatory core

Across all three backend sub-types, these never drop off:

Systems thinking, trade-off analysis, NFR literacy, failure mode imagination, observability, cost of change, communication, knowing what's missing, data integrity & consistency.

Nine skills. A reviewer who masters only these will catch the majority of serious backend issues.

### Distinct high-risk areas per sub-type

| Sub-Type | Watch Especially For |
|---|---|
| **New feature** | Schema design, API contract, data integrity, security, scaling (defining trust boundaries and data shapes from scratch) |
| **Enhancement** | Migration & rollout, API versioning, data integrity, cost of change (existing consumers and data at stake) |
| **Tech improvement** | Failure modes, observability, performance reasoning, rollback (easy to make production worse, and measurement defines success) |

---

## Category Scores (12 core + 1 conditional + 1 advisory)

Score each category from `0.0` to `10.0`. Use `0.5` increments only.

---

### PRT — PRD Traceability

**Core question:** Does every PRD requirement have an RFC answer, and does every RFC decision have a PRD justification?

Look for:
- Forward: every PRD user story → which service/endpoint/job implements it
- Forward: every PRD business rule → which validation, transaction, or constraint enforces it
- Forward: every PRD acceptance criterion → which contract, test, or observability signal satisfies it
- Reverse: every new service, table, queue, endpoint → what PRD need drives it
- Reverse: every new dependency or infrastructure component → what user outcome requires it
- Missing forward = coverage gap. Missing reverse = scope creep.

High score:
- Explicit bidirectional matrix. Every story has an RFC section. Every RFC decision ties back.
- Agent knows which PRD requirement each piece of code satisfies.

Low score:
- No PRD link, no mapping attempted, silent drops, unjustified RFC additions.
- Agent doesn't know why it's building what it's building.

**No-PRD exception:** Tech-debt, performance, or infra RFCs may have no PRD. In that case, score PRT on self-contained justification: does the RFC state its own problem, success criteria, scope boundaries, and non-goals clearly enough for an agent to know *why* it's building this? Max PRT score without a PRD: 8.0.

---

### TDC — Technical Decision Completeness

**Core question:** Are all architectural choices made — not deferred?

Look for:
- Every decision explicitly stated with chosen option named
- Alternatives considered and rejected with specific reasons tied to constraints, measurements, or codebase reality
- No "TBD", "to be decided during implementation", "we might use X or Y" left dangling
- Rationale grounded in reality, not abstract preference

High score:
- Agent can implement every choice without asking "which approach?"
- All trade-offs documented with reasoning.

Low score:
- Multiple unresolved decisions or "we'll figure it out."
- Agent must make architectural choices the RFC author should have made.

---

### DMS — Data Model & Schema Specificity

**Core question:** Are schema changes defined with DDL-level precision?
**Applicability:** Required when the RFC adds/changes tables, columns, indexes, constraints, or storage/query patterns.

Look for:
- Full DDL for every new or altered table: column names, types, nullability, defaults, constraints, foreign keys
- Every index justified by a named query it supports — if you can't point at the query, the index shouldn't exist
- Partition/sharding keys with rationale
- Cardinality estimates and growth projections (quantified, not "lots of data")
- Example rows showing real data shapes
- PII classification per column where relevant
- Data retention policy per table

High score:
- DBA can review and approve from the RFC alone.
- Agent can write the migration file directly from the DDL in the RFC.

Low score:
- "Add a new table for orders" without columns, types, constraints, or indexes.
- Agent must invent the schema.

Review output requirement:
- If DMS < 7.0, list exactly which database details are missing in the report (DDL shape, migration/backfill steps, rollback, compatibility window).

---

### ACV — API Contract & Versioning

**Core question:** Are API contracts specified at consumer-ready depth?
**Applicability:** Required when the RFC adds/changes synchronous API endpoints or externally consumed service interfaces.

Look for:
- Exact method, path, and all status codes per endpoint
- Request schema: field names, types, required/optional, nullability, validation rules
- Response schema: field names, types, nullability, example payloads for each scenario
- Error taxonomy: specific error codes, messages, conditions — not just "returns error"
- Authentication and authorization per endpoint: specific middleware, scopes, roles — not "standard auth"
- Idempotency: which endpoints are idempotent, mechanism per endpoint (natural key, idempotency header, dedup window)
- Versioning strategy: additive, breaking, or feature-flagged; deprecation window for breakers; migration plan for consumers
- Rate limits, payload size limits, pagination conventions — matched to existing codebase patterns
- Backward compatibility: which existing consumer contracts are preserved, which change

High score:
- Consumer team can build against this contract without the backend being finished.
- Agent can generate handler code directly from the spec.

Low score:
- "POST /api/orders" with no schema, status codes, or error handling.
- Agent must invent the contract.

Review output requirement:
- If ACV < 7.0, report must enumerate missing endpoint contract fields per endpoint (method/path, auth, request schema, response schema, error taxonomy, idempotency/versioning).

---

### DIC — Data Integrity & Consistency

**Core question:** Are transactional boundaries, idempotency, and consistency guarantees explicitly specified?

Look for:
- Transactional boundaries: which writes must be atomic together, which can't be in same transaction (e.g., external API calls)
- Idempotency keys: defined for any non-idempotent operation, with dedup window and storage
- Race condition handling: every collision point (two users, user + job, two replicas, retry + original) with named resolution mechanism
- Consistency model: strong or eventual, per data path, with reasoning
- Ordering guarantees: if events must be processed in order, how that's guaranteed
- Duplicate event handling: what happens when the same event arrives twice
- Stale read handling: what happens if a reader sees old data during a write
- Partial failure: what happens when step 2 of 3 fails — is step 1 rolled back? compensated? orphaned?

High score:
- "Wrap order update and outbox insert in a single DB transaction; email sent by outbox worker with idempotency key `order_id:event_type`; duplicate events deduped at consumer."
- Agent implements correct transaction boundaries and dedup on the first pass.

Low score:
- "Update the order and send the email" with no transaction boundary.
- Agent writes code that can corrupt data on partial failure.

---

### FMC — Failure Mode & Retry Coverage

**Core question:** Does every external call have defined failure behavior?

Look for:
- Per external call: timeout value, retry policy (attempts + backoff), circuit breaker threshold, dead-letter queue
- What the caller does on timeout-without-confirmation
- What happens when a downstream service is fully down (not just slow)
- Poison message handling: what happens when a message can never be processed
- Partial infrastructure failure: what works and what doesn't when one component is down
- Graceful degradation: which features can run in degraded mode vs. which must fail-fast

High score:
- "500ms timeout, 3 retries with exponential backoff (100/300/900ms), circuit breaker at 50% error rate over 30s window, failed events to DLQ `orders-retry`."
- Agent implements retry, circuit breaker, and DLQ code from the spec.

Low score:
- "Handle errors gracefully."
- Agent writes code with no retry logic, no timeouts, and no DLQ.

**Error response catalog (must be present for score >= 7.0):**
- For each API endpoint: the full error taxonomy — error code, HTTP status, message, when it occurs
- Which error messages are user-facing vs internal-only
- Error response schema: consistent shape across all endpoints (e.g., `{ "error": "CODE", "message": "...", "details": {...} }`)
- Without this, the agent invents error codes or returns inconsistent error shapes.

---

### CSS — Concurrency & Scaling Specificity

**Core question:** Has the author specified correctness-critical concurrency behavior, and quantified load where it is relevant to implementation risk?

Look for:
- Worker/goroutine counts and limits
- Rate limiting: per-user, per-tenant, global, with specific numbers
- Collision behavior: lock strategy, optimistic retry, or conflict resolution on shared writes
- Backpressure strategy: what happens when the system can't keep up
- Cache strategy: what's cached, TTL, invalidation, stampede protection
- Hot path identification: which code paths are performance-critical
- Expected QPS/latency targets only when the RFC is explicitly performance/capacity scoped

High score:
- "For `order_id` writes: optimistic lock with `version` column, retry once on conflict, return `409 CONFLICT` with recoverable message; queue consumer capped at 20 workers with backpressure to DLQ after retry budget."
- Agent implements conflict handling, rate limits, and backpressure behavior from the spec.

Low score:
- "Should handle our traffic."
- Agent uses framework defaults and invents collision handling.

---

### SAS — Security & Authorization Specificity

**Core question:** Are security boundaries and defenses specified, not assumed?

Look for:
- Brief threat model: who are the attackers, what they're trying to do
- AuthN/AuthZ at every trust boundary: specific middleware/guard, JWT scopes, role checks
- Ownership validation: "user must own the resource" — specify how (e.g., `order.user_id = jwt.sub`)
- Input validation rules per field: length, charset, enum membership, format, cross-field rules
- Injection surface analysis: SQL (parameterized? ORM?), command injection, SSRF on outbound URLs
- Secrets management: where stored, how rotated, what logs they must not appear in
- Audit logging: what actions are logged, to where, with what fields
- Rate limiting for abuse: specific limits per user-triggered endpoint
- Tenancy isolation: how data is scoped per tenant, where isolation is enforced

High score:
- "Bearer JWT required, scope `orders:write`, user must own the order (checked via `order.user_id = jwt.sub`), input validated against JSON schema §X, audit log emitted to `audit.orders`."
- Agent writes secure code by default.

Low score:
- "Authenticated endpoint."
- Agent writes code without input validation, ownership checks, or audit logging.

---

### MRP — Migration & Rollout Plan

**Core question:** Can this be deployed without downtime, and rolled back if it breaks?

Look for:
- Zero-downtime migration strategy: add column nullable → backfill → dual-write → switch reads → drop old column
- Shape of schema DURING migration (not just start and end states)
- Backfill plan: batch size, rate limit, expected duration, impact on production traffic
- Feature flag: name, default state, targeting rules, kill-switch behavior
- Rollout stages: internal → 1% → 10% → 100% with go/no-go evidence gates
- Rollback trigger: specific metric thresholds or error patterns
- Rollback mechanism: what happens to data written during rollout if rolled back
- Consumer notification: which downstream services need to know about the change and when
- Backward compatibility window: how long old behavior is maintained

High score:
- "Add column nullable → backfill via batched job (10k rows/min) → dual-write for 24h → switch reads → drop old column after 7d; behind flag `new_order_schema`; rollback: toggle flag, reads fall back to old column."
- Agent generates migration scripts, feature flag checks, and rollback procedures.

Low score:
- "Deploy the migration."
- Agent writes one-shot migration with no rollback path.

**Configuration contract (must be present):**
- Every new environment variable: name, type, default value, required/optional
- Every new config key or feature flag: name, default state, who provisions it
- Every secret reference: where stored (vault/env/config), rotation policy
- Without this, the agent writes code that compiles but doesn't start in any environment.

---

### OBS — Observability Definition

**Core question:** Will the team know when something goes wrong — and debug it at 3am?

Look for:
- Specific metric names with tags — RED metrics (Rate, Errors, Duration) and USE metrics (Utilization, Saturation, Errors) where applicable
- Structured log fields: what's logged, at what level, with what fields
- Trace span names: around cross-service calls, with parent-child relationships
- Alert thresholds: per SLO-relevant failure mode, with escalation path
- SLO targets: availability, latency, error budget
- Dashboard location or specification
- "How to debug this in prod" paragraph for complex features

High score:
- "Emit `order_cancel_requests_total` counter tagged by `reason`, p95 latency metric, alert if p95 > 500ms for 5min, SLO 99.9% success/30d, dashboard `orders-service` panel §3."
- Agent includes correct instrumentation in generated code.

Low score:
- "Add logging."
- Agent generates code with no metrics, no structured logs, and no trace spans.

---

### RCS — Resource & Cost Notes (Advisory, not scored)

**Core question:** Are there obvious infrastructure or cost risks the team should track outside this RFC?

This section is advisory. It informs rollout planning but does not change category scores, score caps, or the final readiness verdict.

Optional notes:
- Compute requirements: pod count, CPU/memory per pod, autoscale range
- DB load impact: query count delta, connection count delta, current headroom
- Network egress: cross-AZ, cross-region, external API calls
- Storage growth: per table, per month, retention impact
- New infrastructure components: what, why, estimated cost
- Cloud bill impact: even a rough estimate
- Licensing costs for new dependencies

Strong advisory note:
- "Adds ~200 req/s to orders DB (current 1500/s, headroom OK), new Redis instance `cache.t3.small` (~$25/mo), storage growth +5GB/month for audit logs."
- Team can route this to infra/finops planning without blocking implementation RFC readiness.

Weak advisory note:
- "No significant cost impact" without reasoning.
- Potential resource growth is unknown; flag for follow-up in infra planning.

---

### SBC — Service Boundary & Coupling

**Core question:** Is it clear where the change lives, what it calls, and what new coupling is introduced?

Look for:
- Which service or module owns this change — named, not implied
- What cross-service calls are introduced: sync vs async, why
- What new coupling is created and whether it's justified
- Bounded-context discipline: does this respect existing service boundaries or blur them
- Dependency direction: who calls whom, and is the dependency acceptable
- For monoliths: which module owns this, what internal APIs are touched

High score:
- "Logic lives in orders-service; synchronous call to user-service for authorization only (cached 60s); notification via async event to `orders.cancelled` topic (consumed by notification-service); no new sync dependencies added."
- Agent knows exactly where to put the code and what external calls to make.

Low score:
- "Orders service calls user service calls notification service" with no rationale.
- Agent creates a chain of synchronous cross-service calls.

---

### CPA — Consistency & Pattern Alignment

**Core question:** Does the RFC align with existing codebase patterns — or deviate with explicit justification?

Look for:
- Repository/data access pattern: matches existing (repository pattern, ORM usage, raw SQL conventions)
- Error handling pattern: matches existing (error types, wrapping, propagation)
- Service layer pattern: matches existing (dependency injection, interface contracts)
- Queue/event pattern: matches existing (topic naming, message schema, consumer pattern)
- Configuration pattern: matches existing (env vars, config files, feature flags)
- References specific files to mirror: "follow the pattern in `orders/repository.go`"
- If deviating: explicit justification and ADR reference

High score:
- "Follows existing repository pattern in `orders/repository.go`" or "intentionally deviates from current sync approach because Z; deviation documented in ADR-42."
- Agent generates code that looks like it belongs in the existing codebase.

Low score:
- Silently introduces a new ORM, queue technology, or auth pattern alongside the old one.
- Agent generates code in a style that clashes with every neighboring file.

---

### CDG — Compliance & Data Governance [CONDITIONAL]

**Trigger conditions:** Score this category ONLY when the RFC touches PII, payment data, health records, user-generated content retention, authentication logs, audit trails, or cross-border data transfer. If none of these apply, mark as `N/A — no compliance trigger`.

**Core question:** Is regulated data handled correctly?

Look for:
- Data classification: which fields are PII, which are sensitive, which are public
- Legal basis for processing: consent, legitimate interest, contractual necessity (cite UU PDP / GDPR as applicable)
- Retention policy: per data type, with automated enforcement
- Right-to-delete path: what happens when a user requests deletion, which tables are affected
- Encryption: at rest and in transit, per data classification
- Access control: who can query regulated data, audit logging for access
- Cross-border transfer: where data is stored, whether it crosses jurisdictions
- Audit trail: what actions are logged, retention of audit logs themselves

High score:
- Data classified per policy, retention period specified, right-to-delete path defined, access logged, encryption explicit, legal basis cited.
- Agent generates code with correct PII handling from the spec.

Low score:
- PII handled without classification, no retention policy, no audit trail.
- Agent stores PII in plaintext logs.

---

## Relative weighting by sub-type

| Category | New Feature | Enhancement | Tech Improvement |
|---|:---:|:---:|:---:|
| PRT — PRD Traceability | High | High | Medium |
| TDC — Technical Decisions | High | High | High |
| DMS — Data Model & Schema | **Highest** | High | Medium |
| ACV — API Contract & Versioning | **Highest** | **Highest** | Low (unless API changes) |
| DIC — Data Integrity | **Highest** | **Highest** | Medium |
| FMC — Failure Modes | **Highest** | **Highest** | **Highest** |
| CSS — Concurrency & Scaling | High | Medium | **Highest** |
| SAS — Security | **Highest** | Medium | **Highest** (when security) |
| MRP — Migration & Rollout | Medium | **Highest** | **Highest** |
| OBS — Observability | High | Medium | **Highest** |
| RCS — Resource & Cost (Advisory, not scored) | Advisory | Advisory | Advisory |
| SBC — Service Boundary | High | High | Medium |
| CPA — Pattern Alignment | High | **Highest** | Medium |
| CDG — Compliance (conditional) | High (when triggered) | Medium | Medium |

## Score caps (non-negotiable gates)

Apply before finalizing the overall score. Strictest cap wins. Backend has more caps than frontend because the blast radius of bad decisions is larger — corrupted data and broken contracts are near-impossible to undo.

- `PRT < 5.0` → overall max **6.0** — coverage is the foundation
- `TDC < 5.0` → overall max **6.0** — unresolved decisions aren't an RFC
- `DMS < 5.0` → overall max **6.0** — schema is forever; bad schema cascades
- `DIC < 5.0` → overall max **6.5** — data corruption is the worst outcome
- `SAS < 5.0` → overall max **6.5** — security gaps are unacceptable
- `FMC < 4.0` → overall max **7.0** — optimistic RFCs ship fragile code
- `CDG < 5.0` (when triggered) → overall max **6.0** — regulatory risk is not negotiable
- Any 3 categories below `5.0` → overall max **6.5**
- To earn `9.0+`: all categories must be `>= 7.5`; `DMS`, `DIC`, and `FMC` must be `>= 8.5`; no dangling decisions
- To earn `10.0`: every decision resolved, all categories `>= 9.0`, zero vague words in spec sections

## Rating bands

| Score | Rating | Agentic Verdict |
|------:|--------|-----------------|
| 8.5–10.0 | **Agentic-Ready** | **PROCEED** — agent can execute |
| 7.0–8.4 | **Strong** | **PROCEED with notes** — minor gaps, address inline |
| 5.5–6.9 | **Needs Work** | **HOLD** — revise noted categories, re-review |
| 3.5–5.4 | **Incomplete** | **BLOCKED** — significant rework needed |
| 0.0–3.4 | **Insufficient** | **BLOCKED** — not a real RFC yet |

## Overall score

After scoring categories, decide the overall score using judgment, not arithmetic.

The overall score answers:

> If Claude Code were given this RFC, how likely is it to produce correct, production-quality backend code — correct schema, correct transactions, correct retry logic, correct security — without asking the author a question?

## Confidence rules

- `High`: all sections present; specifications are direct and consistent
- `Medium`: most sections present but 1–2 are thin or missing
- `Low`: multiple critical sections absent or contradictory

Low-confidence assessments must avoid top-end scores regardless of what is present.

## Backend-Specific Deep-Dives

These are mandatory review sections that go beyond category scoring.

### Data Integrity Deep-Dive

For every write path in the RFC:
1. What's the transaction boundary?
2. What happens on partial failure?
3. Is there an idempotency key?
4. What consistency guarantee is this making?
5. What happens if the same event/request arrives twice?

Report as a table:

| Write Path | Transaction Scope | Partial Failure | Idempotency | Consistency | Duplicate Handling |
|---|---|---|---|---|---|

### Concurrency Collision Map

For every shared resource:
1. Who writes to it? (List all actors)
2. What's the collision scenario?
3. What's the resolution mechanism?
4. What happens when the lock can't be acquired / the optimistic check fails?

### API Contract Completeness Check

For every endpoint:
1. Is the full request schema specified?
2. Is the full response schema specified (including error responses)?
3. Are all status codes documented?
4. Is auth specified (not "standard auth")?
5. Is idempotency defined?
6. Are example payloads provided?

### Async Job / Event Consumer Spec

For any RFC with background workers, cron jobs, queue consumers, or event handlers:

| Job/Consumer | Trigger | Input Shape | Retry Policy | DLQ | Concurrency Limit | Idempotency Key | Timeout |
|---|---|---|---|---|---|---|---|

For each, check:
1. What triggers the job? (cron schedule, queue message, event, API call)
2. What is the exact input shape? (message schema, event payload fields with types)
3. What is the retry policy? (attempts, backoff, max delay)
4. Where do failed messages go? (DLQ name, retention, alerting)
5. What is the concurrency limit? (max workers, max in-flight messages)
6. What is the idempotency key? (how to dedup re-delivered messages)
7. What is the processing timeout per message?
8. What happens on poison messages that can never be processed?

If any of these are missing for a job/consumer described in the RFC → flag for FMC and DIC.

---

### Compliance Trigger Check

Scan the RFC for these triggers:
- Stores user name, email, phone, address, IP, device ID → PII
- Stores payment card, bank account → PCI scope
- Stores health data → HIPAA scope (if applicable)
- User content with retention → content governance
- Auth/session data → access audit requirements
- Cross-border data transfer → jurisdictional requirements

If any trigger found → CDG category is active and must be scored.

## Scoring Calibration Examples

These examples anchor what a low vs high score looks like for critical categories.
Use them to calibrate your judgment.

### DMS — Data Model & Schema: 4.0 vs 8.0

**Score 4.0 example:**
> "We'll create an `orders` table with columns for order ID, user, amount, status, and timestamps. We'll add indexes for common queries."

Why 4.0: no column types, no nullability, no constraints, no specific index definitions, no DDL. Agent must invent the entire schema.

**Score 8.0 example:**
> ```sql
> CREATE TABLE orders (
>   id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
>   user_id UUID NOT NULL REFERENCES users(id),
>   amount_cents BIGINT NOT NULL CHECK (amount_cents > 0),
>   currency VARCHAR(3) NOT NULL DEFAULT 'IDR',
>   status VARCHAR(20) NOT NULL DEFAULT 'pending' CHECK (status IN ('pending','confirmed','cancelled','refunded')),
>   created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
>   updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
> );
> CREATE INDEX idx_orders_user_status ON orders(user_id, status);  -- supports: GET /users/:id/orders?status=X
> CREATE INDEX idx_orders_created ON orders(created_at DESC);       -- supports: admin dashboard recent orders
> ```
> Expected volume: ~50k rows/month, ~2M rows at steady state. PII: user_id is a reference, no direct PII in this table. Retention: 7 years (financial regulation).

Why 8.0: DDL-level precision, indexes justified by named queries, volume quantified, PII classified, retention stated.

### DIC — Data Integrity: 4.0 vs 8.0

**Score 4.0 example:**
> "The service updates the order and sends a notification email."

Why 4.0: no transaction boundary, no partial failure handling, no idempotency. Agent doesn't know if the email is in the same transaction as the DB write.

**Score 8.0 example:**
> "Order update and outbox insert wrapped in a single DB transaction. Outbox worker polls every 5s, sends email with idempotency key `order:{id}:status_change:{new_status}`. Duplicate deliveries are deduped by the email service using this key (TTL 24h). If email send fails after 3 retries, message moves to DLQ `orders-email-dlq`."

Why 8.0: transaction boundary explicit, idempotency key defined with TTL, retry policy stated, DLQ named.

### FMC — Failure Modes: 4.0 vs 8.0

**Score 4.0 example:**
> "Handle errors gracefully. Retry failed requests."

Why 4.0: no timeout values, no retry counts, no backoff, no circuit breaker, no DLQ. "Gracefully" is not a spec.

**Score 8.0 example:**
> "Payment gateway call: 3s timeout, 2 retries with exponential backoff (500ms, 1.5s), circuit breaker opens at 40% error rate over 60s window (half-open after 30s). On persistent failure: return 503 to caller with `{ error: 'PAYMENT_GATEWAY_UNAVAILABLE', retry_after: 30 }`. Failed events → DLQ `payments-retry` with 7d retention. Alert: PagerDuty if DLQ depth > 100."

Why 8.0: per-call timeout, retry with specific backoff, circuit breaker with specific thresholds, specific error response, DLQ with retention, alert threshold.

### SAS — Security: 4.0 vs 8.0

**Score 4.0 example:**
> "Authenticated endpoint. Only authorized users can access."

Why 4.0: no middleware named, no scopes, no ownership check, no input validation, no injection analysis.

**Score 8.0 example:**
> "Bearer JWT required (validated by `AuthMiddleware`). Scope: `orders:write`. Ownership: `order.user_id == jwt.sub` (enforced in repository layer). Input validation: `amount` > 0 and <= 999999999 (cents), `currency` in ['IDR','USD','SGD'], `reference_id` max 64 chars alphanumeric+dash. SQL injection: all queries via sqlc (parameterized). Rate limit: 50 req/min per user on POST endpoints. Audit: all mutations logged to `audit.orders` with user_id, action, timestamp, diff."

Why 8.0: middleware named, scope specified, ownership check with enforcement point, per-field validation, injection mitigation named, rate limit quantified, audit logging specified.

---

### Frontend calibration examples

### CNT — Contract Specificity: 4.0 vs 8.0

**Score 4.0 example:**
> "The OrderCard component displays order information including status and amount."

Why 4.0: no prop types, no state shape, no event payloads. Agent invents everything.

**Score 8.0 example:**
> ```typescript
> interface OrderCardProps {
>   order: {
>     id: string;
>     status: 'pending' | 'confirmed' | 'cancelled' | 'refunded';
>     amountCents: number;
>     currency: 'IDR' | 'USD' | 'SGD';
>     createdAt: string; // ISO 8601
>     userName: string;
>   };
>   onCancel?: (orderId: string) => Promise<void>; // only shown when status === 'pending'
>   isLoading?: boolean; // default: false
> }
> ```
> Analytics event on cancel: `order_cancelled { order_id, reason, source: 'order_card' }`

Why 8.0: full TypeScript interface, conditional behavior specified, analytics event with exact name and properties.

### FMC (Frontend) — Failure Modes: 4.0 vs 8.0

**Score 4.0 example:**
> "Show an error message if the API call fails."

Why 4.0: which error message? Which API call? What about timeout vs 403 vs 500?

**Score 8.0 example:**
> "On `GET /orders` failure: 401 → redirect to login; 403 → show 'You don't have permission to view orders' inline banner; 429 → show 'Too many requests, please wait' toast with retry button (retry after `Retry-After` header); 500 → show 'Something went wrong' toast with retry button; timeout (10s) → show 'Connection timed out' toast with retry; offline → show cached data with 'You're offline — showing cached data' banner."

Why 8.0: per-status-code behavior, specific error messages, retry mechanism, offline handling, timeout value.

---

## Required evidence style

For every category, cite concrete evidence:

- exact section name or quote from the RFC
- or explicit absence: "no retry policy defined despite describing 3 external service calls"
- short explanation of why that evidence helps or hurts agentic execution

Avoid generic claims. Identify the specific schema, endpoint, or transaction boundary that's missing.
