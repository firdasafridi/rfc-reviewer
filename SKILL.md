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
- Prioritize implementation-contract clarity over infrastructure budgeting detail. Missing pod sizing, CPU/memory sizing, DB load estimates, storage growth projections, Redis memory estimates, or cloud-cost estimates should not block readiness by default unless the RFC is explicitly a capacity/cost planning RFC.
- If a section is absent, record that absence explicitly. Do not skip silently or assume it exists elsewhere.
- Use `0.5` score increments for category scores and the overall score.
- If the RFC is clearly too thin to assess reliably, say so and explain the limiting factors.

## Evidence normalization

Before scoring, normalize the source content:

- Treat sections titled like `Feedback`, `Comments`, `Reviewer Notes`, inline review threads, or conversational Q&A as non-authoritative discussion context.
- Do not use those sections as primary scoring evidence.
- Only score a point from those sections if the RFC explicitly promotes it into the authoritative spec body (for example: decision, contract, rollout, or acceptance criteria sections).
- If the fetched source mixes RFC content and discussion threads, state in the report that discussion sections were excluded from scoring evidence.

## Non-Negotiable SOP

Three phases. Within each phase, steps can run in parallel.

### Phase 1 — Intake (do first, before any scoring)

1. Read the full RFC content.
2. Normalize evidence: exclude non-authoritative feedback/comment sections (`Feedback`, `Comments`, `Reviewer Notes`, review threads) unless explicitly promoted into authoritative RFC sections.
3. Identify RFC type (`frontend`, `backend`, `full-stack`) and sub-type (frontend: `new-feature`/`enhancement`/`performance`; backend: `new-feature`/`enhancement`/`tech-improvement`; full-stack: both).
4. Read the rubric file(s) for that type and `report-template.md`.
5. Build an evidence log — note section by section what is present, thin, or absent.

### Phase 2 — Score & Deep-Dive (can run in parallel across categories)

6. **Run PRD → RFC traceability check** (forward + reverse). If no PRD, score PRT on self-contained justification (problem statement, success criteria, scope, non-goals); max 8.0.
7. Score all rubric categories `0.0`–`10.0` in `0.5` increments. Cite at least one concrete evidence per category.
8. **Run the Decision Closure protocol** on every major technical decision (see below).
9. **Run type-specific deep-dives:**
   - Frontend: UI State audit, Data flow trace, Performance budget check, Accessibility review, Pattern alignment check
   - Backend: Data integrity deep-dive, Concurrency collision map, API contract completeness check, Async job spec, Compliance trigger check
   - Full-stack: ALL of the above PLUS cross-layer contract verification, rollout compatibility matrix, end-to-end data flow trace
   - **Optimization:** for categories scoring `>= 7.0`, abbreviate deep-dive to one-line evidence in the scorecard instead of full tables.
10. Apply score caps from the rubric before finalizing the overall score.

### Phase 3 — Report

11. Write the full report using `report-template.md`. Include the Task Manifest.
12. Reply in chat with a compact summary:
    - Overall score and rating band (Agentic-Ready / Strong / Needs Work / Incomplete / Insufficient)
    - **Agentic Readiness Verdict**: PROCEED / HOLD / BLOCKED — one word with a one-line reason
    - RFC type + sub-type
    - 3 strongest categories
    - 3 biggest gaps (cite specific section or decision, not generic)
    - Top 3 concrete improvement actions before the RFC can be merged
    - Decision closure verdict: X of Y decisions resolved, X dangling
    - PRD traceability: X of Y PRD items covered, X missing
    - Path to saved report

## Decision Closure Assessment — Protocol

This is the most critical differentiator of this skill. Applies to ALL RFC types.
For EVERY major technical decision, run all three checks:

### Check 1 — Resolution & Alternatives
- Is the decision clearly stated with the chosen option named (not buried in prose)?
- Were alternatives listed and rejected with specific reasons tied to constraints, measurements, or codebase reality — not dismissed in one sentence?
- Does the decision reference specific files, modules, services, or tables? Could you find the code from the RFC alone?

### Check 2 — Specification & Failure Handling
- Is the resulting interface fully specified? (API schema, component props, state shape, function signature — types, fields, nullability, error codes all pinned.)
- What happens on the unhappy path? Backend: partial failure, retry, concurrent write. Frontend: network failure, empty data, stale cache, navigation mid-request.
- Are there ordering dependencies? Acceptance criteria per implementation chunk?

### Check 3 — Challenge Protocol
- **Scale**: Does this hold at 10x load? What breaks first?
- **Reversibility**: How hard to change after code is written? After data migration? After user adoption?
- **Consistency**: Conflicts with other decisions in the RFC?
- **Agent implementability**: Could an agent implement this without asking the author a question? What would it guess?

### Scoring decisions

- **Resolved**: decision made, alternatives rejected with reason, interface specified, failure handled. Agent can implement.
- **Partial**: decision made but interface incomplete or failure handling missing. Agent would guess on details.
- **Dangling**: "we could do X or Y" without resolution. Agent cannot proceed.

For each Partial or Dangling: list exactly what is missing, propose a concrete resolution using the RFC's own context, and write specific open questions for the author.

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
- `rubric-backend.md` — backend RFC rubric: 12 core + 1 conditional scored categories (+ resource/cost advisory), 19 reviewer skills, skill matrix by sub-type, backend-specific score caps, data integrity/concurrency/API/compliance deep-dives
- `rubric-fullstack.md` — full-stack RFC rubric: 18 unified scored categories (+ resource/cost advisory) with explicit merge rules for shared categories, cross-layer deep-dives (contract verification, rollout compatibility, end-to-end flow), cross-layer score caps
- `report-template.md` — unified report template that adapts per RFC type
