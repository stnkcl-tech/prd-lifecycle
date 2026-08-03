# AI Tools Workspace

A personal workspace for turning ideas into products — or deciding not to.

## What this is

This folder is a structured home for product experiments. It helps you:
- Start new ideas in a consistent way
- Follow a discovery process before building
- Reuse shared guides and skills across projects
- Keep finished, active, and archived projects organized
- Move projects to their own repositories when they grow

## Who this is for

You, if you want to:
- Test product ideas without over-engineering
- Document why an idea was killed or pivoted
- Keep a library of reusable product and engineering guides

## Folder guide

```
.
├── .agents/skills/     ← Reusable guides for product and engineering work
├── projects/           ← Where the work happens
│   ├── active/         ← Projects being actively built
│   ├── archive/        ← Stopped or killed experiments
│   ├── bugs/           ← Workspace-level bug records when needed
│   ├── experiments/    ← New, unvalidated ideas
│   └── templates/      ← Starter template for new projects
├── scripts/            ← Small helper scripts
└── README.md           ← Workspace overview
```

The PM skills in `.agents/skills/` are based on the open-source `phuryn/pm-skills` repository (https://github.com/phuryn/pm-skills). You don't need to clone `phuryn/pm-skills` to use this workspace — the PM skills are already included in `.agents/skills/`. Clone the upstream repo only if you plan to sync future updates manually.

## How projects move through the workspace

```
experiments/ → active/ → (its own git repo) → archive/
```

1. **Every idea starts in `projects/experiments/`**
   - Use the default template.
   - Answer three questions first:
     - What problem will this solve?
     - Who has this problem?
     - What solutions do they use today?
   - Choose a phase: **Discovery** (finding product-market fit), **Delivery** (problems validated, moving into build/execution), or **Growth** (scaling something that works).

2. **Validate before building**
   - Run small experiments.
   - If the idea fails, document what you learned and move it to `archive/`.
   - If it shows promise, move it to `active/`.

3. **Active projects can become standalone repos**
   - Once a project needs its own home on GitHub, give it its own repository.
   - It can still live inside `projects/active/` for day-to-day work.

4. **Archive what you stop**
   - Move killed or paused experiments to `projects/archive/`.
   - They keep their own git history, so nothing is lost.

## Shared skills

The `.agents/skills/` folder contains reusable guides for common tasks:

- **Product skills** (`pilot-pm-*`): For early-stage discovery, interviews, PRDs, user stories, and validation.
- **Growth skills** (`growth-pm-*`): For scaling, analytics, and competitive analysis.
- **Engineering skills** (`eng-*`): For coding, testing, deployment, and design.
- **Flow skills** (`new-project`, `product-discovery`, `prototype-builder`, `full-stack-builder`): Step-by-step workflows.

These skills are reference material. Agents use them to stay consistent across projects.

## Quick start

1. **Start a new idea**
   ```bash
   cp -r projects/templates/default projects/experiments/<your-idea-name>
   cd projects/experiments/<your-idea-name>
   git init
   ```

2. **Fill out `AGENTS.md`**
   - Pick Discovery, Delivery, or Growth.
   - Answer the three key questions.

3. **Decide what to do next**
   - Run product discovery.
   - Build a quick prototype.
   - Or just let the idea sit until you have more clarity.

## Git setup
- The workspace root is one git repository. It tracks the shared structure, skills, and templates.
- Each project is its own git repository. This keeps projects portable.
- The root workspace ignores individual project contents so they don't get mixed together.

## Handing off between sessions

Each project keeps a `where-we-left-off.md` at its root so the next session can pick up where the last one ended. Workspace-level handover lives at the root too, but it is gitignored and local-only so the shared repo stays fork-clean and project-agnostic.

- Say **"summarize what we've been doing"**, **"save for now"**, or **"continue later"** and the agent will rewrite the file(s) with: what was accomplished, current task status, and what's next.
- Starting a session? Ask the agent to read the relevant `where-we-left-off.md` first instead of re-explaining context.

## Workspace rules
- **Ask first, assume never.** Don't guess requirements.
- **Start small.** Build the smallest testable version.
- **Document learnings.** Especially when something fails.
- **One project = one repository.**
- **Research is private by default.** The `research/` folder in each project is gitignored.

## License
Workspace structure and shared skills are intended to be lightweight and reusable. Individual projects may have their own licenses.
