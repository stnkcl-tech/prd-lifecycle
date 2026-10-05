---
name: report-writer
description: "Convert git commit activity in the Selangkah moneywise repo into a PM-ready weekly report. Use at end of sprint, before client status updates, or when assessing velocity and focus."
---

# Report Writer

## Purpose

Turn the last 7 days of git activity into a client-ready weekly report. Group work by feature, surface risks, and predict next-week focus based on open branches and in-flight items.

## When to use

- End of sprint / weekly ritual
- Before client status updates
- When velocity or focus needs assessment

## Inputs

Before invoking this skill, gather:

1. **Branch name** to report on (provided by the user in chat; defaults to `staging` if not specified).

2. Fetched refs (do **NOT** `git reset --hard` — the working tree often carries uncommitted in-flight work; analyze the checked-out branch as-is):
   ```bash
   git fetch origin
   ```

3. Recent commit activity (last 7 days):
   ```bash
   git log --since="7 days" --pretty=format:"%h %s %b" --name-only
   ```

4. Open branches:
   ```bash
   git branch -r
   ```

5. Uncommitted work in flight (moneywise often carries uncommitted changes — surface them explicitly):
   ```bash
   git status --short
   ```

6. **Notion backlog movement** (OPTIONAL — run only if a Notion backlog data source is configured for this project; moneywise does not have one by default):
   - Include items from all sprints edited within the report window; do not ask the user to assign or filter sprints.
   - Query the backlog data source for items with current status `Ready to Test`, `Ready to Deploy`, or `Live` and `last_edited_time` within the last 7 days.
   - Query for items with `Category = Bug` created or edited in the last 7 days.
   - Query for items with `Status = Blocked` and `last_edited_time` within the last 7 days (older blocked items are intentionally excluded from the report).
   - Sprint assignment is out of scope for this report: include the `Sprint` column for context, but do not prompt the user to assign, reassign, or filter by sprint.
   - Sort backlog tables by **Priority ascending** (`P0 → P1 → P2 → P3`) using:
     ```json
     "sorts": [{"property": "Priority", "direction": "ascending"}]
     ```
   - Example filters:
     ```json
     {
       "filter": {"and": [
         {"property": "Status", "status": {"equals": "Ready to Test"}},
         {"timestamp": "last_edited_time", "last_edited_time": {"on_or_after": "2026-07-04"}}
       ]},
       "sorts": [{"property": "Priority", "direction": "ascending"}]
     }
     ```
   - Do **not** filter by `Sprint` unless the user explicitly asks for a sprint-specific report.
   - If the Notion integration is unavailable or no backlog is configured, explicitly state that backlog movement could not be retrieved / was skipped.

7. Optional: previous weekly report for the same branch:
   ```bash
   ls -t docs/product/weekly-reports/*-BRANCH.md | head -1
   ```

## Workflow

1. Read the last 7 days of git activity.
2. If a Notion backlog is configured: query it for backlog status movement and bugs, sorted by Priority ascending (`P0` first). Otherwise note the section as skipped.
3. Note uncommitted in-flight work from `git status` (what it is and how long it has been uncommitted, if determinable).
4. Group commits by feature or domain, not by author.
5. For each shipped or progressed item, write one client-ready sentence summarizing the outcome.
6. Identify risk signals:
   - Reverts
   - Hotfixes
   - Large refactors
   - Repeated commits on the same file
   - Commits without clear feature context
   - Long-lived uncommitted work
   - Tasks stuck in `Blocked` or bouncing between statuses
7. Compare against the previous weekly report to highlight trends (e.g., velocity up/down, focus shifts).
8. Predict next-week work based on open branches, recent in-flight commits, unresolved items, and backlog state (if available).
9. Build the **Appendix: Files Updated per Shipped/Progressed Item** table by aggregating the `Files touched` and `Commits` collected for each feature.
10. Explicitly state anything that cannot be determined from the provided inputs.

## Output

Write to:

```text
docs/product/weekly-reports/YYYY-MM-DD-BRANCH.md
```

Use the report date and branch name as the filename (e.g., `2026-10-05-staging.md`).

> ⚠️ `docs/` is a **git submodule** (`stnkcl-tech/selangkah-docs`). The file physically lands in the docs repo — commit it to the submodule's `main`, push, then commit the updated submodule pointer in the moneywise repo (see `.local/AGENTS.md` → Documentation & Specs Management).

Use this structure:

```markdown
# Weekly Report — BRANCH — YYYY-MM-DD

## Summary
- Analysis period: [start date] to [end date]
- Total commits: N
- Features progressed: N
- Uncommitted work in flight: [N files — summary / none]
- Risk level: [Low / Medium / High]

## Shipped / Progressed This Week

### [Feature Name]
- **Status:** [Shipped / In Progress / Merged]
- **Summary:** One client-ready sentence.
- **Confidence:** High/Medium/Low

## Notion Backlog Movement This Week
> Only when a Notion backlog is configured; otherwise state it was skipped.
> Movement is inferred from current `Status` + `last_edited_time` within the report window. Notion does not expose a status-change history API, so an item edited this week that currently shows a given status is listed under that status.

### New / Updated Bugs This Week (`Category = Bug`)
*X items created or edited this week.*

| ID | Title | Sprint | Priority | Status | Last Edited |
|---|---|---|---|---|---|
| ... | ... | ... | ... | ... | ... |

### Ready for QA (`Ready to Test`)
*X items currently Ready to Test and edited this week.*

| ID | Title | Sprint | Priority | Last Edited |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |

### Ready to be Shipped (`Ready to Deploy`)
*X items currently Ready to Deploy and edited this week.*

| ID | Title | Sprint | Priority | Last Edited |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |

### Shipped to Production (`Live`)
*X items currently Live and edited this week.*

| ID | Title | Sprint | Priority | Last Edited |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |

### Currently Blocked (`Status = Blocked`)
*Only items with `Status = Blocked` AND `last_edited_time` within the report window. Older blocked items are intentionally excluded.*

| ID | Title | Sprint | Priority | Last Edited |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |

If no blocked items were edited this week, state: "No blocked items were edited this week."

## Risk Flags
- ...

## Trends (vs. Previous Week)
- ...

## Next Week Predictions
- ...

## Open Questions / Blockers
- ...

## Appendix: Files Updated per Shipped/Progressed Item
> Technical reference for engineering / tech lead review.

| No. | Item | Files Updated | Commit IDs |
|---|---|---|---|
| 1 | [Feature Name] | `path/to/file`, `path/to/file` | `abc123`, `def456` |
```

## Confidence Levels

| Level | Definition | PM Action |
|-------|-----------|-----------|
| **High** | Clear code + tests + active usage | Report to client as "shipped and working" |
| **Medium** | Code exists, unclear if released / fully tested | Flag for engineer validation before client communication |
| **Low** | Comments, stubs, inferred intent, or TODOs | Treat as "not built yet" in planning |

## Output Standards

- Plain English first; jargon only with parenthetical explanation.
- Every claim cites a specific commit hash or file path.
- Include generation date in the header.
- Sections must copy-paste cleanly into Linear, Notion, or GSheet.
- Never omit "I cannot determine..." when applicable.

## Success Criteria

- [ ] Commits grouped by feature, not by author
- [ ] Client-ready one-sentence summaries per shipped item
- [ ] Uncommitted work in flight surfaced
- [ ] Notion backlog movement section populated — or explicitly marked skipped when no backlog is configured
- [ ] Status movement split into Ready for QA, Ready to be Shipped, and Shipped to Production (when backlog available)
- [ ] New/updated bugs and blocked items surfaced (when backlog available)
- [ ] Risk flags for reverts, hotfixes, or large refactors
- [ ] Next-week predictions based on open work and backlog state
- [ ] Output saved to `docs/product/weekly-reports/YYYY-MM-DD-BRANCH.md` (submodule commit workflow followed)

## Related Skills

- Use **capability-analyzer** for a capability-oriented view of the same commits.
- Use **tech-lead-diagrammer** to visualize any architecture changes mentioned.
- Use **prd-generator** to turn reported gaps or requests into feature requirements.
