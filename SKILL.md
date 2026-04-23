---
name: rfc-reviewer
description: Review an engineering RFC for agentic-execution readiness — the go/no-go gate before an AI agent or engineer begins implementation. Use when the user wants to know if an RFC is ready for agent-driven coding, wants technical decisions challenged before merge, or wants a scored assessment with concrete improvement recommendations.
argument-hint: [rfc-path-or-url]
disable-model-invocation: true
allowed-tools: Read WebFetch Write Bash(textutil -convert txt -stdout *)
---

# RFC Review

Use this skill when the user wants a real RFC assessment, not a surface-level checklist scan.
Your job is to review the RFC the way a principal engineer would — catching gaps, ambiguities,
unresolved decisions, and missing specifications before a single line of code is written.

The goal of this review is **agentic execution readiness**: after this review, the score tells you
whether an AI agent or engineer can implement code directly from this RFC — without asking the
author clarifying questions. The score is the go/no-go gate before implementation begins.

This skill produces **the assessment report and score only**. It does not generate code.
It is the guardrail that runs before the implementation phase.

## RFC types

This skill supports three RFC types, each with a different rubric:

| Type | When to use | Rubric file |
|------|-------------|-------------|
| **frontend** | Frontend-only changes: UI, components, state, styling, client-side performance. No backend or data layer changes. | `rubric-frontend.md` |
| **backend** | Backend-only changes: APIs, services, data layer, workers, infrastructure. No frontend changes. | `rubric-backend.md` |
| **full-stack** | Touches both frontend and backend: new features spanning UI + API + data, or cross-cutting changes. | `rubric-fullstack.md` (references both frontend and backend rubrics for detailed criteria; adds cross-layer merge rules, compatibility checks, and unified scoring) |

## Input handling

The skill accepts five input forms:

1. **Text file** — user passes a path to a `.md`, `.txt`, `.rst`, or similar plain-text file; read with the Read tool.
2. **PDF** — user passes a `.pdf` path; read with the Read tool. For large PDFs, read in page ranges (e.g. `pages: "1-10"`).
3. **Word document** — user passes a `.docx` or `.doc` path; extract text with:
   ```
   textutil -convert txt -stdout <path>
   ```
4. **URL** — user passes a Confluence, Notion, or Google Docs URL:
   - **Public URL**: fetch with WebFetch.
   - **Confluence via MCP**: if a Confluence MCP is installed and available, use its page-fetch tool to retrieve authenticated content. Add the MCP tool name to `allowed-tools` in the SKILL.md frontmatter once the MCP is set up.
   - If access fails by any method, ask the user to export or paste the content.
5. **Pasted content** — user pastes RFC text directly; analyze what was provided.

If no argument is passed, ask the user to provide the RFC as a file path, URL, or pasted content.

## Default output path

- If the user passed a path argument, write the report to `rfc-review-report.md` in the same directory.
- If no argument, write to `rfc-review-report.md` in the current directory.
- Never write outside the current working directory.
- Reject absolute paths and parent traversal such as `../`.

## Assessment rules

- This is a judgment-based review. Do not tick boxes mechanically.
- Cite specific sections, sentences, or code/schema examples from the RFC. Do not make generic claims.
- Focus on whether an AI agent or engineer can start implementing without a clarification meeting.
- If a section is absent, record that absence explicitly. Do not skip silently or assume it exists elsewhere.
- Use `0.5` score increments for category scores and the overall score.
- If the RFC is clearly too thin to assess reliably, say so and explain the limiting factors.

## Non-Negotiable SOP

Four phases. Within each phase, steps can run in parallel.

### Phase 1 — Intake (do first, before any scoring)

1. Read the full RFC content.
2. Identify the RFC type (`frontend`, `backend`, `full-stack`) and sub-type:
   - Frontend: `new-feature`, `enhancement`, or `performance`
   - Backend: `new-feature`, `enhancement`, or `tech-improvement`
   - Full-stack: identify both frontend and backend sub-types
3. Read the rubric file(s) for that type and `report-template.md`.
4. Build an evidence log — note section by section what is present, thin, or absent.

### Phase 2 — Score (can run in parallel across categories)

5. **Run PRD → RFC traceability check:**
   - Forward: every PRD requirement → which RFC section implements it?
   - Reverse: every RFC decision → which PRD requirement drives it?
   - Produce a traceability matrix. Missing mappings are gaps.
   - **If no PRD exists** (tech-debt, performance, infra RFCs): PRT is scored on whether
     the RFC states its own problem/motivation clearly enough that an agent knows *why*
     it's building this. Score against self-contained justification instead of traceability.
     The RFC must have: problem statement, success criteria, scope boundaries, and
     explicit non-goals. If these are present and clear, PRT can score up to 8.0 without
     a PRD. If absent, PRT scores as if the PRD link were missing.
6. Score all rubric categories from `0.0` to `10.0` using `0.5` increments only.
   For each category, cite at least one concrete piece of evidence.
7. **Run the Decision Closure protocol** on every major technical decision (see below).

### Phase 3 — Deep-Dive (only for categories that scored below 7.0, plus mandatory deep-dives)

8. **Run type-specific deep-dives:**
   - Frontend: UI State audit, Data flow trace, Performance budget check, Accessibility review, Pattern alignment check
   - Backend: Data integrity deep-dive, Concurrency collision map, API contract completeness check, Async job spec, Compliance trigger check
   - Full-stack: ALL of the above PLUS cross-layer contract verification, cross-layer rollout compatibility matrix, end-to-end data flow trace (see `rubric-fullstack.md`)
   - **Optimization:** deep-dive sections for categories scoring >= 7.0 can be abbreviated to one-line evidence in the scorecard instead of full tables.
9. Apply score caps from the rubric before finalizing the overall score.

### Phase 4 — Report

10. Write the full report using `report-template.md`. Include the Task Manifest.
11. Reply in chat with a compact summary:
    - Overall score and rating band (Agentic-Ready / Strong / Needs Work / Incomplete / Insufficient)
    - **Agentic Readiness Verdict**: PROCEED / HOLD / BLOCKED — one word with a one-line reason
    - RFC type + sub-type
    - 3 strongest categories
    - 3 biggest gaps (cite specific section or decision, not generic)
    - Top 3 concrete improvement actions before the RFC can be merged
    - Decision closure verdict: X of Y decisions resolved, X dangling
    - PRD traceability: X of Y PRD items covered, X missing
    - Path to saved report

## Decision Closure Assessment — Detailed Protocol

This is the most critical differentiator of this skill. Applies to ALL RFC types.
For EVERY technical decision in the RFC, run both steps: **resolution check** then **challenge protocol**.

### Step 1 — Resolution check (what must be present)

**1. Decision identity**
- Is the decision clearly stated? Not buried in prose.
- Is the chosen option explicitly named?

**2. Alternatives considered**
- Were alternatives listed?
- Is each alternative rejected with a specific reason tied to a constraint, measurement, or codebase reality?

**3. Grounding in existing code**
- Does the decision reference specific files, modules, services, components, or tables?
- Could you find the code being discussed from the RFC alone?

**4. Interface specification**
- Is the resulting interface (API, schema, component props, state shape, function signature) fully specified?
- Types, fields, nullability, error codes — all pinned down?

**5. Failure handling**
- What happens when this decision encounters the unhappy path?
- For backend: partial failure, retry, timeout, concurrent write, stale read?
- For frontend: network failure, empty data, slow response, stale cache, user navigates away mid-request?

**6. Implementation constraints**
- Are there ordering dependencies?
- Are acceptance criteria stated per implementation chunk?

### Step 2 — Challenge protocol (probe for holes)

After the resolution check, actively challenge each major technical decision:

- **Alternative challenge**: Was the rejected alternative genuinely evaluated, or dismissed in one sentence?
- **Scale challenge**: Does this decision hold at 10x current load/users? What breaks first?
- **Failure challenge**: What's the worst thing that happens if this specific choice is wrong?
- **Reversibility challenge**: How hard is it to change this decision after code is written? After data is migrated? After users have adopted it?
- **Consistency challenge**: Does this decision conflict with any other decision in the same RFC?
- **Codebase challenge**: Does this decision match existing patterns, or introduce a new one? If new, is that justified?
- **Agent challenge**: Could an AI agent implement this without asking the author a question? What would it guess?

### Scoring decisions

- **Resolved**: decision made, alternatives rejected with reason, interface specified, failure handled. Agent can implement.
- **Partial**: decision made but interface incomplete, or failure handling missing. Agent would guess on details.
- **Dangling**: "we could do X or Y" without resolution, or decision buried as ambiguity. Agent cannot proceed.

For each Partial or Dangling decision, include:
- A **gap list**: exactly what is missing
- A **resolution suggestion**: a concrete proposed resolution based on the RFC's own context
- **Open questions**: specific questions to resolve with the author before merge

## Score discipline

- Category scores follow the rubric anchors, not general impression.
- `9.0+` requires at least 2 strong, non-conflicting evidence points per category.
- `10.0` is extremely rare.
- Always apply score caps from the type-specific rubric.

## Report requirements

- Make the full report concrete and evidence-backed.
- Quote or reference specific RFC sections whenever possible.
- Include: PRD traceability matrix, strengths, gaps, per-decision assessment, type-specific deep-dives, improvement recommendations.
- For each gap, write a concrete improvement suggestion — not generic "add more detail" but the actual missing spec or the specific question the author needs to answer.
- Make recommendations sequenced and actionable.
- End the report with a **Dangling Decisions** section and an **Open Questions** section.
- The report should be uncomfortable to read if the RFC is not ready — that discomfort is the point.

## Rubric files

- `rubric-frontend.md` — frontend RFC rubric: 11 categories, 17 reviewer skills, skill matrix by sub-type, frontend-specific score caps
- `rubric-backend.md` — backend RFC rubric: 13 core + 1 conditional categories, 19 reviewer skills, skill matrix by sub-type, backend-specific score caps, data integrity/concurrency/API/compliance deep-dives
- `rubric-fullstack.md` — full-stack RFC rubric: 19 unified categories with explicit merge rules for shared categories, cross-layer deep-dives (contract verification, rollout compatibility, end-to-end flow), cross-layer score caps
- `report-template.md` — unified report template that adapts per RFC type
