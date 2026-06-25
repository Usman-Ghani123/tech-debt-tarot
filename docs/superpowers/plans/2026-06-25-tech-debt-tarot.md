# Tech Debt Tarot — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build `index.html` — a single-file tarot web app with 21 dev-themed cards, three-card spread, CSS 3D flip animation, and post-flip glitch reveal.

**Architecture:** Single `index.html` with inline `<style>` and `<script>`. App state is a plain JS object. Card data is a JS array literal. No build step; only external resources are two Google Fonts via CDN.

**Tech Stack:** HTML5, CSS3 (custom properties, 3D transforms, `@keyframes`), vanilla ES6 JS, Google Fonts (Cinzel + JetBrains Mono via CDN).

## Global Constraints

- Delivery: `index.html` at repo root — no other files created
- No npm, no build tools, no JS frameworks
- External deps: Google Fonts CDN only (`fonts.googleapis.com`)
- Exactly 21 cards (ids 0–20)
- Spread always draws 3 unique cards per draw
- Button disabled (`pointer-events: none`, opacity 0.4) while `state.phase === 'drawing'`
- Desktop-first; `@media (max-width: 600px)` stacks cards vertically

---

### Task 1: HTML Skeleton + CSS Foundation

**Files:**
- Create: `index.html`

**Interfaces:**
- Produces: `index.html` with `<style>` and `<script>` blocks, Google Fonts loaded, CSS custom properties defined, body layout applied

- [ ] **Step 1: Create `index.html` with full skeleton**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Tech Debt Tarot</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Cinzel:wght@400;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
  <style>
    :root {
      --bg: #0a0a12;
      --card-back-from: #1a0a2e;
      --card-back-to: #0d0520;
      --gold: #c9a84c;
      --card-front-bg: #0d0d1a;
      --wisdom: #e8e0d0;
      --green: #4afa9a;
      --red: #fa4a6a;
      --card-w: 160px;
      --card-h: 280px;
    }

    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      background: var(--bg);
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 3rem 1rem 4rem;
      color: var(--wisdom);
    }
  </style>
</head>
<body>
  <script>
    // app code goes here
  </script>
</body>
</html>
```

- [ ] **Step 2: Open in browser and verify**

Open `index.html` directly in browser (double-click or `file://` URL).
Expected: solid `#0a0a12` near-black background, no console errors, page title "Tech Debt Tarot".

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add HTML skeleton and CSS foundation"
```

---

### Task 2: Header + Draw Button

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: CSS variables from Task 1
- Produces: `.header`, `.title`, `.subtitle`, `#draw-btn` elements + styles; `@keyframes shake`

- [ ] **Step 1: Add header + button HTML** (inside `<body>`, before `<script>`)

```html
<header class="header">
  <h1 class="title">&#10022; TECH DEBT TAROT &#10022;</h1>
  <p class="subtitle">// consult the ancient wisdom of your codebase</p>
</header>

<button class="draw-btn" id="draw-btn">&#10022; CONSULT THE STACK &#10022;</button>
```

- [ ] **Step 2: Add header + button CSS** (inside `<style>`)

```css
.header {
  text-align: center;
  margin-bottom: 2rem;
}

.title {
  font-family: 'Cinzel', serif;
  font-size: 2.4rem;
  font-weight: 700;
  color: var(--gold);
  text-shadow: 0 0 20px rgba(201,168,76,0.45), 0 0 50px rgba(201,168,76,0.2);
  letter-spacing: 0.08em;
}

.subtitle {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.72rem;
  color: rgba(232,224,208,0.38);
  margin-top: 0.6rem;
  letter-spacing: 0.05em;
}

.draw-btn {
  font-family: 'Cinzel', serif;
  font-size: 0.95rem;
  font-weight: 600;
  color: var(--gold);
  background: transparent;
  border: 1px solid var(--gold);
  padding: 0.8rem 2.2rem;
  cursor: pointer;
  letter-spacing: 0.18em;
  margin-bottom: 3rem;
  transition: background 0.2s ease, box-shadow 0.2s ease;
}

.draw-btn:hover:not(:disabled) {
  background: rgba(201,168,76,0.07);
  box-shadow: 0 0 22px rgba(201,168,76,0.28);
}

.draw-btn:disabled {
  opacity: 0.4;
  pointer-events: none;
  cursor: default;
}

.draw-btn.shaking {
  animation: shake 0.3s ease;
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  20%       { transform: translateX(-4px); }
  40%       { transform: translateX(4px); }
  60%       { transform: translateX(-3px); }
  80%       { transform: translateX(3px); }
}
```

- [ ] **Step 3: Verify in browser**

Reload. Expected: gold glowing title in Cinzel, muted monospace subtitle, gold-bordered button. Hover the button — it glows slightly. No console errors.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add header and draw button"
```

---

### Task 3: Spread Layout + Card Slots

**Files:**
- Modify: `index.html`

**Interfaces:**
- Produces: `.spread` flex container, 3 `.slot` columns each with `.slot-label` and `.card-slot[data-slot]`; empty placeholder `.card-slot-empty` in each slot initially

- [ ] **Step 1: Add spread HTML** (after `.draw-btn`, before `<script>`)

```html
<div class="spread">
  <div class="slot">
    <div class="slot-label">Legacy Code</div>
    <div class="card-slot" data-slot="0"><div class="card-slot-empty"></div></div>
  </div>
  <div class="slot">
    <div class="slot-label">Current Sprint</div>
    <div class="card-slot" data-slot="1"><div class="card-slot-empty"></div></div>
  </div>
  <div class="slot">
    <div class="slot-label">Tech Debt Ahead</div>
    <div class="card-slot" data-slot="2"><div class="card-slot-empty"></div></div>
  </div>
</div>
```

- [ ] **Step 2: Add spread + slot CSS**

```css
.spread {
  display: flex;
  gap: 2rem;
  align-items: flex-start;
  justify-content: center;
}

.slot {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.65rem;
}

.slot-label {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.58rem;
  letter-spacing: 0.2em;
  color: rgba(201,168,76,0.45);
  text-transform: uppercase;
  text-align: center;
}

.card-slot {
  width: var(--card-w);
  height: var(--card-h);
}

.card-slot-empty {
  width: 100%;
  height: 100%;
  border: 1px dashed rgba(201,168,76,0.18);
  border-radius: 4px;
}
```

- [ ] **Step 3: Verify in browser**

Reload. Expected: three labeled columns side by side, each with a faint dashed gold rectangle (empty slot placeholder). Labels read "Legacy Code", "Current Sprint", "Tech Debt Ahead".

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add three-card spread layout"
```

---

### Task 4: Card CSS — 3D Flip, Back Face, Front Face, Animations

**Files:**
- Modify: `index.html`

**Interfaces:**
- Produces: `.card-container` (perspective wrapper), `.card` (flip target), `.card-back`, `.card-front`, `.card-content` + all child element styles; `@keyframes fadeSlideIn`, `@keyframes glitch`
- Consumed by Task 7: `flipCard()` adds `.flipped` to `.card`; glitch applies `.glitching` to `.card-content`

- [ ] **Step 1: Add card CSS** (append to `<style>`)

```css
.card-container {
  width: var(--card-w);
  height: var(--card-h);
  perspective: 900px;
}

.card-container.entering {
  animation: fadeSlideIn 0.4s ease both;
}

.card {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  transition: transform 0.7s ease-in-out;
}

.card.flipped {
  transform: rotateY(180deg);
}

.card-back,
.card-front {
  position: absolute;
  inset: 0;
  backface-visibility: hidden;
  -webkit-backface-visibility: hidden;
  border: 1px solid var(--gold);
  border-radius: 4px;
  overflow: hidden;
}

.card-back {
  background: radial-gradient(ellipse at 50% 40%, var(--card-back-from) 0%, var(--card-back-to) 100%);
  display: flex;
  align-items: center;
  justify-content: center;
}

.card-back::before {
  content: '';
  position: absolute;
  inset: 7px;
  border: 1px solid rgba(201,168,76,0.25);
  border-radius: 2px;
}

.card-back-sigil {
  font-family: 'Cinzel', serif;
  font-size: 2.8rem;
  color: var(--gold);
  opacity: 0.55;
  text-shadow: 0 0 18px rgba(201,168,76,0.5);
  user-select: none;
}

.card-front {
  background: var(--card-front-bg);
  transform: rotateY(180deg);
}

.card-content {
  display: flex;
  flex-direction: column;
  height: 100%;
  padding: 0.75rem 0.7rem;
}

.card-content.glitching {
  animation: glitch 0.15s ease;
}

.card-header-row {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 0.45rem;
}

.card-number {
  font-family: 'Cinzel', serif;
  font-size: 0.7rem;
  color: rgba(201,168,76,0.55);
  letter-spacing: 0.05em;
}

.card-orientation {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.5rem;
  letter-spacing: 0.08em;
  padding: 2px 5px;
  border-radius: 2px;
  text-transform: uppercase;
}

.card-orientation.upright {
  color: var(--green);
  border: 1px solid var(--green);
}

.card-orientation.reversed {
  color: var(--red);
  border: 1px solid var(--red);
}

.card-name {
  font-family: 'Cinzel', serif;
  font-size: 0.82rem;
  font-weight: 600;
  color: var(--gold);
  line-height: 1.3;
  margin-bottom: 0.6rem;
}

.card-wisdom {
  font-family: Georgia, 'Times New Roman', serif;
  font-size: 0.68rem;
  color: var(--wisdom);
  line-height: 1.55;
  flex: 1;
}

.card-divider {
  border: none;
  border-top: 1px solid rgba(201,168,76,0.18);
  margin: 0.55rem 0;
}

.card-trace {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.52rem;
  color: var(--green);
  line-height: 1.45;
  white-space: pre-wrap;
  word-break: break-all;
}

@keyframes fadeSlideIn {
  from { opacity: 0; transform: translateY(18px); }
  to   { opacity: 1; transform: translateY(0); }
}

@keyframes glitch {
  0%   { transform: translate(0, 0);       opacity: 1; }
  12%  { transform: translate(-3px, 0);    opacity: 0.75; }
  25%  { transform: translate(3px, 1px);   opacity: 1; }
  37%  { transform: translate(-2px, -1px); opacity: 0.85; }
  50%  { transform: translate(0, 2px);     opacity: 1; }
  62%  { transform: translate(2px, 0);     opacity: 0.8; }
  75%  { transform: translate(-1px, 1px);  opacity: 1; }
  100% { transform: translate(0, 0);       opacity: 1; }
}
```

- [ ] **Step 2: Temporarily add a test card** (paste after `.spread` in body, remove after verify)

```html
<!-- TEMP: remove after verify -->
<div class="card-container" style="margin-top:2rem">
  <div class="card" id="test-card">
    <div class="card-back"><span class="card-back-sigil">&#9672;</span></div>
    <div class="card-front">
      <div class="card-content">
        <div class="card-header-row">
          <span class="card-number">I</span>
          <span class="card-orientation upright">upright</span>
        </div>
        <div class="card-name">The Refactorer</div>
        <div class="card-wisdom">Clean boundaries emerge. The coupling dissolves. Ship it.</div>
        <hr class="card-divider">
        <pre class="card-trace">RefactorComplete: coupling dissolved
  at CleanCode.emerge (soul.js:1)
  at BoundedContext.hold (domain.js:42)</pre>
      </div>
    </div>
  </div>
</div>
```

- [ ] **Step 3: Verify card back**

Reload. Below the spread: purple/black radial gradient card, gold border, inner border, gold `◈` sigil. Face-down (back visible).

- [ ] **Step 4: Force flip in DevTools console, verify front**

```js
document.getElementById('test-card').classList.add('flipped')
```

Expected: smooth 0.7s 3D rotation to front face. Front shows number "I" (top-left, muted gold), green "UPRIGHT" tag (top-right), gold "The Refactorer" name in Cinzel, wisdom text in serif, divider, green monospace stack trace.

- [ ] **Step 5: Remove the temp test card from HTML**

Delete the `<!-- TEMP -->` block added in Step 2.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add card CSS with 3D flip, back/front faces, and animation keyframes"
```

---

### Task 5: Card Data Array

**Files:**
- Modify: `index.html`

**Interfaces:**
- Produces: `const CARDS` (global, in `<script>`) — 21 objects with shape `{ id: Number, name: String, number: String, upright: {text: String, trace: String}, reversed: {text: String, trace: String} }`
- Consumed by Task 6: `drawSpread()` uses `CARDS`

- [ ] **Step 1: Add CARDS array to `<script>` block**

```js
const CARDS = [
  {
    id: 0, name: "The Fool's Deploy", number: "0",
    upright: {
      text:  "Ship early, learn fast. The unknown is not the enemy.",
      trace: "EarlyShipSuccess: iteration 1 deployed\n  at Courage.launch (unknown.js:1)\n  at Pipeline.run (ci.js:200)"
    },
    reversed: {
      text:  "You pushed to main without a PR. On a Friday.",
      trace: "DirectPushError: branch protection was disabled\n  at Git.push (main.js:1)\n  at Friday.afternoon (deploy.js:17)"
    }
  },
  {
    id: 1, name: "The Refactorer", number: "I",
    upright: {
      text:  "Clean boundaries emerge. The coupling dissolves. Ship it.",
      trace: "RefactorComplete: coupling dissolved, tests green\n  at CleanCode.emerge (soul.js:1)\n  at BoundedContext.hold (domain.js:42)"
    },
    reversed: {
      text:  "You will rewrite this twice and regret both times.",
      trace: "ThirdRewriteError: estimation exceeded\n  at History.repeat (v3.js:1)\n  at Hubris.check (iteration.js:∞)"
    }
  },
  {
    id: 2, name: "The Legacy Monolith", number: "II",
    upright: {
      text:  "Ancient wisdom lives in undocumented code. Read before you delete.",
      trace: "AncientWisdom: comment from 2008 still valid\n  at Read.before.delete (legacy.js:1)\n  at Respect.earn (history.js:144)"
    },
    reversed: {
      text:  "Nobody knows what this does. Nobody will ask.",
      trace: "UnknownBehaviorError: bus factor = 0\n  at Documentation.missing (legacy.js:???)\n  at LastEngineer.left (2022.js:1)"
    }
  },
  {
    id: 3, name: "The Infinite Loop", number: "III",
    upright: {
      text:  "Persistence yields results. Keep iterating.",
      trace: "ConvergenceReached: loop terminated at step 7\n  at Persistence.pay (loop.js:7)\n  at Algorithm.halt (math.js:101)"
    },
    reversed: {
      text:  "You have been debugging this for six hours. It is a typo.",
      trace: "TypoException: semicolon missing since hour 1\n  at StareAt.screen (6hours.js:1)\n  at Variable.name (typo.js:∞)"
    }
  },
  {
    id: 4, name: "The Pull Request", number: "IV",
    upright: {
      text:  "Your peers sharpen your code. Invite their gaze.",
      trace: "PeerReviewInsight: line 47 refactored\n  at Colleague.sharpen (review.js:1)\n  at Code.improve (pr.js:3)"
    },
    reversed: {
      text:  "LGTM. Nobody read it.",
      trace: "LGTMError: approved in 4 seconds\n  at Nobody.read (pr.js:1)\n  at Rubber.stamp (review.js:0)"
    }
  },
  {
    id: 5, name: "The Technical Debt", number: "V",
    upright: {
      text:  "A conscious shortcut, documented and bounded. Acceptable.",
      trace: "DebtAcknowledged: TECH-∞ created\n  at Conscious.shortcut (todo.js:1)\n  at Ticket.filed (jira.js:42)"
    },
    reversed: {
      text:  "This was supposed to be temporary. In 2019.",
      trace: "TemporaryCodeError: context.year = 2019\n  at TODO.wait (fixlater.js:1)\n  at Hack.persist (temporary.js:∞)"
    }
  },
  {
    id: 6, name: "The Null Pointer", number: "VI",
    upright: {
      text:  "The void is acknowledged. Handle it gracefully.",
      trace: "NullHandled: optional chaining applied\n  at Optional.unwrap (null-safe.js:1)\n  at EdgeCase.survive (handler.js:23)"
    },
    reversed: {
      text:  "You assumed it would never be null. It was null.",
      trace: "NullAssumptionError: field was always null\n  at HopeService.get (optimism.js:47)\n  at Production.run (live.js:∞)"
    }
  },
  {
    id: 7, name: "The Standup", number: "VII",
    upright: {
      text:  "Brief alignment moves the team forward.",
      trace: "MeetingComplete: 8 minutes, 3 blockers surfaced\n  at Update.brief (standup.js:1)\n  at Timebox.hold (sync.js:900)"
    },
    reversed: {
      text:  "This could have been an email. It is now 47 minutes long.",
      trace: "MeetingOverflowError: 47 minutes elapsed\n  at Email.became (meeting.js:1)\n  at Agenda.missing (calendar.js:0)"
    }
  },
  {
    id: 8, name: "The Hotfix", number: "VIII",
    upright: {
      text:  "Swift action prevents catastrophe. The system holds.",
      trace: "HotfixDeployed: p0 resolved in 12 minutes\n  at Swift.action (oncall.js:1)\n  at Incident.close (production.js:200)"
    },
    reversed: {
      text:  "The fix ships at 4:57 PM on a Friday.",
      trace: "FridayDeployError: 4:57 PM, no rollback plan\n  at Courage.misplace (hotfix.js:1)\n  at OnCall.phone (friday.js:17)"
    }
  },
  {
    id: 9, name: "The Abandoned Dependency", number: "IX",
    upright: {
      text:  "Stand on giants’ shoulders — but check they still maintain it.",
      trace: "DependencyAudit: maintainer active, 12 contributors\n  at Due.diligence (package.js:1)\n  at npm.audit (clean.js:0)"
    },
    reversed: {
      text:  "847 open issues. Last commit: 2020. You ship tomorrow.",
      trace: "PackageAbandonedError: issue #847 still open\n  at LastCommit.date (2020.js:1)\n  at Ship.tomorrow (deadline.js:9)"
    }
  },
  {
    id: 10, name: "The Merge Conflict", number: "X",
    upright: {
      text:  "Two truths reconciled become stronger than either.",
      trace: "MergeResolved: histories reconciled\n  at Conflict.face (git.js:1)\n  at Truth.reconcile (merge.js:42)"
    },
    reversed: {
      text:  "You accepted both sides. Production burns.",
      trace: "AcceptBothError: <<<<<<< HEAD in production\n  at Merge.panic (conflict.js:1)\n  at Fire.start (main.js:1)"
    }
  },
  {
    id: 11, name: "The Architect", number: "XI",
    upright: {
      text:  "The system reveals its elegant, minimal structure.",
      trace: "ArchitectureRevealed: 3 modules, clear interfaces\n  at Pattern.emerge (system.js:1)\n  at Complexity.tame (design.js:7)"
    },
    reversed: {
      text:  "Six abstraction layers. One button. No docs.",
      trace: "AbstractionLimitError: depth = 6\n  at Factory.of.factory (pattern.js:1)\n  at Button.click (facade/proxy/adapter/bridge.js:∞)"
    }
  },
  {
    id: 12, name: "The Deployment", number: "XII",
    upright: {
      text:  "Release with confidence. The pipeline is green.",
      trace: "DeploySuccess: all checks passed\n  at Pipeline.green (ci.yml:200)\n  at Confidence.deploy (main.js:1)"
    },
    reversed: {
      text:  "Works on my machine.",
      trace: "EnvDivergenceError: works on localhost only\n  at My.machine (works.js:1)\n  at Production.differ (env.js:∞)"
    }
  },
  {
    id: 13, name: "The Stack Overflow", number: "XIII",
    upright: {
      text:  "Others have walked this path. Their wisdom awaits you.",
      trace: "AnswerFound: 2847 upvotes, accepted\n  at Others.solved (stackoverflow.com:1)\n  at Search.before.ask (google.js:1)"
    },
    reversed: {
      text:  "The accepted answer is from 2011. The API no longer exists.",
      trace: "DeprecatedAnswerError: API removed in v3\n  at Answer.age (2011.js:1)\n  at Migration.guide (404.js:1)"
    }
  },
  {
    id: 14, name: "The Codebase", number: "XIV",
    upright: {
      text:  "Balance: new features retire old ones. The system breathes.",
      trace: "EntropyReduced: feature added, dead code removed\n  at Balance.keep (codebase.js:1)\n  at System.breathe (cleanup.js:42)"
    },
    reversed: {
      text:  "40,000 lines. No tests. One file.",
      trace: "FileSizeLimitError: 40000 lines, 0 tests\n  at God.object (everything.js:1)\n  at Fear.refactor (untouched.js:∞)"
    }
  },
  {
    id: 15, name: "The Sprint", number: "XV",
    upright: {
      text:  "Contained chaos becomes velocity. The timebox is sacred.",
      trace: "SprintComplete: velocity 42, scope held\n  at Timebox.sacred (sprint.js:1)\n  at Team.focus (velocity.js:8)"
    },
    reversed: {
      text:  "Scope creep arrives on day two. It never leaves.",
      trace: "ScopeCreepError: entered day 2, never terminated\n  at Feature.added (sprint.js:3)\n  at Stakeholder.idea (slack.js:∞)"
    }
  },
  {
    id: 16, name: "The Senior Dev", number: "XVI",
    upright: {
      text:  "Hard-won experience illuminates the path. Ask them.",
      trace: "KnowledgeTransferred: pair session complete\n  at Experience.share (mentor.js:1)\n  at Wisdom.document (runbook.js:42)"
    },
    reversed: {
      text:  "They left six months ago. Nobody has the password.",
      trace: "OffboardingError: last known holder departed\n  at Knowledge.vanish (bus.factor:1)\n  at Password.lost (vault.js:404)"
    }
  },
  {
    id: 17, name: "The Unit Test", number: "XVII",
    upright: {
      text:  "Your assumptions, made explicit and verified.",
      trace: "TestPassed: assumption verified in 12ms\n  at Assert.truth (test.js:1)\n  at Confidence.earn (suite.js:42)"
    },
    reversed: {
      text:  "100% coverage. All mocks. Zero real confidence.",
      trace: "MockConfidenceError: coverage=100, reality=0\n  at Mock.everything (test.js:1)\n  at Real.database (untested.js:∞)"
    }
  },
  {
    id: 18, name: "The README", number: "XVIII",
    upright: {
      text:  "Documentation is an act of kindness to your future self.",
      trace: "DocumentationFound: setup succeeded first try\n  at Kindness.future.self (readme.md:1)\n  at Instructions.work (setup.js:1)"
    },
    reversed: {
      text:  "Last updated: three years ago. The setup steps fail.",
      trace: "StaleDocError: last updated 3 years ago\n  at Step.3.fail (readme.md:47)\n  at Tool.deprecated (2021.js:1)"
    }
  },
  {
    id: 19, name: "The Environment Variable", number: "XIX",
    upright: {
      text:  "Configuration externalised. The system adapts.",
      trace: "ConfigExternalised: 12-factor compliant\n  at DotEnv.load (config.js:1)\n  at Secret.safe (vault.js:3)"
    },
    reversed: {
      text:  "It was hardcoded in production.",
      trace: "HardcodedSecretError: committed in 2018, still there\n  at String.literal (production.js:1)\n  at Audit.found (git.history:∞)"
    }
  },
  {
    id: 20, name: "The Root Cause", number: "XX",
    upright: {
      text:  "The true source reveals itself at last. Fix the cause.",
      trace: "RootCauseFound: 5-whys complete\n  at Symptom.ignore (never.js:0)\n  at Cause.fix (postmortem.js:1)"
    },
    reversed: {
      text:  "It is DNS. It is always DNS.",
      trace: "DNSPropagationError: it was always DNS\n  at Check.dns.last (lesson.js:1)\n  at DNS.it.always.is (always.js:∞)"
    }
  }
];
```

- [ ] **Step 2: Verify in DevTools console**

Open `index.html` in browser, open DevTools console (F12):
```
> CARDS.length
21
> CARDS[0].name
"The Fool's Deploy"
> CARDS[20].reversed.text
"It is DNS. It is always DNS."
> CARDS[6].reversed.trace
"NullAssumptionError: field was always null\n  at HopeService.get (optimism.js:47)\n  at Production.run (live.js:∞)"
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add all 21 card data objects with upright/reversed text and stack traces"
```

---

### Task 6: Draw Logic — State, drawSpread, resetSpread, Event Handler

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `CARDS` array (Task 5), `.card-slot` elements (Task 3), `#draw-btn` (Task 2)
- Produces: `state` object; `drawSpread()` — selects 3 unique cards, renders face-down DOM cards, schedules flips; `resetSpread()` — clears slots, resets state; click handler on `#draw-btn`
- Consumed by Task 7: `flipCard(index)` must be defined before `drawSpread()` calls it via `setTimeout`

- [ ] **Step 1: Add state + resetSpread + drawSpread + event handler to `<script>`** (after CARDS array; add a `// flipCard defined below` comment placeholder — replace in Task 7)

```js
const state = { drawn: [], phase: 'idle' };

function resetSpread() {
  document.querySelectorAll('.card-slot').forEach(function(slot) {
    slot.innerHTML = '<div class="card-slot-empty"></div>';
  });
  state.drawn = [];
}

function drawSpread() {
  state.phase = 'drawing';

  var btn = document.getElementById('draw-btn');
  btn.disabled = true;
  btn.classList.add('shaking');
  setTimeout(function() { btn.classList.remove('shaking'); }, 300);

  // 3 unique cards, each randomly upright or reversed
  var shuffled = CARDS.slice().sort(function() { return Math.random() - 0.5; });
  state.drawn = shuffled.slice(0, 3).map(function(card) {
    return { card: card, orientation: Math.random() < 0.5 ? 'upright' : 'reversed' };
  });

  // Render face-down cards into slots
  document.querySelectorAll('.card-slot').forEach(function(slot, i) {
    slot.innerHTML = '';

    var container = document.createElement('div');
    container.className = 'card-container entering';
    container.style.animationDelay = (i * 80) + 'ms';

    var cardEl = document.createElement('div');
    cardEl.className = 'card';
    cardEl.id = 'card-' + i;

    var back = document.createElement('div');
    back.className = 'card-back';
    back.innerHTML = '<span class="card-back-sigil">&#9672;</span>';

    var front = document.createElement('div');
    front.className = 'card-front';

    cardEl.appendChild(back);
    cardEl.appendChild(front);
    container.appendChild(cardEl);
    slot.appendChild(container);
  });

  // Staggered flips: 600ms, 900ms, 1200ms
  [0, 1, 2].forEach(function(i) {
    setTimeout(function() { flipCard(i); }, 600 + i * 300);
  });

  // Re-enable button after last flip settles: 1200 + 700 (flip) + 200 (buffer) = 2100ms
  setTimeout(function() {
    btn.textContent = '✦ DRAW AGAIN ✦';
    btn.disabled = false;
    state.phase = 'revealed';
  }, 2100);
}

document.getElementById('draw-btn').addEventListener('click', function() {
  if (state.phase === 'drawing') return;
  resetSpread();
  drawSpread();
});
```

- [ ] **Step 2: Add stub for flipCard** (before drawSpread, so the setTimeout calls don't error)

```js
function flipCard(index) {
  // implemented in Task 7
  console.log('flipCard', index, state.drawn[index]);
}
```

- [ ] **Step 3: Verify face-down draw in browser**

Reload. Click "✦ CONSULT THE STACK ✦".
Expected:
- Button shakes briefly, becomes disabled/dimmed
- 3 face-down cards fade+slide in with gold border and `◈` sigil (staggered ~80ms apart)
- DevTools console logs `flipCard 0`, `flipCard 1`, `flipCard 2` at ~600ms intervals
- At ~2.1s button re-enables as "✦ DRAW AGAIN ✦"
- Clicking "Draw Again" clears cards and immediately starts a new draw

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add draw logic, state management, and button event handler"
```

---

### Task 7: Flip Logic + Glitch Effect

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `state.drawn[index]` (Task 6), `#card-{index}` DOM elements (Task 6), `.card-front` (Task 4), `@keyframes glitch` (Task 4)
- Produces: `flipCard(index)` — replaces the stub from Task 6; injects card HTML into `.card-front`, adds `.flipped` to `.card`, fires glitch on `transitionend`

- [ ] **Step 1: Replace the `flipCard` stub with the real implementation**

Find and replace the stub function:

```js
// REMOVE this stub:
function flipCard(index) {
  // implemented in Task 7
  console.log('flipCard', index, state.drawn[index]);
}
```

Replace with:

```js
function flipCard(index) {
  var drawn = state.drawn[index];
  var cardEl = document.getElementById('card-' + index);
  if (!cardEl) return;

  var front = cardEl.querySelector('.card-front');
  var data = drawn.card[drawn.orientation];

  front.innerHTML =
    '<div class="card-content">' +
      '<div class="card-header-row">' +
        '<span class="card-number">' + drawn.card.number + '</span>' +
        '<span class="card-orientation ' + drawn.orientation + '">' + drawn.orientation + '</span>' +
      '</div>' +
      '<div class="card-name">' + drawn.card.name + '</div>' +
      '<div class="card-wisdom">' + data.text + '</div>' +
      '<hr class="card-divider">' +
      '<pre class="card-trace">' + data.trace + '</pre>' +
    '</div>';

  cardEl.classList.add('flipped');

  cardEl.addEventListener('transitionend', function handler(e) {
    if (e.propertyName !== 'transform') return;
    cardEl.removeEventListener('transitionend', handler);
    var content = cardEl.querySelector('.card-content');
    if (!content) return;
    content.classList.add('glitching');
    setTimeout(function() { content.classList.remove('glitching'); }, 150);
  });
}
```

- [ ] **Step 2: Verify full draw sequence in browser**

Reload. Click "✦ CONSULT THE STACK ✦".
Expected:
- Button shakes → 3 face-down cards appear
- At ~600ms: card 1 flips (0.7s smooth 3D rotation), reveals content, brief glitch on text
- At ~900ms: card 2 flips + glitches
- At ~1200ms: card 3 flips + glitches
- Revealed card shows: Roman numeral (top-left), orientation tag (green=UPRIGHT / red=REVERSED), gold card name, wisdom text, gold divider, green monospace stack trace
- Button becomes "✦ DRAW AGAIN ✦"

- [ ] **Step 3: Verify reversed card appears correctly**

Draw several times until a reversed card appears (roughly 50% of cards each draw).
Expected: orientation tag shows "reversed" in red border. Text is the shorter, punchy reversed text. Stack trace is the reversed trace.

- [ ] **Step 4: Verify no-repeat guarantee**

In DevTools console, after a draw:
```js
state.drawn.map(d => d.card.id)
// e.g. [3, 17, 9] — all unique
```

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add card flip animation and post-flip glitch reveal"
```

---

### Task 8: Mobile Responsiveness + Final Verification

**Files:**
- Modify: `index.html`

**Interfaces:**
- Produces: `@media (max-width: 600px)` rules — cards stack vertically, title/button font sizes reduced

- [ ] **Step 1: Add mobile media query** (append to `<style>`)

```css
@media (max-width: 600px) {
  .spread {
    flex-direction: column;
    align-items: center;
  }
  .title {
    font-size: 1.6rem;
  }
  .draw-btn {
    font-size: 0.8rem;
    padding: 0.7rem 1.5rem;
  }
}
```

- [ ] **Step 2: Verify mobile layout**

In Chrome DevTools, toggle device toolbar (Ctrl+Shift+M / Cmd+Shift+M), set width to 375px.
Expected: smaller title, cards stack vertically (one per row), all text readable, button fits width.

- [ ] **Step 3: Final end-to-end checklist**

Reload in normal desktop window. Check each item:

- [ ] Page loads: dark background, gold glowing title, muted subtitle, styled button, 3 labeled empty slots
- [ ] Fonts correct: title uses Cinzel (decorative serifs), stack traces use JetBrains Mono
- [ ] Click button: shakes, becomes disabled, 3 face-down purple/gold cards appear
- [ ] Cards flip 1→2→3 with ~300ms stagger, each with 0.7s smooth 3D rotation
- [ ] Brief glitch fires on each card text after flip completes
- [ ] Card content: number, orientation tag (colored correctly), name in Cinzel, wisdom text, divider, green trace
- [ ] Button re-enables as "DRAW AGAIN" after last flip
- [ ] Click "Draw Again": board resets, new spread starts immediately
- [ ] Draw 5+ times: no crashes, no duplicate cards within any single spread
- [ ] Console (F12): zero JS errors throughout

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add mobile responsiveness and complete end-to-end verification"
```

---

## Spec Coverage Check

| Spec requirement | Task |
|---|---|
| Single `index.html`, no other files | Task 1 |
| Google Fonts CDN (Cinzel + JetBrains Mono) | Task 1 |
| Dark `#0a0a12` background, gold `#c9a84c` palette | Tasks 1–2 |
| Button shake animation on click | Task 2 |
| Three-card spread (Legacy Code / Current Sprint / Tech Debt Ahead) | Task 3 |
| Card back: purple gradient, gold border, inner border, `◈` sigil | Task 4 |
| Card front: number, orientation tag, name, wisdom, divider, stack trace | Task 4 |
| `@keyframes glitch` CSS animation | Task 4 |
| 21 cards, upright + reversed text + stack trace per orientation | Task 5 |
| Button disabled during draw (`phase === 'drawing'`) | Task 6 |
| Face-down cards fade+slide in with stagger | Task 6 |
| 3 unique cards per spread | Task 6 |
| 50/50 upright/reversed per card | Task 6 |
| Staggered CSS 3D flips (600ms, 900ms, 1200ms) | Tasks 6–7 |
| Glitch fires on `.card-content` after `transitionend` | Task 7 |
| `transitionend` filtered by `e.propertyName === 'transform'` | Task 7 |
| Button becomes "Draw Again", re-enables after all flips | Tasks 6–7 |
| Mobile: cards stack vertically at ≤600px | Task 8 |
| No backend, no persistence, no build step | Throughout |
