# RFC Review Rubric — Frontend

Frontend RFC review assesses whether an AI agent or engineer can implement
the proposed frontend changes — without asking the author clarifying questions
about UI states, component contracts, data flow, or interaction patterns.

Score what is actually present in the RFC. Do not score intent or "they'll
figure it out during implementation."

## Core question

How likely is an AI coding agent (or a human frontend engineer) to:

- build every UI state (loading, empty, error, partial, success) correctly on the first pass
- implement component contracts, props, and state management without guessing
- handle all user interaction paths including edge cases and error recovery
- produce accessible, performant code that matches existing codebase patterns
- validate their implementation against concrete acceptance criteria

## Frontend sub-type calibration

Identify which sub-type best describes this RFC:

- `new-feature`: new page, view, or standalone feature; deepest coverage needed across all skills
- `enhancement`: modifying existing pages or components; regression, migration, and pattern alignment are critical
- `performance`: optimizing existing frontend; trade-offs, observability, and rollback are critical

## The 17 Reviewer Skills

These skills are the lenses applied during review. Each produces evidence for scoring.

| # | Skill | New Feature | Enhancement | Performance |
|---|---|:---:|:---:|:---:|
| 1 | Systems thinking & dependency mapping | M | **M** | M |
| 2 | Requirements & scope interrogation | **M** | M | O |
| 3 | User flow & state modeling | M | M | O |
| 4 | Trade-off analysis | M | M | **M** |
| 5 | Non-functional requirements literacy | M | M | M |
| 6 | Security & privacy threat modeling | **M** | O | O |
| 7 | Performance reasoning | M | M | **M** |
| 8 | Data flow & state management critique | M | M | M |
| 9 | API contract & backend coordination | M | O | O |
| 10 | Failure mode imagination | **M** | **M** | **M** |
| 11 | Migration & rollout reasoning | O | **M** | **M** |
| 12 | Observability thinking | M | M | **M** |
| 13 | Accessibility & inclusive design | **M** | O | O |
| 14 | Abstraction judgment | **M** | M | O |
| 15 | Cost of change / reversibility | M | **M** | M |
| 16 | Communication & written critique | M | M | M |
| 17 | Knowing what's missing | **M** | **M** | **M** |

**M** = Mandatory for this sub-type
**Bold M** = Particularly high-leverage for this sub-type
**O** = Optional / context-dependent (see activation rules below)

### When "Optional" becomes mandatory

- **Security** (enhancement & performance): mandatory if RFC touches auth, user data, input handling, third-party integrations, caching of sensitive data, or session management.
- **Requirements interrogation** (performance): mandatory if the perf target isn't quantified. "Make it faster" is not a requirement.
- **User flow** (performance): mandatory if optimization changes timing of what users see (deferred hydration, skeleton screens, progressive loading).
- **API contract** (enhancement / performance): mandatory if RFC adds, changes, removes, or batches any API call.
- **Accessibility** (enhancement / performance): mandatory if RFC touches DOM structure, focus order, visible content, or animations. Lazy-loading and virtualization often break screen readers.
- **Migration & rollout** (new feature): mandatory if the feature is large or risky.
- **Abstraction judgment** (performance): mandatory if the proposal introduces caching layers, new state stores, or restructures data flow.

---

## Category Scores (11 categories)

Score each category from `0.0` to `10.0`. Use `0.5` increments only.

---

### PRT — PRD Traceability

**Core question:** Does every PRD requirement have an RFC answer, and does every RFC decision have a PRD justification?

Look for:
- Forward traceability: every PRD user story → which RFC section implements it
- Forward traceability: every PRD business rule → which RFC technical decision enforces it
- Forward traceability: every PRD acceptance criterion → which contract or test satisfies it
- Reverse traceability: every RFC technical decision → which PRD requirement drives it
- Reverse traceability: every new dependency, abstraction, or system → what PRD need does it serve
- Missing forward mapping = RFC has a coverage gap
- Missing reverse mapping = scope creep or PRD was under-specified

High score:
- Explicit traceability matrix present. Bidirectional coverage.
- Agent knows which PRD requirement each piece of code satisfies.

Low score:
- No PRD link, no mapping attempted, silent drops, or unjustified RFC additions.
- Agent doesn't know why it's building what it's building.

**No-PRD exception:** Tech-debt, performance, or infra RFCs may have no PRD. In that case, score PRT on self-contained justification: does the RFC state its own problem, success criteria, scope boundaries, and non-goals clearly enough for an agent to know *why* it's building this? Max PRT score without a PRD: 8.0.

---

### TDC — Technical Decision Completeness

**Core question:** Are all architectural and design choices made — not deferred?

Look for:
- Every decision explicitly stated with chosen option named
- Alternatives considered and rejected with specific reasons tied to constraints or measurements
- No "TBD", "to be decided during implementation", "we could use X or Y" left dangling
- Rationale grounded in codebase reality, not abstract preference

High score:
- Agent can implement every choice without asking "which approach?"
- All trade-offs documented.

Low score:
- Multiple unresolved decisions or "we'll figure it out."
- Agent must make architectural choices the RFC author should have made.

---

### CNT — Contract Specificity

**Core question:** Are API shapes, component props, state structures, and event payloads fully specified?

Look for:
- Component prop contracts: name, type, required/optional, default value, validation
- State structures: shape, ownership (which store/context), update patterns
- API request/response shapes consumed by frontend: field names, types, nullability, example payloads
- Event payloads: analytics events with exact names and properties, custom events
- Conditional fields: example payloads for each branch

High score:
- Agent can generate TypeScript interfaces directly from the RFC.
- No guessing about what data flows where.

Low score:
- "Component takes user data" without specifying which fields, types, or optionality.
- Agent invents prop types and state shapes.

**Data fetching strategy (must be present for score >= 7.0):**
- How is data fetched? (SWR, React Query, Pinia actions, plain fetch, RTK Query, Apollo, etc.)
- Caching strategy: what's cached, TTL, cache key structure
- Refetch triggers: when does data refetch? (on mount, on focus, on interval, on event)
- Stale-while-revalidate: is stale data shown while fresh data loads?
- Optimistic updates: any mutations that update UI before server confirms?
- Without this, the agent guesses the fetching pattern — the #1 source of frontend data bugs.

---

### SCB — Scope Boundaries

**Core question:** Is it crystal clear which files, components, and folders are in scope — and what is explicitly out?

Look for:
- Named files to create or modify (specific paths, not "update the inbox")
- Named components to touch, with reason for each
- Named components that are explicitly NOT touched (non-goals)
- Integration scope: which other systems, pages, or shared components are affected
- For enhancements: impact analysis of shared code (a Button used in 47 places)

High score:
- Agent can produce a file-by-file implementation plan from the RFC alone.
- No surprise regressions from touching shared code.

Low score:
- "Update the checkout flow" with no file list.
- Agent must search the codebase to figure out what exists and what to change.

---

### DEP — Dependency Explicitness

**Core question:** Are all dependencies (packages, services, backend endpoints, feature flags, shared components) named with confirmed status?

Look for:
- External packages: name, version, bundle size impact, why this package
- Backend endpoints consumed: method, path, availability status (exists / needs building / blocked)
- Shared components or design system components used
- Feature flag name and who provisions it
- Cross-team dependencies with owning team and status (confirmed / assumed / unvalidated)

High score:
- Agent can start building knowing every external piece is either available or explicitly blocked.
- No "we need an endpoint for X" without checking if it exists.

Low score:
- Implicit dependencies that will surface mid-implementation.
- Agent writes code calling endpoints that don't exist yet.

---

### FMC — Failure Mode Coverage

**Core question:** Does every external interaction have defined failure behavior?

Look for:
- API failure handling per endpoint: what does the UI show on 400, 401, 403, 404, 429, 500, timeout, offline?
- Network resilience: slow 3G behavior, offline state, retry options
- Third-party script failures: what happens if analytics, CDN, or embedded widget fails to load?
- Race conditions: user clicks fast, navigates away mid-request, submits form twice
- Stale data: what if cached data is outdated? What if WebSocket reconnects with stale state?
- Graceful degradation: what works without JavaScript? Without certain browser features?

High score:
- Agent implements error boundaries, retry logic, and fallback UI for every external interaction.
- No "handle errors gracefully" — specific behavior per failure type.

Low score:
- Only happy path described.
- Agent builds code that shows blank screens or infinite spinners on failure.

**Error message catalog (must be present for score >= 7.0):**
- For each error the user can encounter: the exact error message string or i18n key
- Error code → user-facing message mapping (e.g., `INSUFFICIENT_BALANCE` → "Your balance is too low to complete this action")
- Which errors are user-facing (shown in UI) vs logged-only (silent to user, toast to retry)
- Without this, the agent invents copy or shows raw error codes to users.

---

### NFS — Non-Functional Specificity

**Core question:** Are performance, accessibility, security, i18n, and browser targets quantified?

Look for:
- **Performance**: LCP, INP, CLS targets with specific numbers. Bundle size budget. Code-splitting strategy.
- **Accessibility**: WCAG level (AA minimum), keyboard navigation flow, screen reader behavior, focus management, color contrast ratios, ARIA usage
- **Browser support**: specific versions (Chrome/Safari/Firefox/Edge, last N versions, iOS Safari)
- **Responsive**: breakpoints, mobile-specific behavior, touch targets
- **i18n/l10n**: RTL support, string externalization, date/number formatting
- **Security**: input sanitization rules, no `v-html`/`dangerouslySetInnerHTML` without sanitization, HTTPS-only resources, CSRF handling

High score:
- "LCP < 2.5s on throttled 3G; WCAG AA verified with axe-core; Chrome/Safari/Firefox last 2 versions"
- Agent knows exactly what non-functional tests to write.

Low score:
- "Should be fast and accessible."
- Agent ships code with no performance optimization and no a11y testing.

---

### TPS — Test Plan Specificity

**Core question:** Are acceptance criteria specific enough to become automated tests without interpretation?

Look for:
- Given/When/Then per story or interaction, including at least one failure path
- Unit tests: which components, which logic, what assertions
- Integration tests: component + store, component + API mock, full user flow
- E2E tests: named scenarios per user journey
- Visual regression: if design fidelity matters, are snapshot/screenshot tests planned?
- Accessibility tests: axe-core integration, keyboard navigation test scenarios
- Performance tests: Lighthouse CI targets, bundle size assertions

High score:
- QA or agent can write the full test suite from the RFC.
- Named scenarios: "empty inbox", "500 error on room load", "rapid room switching", "keyboard-only navigation."

Low score:
- "Works correctly" or "100% coverage."
- Agent generates trivial happy-path tests.

---

### ROL — Rollout & Rollback Plan

**Core question:** Is there a concrete plan to ship safely, measure success, and revert if it breaks?

Look for:
- Feature flag name, default state, targeting rules, kill-switch behavior
- Rollout stages with named audiences and evidence gates (internal → 1% → 10% → 100%)
- Rollback mechanism: "revert the PR" is acceptable for frontend-only; describe what happens to users mid-session
- Stop conditions: specific metric regressions or error patterns that trigger rollback
- Cache invalidation: old JS/CSS bundles, service workers, CDN caches
- For enhancements: backward compatibility with saved user state (localStorage, cookies)
- Blast radius: worst case, what breaks, who is affected

High score:
- Team can execute rollout and rollback without calling the author.
- Agent generates feature flag checks and rollout configuration.

Low score:
- "Deploy to prod" with no flags, no stages, no rollback plan.
- Agent ships code without safety controls.

**Configuration contract (must be present):**
- Every new environment variable or build-time config: name, type, default value
- Feature flag name, default state, who provisions it, targeting rules
- Any API base URL or endpoint config that differs per environment
- Without this, the agent writes code that works locally but breaks in staging/production.

---

### OBS — Observability Definition

**Core question:** How will the team know if this works — and know quickly when it breaks?

Look for:
- Specific metric names to emit (not "we'll add metrics")
- Analytics events: exact event names, properties, when they fire
- Error monitoring: which errors are tracked, severity levels, alert thresholds
- Performance monitoring: which Core Web Vitals are tracked, dashboards
- For performance RFCs: before/after measurement methodology, Lighthouse CI integration
- User-facing success metrics tied back to PRD (conversion, task completion, perceived speed)

High score:
- On-call can debug from the observability described here.
- Agent includes correct analytics and error tracking in generated code.

Low score:
- "Add logging" without naming what to log.
- Agent generates code with no observability — failures are silent.

---

### CPA — Consistency & Pattern Alignment

**Core question:** Does the RFC align with existing codebase patterns — or deviate with explicit justification?

Look for:
- State management: matches existing pattern (Redux/Vuex/Context/Pinia) or justifies deviation
- Folder structure: follows existing conventions or explicitly proposes new structure with reason
- Component patterns: matches existing composition, naming, prop conventions
- Styling approach: matches existing CSS/SCSS/CSS-in-JS/Tailwind pattern
- Error handling pattern: matches existing error boundary, toast, retry patterns
- For enhancements: no silent introduction of new patterns alongside old ones (creating parallel systems)
- References specific files to mirror: "follow the pattern in `src/components/Chat/ChatRoom.vue`"

High score:
- Agent generates code that looks like it belongs in the existing codebase.
- Explicit: "follows existing pattern X in Y" or "intentionally deviates because Z."

Low score:
- Silently introduces React hooks in an options-API Vue codebase.
- Agent generates code in a style that clashes with every neighboring file.

---

## Relative weighting by sub-type

| Category | New Feature | Enhancement | Performance |
|---|:---:|:---:|:---:|
| PRT — PRD Traceability | High | High | Medium |
| TDC — Technical Decisions | High | High | High |
| CNT — Contract Specificity | **Highest** | High | Medium |
| SCB — Scope Boundaries | High | **Highest** | Medium |
| DEP — Dependencies | High | Medium | Low |
| FMC — Failure Modes | **Highest** | **Highest** | High |
| NFS — Non-Functional | High | Medium | **Highest** |
| TPS — Test Plan | High | High | High |
| ROL — Rollout & Rollback | Medium | **Highest** | **Highest** |
| OBS — Observability | Medium | Medium | **Highest** |
| CPA — Pattern Alignment | High | **Highest** | Medium |

## Score caps (non-negotiable gates)

Apply before finalizing the overall score. Strictest cap wins.

- `PRT < 5.0` → overall max **6.0** — coverage is the foundation
- `TDC < 5.0` → overall max **6.0** — an RFC with unresolved decisions isn't an RFC yet
- `CNT < 5.0` → overall max **6.5** — agents will hallucinate shapes
- `FMC < 4.0` → overall max **7.0** — optimistic RFCs ship broken code
- `CPA < 4.0` (for enhancements) → overall max **7.0** — violating patterns causes regressions
- `NFS < 4.0` (for performance RFCs) → overall max **6.0** — can't optimize what you haven't quantified
- `ROL < 4.0` (for enhancements/performance) → overall max **7.0** — risky changes need rollback plans
- Any 3 categories below `5.0` → overall max **6.5**
- To earn `9.0+`: all categories must be `>= 7.5`; `CNT` and `FMC` must be `>= 8.5`; no dangling decisions
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

> If Claude Code were given this RFC, how likely is it to produce correct, working frontend code — all UI states, all edge cases, matching existing patterns — without asking the author a question?

## Confidence rules

- `High`: all sections present; specifications are direct and consistent
- `Medium`: most sections present but 1–2 are thin or missing
- `Low`: multiple critical sections absent or contradictory

Low-confidence assessments must avoid top-end scores regardless of what is present.

## Frontend-Specific Deep-Dives

These are mandatory review sections that go beyond category scoring.

### UI State Audit

For every data-driven component in the RFC, verify these 5 states are specified:
1. **Loading** — what the user sees while data is being fetched
2. **Empty** — what the user sees when data returns but is empty
3. **Error** — what the user sees when the request fails
4. **Partial** — what the user sees when some data loaded but some failed
5. **Success** — the happy path (this is the one most RFCs have)

Score: count of components × states defined ÷ components × 5. Report missing states.

### Performance Budget Check

For performance and new-feature RFCs:
- Is there a bundle size budget? What's the delta?
- Is there a Core Web Vitals budget (LCP, INP, CLS)?
- Is code-splitting planned for route-level or component-level?
- Are images optimized (WebP/AVIF, proper sizing, lazy loading)?
- Are third-party scripts accounted for?

### Accessibility Review

For new-feature RFCs and enhancements touching DOM:
- Is keyboard navigation flow described?
- Are focus management rules specified (modal open/close, route change)?
- Are ARIA labels specified for non-semantic interactive elements?
- Is heading hierarchy correct?
- Are color contrast ratios verified?
- Is motion sensitivity addressed (prefers-reduced-motion)?

### Data Flow Trace

For each major user interaction, trace the data path through the frontend:

```
User action → Component → Store/Service call → API request → Response handling → State update → Re-render
```

Check:
- Is the full frontend data path documented?
- Are there gaps where data shape changes without a transformation being specified?
- Are side effects (analytics, cache invalidation, other store updates) shown?

---

### Pattern Alignment Check

For enhancement RFCs:
- List every existing pattern the RFC touches
- For each: does the RFC follow it, extend it, or replace it?
- If replacing: is the old pattern cleaned up or does it create a parallel system?
- Are existing analytics events preserved?

## Required evidence style

For every category, cite concrete evidence:

- exact section name or quote from the RFC
- or explicit absence: "no empty state defined for the inbox list component"
- short explanation of why that evidence helps or hurts agentic execution

Avoid generic claims such as "add more detail to UI states" unless you identify
the specific component and the specific missing state.

## Frequently Overlooked Issues

Flag these if not addressed in the RFC:
- Memory leaks from event listeners and subscriptions not cleaned up
- Race conditions from rapid user clicks or navigation during fetch
- Timezone handling (store UTC, display local)
- Keyboard trap in modals
- Missing focus return after dialog close
- Hardcoded strings that should be i18n
- Unhandled promise rejections
- Hydration mismatches in SSR
- Missing debounce on expensive handlers (search, resize, scroll)
- Safari iOS quirks (100vh, date inputs, scroll behavior)
- Old service worker caches after deploy
