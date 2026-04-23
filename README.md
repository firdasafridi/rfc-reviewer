# rfc-reviewer

A skill for AI coding assistants that reviews engineering RFCs the way a principal engineer would. It scores whether an agent or developer can pick up the RFC and start implementing without scheduling a meeting to ask "what did you mean by this?"

## What it does

You point it at an RFC, it reads the whole thing, and tells you if it's ready for someone (or something) to implement. The output is a report with:

- A verdict: PROCEED, HOLD, or BLOCKED
- Scores per category (0.0 to 10.0, in 0.5 increments)
- A traceability matrix mapping PRD requirements to RFC sections (and back)
- Every technical decision checked against a 7-point closure protocol
- A list of what to fix, in priority order

## Supported RFC types

| Type | Sub-types | Scored categories |
|------|-----------|-------------------|
| Frontend | new-feature, enhancement, performance | 11 |
| Backend | new-feature, enhancement, tech-improvement | 13-14 |
| Full-stack | new-feature, enhancement, tech-improvement | 19 |

Each type uses its own rubric. A frontend RFC gets evaluated differently from a backend one.

## Installation

### Claude Code

Clone into the skills directory:

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/firdasafridi/rfc-reviewer.git ~/.claude/skills/rfc-reviewer
```

Or copy files manually if you already have the repo:

```bash
mkdir -p ~/.claude/skills/rfc-reviewer
cp SKILL.md rubric-*.md report-template.md ~/.claude/skills/rfc-reviewer/
```

### OpenCode

```bash
mkdir -p ~/.config/opencode/skills
git clone https://github.com/firdasafridi/rfc-reviewer.git ~/.config/opencode/skills/rfc-reviewer
```

Or copy manually:

```bash
mkdir -p ~/.config/opencode/skills/rfc-reviewer
cp SKILL.md rubric-*.md report-template.md ~/.config/opencode/skills/rfc-reviewer/
```

> OpenCode also scans `~/.claude/skills/`, so cloning there once covers both tools.

### Cursor

```bash
mkdir -p ~/.cursor/rules
git clone https://github.com/firdasafridi/rfc-reviewer.git /tmp/rfc-reviewer
cp /tmp/rfc-reviewer/SKILL.md ~/.cursor/rules/rfc-reviewer.mdc
```

For project-level usage:

```bash
mkdir -p .cursor/rules
cp SKILL.md .cursor/rules/rfc-reviewer.mdc
cp rubric-*.md report-template.md .cursor/rules/
```

### GitHub Copilot

```bash
mkdir -p .github
git clone https://github.com/firdasafridi/rfc-reviewer.git /tmp/rfc-reviewer
cp /tmp/rfc-reviewer/SKILL.md .github/copilot-instructions.md
```

For organization-wide usage:

```bash
mkdir -p ~/.config/github-copilot/instructions
cp SKILL.md rubric-*.md report-template.md ~/.config/github-copilot/instructions/
```

### Antigravity

```bash
mkdir -p ~/.antigravity/skills
git clone https://github.com/firdasafridi/rfc-reviewer.git ~/.antigravity/skills/rfc-reviewer
```

Or copy manually:

```bash
mkdir -p ~/.antigravity/skills/rfc-reviewer
cp SKILL.md rubric-*.md report-template.md ~/.antigravity/skills/rfc-reviewer/
```

### Windsurf

```bash
mkdir -p ~/.codeium/windsurf/rules
git clone https://github.com/firdasafridi/rfc-reviewer.git /tmp/rfc-reviewer
cp /tmp/rfc-reviewer/SKILL.md ~/.codeium/windsurf/rules/rfc-reviewer.md
```

For project-level usage:

```bash
mkdir -p .windsurf/rules
cp SKILL.md .windsurf/rules/rfc-reviewer.md
cp rubric-*.md report-template.md .windsurf/rules/
```

### Other tools

If your tool supports custom instructions or system prompts, paste the contents of `SKILL.md` into the instruction field. Make sure the rubric files (`rubric-backend.md`, `rubric-frontend.md`, `rubric-fullstack.md`) and `report-template.md` are accessible in the working directory.

## Usage

### Claude Code / OpenCode

```
/rfc-reviewer <path-or-url>
```

### Cursor / Copilot / Windsurf / others

```
Review this RFC using the rfc-reviewer skill: <path-or-url>
```

### Accepted inputs

- Markdown files: `./my-rfc.md`
- PDF files: `./my-rfc.pdf`
- Word documents: `./my-rfc.docx`
- Public URLs: `https://docs.example.com/my-rfc`
- Confluence, Notion, or Google Docs (if you have MCP configured)
- Pasted content in chat

The report gets written to `rfc-review-report.md` in the same directory as the input.

## Scoring

| Score | Rating | What it means |
|-------|--------|---------------|
| 8.5-10.0 | Agentic-ready | An agent can implement this without guessing |
| 7.0-8.4 | Strong | Mostly there, a few things need clarifying |
| 5.5-6.9 | Needs work | Gaps that would force guessing on important details |
| 3.5-5.4 | Incomplete | Too many missing specs and open questions |
| 0.0-3.4 | Insufficient | Not enough to work with, needs a rewrite |

## Project structure

```
rfc-reviewer/
├── SKILL.md              # How the skill operates (4-phase process)
├── rubric-backend.md     # Backend evaluation rubric, 19 reviewer skills
├── rubric-frontend.md    # Frontend evaluation rubric, 17 reviewer skills
├── rubric-fullstack.md   # Full-stack rubric with cross-layer merge rules
└── report-template.md    # Output template for the assessment report
```

## How it works

The assessment runs in four phases:

1. Read the full RFC, figure out the type (frontend/backend/full-stack), load the right rubric, start collecting evidence
2. Check PRD traceability, score each category, run the decision closure protocol on every technical choice
3. Deep-dive into any category that scored below 7.0 with type-specific audits
4. Write the report and deliver a verdict

## Design choices

The scoring is judgment-based, not a checklist. Every score has to point at a specific section of the RFC (or the absence of one). Different RFC types get different rubrics because evaluating a frontend performance RFC the same way you'd evaluate a backend data migration makes no sense.

The tool is deliberately strict. If something is unclear enough that an implementer would have to guess, the score reflects that. Comfortable reports that hide gaps help nobody.

## License

[MIT](LICENSE) - Firda Safridi
