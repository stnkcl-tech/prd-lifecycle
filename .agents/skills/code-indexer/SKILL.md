---
name: code-indexer
description: "Map the structure and capabilities of the Selangkah moneywise repo into a PM-readable master index. Use when onboarding the repo, refreshing the capability index after structural changes, or when the current master index is stale."
---

# Code Indexer

## Purpose

Build a living, plain-English map of the moneywise repo (Selangkah — a financial mindset and planning platform) so non-technical stakeholders can understand what the codebase supports, where each capability lives, and how confident we are in it.

This skill is the foundation for every other analysis skill in this workspace. Run it before `capability-analyzer`, `prd-generator`, `report-writer`, or `tech-lead-diagrammer` when the index is missing or outdated.

## When to use

- First-time indexing of the moneywise repo
- Weekly refresh when file structure or domains have visibly changed
- After major refactors, renames, or new domain additions
- When another skill cannot find `docs/specs/capability-index/master-index-BRANCH.md`

## Repo Layout Reference

```
.
├── web/            # Static HTML entry points (one folder per route: /, /quiz/, /quiz/laporan/, /health/, ...)
├── src/modules/    # Pure TypeScript business logic (archetype/, health/, storage/)
├── e2e/            # Playwright E2E specs
├── vite.config.ts  # Multi-page app config + trailing-slash middleware
├── package.json    # Scripts; version is the release version (v-badge sweep applies)
└── docs/           # [Git submodule → stnkcl-tech/selangkah-docs] All outputs of this skill land here
```

## Inputs

Before invoking this skill, gather:

1. **Branch name** to index (provided by the user in chat; defaults to `staging` — the active development branch — if not specified).

2. Fetched refs (do **NOT** `git reset --hard` — the working tree often carries uncommitted in-flight work; analyze the checked-out branch as-is):
   ```bash
   git fetch origin
   git rev-parse HEAD
   ```

3. Directory skeleton (full, no cap):
   ```bash
   find . -maxdepth 3 -type d \
     -not -path './node_modules*' -not -path './.git*' -not -path './dist*' \
     -not -path './docs*' -not -path './test-results*' -not -path './research*' \
     -not -path './references*' | sort
   ```

4. Key project metadata and entry points:
   ```bash
   cat README.md package.json vite.config.ts tsconfig.json wrangler.json 2>/dev/null || true
   ```

5. Source files grouped by top-level folder, capped per folder:
   ```bash
   for dir in web src e2e; do
     echo "=== $dir ==="
     find "$dir" -type f \( -name "*.ts" -o -name "*.html" -o -name "*.css" \) | head -40
   done
   ```

6. Optional: existing `docs/specs/capability-index/master-index-BRANCH.md` if this is a refresh.

## Why this input shape

A flat `head -100` file list can silently truncate and hide entire domains. The hybrid approach keeps the full directory skeleton visible while limiting deep file noise. The agent can always request more files for a specific domain if coverage looks thin.

## Workflow

1. Read the directory skeleton to understand top-level structure and domains (`web/` folders are the route/domain boundaries).
2. Read key metadata to identify framework, tech stack, and entry points (Vite MPA, no backend beyond a Google Apps Script capture endpoint).
3. Read sampled source files per folder to infer responsibilities.
4. Identify 5-10 plain-English feature domains (e.g., Archetype Quiz, Personalized Report, Financial Health Score, Landing Pages, Telemetry & Data Capture, E2E Test Suite).
5. For each domain, summarize:
   - What it does (one sentence)
   - Key files and their roles
   - Confidence level: **High**, **Medium**, or **Low**
   - Gaps, TODOs, stubs, or unclear areas
6. Flag cross-cutting concerns (localStorage persistence contract, feature toggles, Apps Script telemetry, shared copy/voice conventions, mobile-first CSS, print styles, version badges).
7. If refreshing, diff against the previous `master-index-BRANCH.md` and highlight structural changes.
8. Record the indexed `HEAD` commit hash in the output header so `capability-analyzer` can use it as a baseline.
9. Explicitly state anything that cannot be determined from the provided inputs.

## Output

Write to:

```text
docs/specs/capability-index/master-index-BRANCH.md
```

Replace `<BRANCH>` with the branch name being indexed (e.g., `master-index-staging.md`).

> ⚠️ `docs/` is a **git submodule** (`stnkcl-tech/selangkah-docs`). The file physically lands in the docs repo — commit it to the submodule's `main`, push, then commit the updated submodule pointer in the moneywise repo (see `.local/AGENTS.md` → Documentation & Specs Management).

Use this structure:

```markdown
# Master Capability Index — BRANCH — YYYY-MM-DD
- Indexed HEAD: `hash`

## Executive Summary
- Repo: moneywise (Selangkah)
- Total domains identified: N
- Overall confidence: [High / Mixed / Low]

## Domain: [Domain Name]

**What it does:** ...

**Key files:**
| File | Role | Confidence |
|------|------|------------|
| ...  | ...  | High       |

**Gaps / TODOs:** ...

---

## Cross-Cutting Concerns
- ...

## Gaps & Open Questions
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
- Include generation date and indexed HEAD hash in the header.
- Sections must copy-paste cleanly into Linear, Notion, or GSheet.
- Never omit "I cannot determine..." when applicable.

## Success Criteria

- [ ] Full directory skeleton reviewed
- [ ] Plain-English feature domains identified
- [ ] File-to-function mapping clear enough for a non-technical PM
- [ ] Gaps and TODOs flagged explicitly
- [ ] Cross-cutting concerns called out
- [ ] Indexed HEAD hash recorded in the header
- [ ] Output saved to `docs/specs/capability-index/master-index-BRANCH.md` (submodule commit workflow followed)

## Related Skills

- Use **capability-analyzer** to compare this index against recent commits.
- Use **prd-generator** to write feature PRDs grounded in this index.
- Use **tech-lead-diagrammer** to visualize domains found here.
- Use **pilot-pm-create-prd** for a comprehensive 8-section PRD when code reality is less important than product strategy.
