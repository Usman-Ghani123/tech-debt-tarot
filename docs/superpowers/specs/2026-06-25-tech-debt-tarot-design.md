# Tech Debt Tarot — Design Spec
**Date:** 2026-06-25

## Overview

Single-page web app delivered as one `index.html` file. Users draw a three-card tarot spread; each card is a piece of software engineering wisdom or warning. Mystical outer shell "glitches" into developer-speak on card reveal. No backend, no build step, runs entirely client-side.

---

## Architecture

### Files
- `index.html` — the entire app (inline `<style>` + inline `<script>`)

### External Dependencies (CDN only)
- Google Fonts: **Cinzel** (mystical titles/headers)
- Google Fonts: **JetBrains Mono** (stack traces, monospace dev content)

### App State
```js
{
  drawn: [],          // array of 3 { card, orientation: "upright"|"reversed" }
  phase: "idle" | "drawing" | "revealed"
}
```

### Card Data Structure
```js
{
  id: Number,
  name: String,
  number: String,       // Roman numeral or "0"
  upright:  { text: String, trace: String },
  reversed: { text: String, trace: String }
}
```

---

## Visual Design

### Color Palette
| Role | Value |
|------|-------|
| Page background | `#0a0a12` |
| Card back gradient | `#1a0a2e → #0d0520` |
| Gold (borders, titles) | `#c9a84c` |
| Card front background | `#0d0d1a` |
| Wisdom text | `#e8e0d0` (warm off-white, serif) |
| Stack trace text | `#4afa9a` (terminal green, JetBrains Mono) |
| Upright tag | `#4afa9a` (green) |
| Reversed tag | `#fa4a6a` (red) |

### Typography
- **Cinzel** — page title, card names, card number
- **Georgia / serif** — wisdom text body
- **JetBrains Mono** — stack traces, subtitle, orientation tags

### Page Layout
```
┌─────────────────────────────────────────┐
│  ✦ TECH DEBT TAROT ✦                    │  Cinzel, gold, text-shadow glow
│  consult the ancient wisdom of your     │  JetBrains Mono, muted
│  codebase                               │
│                                         │
│       [ ✦ CONSULT THE STACK ✦ ]        │  ritual button
│                                         │
│  LEGACY CODE  CURRENT SPRINT  TECH      │  spread position labels
│                               DEBT AHEAD│
│  ┌─────┐      ┌─────┐        ┌─────┐   │  3 card slots, ~160×280px
│  │     │      │     │        │     │   │
│  └─────┘      └─────┘        └─────┘   │
└─────────────────────────────────────────┘
```

### Spread Position Labels
- **Position 1:** LEGACY CODE
- **Position 2:** CURRENT SPRINT
- **Position 3:** TECH DEBT AHEAD

### Card Dimensions
~160px wide × 280px tall (≈ 1:1.75 ratio, traditional tarot proportions)

### Card Back Design
Pure CSS: deep purple radial gradient, gold border, a Unicode sigil (`◈`) centered in Cinzel.

### Revealed Card Layout
```
┌───────────────────┐
│  I        UPRIGHT │   number (Cinzel) + orientation tag (mono, colored)
│                   │
│   The Refactorer  │   card name (Cinzel, gold, large)
│                   │
│  Clean boundaries │   wisdom text (Georgia, off-white)
│  emerge. The      │
│  coupling         │
│  dissolves.       │
│  ──────────────── │   divider
│  > RefactorError  │   stack trace (JetBrains Mono, green)
│    at soul.js:1   │
└───────────────────┘
```

---

## Interactions & Animations

### Draw Sequence
1. User clicks "Consult the Stack"
2. Button shakes (CSS keyframe, 300ms) — the ritual
3. 3 unique cards selected at random from deck; each independently upright or reversed (50/50)
4. All 3 face-down cards fade+slide in (`opacity 0→1`, `translateY 20px→0`, 400ms)
5. Cards flip sequentially: card 1 at `t=600ms`, card 2 at `t=900ms`, card 3 at `t=1200ms`
6. Each flip: CSS 3D `rotateY(0→180deg)`, 700ms ease-in-out
7. On each flip completion: glitch animation fires on card text (150ms)
8. After all 3 revealed: button text → "Draw Again", phase = `revealed`
9. "Draw Again" resets to blank slots, phase = `idle`

### CSS 3D Flip
```css
.card { transform-style: preserve-3d; transition: transform 0.7s ease-in-out; }
.card.flipped { transform: rotateY(180deg); }
.card-back  { backface-visibility: hidden; }
.card-front { backface-visibility: hidden; transform: rotateY(180deg); }
```

### Glitch Effect
```css
@keyframes glitch {
  0%   { transform: translate(0);        color: #4afa9a; }
  20%  { transform: translate(-2px,1px); color: #fa4a6a; }
  40%  { transform: translate(2px,-1px); color: #4af0fa; }
  60%  { transform: translate(-1px,2px); color: #fa4a6a; }
  100% { transform: translate(0);        color: inherit; }
}
```

### No-Repeat Guarantee
Within a single spread, the 3 drawn cards are always unique. On re-draw, the full 21-card deck reshuffles.

---

## The 21 Cards

| # | Name | Upright Text | Reversed Text |
|---|------|-------------|--------------|
| 0 | The Fool's Deploy | "Ship early, learn fast. The unknown is not the enemy." | "You pushed to main without a PR. On a Friday." |
| I | The Refactorer | "Clean boundaries emerge. The coupling dissolves. Ship it." | "You will rewrite this twice and regret both times." |
| II | The Legacy Monolith | "Ancient wisdom lives in undocumented code. Read before you delete." | "Nobody knows what this does. Nobody will ask." |
| III | The Infinite Loop | "Persistence yields results. Keep iterating." | "You have been debugging this for six hours. It is a typo." |
| IV | The Pull Request | "Your peers sharpen your code. Invite their gaze." | "LGTM. Nobody read it." |
| V | The Technical Debt | "A conscious shortcut, documented and bounded. Acceptable." | "This was supposed to be temporary. In 2019." |
| VI | The Null Pointer | "The void is acknowledged. Handle it gracefully." | "You assumed it would never be null. It was null." |
| VII | The Standup | "Brief alignment moves the team forward." | "This could have been an email. It is now 47 minutes long." |
| VIII | The Hotfix | "Swift action prevents catastrophe. The system holds." | "The fix ships at 4:57 PM on a Friday." |
| IX | The Abandoned Dependency | "Stand on giants' shoulders — but check they still maintain it." | "847 open issues. Last commit: 2020. You ship tomorrow." |
| X | The Merge Conflict | "Two truths reconciled become stronger than either." | "You accepted both sides. Production burns." |
| XI | The Architect | "The system reveals its elegant, minimal structure." | "Six abstraction layers. One button. No docs." |
| XII | The Deployment | "Release with confidence. The pipeline is green." | "Works on my machine." |
| XIII | The Stack Overflow | "Others have walked this path. Their wisdom awaits you." | "The accepted answer is from 2011. The API no longer exists." |
| XIV | The Codebase | "Balance: new features retire old ones. The system breathes." | "40,000 lines. No tests. One file." |
| XV | The Sprint | "Contained chaos becomes velocity. The timebox is sacred." | "Scope creep arrives on day two. It never leaves." |
| XVI | The Senior Dev | "Hard-won experience illuminates the path. Ask them." | "They left six months ago. Nobody has the password." |
| XVII | The Unit Test | "Your assumptions, made explicit and verified." | "100% coverage. All mocks. Zero real confidence." |
| XVIII | The README | "Documentation is an act of kindness to your future self." | "Last updated: three years ago. The setup steps fail." |
| XIX | The Environment Variable | "Configuration externalised. The system adapts." | "It was hardcoded in production." |
| XX | The Root Cause | "The true source reveals itself at last. Fix the cause." | "It is DNS. It is always DNS." |

Each card has a unique thematic fake stack trace for both upright and reversed orientations (e.g. reversed Null Pointer: `NullAssumptionError: field was always null` / `at HopeService.get (optimism.js:47)`).

---

## Out of Scope

- Backend / persistence of any kind
- Card history across page reloads
- Sharing / screenshot features
- Mobile-specific layout optimisations (responsive is fine, mobile-first is not required)
