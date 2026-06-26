# claude-custom-command

Two things live here: a reusable Claude Code custom command that brings structure to any development task, and a demo app built entirely with that command to prove the workflow works end to end.

---

## The `/smart-workflow` command

`/smart-workflow` is a plan-first adaptive workflow for Claude Code. Instead of jumping straight into code, it scopes the work, builds a plan for your approval, classifies which tools the plan actually needs, then executes task by task.

**What makes it different from just prompting Claude:**

- It never writes code before you approve a plan
- It only activates plugins the task actually calls for (frontend design, live docs, code simplification) rather than running everything by default
- It auto-triggers a codebase graph visualization when a changeset touches more than 5 source files, so you can explore what changed in Obsidian before closing the work
- It runs `/compact` between tasks to keep context from ballooning on long builds
- It ends every build with a code review pass before declaring done

The command lives at `.claude/commands/smart-workflow.md`. Copy it to any project and it works. See [SETUP.md](SETUP.md) for full installation steps including the required plugins.

---

## `index.html` — Tech Debt Tarot

The tarot app is a single-file web app built to demonstrate the smart-workflow command on a real creative task.

It is a developer-themed tarot reading tool. Click "Consult the Stack" and three cards are drawn from a 21-card arcana, each themed around a software development experience: merge conflicts, hotfixes shipped on Friday afternoons, abandoned dependencies, 100% test coverage with zero real confidence, and nineteen others. Each card lands either upright (the constructive reading) or reversed (the cautionary one), and flips with a glitch animation to reveal the card's wisdom and a fake-but-plausible stack trace.

**Why it is useful beyond being fun:**

- It is a working proof that the workflow handles a full creative-technical build: design direction ran before any markup was written, the frontend-design skill shaped the visual direction (dark mystical aesthetic with gold accents), and the result is a polished single-file deliverable with no external dependencies beyond Google Fonts
- The card content doubles as a low-key reference: the reversed readings call out real antipatterns (pushing directly to main, mocking everything, shipping at 4:57 PM on a Friday) in a format that sticks better than a lint rule
- The file is self-contained. No build step, no npm install. Open it in a browser and it works

To run it, open `index.html` directly in any browser.

---

## Setup

See [SETUP.md](SETUP.md) for step-by-step installation of all required plugins and tools.
