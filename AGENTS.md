# AGENTS.md — AI Tools Workspace

## Project Overview

This is a local personal workspace with a comprehensive skill library for product and engineering work. Every new project goes through product discovery before any solution is built. Discovery may or may not include proper lengthy research. Quick experiments without research is still considered discovery. Approach is based on available skills in the root `.agents/skills/` folder.

The workspace contains three main asset groups:

1. **`.agents/skills/`** — A flat collection of skills organized by discipline and lifecycle stage:
   - **59 `pilot-pm-*` skills** — Product management for discovery, validation, and building toward PMF (0–100K users)
   - **9 `growth-pm-*` skills** — Product management for scaling, optimization, and expansion (100K+ users)
   - **21 `eng-*` skills** — Engineering practices from coding discipline to deployment, monitoring, and performance
   - **2 `prd-*` skills** — Product design, copy, and other pre-engineering product outputs
   - **1 `core-*` skill** — Workspace-wide documentation standards
   - **1 `full-stack-builder` flow** — Orchestrated end-to-end feature development from PRD to shipped code
   - **3 additional product/orchestration skills** — `new-project`, `product-discovery`, `prototype-builder`

2. **`phuryn/pm-skills`** — The open-source upstream source for the PM skills (https://github.com/phuryn/pm-skills). It is a Claude Code plugin marketplace that packages the PM skills into installable plugins plus slash-commands that chain skills into end-to-end workflows. If you maintain the upstream reference for syncing, clone it to a location of your choice outside this workspace.

3. **`projects/`** — Active workspace for experiments and shipped products.

---

# Agent Behavior & Thinking Guidelines

## Default Posture

These rules apply to all agents operating anywhere in this workspace unless overridden by a project-level `AGENTS.md`.

1. **Ask first, assume never**. Ask clarifying questions before making assumptions about requirements, constraints, or user intent.
2. **Simplicity over cleverness**. Prefer simple, minimal solutions. Avoid over-engineering.
3. **Minimal changes**. Make the smallest change necessary to achieve the goal. Do not refactor unrelated code.
4. **Token efficiency**. Reference shared resources (skills, configs) from the root rather than duplicating them.
5. **When stuck, ask**. If you are uncertain about the correct approach, ask the user rather than hallucinating a solution.
6. **Verify non-trivial plans**. Before implementing complex changes, summarize your plan and get user confirmation.
7. **Preserve existing logic**. When refactoring, do not change behavior unless explicitly requested.
8. **Follow existing style**. Match the coding style, naming conventions, and formatting of the codebase you are editing.
9. **Document your work**. Follow the **core-documentation** skill for all written communication — code comments, READMEs, PR descriptions, and project docs.
10. **Gatekeep implementation**. Do not start writing code, building, shipping, or implementing any changes (including project scaffolding) until the user explicitly says "approve", "OK", "go build this," "start building," "implement this," or an equivalent explicit go-ahead. Planning, discovery, design, analysis, documentation, and research are welcome without this signal; implementation, file writing, any kind of executions are not.

## Multimodal Input Discipline

- When the user attaches an image, video frame, screenshot, mock, or clip, read the file/frame in the current session and base decisions on it. Do not paraphrase a guessed description.
- Use screenshots/frames as ground truth for visual claims; cite the file path in the report.
- For design parity work, attach the reference image and reference the path; do not invent colors, spacing, or typography.
- After a UI change, re-read the resulting state (post-change frame) before claiming it is correct. Do not rely on memory of the pre-change state.

## Solver Loop

For non-trivial work:
1. Define the outcome in operational terms.
2. Inspect the repo and current environment before choosing an approach.
3. Find the spine: entry points, data flow, state boundaries, persistence, and user-visible behavior.
4. Build the smallest vertical slice that proves the solution works.
5. Verify at the surface where the user experiences the change.
6. Expand scope only after the core slice is working.

## Scope Control

- Do exactly the slice the user asked for.
- Do not turn planning into implementation or explanation into edits.
- Do not broaden scope with opportunistic cleanup, refactors, or polish unless needed for the requested outcome.
- If scope changes during the work, say what changed and why before continuing beyond the original slice.
- If unrelated or unexpected edits appear, stop and ask before proceeding.

## Stuck Loop And Retry Policy

- After two failed verification attempts on the same hypothesis, stop repeating the same fix.
- Document evidence from those attempts, then switch strategy: a smaller patch, reading a wider area of the codebase, or one concrete forked question to the user.
- Do not loop on identical reasoning without changing inputs (new reads, new command, or narrower scope).
- Compress raw evidence from the failing attempt before starting the next iteration.

## Mid Task Checkpointing

- On long or multi-step work, checkpoint before expanding scope: restate the goal, list files touched, checks already run, and what remains.
- Prefer re-reading authoritative files over relying on conversation memory for exact APIs, signatures, or line-level detail.

## Context Management

- Decide retention vs. compression per slice before loading it. Pick: keep verbatim / keep summary / drop.
- Compress after each iteration. Replace raw search/fetch output with a 2–4 line summary; never accumulate more than a few raw blocks of any single source.
- Prefer targeted `Grep` / `Read` over full re-ingest when a slice answer suffices.
- Offload deep recipes to skills instead of inlining them into the always-on prompt.
- For very large work, plan a 4–6 line loader plan first: in-context at start, what to add verbatim, what to summarize, what to drop, when to compress.

## Tool Discipline

- Do not invent tool names, wrappers, or APIs that are not present in the current environment.
- Do not promise browser, canvas, subagent, MCP, or other tool-based output until the tool path is confirmed in the current runtime.
- Prefer direct tools over shell when the environment exposes a dedicated tool for the action.
- Parallelize independent reads, greps, and searches; serialize when the next step depends on the result of a read or edit.

## Freshness And Honesty

- When facts may be stale or fast-moving, check current docs or web sources before speaking with confidence.
- If you did not verify a claim, say that directly instead of implying certainty.
- Do not use fake `<think>` blocks, inflated self-descriptions, or confident filler in place of grounded evidence.
- When uncertain, name the cheapest check that would resolve it (one command, one file read, or one doc lookup) and run it when tools allow.
- For visual claims, ground in the actual attached image/frame, not in a memory or guessed description; if the user did not attach one and the claim needs it, say so.

## Communication

- Lead with actions, findings, and results.
- Keep progress updates short and high signal.
- Prefer milestone updates over step-by-step narration.
- Report new information, blockers, scope changes, and verification results.
- When blocked, state the blocker, evidence, and smallest next step; if two attempts on the same hypothesis failed, switch strategy per the stuck-loop policy instead of retrying blindly.

## Local Development Server Persistence

When the user asks to **build and run locally**, **run the web locally**, **serve locally**, or any equivalent phrase, start the local development server in a way that survives the agent's tool timeout and remains available for the user to view in their browser.

- **Do not** rely solely on `Shell(run_in_background=true)` for long-running servers — background tasks may be killed after a timeout.
- **Do** detach the server process using `nohup` (or equivalent) so it is re-parented to PID 1 and keeps running after the shell tool returns.
- Preferred pattern:
  ```bash
  cd <project-dir>
  nohup <server-command> > /tmp/<project>-server.log 2>&1 &
  echo $! > /tmp/<project>-server.pid
  ```
- Verify the server is reachable with `curl` before reporting it ready.
- Report the URL, PID, and log file path to the user.

## Handover Protocol

When the user says "summarize what we've been doing", "save for now", or "continue later", the agent must:
1. **Ask which scope the handover covers** — in this multi-repo workspace, a session can touch a project, the workspace root, or both. Ask: project-related, root-related, or both?
2. Ask the user what the next item to discuss is.
3. Generate or update the relevant `where-we-left-off.md` handover file(s) — one at the **project's root** for project scope, and one at the **workspace root** for root scope. Each file summarizes:
   - What has been accomplished in the current session.
   - The current status of the active tasks.
   - The next items to be groomed or implemented.

**Scope rules:**
- **Project handover** — kept in the project repo, is committed/pushed with the project, and contains project-specific details. It is read first when starting work on that project.
- **Root handover** — kept at the workspace root, is **gitignored and local-only** so the workspace root stays fork-clean and project-agnostic. It only covers workspace-level changes, not project specifics.

The `where-we-left-off.md` file will be rewritten on every summary/save/continue request.

---

# Directory Structure

```
.
├── .agents/
│   └── skills/                          # 96 flat skill directories (agent-agnostic)
│       ├── pilot-pm-create-prd/SKILL.md
│       ├── pilot-pm-user-stories/SKILL.md
│       ├── eng-coding-discipline/SKILL.md
│       ├── eng-api-design/SKILL.md
│       ├── full-stack-builder/SKILL.md
│       └── ... (83 more skills)
├── projects/                            # Project workspace
│   ├── active/                          # Graduated, ongoing projects
│   ├── archive/                         # Stopped or killed experiments
│   ├── bugs/                            # Workspace-level bug records when needed
│   ├── experiments/                     # New, unvalidated ideas
│   └── templates/                       # Scaffolding templates
├── scripts/                             # Workspace utilities
│   └── generate-image.py               # Pilot image generation for UI design
└── README.md                            # Workspace overview

The PM skills in `.agents/skills/` are based on the open-source `phuryn/pm-skills` repository (https://github.com/phuryn/pm-skills). If you maintain the upstream reference for syncing, clone it to a location of your choice outside this workspace.
```

---

# Workspace Conventions & Project Lifecycle

## Directory Usage

| Directory | Purpose |
|-----------|---------|
| `projects/experiments/` | New, unvalidated ideas. Every new project starts here. |
| `projects/active/` | Projects that have graduated from experiments and are under active development. |
| `projects/templates/` | Scaffolding templates for bootstrapping new projects. |
| `projects/archive/` | Stopped or killed experiments. Moved here when an idea is no longer pursued. |
| `projects/bugs/` | Workspace-level bug records. Used when a central bug log is needed across projects. |

## Project Phase Selection (Required at Initiation)

**When initiating any new project, the agent MUST ask the user:**

> "Are we in **discovery** mode (finding PMF, validating ideas, building MVP for 0–100K users), **delivery** mode (problems validated, moving straight into build/execution), or **growth** mode (scaling, optimizing, expanding beyond 100K users)?"

This determines which PM skill set to invoke for all product work:

| Phase | Skill Prefix | Typical Use Cases |
|-------|-------------|-------------------|
| **Discovery / Pilot** | `pilot-pm-*` | Customer interviews, ideation, assumption validation, PRDs, user stories, GTM strategy, pricing, MVP scoping |
| **Delivery / Build** | `eng-*` + selected `pilot-pm-*` + `prd-*` | Requirements docs, user/job stories, product design, copy, implementation, testing, deployment — when problems are already validated and the goal is to ship |
| **Growth / Scale** | `growth-pm-*` | A/B test analysis, cohort retention, competitive battlecards, market expansion, PESTLE / Porter's analysis |

`prd-*` skills are pre-engineering product skills. Use them after discovery requirements are set and before engineering implementation begins.

After the user selects a phase, the agent MUST confirm the project has clear answers to:

> 1. What problems will be solved?
> 2. Who experiences these problems?
> 3. What are currently available solutions that are used?

### Recording the Response

Record both the phase and the answers in the project's `AGENTS.md`:

```markdown
## Project Phase
- **Current phase**: Discovery (pilot-pm-*) | Delivery (prd-* / eng-* / pilot-pm-execution) | Growth (growth-pm-*)
- **Selected on**: <date>
- **Rationale**: <user's reason>

## Project Key Questions
1. **What problems will be solved?**
   <answer>
2. **Who experiences these problems?**
   <answer>
3. **What are currently available solutions that are used?**
   <answer>
```

The agent must reference this phase in all subsequent PM work. If the user later switches phases (e.g., from discovery to growth after finding PMF), update the `AGENTS.md` and adjust skill references accordingly.

## Project Creation (Manual)

To start a new project:

```bash
# 1. Ask project name
# 2. Ask Discovery, Delivery, or Growth
# 3. Ask the 3 key questions
# 4. Only then scaffold:
cp -r projects/templates/default projects/experiments/<project-name>
cd projects/experiments/<project-name>
git init
# Edit AGENTS.md, README.md, and .gitignore to match your project
git add AGENTS.md README.md .gitignore
git commit -m "init: scaffold project with phase and key questions"
```

The agent **must** perform the phase selection step and collect all 3 key questions **before** scaffolding the project. The project directory should only be created once `AGENTS.md` can be fully filled out.

## Project Structure Template

Each project should follow this layout:

```
projects/experiments/<project-name>/
├── AGENTS.md              # Project-specific agent instructions (inherits root)
├── README.md              # Public-facing project description
├── .gitignore             # Git ignore rules
├── where-we-left-off.md   # Handover file (see Handover Protocol)
├── research/              # Research notes, drafts, data (gitignored by default)
└── src/                   # Software source code (committed)
```

## Git Policy

- **One project = one repository**. The workspace root is a git repository that tracks shared structure, skills, and templates.
- **Research is private by default**. The `research/` directory is gitignored. Only commit it if the user explicitly requests it.
- **Software is public by default**. Code in `src/` and documentation in `README.md` are intended for public GitHub repos unless the user says otherwise.

## Skill Usage Policy

- Skills live in `.agents/skills/` at the workspace root and are the single source of truth.
- **Default behavior**: Reference skills from the root workspace using relative paths (`../../../.agents/skills/`).
- **Agent instruction**: At the start of every new project, ask the user: "Should I copy specific skills into this project folder, or reference them from the root workspace?"
- Never silently duplicate skills. Token efficiency is prioritized.

### Skill Reference by Phase

When the project is in **Discovery** phase, prefer `pilot-pm-*` skills:
- `pilot-pm-interview-script` for customer discovery
- `pilot-pm-identify-assumptions-new` for new product risk assessment
- `pilot-pm-create-prd` for requirements documents
- `pilot-pm-user-stories` or `pilot-pm-job-stories` for backlog items
- `pilot-pm-beachhead-segment` for first market selection
- `pilot-pm-lean-canvas` or `pilot-pm-startup-canvas` for business model
- `pilot-pm-gtm-strategy` for launch planning

When the project is in **Delivery** phase, problems are already validated and the focus is execution. Prefer:
- `pilot-pm-create-prd` to lock final requirements
- `pilot-pm-user-stories` or `pilot-pm-job-stories` for backlog items
- `pilot-pm-test-scenarios` for acceptance criteria and QA cases
- `prd-product-design` for UI/UX design and design systems before implementation
- `prd-copywriter` for user-facing product copy before implementation
- `eng-coding-discipline` for all implementation work
- `eng-front-end-design` for UI/UX build
- `eng-deployment` for shipping to production
- `full-stack-builder` for end-to-end feature development

When the project is in **Growth** phase, prefer `growth-pm-*` skills:
- `growth-pm-ab-test-analysis` for experiment evaluation
- `growth-pm-cohort-analysis` for retention analysis
- `growth-pm-competitive-battlecard` for sales enablement
- `growth-pm-pestle-analysis` or `growth-pm-porters-five-forces` for strategic expansion

For **engineering work**, invoke `eng-*` skills regardless of phase:
- `eng-coding-discipline` for all code changes
- `eng-api-design` for API contracts
- `eng-auth-implementation` for authentication
- `eng-testing-debugging` for test strategy
- `eng-deployment` for shipping to production
- `full-stack-builder` for end-to-end feature development

### Kimi CLI Skill Discovery

Kimi CLI discovers skills relative to the **project root** (nearest `.git` ancestor). Since each project is its own git repo, `.kimi/skills/` at the workspace root is **not** automatically discovered when working inside a project.

| Skill Location | Scope | Discovered in projects? |
|----------------|-------|------------------------|
| `~/.kimi/skills/` | User-level | ✅ Yes, everywhere |
| `.agents/skills/` (workspace root) | Workspace | ❌ No — only at root level |
| `projects/experiments/*/.kimi/skills/` | Project-level | ✅ Yes, within that project only |

**Current setup:**
- All `eng-*`, `core-*`, and `full-stack-builder` skills live in `~/.kimi/skills/` (global, available in all projects)
- PM skills (`pilot-pm-*` and `growth-pm-*`) and pre-engineering product skills (`prd-*`) live in `.agents/skills/` at workspace root (reference manually or copy into projects as needed)

**To use PM skills in a project:**
1. Copy or symlink specific skills into the project's `.kimi/skills/` folder
2. Or reference them manually by path from the workspace root
3. Or add the workspace path to `extra_skill_dirs` in `~/.kimi/config.toml` (⚠️ adds ~65 skill descriptions to every conversation's system prompt)

## Project Graduation

When a project moves from `experiments/` to `active/`:
1. Move the directory: `mv projects/experiments/<name> projects/active/<name>`
2. Update the project's `AGENTS.md` to reflect its new status.
3. If the project has found PMF and is entering growth mode, update the **Project Phase** from Discovery or Delivery to Growth.
4. Ensure the project's git remote is configured for the target repository.

## Project Archiving

When a project is stopped or killed, move it from `experiments/` or `active/` to `archive/`:

```bash
mv projects/experiments/<name> projects/archive/<name>
```

Then update the project's `README.md` and `AGENTS.md`:

1. **`README.md`**: Fill it out as usual (Overview, Problem, Who it's for, Getting Started, Project Structure). Then add a **Product Learning** section directly under **Project Structure** containing:
   - **Idea summary**: What the project was, in simple terms, what problem it solved, and who it was for.
   - **Research/validation summary**: What was tested and how.
   - **Validation outcome**: The decision (kill/pivot/keep) and a concise summary of the key learnings.
   - **Recommendations**: Alternative directions or next steps.

2. **`AGENTS.md`**: Update the status to "Archived" and note the date and decision.

3. Commit the final state in the project's own repository.

The detailed research and raw notes should stay in the project's `research/` folder (gitignored by default).

---

# Skills Maintenance Workflow

This workspace utilizes two representations of the same skills. The publicly available `phuryn/pm-skills` as the upstream source, and `.agents/skills/` as the vendor-agnostic downstream copy.

## When to Sync

- After pulling updates from the upstream `phuryn/pm-skills` repository
- When adding a new skill to the Claude plugin structure
- When modifying a skill and both representations must stay consistent

## Sync Process (Manual)

1. **Identify changes** — Compare `<path-to-your-pm-skills-clone>/*/skills/<skill-name>/SKILL.md` against `.agents/skills/<skill-name>/SKILL.md`
2. **Copy and adapt** — Copy the updated Claude skill into `.agents/skills/<skill-name>/SKILL.md`
3. **Apply vendor-agnostic substitution** — Replace `$ARGUMENTS` with `the user's request` (or equivalent phrasing suitable for the target model)
4. **Manual review** — Check frontmatter consistency, description quality, and word count
5. **No automated validation** — `.agents/skills/` has no validator script. Quality checks are manual.

---

# Code Style Guidelines

## Skills (`SKILL.md`)

- **Naming**: Directory name must exactly match the skill `name` in frontmatter.
- **Prefixes**:
  - `pilot-pm-*` — Product management for discovery / 0–100K users
  - `growth-pm-*` — Product management for growth / 100K+ users
  - `eng-*` — Engineering practices and implementation
  - `prd-*` — Product design, copy, and other pre-engineering product outputs
  - `core-*` — Workspace-wide cross-cutting concerns
- **Frontmatter**: Every `SKILL.md` must start with:
  ```yaml
  ---
  name: <skill-directory-name>
  description: "<trigger phrase>. Use when ..."
  ---
  ```
- **Description quality**: Should include explicit trigger phrases (e.g., "Use when writing a PRD..."). Descriptions under 30 characters should be flagged during manual review.
- **Content length**: Aim for 50–3000 words. Skills shorter than 50 words or longer than 3000 words should be flagged during manual review.
- **Language**: Skills are nouns representing domain knowledge (e.g., `pilot-pm-create-prd`, `eng-api-design`).

## Commands (`commands/*.md`)

- **Naming**: Files are verbs representing workflows (e.g., `discover.md`, `write-prd.md`).
- **Frontmatter**: Every command must start with:
  ```yaml
  ---
  description: <what the command does>
  argument-hint: "<expected user input>"
  ---
  ```
- **Cross-references**: Commands may reference skills with bold formatting: `**skill-name** skill`. These references must point to skills inside the same plugin only.

## Agent-Agnostic Adaptation (`.agents/skills/`)

The flat skills in `.agents/skills/` are derived from the Claude plugin skills but with one systematic substitution:
- Claude plugin skills use `$ARGUMENTS` for command argument substitution.
- `.agents/skills/` versions replace `$ARGUMENTS` with `the user's request` (or similar phrasing) so they work in conversational agent contexts that do not support the `$ARGUMENTS` macro.

---

# Notes for AI Agents

- If you are asked to edit a skill, check whether the same skill exists in both `.agents/skills/` and your local clone of `phuryn/pm-skills` (`<path-to-your-pm-skills-clone>/*/skills/`). Apply the same conceptual change to both if appropriate.
- The workspace root is a git repository that tracks shared structure, skills, and templates; each project also has its own dedicated git repository.
- There is no package manager or dependency lock file. Do not attempt to run `npm`, `pip`, `cargo`, etc. at the root.
- When a user asks for PM help, check the project's `AGENTS.md` for the **Project Phase** before choosing between `pilot-pm-*`, `prd-*`, `eng-*`, and `growth-pm-*` skills.

---

# Key Conventions

- **Skills = nouns, Commands = verbs**. Keep this distinction consistent.
- **No cross-plugin references** in commands. Suggest follow-ups in natural language only.
- **One change per PR** when contributing upstream to `phuryn/pm-skills`.
- **MIT License** applies to the `pm-skills` content.

---

# Security Considerations

- This repository contains only documentation, prompts, and structured markdown. There are no secrets, credentials, or executable build artifacts.
- The validation script (`validate_plugins.py` inside your local clone of `phuryn/pm-skills`) only reads files and prints to stdout. It is specific to the Claude plugin structure and not used for `.agents/skills/`.
- Do not commit sensitive data into skill files or manifests.
- Before destructive or high-impact actions (`rm -rf`, dropping databases, production deploys, irreversible data migration, or changing secrets and credentials): obtain explicit user confirmation when the environment allows; do not proceed on assumption.
- Never echo, log, or commit secrets, API keys, tokens, or passwords in chat or code unless the user explicitly requests a redacted pattern.

---

# Deployment / Distribution

- **Claude Code / Cowork**: The `pm-skills` folder is designed to be installed as a marketplace from GitHub (`phuryn/pm-skills`).
- **Other assistants**: Skills from `.agents/skills/` (or the flat `skills/` inside each plugin) can be copied into tool-specific directories:
  - Gemini CLI → `.gemini/skills/`
  - OpenCode → `.opencode/skills/`
  - Cursor → `.cursor/skills/`
  - Codex CLI → `.codex/skills/`
  - Kiro → `.kiro/skills/`
  - Kimi → `.kimi/skills/`
