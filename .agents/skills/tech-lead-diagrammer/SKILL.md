---
name: tech-lead-diagrammer
description: "Analyze the Selangkah moneywise repo like a tech lead and produce architecture, data-model, or user-journey diagrams. Use for onboarding, architecture reviews, or explaining complex flows in Mermaid or ASCII."
---

# Tech Lead Diagrammer

## Purpose

Produce a single, focused diagram based on the user's request by reading only the files needed to answer it. Act like an experienced tech lead who can explain a system visually without drowning the reader in detail.

## When to use

- Onboarding a new engineer or PM
- Before architecture reviews
- When a specific flow or relationship is hard to explain in text
- After major refactors to re-document a domain

## Inputs

The user provides in chat:

1. **Branch name** to analyze (defaults to `staging` if not specified).
2. **Diagram request** — what to diagram. Examples:
   - "System architecture"
   - "Data model of quiz profiles and storage"
   - "User journey: homepage → quiz → reflection → report"
   - "Flow: health check scoring and gating"
   - "Sequence: claim → WhatsApp handoff → Apps Script capture"
   - "Component structure of the report page"
3. **Output format** — `mermaid` (default) or `ascii`.

Before invoking, gather:

1. Fetched refs (do **NOT** `git reset --hard` — analyze the checked-out branch as-is):
   ```bash
   git fetch origin
   ```

2. **Existing capability index** (if available):
   ```bash
   cat docs/specs/capability-index/master-index-BRANCH.md
   ```
   If this file exists, use it as the primary source. Do not re-analyze the full directory skeleton unless the index is missing or does not cover the requested topic.

3. **Additional files based on the request:**
   - For system architecture: `README.md`, `package.json`, `vite.config.ts`, `wrangler.json`
   - For data models: `src/modules/*/types.ts` and `src/modules/*/content.ts`
   - For user journeys / page flows: the relevant `web/<route>/index.html` + its module script (e.g., `web/quiz/main.ts`, `web/quiz/laporan/laporan.ts`, `web/health/health.ts`)
   - For capture/telemetry flows: the POST logic in the page module + any Apps Script draft under `research/` or `docs/specs/`
   - If unsure which files are relevant, read the directory skeleton:
     ```bash
     find web src -maxdepth 3 -type d | sort
     ```

## Workflow

1. Parse the user's diagram request.
2. Check for `docs/specs/capability-index/master-index-BRANCH.md`.
   - If it exists, read it and use its domain breakdown to scope the diagram.
   - If it does not exist, gather the directory skeleton and key files to build context.
3. Read only the additional files needed for the specific diagram.
4. Produce the diagram in the requested format:
   - **Mermaid:** Use valid Mermaid syntax (`graph TD`, `erDiagram`, `sequenceDiagram`, `flowchart LR`, etc.).
   - **ASCII:** Use boxes, arrows, and indentation for clarity.
5. Accompany the diagram with a short plain-English explanation.
6. Cite the file paths used to build the diagram.
7. Explicitly state anything that cannot be determined from the code.

## Output

Write to:

```text
docs/specs/capability-index/by-domain/diagram-<slug>-BRANCH.md
```

- `<slug>` is a kebab-case summary of the request (e.g., `system-architecture`, `quiz-user-journey`, `data-model-profiles`).
- `<BRANCH>` is the branch name.

> ⚠️ `docs/` is a **git submodule** (`stnkcl-tech/selangkah-docs`). The file physically lands in the docs repo — commit it to the submodule's `main`, push, then commit the updated submodule pointer in the moneywise repo (see `.local/AGENTS.md` → Documentation & Specs Management).

### File structure

````markdown
# Diagram: [Request Title] — BRANCH — YYYY-MM-DD

## Request
[Repeat the user's request]

## Format
Mermaid / ASCII

## Diagram

```mermaid
...
```

## Explanation
[Plain-English walkthrough of what the diagram shows]

## File References
- `web/quiz/main.ts`
- `src/modules/archetype/engine.ts`
- ...

## Gaps / Open Questions
- [Anything that could not be determined]
````

## Output Standards

- Plain English first; jargon only with parenthetical explanation.
- Every claim cites a specific file path.
- Include generation date and branch in the header.
- Mermaid diagrams must use valid syntax.
- ASCII diagrams must be readable in a fixed-width font.
- Never omit "I cannot determine..." when applicable.

## Success Criteria

- [ ] User's specific diagram request addressed
- [ ] Output in requested format (Mermaid or ASCII)
- [ ] File saved to `docs/specs/capability-index/by-domain/diagram-<slug>-BRANCH.md` (submodule commit workflow followed)
- [ ] Relevant file paths cited
- [ ] Plain-English explanation included

## Related Skills

- Run **code-indexer** first if no capability index exists.
- Use **capability-analyzer** to understand which domains have recently changed.
- Use **prd-generator** to turn diagram findings into requirements.
