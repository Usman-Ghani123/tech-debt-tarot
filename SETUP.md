# Setup guide

This repo includes a `/smart-workflow` custom command for Claude Code. It runs a plan-first development workflow that figures out which tools each task actually needs before writing a single line of code.

To use it, you need four plugins installed. Here's how.

---

## Prerequisites

- Claude Code CLI installed and authenticated (`claude --version` should work)
- Node.js (ships with Claude Code, so you probably already have it)

---

## Step 1: Add the official plugin marketplace

Claude Code ships without the community marketplace configured. Run this once:

```bash
claude plugins marketplace add anthropics/claude-plugins-official
```

---

## Step 2: Install the required plugins

```bash
claude plugins install superpowers@claude-plugins-official
claude plugins install frontend-design@claude-plugins-official
claude plugins install context7@claude-plugins-official
claude plugins install code-simplifier@claude-plugins-official
```

What each one does:

- **superpowers** - the core engine. Handles brainstorming, plan writing, TDD, code review, and subagent coordination.
- **frontend-design** - runs design direction before any HTML or CSS gets written, so you don't end up with generic-looking components.
- **context7** - pulls live docs for third-party libraries into the task brief, so the agent isn't guessing at API shapes.
- **code-simplifier** - runs a cleanup pass after refactor tasks finish.

---

## Step 3: Add the custom command to your project

The command file lives at `.claude/commands/smart-workflow.md` in this repo. Copy it to the same path in your own project:

```bash
# From your project root
cp path/to/this-repo/.claude/commands/smart-workflow.md .claude/commands/smart-workflow.md
```

Or drop it in `~/.claude/commands/` to make it available in every project on your machine.

---

## Step 4 (optional): Install caveman

Caveman keeps Claude's responses terse. It's not required for the workflow to function, but it cuts down on filler in long builds.

```bash
claude plugins marketplace add JuliusBrussee/caveman
claude plugins install caveman@caveman
```

---

## Restart Claude Code

After installing plugins, restart your Claude Code session. Plugins load at session start.

---

## Using the command

```
/smart-workflow <describe what you want to build>
```

The command scopes the work, builds a plan for your approval, classifies which plugins each task needs, then executes phase by phase. It runs `/compact` between tasks to keep context from ballooning on longer builds.
