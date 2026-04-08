---
name: xperion-landing-page
description: Create single-file React/Babel landing pages for xPerion AI products, accelerators, and agents — following the proven VoyagerAI design system. Use this skill whenever the user wants to build a product landing page, demo page, accelerator showcase, or agent portfolio page for xPerion or Coforge branded products. Also trigger when the user mentions "landing page like VoyagerAI", "xPerion page", "product demo page", "agent showcase", "accelerator landing page", or asks to build a single-file HTML app with React inline. This skill covers the complete architecture: design tokens, layout system, data-driven pipeline diagrams, interactive demos, use-case cards, and the full xPerion visual language.
---

# xPerion Landing Page Design System

Build production-grade, single-file React landing pages for AI products, accelerators, and agents — following the xPerion design language established by the VoyagerAI reference implementation.

> **Reference**: See `references/DESIGN_TOKENS.md` for the complete color, typography, shadow, and animation token system.
> **Reference**: See `references/COMPONENT_PATTERNS.md` for reusable component blueprints and SVG architecture diagram specifications.

---

## 1. Architecture — Single-File React/Babel App

Every xPerion landing page is a **single self-contained HTML file** — no build step, no bundler, no separate CSS/JS files. This makes pages instantly shareable, demo-ready, and zero-dependency.

### Tech Stack (fixed)
```
React 18          — CDN: cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js
ReactDOM 18       — CDN: cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js
Babel Standalone   — CDN: cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.23.9/babel.min.js
Google Fonts      — Inter (400–900) + JetBrains Mono (400–600)
```

### File Structure (within one HTML file)
```
<!DOCTYPE html>
<html>
<head>
  — Meta, title, font links, CDN scripts
  — <style> block: resets, keyframe animations, utility classes
</head>
<body>
  <div id="root"></div>
  <script type="text/babel">
    — Design tokens (const X = {...})
    — Data constants (PIPELINE, USE_CASES, KPIS, CHANNELS)
    — Small reusable components (Badge, DataBlock)
    — SVG Architecture Diagram component
    — Agent/step execution components
    — Main App component (full page layout)
    — ReactDOM.createRoot render call
  </script>
</body>
</html>
```

### Key Rules
- ALL styles are inline JSX `style={{...}}` — no CSS classes except animation utilities
- ALL state lives in the App component via `useState` hooks
- ALL data is defined as top-level constants — NEVER hardcoded in JSX
- Use `<script type="text/babel">` for JSX compilation
- Target viewport: 1440px desktop (no mobile-first — these are demo/presentation pages)

---

## 2. Page Layout — Vertical Section Stack

The page uses a **sidebar + main content** layout:

```
┌──────┬──────────────────────────────────────────────┐
│ Side │  Header (sticky, z-100)                      │
│ bar  ├──────────────────────────────────────────────┤
│ 56px │  Hero Section                                │
│      ├──────────────────────────────────────────────┤
│      │  KPI Strip (grid)                            │
│      ├──────────────────────────────────────────────┤
│      │  Architecture Diagram (interactive SVG)      │
│      ├──────────────────────────────────────────────┤
│      │  Live Demo Section (use-case cards → runner) │
│      ├──────────────────────────────────────────────┤
│      │  Footer                                      │
└──────┴──────────────────────────────────────────────┘
```

### Section Anatomy
Every section follows the same card pattern:
```jsx
<section style={{
  margin: "24px 24px 0",        // Consistent 24px outer margins
  padding: "28px 32px",          // Inner breathing room
  background: X.card,            // White card
  borderRadius: 18,              // Soft rounding
  border: `1px solid ${X.bdr}`,  // Subtle border
  boxShadow: X.sh,               // Light shadow
}}>
```

---

## 3. Design Token System

Read `references/DESIGN_TOKENS.md` for the full token reference. Core principles:

### Color Architecture
- **Brand accent**: `coral` (#E8613C) — used for CTAs, highlights, xPerion branding
- **Semantic palette**: 8 named colors (cyan, purple, blue, amber, coral, pink, green, orange) — each has a base + light background variant
- **Neutral scale**: t1 (darkest text), t2 (secondary), t3 (muted/placeholder), bg (page background), card (white), surf (off-white surface)
- **Every color gets a light bg variant**: e.g., `blue: "#3B82F6"` paired with `blueB: "#EFF6FF"` — used for tinted panels and badges

### Color Assignment Strategy
When building a pipeline with N stages, assign **one unique color per stage** from the semantic palette. This creates visual differentiation across the pipeline while maintaining cohesion. The color carries through from the architecture node → detail panel → agent badge → data block accent.

### Typography
- **Primary**: Inter (weights 400–900) — all UI text
- **Monospace**: JetBrains Mono — KPI stats, data blocks, code-like values, timers
- **Size scale**: 8px (micro labels) → 10px (badges) → 11-12px (body) → 14-15px (section text) → 18-22px (headings) → 30px (hero title)

### Shadows
- `sh`: subtle lift — `0 1px 3px rgba(0,0,0,.06), 0 1px 2px rgba(0,0,0,.04)`
- `shH`: hover/emphasis — `0 4px 14px rgba(0,0,0,.08), 0 2px 4px rgba(0,0,0,.04)`

---

## 4. Data-Driven Architecture

**All content is driven by data constants, NOT hardcoded in JSX.** This is the most important pattern. It makes the page maintainable, QA-friendly, and content-swappable.

### PIPELINE Array
Each entry in the pipeline represents one AI agent or processing stage:
```js
const PIPELINE = [
  {
    id: "ingest",                          // URL-safe slug
    agent: "Data Ingestion Agent",         // Full display name (used in badges, panels, data tab)
    label: "Data Ingestion\nAgent",        // SVG node label (\n splits lines)
    icon: "📡",                            // Emoji icon for the node
    color: X.cyan,                         // Semantic color for this stage
    bg: X.cyanB,                           // Light background variant
    diff: "Prebuilt PSS Connectors",       // Differentiator text
    kpi: "Single Customer 360 View",       // KPI text (keep ≤25 chars for SVG fit)
    desc: "Full description paragraph..."  // Shown in detail panel on click
  },
  // ... one entry per pipeline stage
];
```

**Critical rules for PIPELINE data:**
- `agent` and `label` MUST be consistent — label is just the line-broken version of agent
- `kpi` strings MUST be ≤25 characters — they render in tiny SVG text and will overlap if longer
- `diff` and `kpi` MUST be different strings — they serve different purposes
- Each stage gets a unique `color` from the semantic palette — no duplicates

### USE_CASES Array
Each use case powers the interactive demo:
```js
const USE_CASES = [
  {
    id: "churn",
    title: "Churn Prevention",
    subtitle: "Retain high-value passengers...",
    icon: "🛡️",
    color: X.red,
    bg: X.redB,
    passenger: { name, tier, hub, ltv, churnRisk, miles, ... },
    trigger: { title, signals: [{ s: "source", e: "event", sev: "high|critical|medium" }] },
    offer: { headline, action, miles, credit, lounge },
    result: { before: "81%", after: "19%", metric: "churn risk" },
    steps: [
      { t: "Step description", d: { key: "value", ... } }  // One per PIPELINE stage
    ]
  }
];
```

**Critical: `steps` array length MUST equal `PIPELINE` length** — each step maps 1:1 to a pipeline stage.

### KPIS Array
Drives the KPI strip below the hero:
```js
const KPIS = [
  { stat: "60%", label: "Less time for data modeling", color: X.purple },
  // 5-6 entries, keep labels short
];
```

---

## 5. SVG Architecture Diagram

The interactive pipeline diagram is a **responsive SVG** rendered by React. Read `references/COMPONENT_PATTERNS.md` for the full blueprint.

### Key Specifications
```
Viewport:   1000 x 270
Node size:  108 x 76px, gap: 8px, border-radius: 12
Font:       7.5px for node labels, 6px for KPI labels
```

### Layers (bottom to top)
1. **Connecting lines** — dashed animated lines between nodes with arrow endpoints
2. **Nodes** — rounded rects with icon + 2-line label, click-to-select interaction
3. **Title** — centered "PIPELINE — N AUTONOMOUS AGENTS"
4. **KPI labels** — below each node with dashed connector line
5. **Activation bar** — bottom channel list (Website, Apps, Notifications, SMS, Email, Social, Offline)

### Interaction
- Click a node → it gets colored border + glow + active state
- Detail panel appears below the SVG with icon, agent name, full description, differentiator badge, and KPI badge
- Click again or "Clear selection" to deselect

---

## 6. Interactive Demo System

The demo section has two states: **idle** (card selection) and **running** (live execution).

### Idle State
Grid of use-case cards, each showing:
- Icon + title + subtitle
- Passenger badges (tier, hub, LTV)
- "Launch Demo →" button

### Running State (3 tabs)
When a use case is launched, the demo runner executes pipeline steps sequentially with animated transitions:

**Left Panel** (fixed):
- Trigger alert card (signal list with severity badges)
- Passenger profile card (360 view)
- Agent Pipeline Execution log (progress bar + step-by-step with ⚙️ spinner → ✓ checkmark)

**Right Panel** (tabbed):
- **🤖 Agent Flow** — vertical pipeline map with icons, agent names, differentiator badges
- **📱 Experience** — rendered push notification mock + offer card with "Claim" CTA
- **📊 Data** — pipeline data flow showing key-value data blocks per agent step

### Demo Execution Logic
```js
for (let i = 0; i < steps.length; i++) {
  setCurrentStep(i);
  setStepState(i, "running");          // Show spinner
  await delay(800 + random * 500);      // Simulated processing
  setStepState(i, "done");              // Show checkmark
}
setPhase("done");
setDemoTab("experience");               // Auto-switch to experience tab
```

---

## 7. Animation System

### CSS Keyframes (defined in `<style>` block)
```css
fadeUp    — opacity 0→1, translateY(14px→0) — section entrance
fadeIn    — opacity 0→1 — general reveal
slideIn   — opacity 0→1, translateX(-14px→0) — list items
pulse     — box-shadow glow pulsing — running agent indicator
spin      — rotate(0→360deg) — loading spinner ⚙️
flowDash  — stroke-dashoffset animation — SVG connecting lines
```

### Utility Classes
```css
.fu         — fadeUp with cubic-bezier(.22,1,.36,1)
.fi         — fadeIn with ease
.si         — slideIn with cubic-bezier(.22,1,.36,1)
.hover-lift — translateY(-3px) + shadow on hover
```

### Staggered Animations
Use `animationDelay` on mapped elements:
```jsx
{items.map((item, i) => (
  <div className="fu" style={{ animationDelay: `${i * 0.06}s` }}>
```

---

## 8. Reusable Component Library

### Badge
Inline label with semantic color:
```jsx
function Badge({ text, color = X.coral, bg }) {
  return <span style={{
    fontSize: 10, fontWeight: 600, padding: "2.5px 9px",
    borderRadius: 10, background: bg || `${color}14`,
    color, letterSpacing: ".03em", whiteSpace: "nowrap"
  }}>{text}</span>;
}
```

### DataBlock
Monospace key-value display for technical data:
```jsx
function DataBlock({ entries, color }) {
  return (
    <div style={{ background: X.surf, borderRadius: 10, padding: "10px 14px",
      fontFamily: mono, fontSize: 10.5, lineHeight: 1.9, border: `1px solid ${X.bdr}` }}>
      {Object.entries(entries).map(([k, v]) => (
        <div key={k}>
          <span style={{ color, fontWeight: 600 }}>{k}:</span>{" "}
          <span style={{ color: X.t2 }}>{String(v)}</span>
        </div>
      ))}
    </div>
  );
}
```

### AgentRow
Step in the pipeline execution log with status indicator:
```jsx
function AgentRow({ step, pipeline, state }) {
  // state: "pending" (dimmed) | "running" (spinner + pulse) | "done" (checkmark)
  // Shows: icon, step title, agent badge, data block (when done)
}
```

---

## 9. QA Checklist

After building a page, verify every item:

### Content QA
- [ ] All agent names consistent across: PIPELINE.agent, PIPELINE.label, detail panel title, badge text, data tab labels
- [ ] PIPELINE.diff and PIPELINE.kpi are DIFFERENT strings for each stage
- [ ] PIPELINE.kpi strings are ≤25 characters (SVG overflow check)
- [ ] USE_CASES.steps.length === PIPELINE.length for every use case
- [ ] No typos in domain-specific terms (PSS names, acronyms, product names)
- [ ] Grammar: "APIs" not "API's", proper capitalization

### Visual QA (screenshot each section at 1440px viewport)
- [ ] **Logo**: transparent background, no artifacts, fits container
- [ ] **Header**: sticky, branding + breadcrumb + search + avatar
- [ ] **Hero**: title hierarchy, CTA buttons, logo container
- [ ] **KPI Strip**: 6 cards evenly distributed, no text wrapping
- [ ] **Architecture Diagram**: all nodes readable, KPI labels NO overlap, animated connectors, activation bar
- [ ] **Architecture Detail Panel**: appears on click, correct icon/name/description/badges
- [ ] **Demo Cards** (idle): both use cases render with correct badges
- [ ] **Demo Running**: progress bar, step-by-step execution, spinner animation
- [ ] **Demo Agent Flow tab**: all 8 agents with correct names + differentiator badges
- [ ] **Demo Experience tab**: push notification mock + offer card + CTA
- [ ] **Demo Data tab**: pipeline data flow with agent names + data blocks
- [ ] **Footer**: xPerion branding, product name, tagline, © year + company

### Interaction QA
- [ ] Click each architecture node → detail panel appears with correct content
- [ ] "Clear selection" button deselects node
- [ ] "Launch Demo" starts pipeline execution
- [ ] Progress bar fills correctly
- [ ] Tab switching works (Agent Flow / Experience / Data)
- [ ] "Claim Offer" / "Confirm Rebooking" CTA transitions to success state
- [ ] "Back to Use Cases" resets demo

---

## 10. Adapting for a New Product

To create a new xPerion landing page for a different product:

1. **Replace PIPELINE array** — define your product's stages/agents with unique ids, names, icons, colors, descriptions, differentiators, and KPIs
2. **Replace USE_CASES array** — define 2-3 demo scenarios with personas, triggers, offers, results, and per-stage step data
3. **Replace KPIS array** — 5-6 headline metrics
4. **Replace CHANNELS array** — activation/output channels relevant to the product
5. **Update hero content** — product name, tagline, description, logo
6. **Update header breadcrumb** — category and product name
7. **Update footer** — product name and tagline

**Do NOT change**: the design tokens, layout structure, component library, animation system, or interaction patterns — these are the xPerion design system and should remain consistent across all products.
