---
name: capability-analyzer
description: "Audit what the Selangkah moneywise repo currently supports with confidence scoring and PM implications. Use before stakeholder meetings, when scoping new features, or after detecting changes in the repo."
---

# Capability Analyzer

## Purpose

Compare recent code changes against the existing capability index to surface what is new, modified, stable, or at risk — and translate those findings into PM implications.

## When to use

- Before client/stakeholder meetings
- When scoping new features ("what exists vs. what we need to build")
- Weekly as part of sprint review prep
- After detecting changes in the repo since the last update

## Inputs

Before invoking this skill, gather:

1. **Branch name** to analyze (provided by the user in chat; defaults to `staging` if not specified).

2. Fetched refs (do **NOT** `git reset --hard` — the working tree often carries uncommitted in-flight work; analyze the checked-out branch as-is, and explicitly note when uncommitted changes are included in the analysis):
   ```bash
   git fetch origin
   git rev-parse HEAD
   git status --short
   ```

3. **Capability baseline** (the commit hash representing the last analyzed state):
   - Prefer the `HEAD` hash recorded in the most recent `docs/specs/capability-index/YYYY-MM-DD-BRANCH-update.md`.
   - If no previous update exists, use the hash recorded in `docs/specs/capability-index/master-index-BRANCH.md`.
   - If neither file records a hash, fall back to `HEAD~7`.

4. Recent git activity since the baseline:
   ```bash
   git log --pretty=format:"%h %s %b" --name-only <BASELINE>..HEAD
   ```

5. Change statistics since the baseline:
   ```bash
   git diff --stat <BASELINE>..HEAD
   ```

6. Current capability index:
   ```bash
   cat docs/specs/capability-index/master-index-BRANCH.md
   ```

7. Optional: previous capability update (`docs/specs/capability-index/YYYY-MM-DD-BRANCH-update.md`) for trend comparison.

## Workflow

1. Read the current `docs/specs/capability-index/master-index-BRANCH.md`.
2. Determine the capability baseline commit hash from the most recent update file (or master index if no update exists).
3. Read the git log and diff statistics between the baseline and current `HEAD`. If `git status` shows uncommitted changes that touch analyzed files, include them as an explicit "Uncommitted changes" note.
4. For each capability in the master index, determine if it was touched, unchanged, newly introduced, or removed since the baseline.
5. Tag every capability with confidence:
   - **High** — clear code + tests + active usage
   - **Medium** — code exists, unclear if released / fully tested
   - **Low** — comments, stubs, inferred intent, or TODOs
6. **Detect major structural changes.** Consider refreshing the master index via `code-indexer` when any of the following are true:
   - A new top-level route/domain is introduced (e.g., a new `web/<route>/` entry point, a new `src/modules/<domain>/` module family).
   - More than **5 capabilities** are newly introduced in a single update.
   - More than **30% of capabilities** in the master index are modified.
   - Large file renames, deletions, or directory restructuring.
   - The diff stat exceeds roughly **100 files** or **5,000 insertions/deletions** since the baseline.
   - The previous master index is older than **3–4 weeks**.
7. If a master-index refresh is triggered:
   - Run the `code-indexer` skill to rewrite `docs/specs/capability-index/master-index-BRANCH.md` from the current `HEAD`.
   - Treat the new master index as the baseline for this update.
   - Include a **Master Index Refreshed** section in the update file explaining why and what changed structurally.
8. Separate findings into:
   - **New** — capabilities not present in the previous index/baseline
   - **Modified** — capabilities with code changes since the baseline
   - **Stable** — capabilities unchanged since the baseline and still present
   - **Deprecated / Removed** — files deleted or logic replaced since the baseline
9. State PM implications explicitly, not just technical descriptions.
10. Flag risks: reverts, hotfixes, large refactors, untested changes, long-lived uncommitted work, or ambiguous TODOs.
11. Record the baseline commit hash and the current `HEAD` hash in the output file so the next run can use them.
12. Explicitly state anything that cannot be determined from the provided inputs.

## Output

Write to:

```text
docs/specs/capability-index/YYYY-MM-DD-BRANCH-update.md
```

Replace `<BRANCH>` with the branch name being analyzed (e.g., `2026-10-05-staging-update.md`).

> ⚠️ `docs/` is a **git submodule** (`stnkcl-tech/selangkah-docs`). The file physically lands in the docs repo — commit it to the submodule's `main`, push, then commit the updated submodule pointer in the moneywise repo (see `.local/AGENTS.md` → Documentation & Specs Management).

Use this structure:

```markdown
# Capability Update — BRANCH — YYYY-MM-DD

## Summary
- Baseline commit: `hash` (from previous update or master index)
- Current HEAD: `hash`
- Uncommitted changes included: [yes — N files / no]
- Analysis period: since baseline (or last 7 days if no baseline recorded)
- Total capabilities: N
- New: N | Modified: N | Stable: N | Deprecated/Removed: N
- Overall risk: [Low / Medium / High]

## Master Index Refreshed (only when code-indexer was triggered)
- Reason: e.g., "New /health/ route introduced; previous master index was 18 days old."
- Structural changes: summary of new/removed/renamed domains
- New master index: `docs/specs/capability-index/master-index-BRANCH.md`

## New Capabilities
### [Capability Name]
- What changed: ...
- PM implication: ...
- Confidence: High/Medium/Low
- Files: ...

## Modified Capabilities
### [Capability Name]
- What changed: ...
- PM implication: ...
- Confidence: High/Medium/Low
- Files: ...

## Stable Capabilities
- [Capability Name] — Confidence: High/Medium/Low

## Deprecated / Removed Capabilities
- [Capability Name] — Reason: ...

## Risk Flags
- ...

## Open Questions for Engineers
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
- Every claim cites a specific file path or commit hash.
- Include generation date in the header.
- Sections must copy-paste cleanly into Linear, Notion, or GSheet.
- Never omit "I cannot determine..." when applicable.

## Success Criteria

- [ ] Baseline commit hash is determined from the previous update or master index
- [ ] Analysis covers changes between the baseline and current `HEAD`
- [ ] Uncommitted changes noted explicitly when present
- [ ] Major structural changes trigger a `code-indexer` master-index refresh
- [ ] Refreshed update files include a **Master Index Refreshed** section
- [ ] Every capability tagged with confidence
- [ ] New, modified, stable, and deprecated sections clearly separated
- [ ] PM implications stated explicitly
- [ ] Risk flags called out
- [ ] Baseline and current HEAD hashes recorded in the output
- [ ] Output saved to `docs/specs/capability-index/YYYY-MM-DD-BRANCH-update.md` (submodule commit workflow followed)

## Related Skills

- Run **code-indexer** first if `docs/specs/capability-index/master-index-BRANCH.md` is missing or stale.
- Use **report-writer** for a commit-oriented weekly summary.
- Use **prd-generator** to turn gaps into feature requirements.
