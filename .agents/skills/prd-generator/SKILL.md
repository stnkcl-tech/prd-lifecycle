---
name: prd-generator
description: "Generate Product Requirements Documents grounded in the existing code reality of the Selangkah moneywise repo. Use when scoping a new feature, inheriting a partially-built feature, or assessing feasibility against existing code."
---

# PRD Generator

## Purpose

Produce a feature PRD that explicitly describes the current state of related capabilities in the moneywise repo (Selangkah) before defining what needs to change. This prevents speculative requirements and surfaces feasibility risks early.

## When to use

- New feature scoped by client or team
- Before writing backlog tickets
- When inheriting partially-built features
- When a feature request needs feasibility assessment against existing code

## Inputs

Before invoking this skill, gather:

1. Feature description from the PM (1-2 sentences, provided in chat).

2. **Branch name** to ground the PRD in (provided by the user in chat; defaults to `staging` if not specified).

3. Fetched refs (do **NOT** `git reset --hard` — the working tree often carries uncommitted in-flight work; analyze the checked-out branch as-is):
   ```bash
   git fetch origin
   ```

4. Relevant code contents. Identify the related domain first, then read key files:
   ```bash
   cat docs/specs/capability-index/master-index-BRANCH.md
   # Then, for the relevant route/module, e.g.:
   find web/quiz src/modules/archetype -type f \( -name "*.ts" -o -name "*.html" \) | head -30
   ```

5. Optional: existing PRDs for related features in `docs/product/prds/`.

## Workflow

1. Parse the feature description and identify the primary domain (route folder in `web/`, module family in `src/modules/`).
2. Read the relevant section of `docs/specs/capability-index/master-index-BRANCH.md`.
3. Read the relevant source files to understand current behavior, data models, UI patterns, and storage/telemetry contracts.
4. Write a "Current State" section describing what already exists in code.
5. Draft requirements tagged as:
   - **Must Have** — required for MVP
   - **Should Have** — important but not launch-blocking
   - **Nice to Have** — desirable if time allows
6. Flag open questions for engineer validation.
7. Mark any speculative requirement with `[NEEDS VALIDATION]`.
8. Identify dependencies, risks, and estimated complexity if determinable from code.
9. **Respect the project guardrails in `.local/AGENTS.md`** for anything user-facing: calm/guilt-free Indonesian brand voice (no shaming, no "disiplin"), value-before-data, mobile-first (360–430px), and Playwright E2E coverage for any flow/selector/copy change.
10. Explicitly state anything that cannot be determined from the provided inputs.

## Output

Write to:

```text
docs/product/prds/[feature-name].md
```

Use kebab-case for `[feature-name]` derived from the feature description.

> ⚠️ `docs/` is a **git submodule** (`stnkcl-tech/selangkah-docs`). The file physically lands in the docs repo — commit it to the submodule's `main`, push, then commit the updated submodule pointer in the moneywise repo (see `.local/AGENTS.md` → Documentation & Specs Management).

Use this structure:

```markdown
# PRD: [Feature Name] — YYYY-MM-DD

## Feature Description
[1-2 sentence description from PM]

## Current State
- What already exists in code:
- Relevant domains:
- Relevant files:
- Confidence in current capability: High/Medium/Low

## Requirements

### Must Have
1. ...
2. ...

### Should Have
1. ...
2. ...

### Nice to Have
1. ...
2. ...

## Open Questions
- [ ] [Question] — [NEEDS VALIDATION] if speculative

## Dependencies & Risks
- ...

## Acceptance Criteria
- [ ] ...
- [ ] ...

## Out of Scope
- ...
```

## Confidence Levels

| Level | Definition | PM Action |
|-------|-----------|-----------|
| **High** | Clear code + tests + active usage | Report to client as "shipped and working" |
| **Medium** | Code exists, unclear if released / fully tested | Flag for engineer validation before client communication |
| **Low** | Comments, stubs, inferred intent, or TODOs | Treat as "not built yet" in planning |

## Output Standards

- Plain English first; jargon only with parenthetical explanation.
- Every claim cites a specific file path.
- Include generation date in the header.
- Sections must copy-paste cleanly into Linear, Notion, or GSheet.
- Never omit "I cannot determine..." when applicable.

## Success Criteria

- [ ] "Current State" section present and accurate
- [ ] Requirements tagged as Must Have / Should Have / Nice to Have
- [ ] Open questions flagged for engineer validation
- [ ] No speculative requirement without [NEEDS VALIDATION] marker
- [ ] Project guardrails (voice, mobile-first, E2E) respected for user-facing changes
- [ ] Output saved to `docs/product/prds/[feature-name].md` (submodule commit workflow followed)

## Related Skills

- Run **code-indexer** first if no capability index exists.
- Use **pilot-pm-create-prd** for a more comprehensive 8-section PRD when code reality is less important than product strategy.
- Use **pilot-pm-user-stories** or **pilot-pm-job-stories** to generate backlog items from this PRD.
- Use **pilot-pm-test-scenarios** to derive acceptance tests.
