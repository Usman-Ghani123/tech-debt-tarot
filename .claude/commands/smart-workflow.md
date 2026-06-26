---
description: Plan-first adaptive workflow — superpowers scopes the task, then activates only the skills the plan actually calls for (frontend design, code-graph visualization, simplification, doc lookup)
---

You are running an adaptive workflow for: $ARGUMENTS

## Step 1 — Plan first, always
Use superpowers brainstorming to clarify scope and intent. Ask only what's
necessary to remove real ambiguity. Present a short design doc / plan summary
and wait for approval before writing any code or generating any files.

Once approved, use superpowers writing-plans to break the work into concrete
tasks.

## Step 2 — Classify what the plan actually needs

**HARD GATE: Complete this step and output the checklist below before touching Step 3. Do not dispatch any implementer subagent until classification is done.**

Read through the approved plan and check each item. Output this block before proceeding:

```
Classification:
[ ] Frontend/UI work (HTML/CSS/components)?              → design-taste-frontend
[ ] Third-party library or API?                          → context7 docs
[ ] Visualization needed? (explicit request OR           → visualization protocol
    changeset >5 source code files)
[ ] Refactor or cleanup task?                            → code-simplifier
```

Mark each `[x]` or `[ ]`. Then follow the per-skill rules below for every item marked `[x]`.

### Per-skill invocation rules

**design-taste-frontend — Frontend / UI work**
Invoke this skill BEFORE dispatching the first implementer subagent that
touches HTML, CSS, or components. Not during. Not after. The skill output
must be included in that task's implementer brief. No implementer subagent
writes frontend markup without the skill having run first.

**context7 — Third-party library or API**
During Step 2, identify all unique third-party libraries across the entire
plan. Pull context7 once per library, listing all required endpoints in a
single call. Store excerpts in the plan brief as a doc cache. Step 3
implementer briefs reference this cache — do not re-pull per task. Only
pull again if a task surfaces an endpoint gap not covered in the initial
pull.

**visualization protocol — Codebase graph**
Two triggers, same protocol, different granularity:

- **Explicit request** (user asked for graph/architecture view): run at
  community/module level. Use `get_architecture_overview_tool` as primary
  artifact. Avoid per-symbol hairball dumps.
- **Auto-trigger** (changeset exceeds 5 source code files — language files
  only, not .md/.json/.yaml/config/markup): run at per-symbol granularity
  AFTER all tasks complete, before declaring work finished. Goal is
  exploration and Q&A, not overview.

Both cases: generate the Obsidian vault (per-symbol .md files tagged by
community/module) and a graphical export (HTML/D3 or equivalent). Follow the
full visualization protocol section below. Verify export is non-empty before
reporting success.

**code-simplifier — Refactor or cleanup**
Run code-simplifier on the touched files AFTER the implementer subagent
commits, BEFORE marking that task complete.

**None of the above apply**
Proceed with plain implementation under superpowers' TDD discipline
(test-driven-development skill), no extra skills invoked.

## Step 3 — Implement
Execute the approved plan task by task, using subagent-driven-development
or executing-plans per superpowers' normal workflow.

For each task, check your Step 2 classification before dispatching:
- Does this task touch frontend? → design-taste-frontend must already be done.
- Does this task hit a third-party API? → reference doc cache from Step 2, not a fresh context7 pull.
- Does this task refactor code? → schedule code-simplifier after the commit.
- Was visualization flagged in Step 2? → explicit request: run community/module level now; auto-trigger (>5 source files): run per-symbol after final task, before closing.

Run requesting-code-review before declaring the work finished.

## Context management during long builds
Multi-task builds accumulate context fast — file reads, tool outputs, and
subagent reasoning across many tasks fill the window even though caveman
keeps individual responses terse (caveman compresses output tokens only,
not accumulated context).

After completing each task in the plan, before starting the next one, run
`/compact` to summarize accumulated history and free up context. Do this
proactively — don't wait until context pressure becomes a problem. A short
pause between tasks to compact is worth it on any build with more than
3-4 sequential tasks.

---

## Visualization protocol (only when a graph/architecture visual is requested)

If the task involves visualizing a codebase (Obsidian, HTML/D3, or other
graph export), do not just take the tool's raw default output. Apply these
practices:

1. **Build/refresh the graph first**
   ```
   code-review-graph build   (or update, if a graph already exists)
   ```

2. **Decide the right granularity for the goal, don't default to per-symbol:**
   - If the goal is a **demo, portfolio piece, or architecture overview** →
     generate at the **community/module level**. Use
     `get_architecture_overview_tool` and community detection output as the
     primary artifact. Avoid dumping all individual symbol nodes into the
     same view — it produces an unreadable hairball.
   - If the goal is **debugging or tracing a specific symbol's impact** →
     per-symbol detail is correct. Use `get_impact_radius_tool` /
     `detect_changes_tool` and keep the view scoped to that symbol's
     neighborhood, not the whole graph.

3. **Surface only the structurally interesting nodes**, not everything:
   - Pull `get_hub_nodes_tool` (most-connected nodes) and
     `get_bridge_nodes_tool` (architectural chokepoints) and make sure these
     are visually prominent in whatever's generated.
   - Don't give equal visual weight to private helpers and public APIs.

4. **Don't mix edge types indiscriminately.** Calls, imports, inheritance,
   and test-coverage edges mean different things — keep them visually
   distinguishable (color/style), or generate separate views per relationship
   type if the export format doesn't support layered filtering.

5. **Generate the actual visual export, then verify it's non-empty before
   reporting success:**
   ```
   code-review-graph visualize --format obsidian
   ```
   Then check `_INDEX.md` (or equivalent index file) for non-zero
   Nodes/Edges/Communities counts. If it's empty, investigate — check the
   working directory matches the built graph's location — before telling me
   it's done. Do not report success on an empty export.

6. **If targeting Obsidian specifically**, structure the vault so both
   granularities are usable:
   - Keep per-symbol files (for backlinks/search) but tag them by
     community/module so Obsidian's native graph filters can toggle between
     "show me the architecture" and "show me this one function's
     connections."
