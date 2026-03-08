---
title: "Community Platform Clickthrough Demo"
type: feat
status: active
date: 2026-03-08
project: bootstrapped_berlin
brainstorm: docs/brainstorms/2026-03-08-community-platform-demo-brainstorm.md
---

## Enhancement Summary

**Deepened on:** 2026-03-08
**Agents used:** 9 (Security Sentinel, Code Simplicity, Frontend Races, Learnings Researcher, Performance Oracle, Architecture Strategist, Frontend Design, Pattern Recognition, CSS Extraction Research)

### Key Improvements

1. **CRITICAL FIX: i18n race condition** — `setLang()` fires in `shared.js` before `app.js` merges translations. Solution: defer language init to explicit `initLang()` call.
2. **Theme flash prevention** — Every HTML page needs a blocking `<script>` in `<head>` before any CSS loads to apply stored theme instantly.
3. **Performance** — Consolidate 3 separate `requestAnimationFrame` loops into 1 combined loop; pre-generate grain frames; replace cursor trail forced reflow with CSS class toggle.
4. **Security** — Whitelist-validate all URL params (`?id=`, `?with=`, `?mode=`); always use `textContent` never `innerHTML` for user-facing data.
5. **JS/CSS load order** — Use `defer` on both `shared.js` and `app.js` in `<head>` for parallel download + guaranteed execution order; wrap page init in `DOMContentLoaded`.
6. **CSS class naming convention** — Flat hyphenated pattern consistent with existing codebase (e.g., `.tab-bar`, `.wizard-step`, `.chat-bubble`).
7. **Missing i18n keys** — Added complete key sets for welcome, profile, discover, challenge, and chat pages.
8. **HTML deduplication** — Inject tab bar and header via JS (`app.js`) instead of duplicating markup across 7 files.
9. **Dead-end fix** — "Let's connect" on founders without chat data shows "Connection request sent" state instead of silent fallback.
10. **Cache busting** — Use `?v=1.0.0` query string on CSS/JS links; bump manually on changes.

### New Considerations Discovered

- `getComputedStyle` for `--accent` in canvas code may return wrong value if script runs before external CSS loads — wrap canvas init in `window.addEventListener('load', ...)`
- Font preloading with `crossorigin` attribute is mandatory even for self-hosted fonts
- The `challenge` field on founder objects is redundant with `tags[0]` — remove it, use `tags[0]` as primary challenge
- Chat `from: 'you'` magic string should be `isSelf: true` boolean for clarity
- Industry i18n keys should use shared namespace (`industry.saas`) not page-scoped (`apply.industry.saas`) since they're referenced from discover + challenge pages too

---

# Community Platform Clickthrough Demo

## Overview

Extend the existing `bootstrapped, berlin` landing page (`index.html`) with a clickable app demo showing the complete user flow — from application to anonymous chat. 6 new HTML pages in an `app/` folder, a shared CSS file extracted from the existing inline styles, and shared JS for theme/i18n/effects. Same design system (Dark Mode, Copper `#c45a3c`, JetBrains Mono/Lora/Inter), all effects (cursor glow, grain, parallax), full i18n (EN/DE). All dummy data, no backend.

## Problem Statement / Motivation

The landing page communicates the *what* of bootstrapped, berlin — but potential members can't experience the *how*. A clickthrough demo lets founders feel the platform's UX before committing to the waitlist. It also serves as a design validation tool and a conversation piece for stakeholder feedback.

## Proposed Solution

Multi-page static HTML demo with extracted shared CSS/JS. Each screen is a standalone `.html` file linked via standard `<a href>` navigation. No build step, no framework, consistent with the existing architecture.

---

## Architecture

### Current State

```
bootstrapped_berlin/
├── index.html          # 1605 lines (CSS: 11-871, HTML: 872-1038, JS: 1039-1602)
├── CLAUDE.md
└── docs/
```

### Target State

```
bootstrapped_berlin/
├── index.html          # Landing page (refactored to use shared.css + shared.js)
├── shared.css          # Extracted design system (~860 lines)
├── shared.js           # Extracted theme/i18n/effects logic (~400 lines)
├── app/
│   ├── app.css         # App-specific styles (tab bar, forms, cards, wizard)
│   ├── app.js          # App-specific JS (wizard, filters, chat, dummy data)
│   ├── apply.html      # 3-step application wizard
│   ├── welcome.html    # Acceptance confirmation
│   ├── profile.html    # Profile setup (onboarding) / view (tab bar)
│   ├── discover.html   # Challenge-based matching (filterable list)
│   ├── challenge.html  # Challenge detail + "Let's connect"
│   └── chat.html       # Chat list + inline conversation view
├── CLAUDE.md
└── docs/
```

### CSS/JS Extraction Strategy

**`shared.css`** — Extract from `index.html` lines 11-871:
- CSS reset & base styles
- CSS custom properties (`:root` dark + `html[data-theme="light"]`)
- Font declarations (mono, sans, serif variables)
- Cursor glow & cursor trail styles
- Grain overlay styles
- Header (glass pill) styles
- Reveal animation styles
- Section atmosphere styles
- Parallax headline styles
- Responsive breakpoints (`@media max-width: 600px`)
- Theme transition timing (`5s cubic-bezier`)

**Do NOT extract** to shared.css:
- Landing-page-specific section styles (hero, manifesto, value grid, audience, CTA form, footer)
- These stay as a `<style>` block in `index.html` or in a separate `landing.css`

**`shared.js`** — Extract from `index.html` lines 1039-1602:
- `applyTheme()` function + localStorage (`bb_theme`)
- `setLang()` function + `translations` object + localStorage (`bb_lang`)
- Cursor glow initialization (lerp, mobile sine-wave)
- Cursor trail pool
- Grain canvas animation
- Scroll reveal (IntersectionObserver)
- Parallax headlines
- Section atmosphere IntersectionObserver
- Header scroll glass effect

**Do NOT extract** to shared.js:
- Magnetic typography (hero-specific — only on index.html)
- Arrow particles (hero-specific)
- Logo canvas (keep in shared.js — logo appears on all pages)
- Waitlist form handler (landing-page-specific)
- `wrapChars` / `setLang` wrapper for magnetic hero (landing-page-specific)

**`app/app.css`** — New styles for app screens:
- Bottom tab bar
- Top bar (simplified header for app screens)
- Form elements (inputs, selects, multi-select tags, buttons)
- Wizard progress indicator
- Card components (discover cards, challenge detail)
- Chat UI (message bubbles, input bar)
- Profile view styles

**`app/app.js`** — New JS for app screens:
- Wizard step logic (show/hide via hash fragments)
- Discover filter logic
- Chat dummy data + message rendering
- Dummy founder data (JSON object)
- i18n translations for all app screens (extends `translations` object)

### Linking Strategy

**Every HTML page `<head>` — blocking theme init (prevents flash):**
```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>bootstrapped, berlin</title>

  <!-- CRITICAL: Blocking theme script BEFORE any CSS to prevent flash -->
  <script>
    (function() {
      try {
        var t = localStorage.getItem('bb_theme');
        if (t) document.documentElement.setAttribute('data-theme', t);
      } catch(e) {}
    })();
  </script>

  <!-- Font preloading (crossorigin mandatory even for self-hosted) -->
  <link rel="preload" href="../fonts/JetBrainsMono.woff2" as="font" type="font/woff2" crossorigin>

  <!-- Shared CSS (render-blocking = prevents FOUC) -->
  <link rel="stylesheet" href="../shared.css?v=1.0.0">
  <!-- Page/app-specific CSS after shared (source order wins on equal specificity) -->
  <link rel="stylesheet" href="app.css?v=1.0.0">
</head>
```

**Every HTML page `<body>` end — script loading:**
```html
  <!-- defer = parallel download, executes after DOM parsed, preserves order -->
  <script src="../shared.js?v=1.0.0" defer></script>
  <script src="app.js?v=1.0.0" defer></script>

  <!-- Page-specific init (DOMContentLoaded fires AFTER all deferred scripts) -->
  <script>
    document.addEventListener('DOMContentLoaded', function() {
      // Merge app translations, THEN init language
      if (window.mergeAppTranslations) window.mergeAppTranslations();
      if (window.initLang) window.initLang();
      // Page-specific initialization here
    });
  </script>
</body>
```

**index.html uses same pattern** but with `shared.css` / `shared.js` paths (no `../` prefix) and landing-page-specific inline styles/scripts.

**Cache busting:** Bump `?v=` query string manually when files change. No build step needed.

---

## User Flow (Detailed)

```
index.html ──[CTA Button]──► app/apply.html#step1

app/apply.html
  #step1 ──[Next]──► #step2
  #step2 ──[Back]──► #step1
  #step2 ──[Next]──► #step3
  #step3 ──[Back]──► #step2
  #step3 ──[Submit]──► #success
  #success ──[Continue]──► app/welcome.html

app/welcome.html ──[Get Started]──► app/profile.html?mode=setup

app/profile.html?mode=setup ──[Continue to Discover]──► app/discover.html

app/discover.html ──[Click card]──► app/challenge.html?id=mika (etc.)

app/challenge.html?id=X ──[Let's connect]──► app/chat.html?with=X

app/chat.html ──[Tab: Discover]──► app/discover.html
              ──[Tab: Profile]──► app/profile.html
```

### Navigation Rules

| Context | Top Bar | Bottom Tab Bar |
|---------|---------|----------------|
| Onboarding (apply, welcome, profile?mode=setup) | Logo (→ index.html) + Theme/Lang toggles | **None** — linear flow only |
| App (discover, challenge, chat, profile) | Logo (→ discover.html) + Theme/Lang toggles | Discover / Chats / Profile |

### State Management

**Decision: No gates, all pages freely accessible.** This is a clickthrough demo, not a real app. Any page can be accessed directly via URL. No localStorage state machine needed for routing.

**What IS persisted in localStorage:**
- `bb_theme` — dark/light preference (existing)
- `bb_lang` — en/de preference (existing)
- Nothing else — no onboarding completion state

---

## Critical Implementation Notes (from Research)

### 1. i18n Race Condition Fix

**Problem:** `shared.js` currently calls `setLang()` at load time (auto-detect IIFE at line 1145). On app pages, `app.js` hasn't merged its translations yet — all app-specific keys render as empty strings on first load.

**Fix:** Split `setLang` into two functions in `shared.js`:
```javascript
// shared.js — PUBLIC API
function setLang(lang) {
  // Updates DOM, stores to localStorage — called on user toggle
  document.querySelectorAll('[data-i18n]').forEach(function(el) { ... });
  try { localStorage.setItem('bb_lang', lang); } catch(e) {}
  document.documentElement.setAttribute('data-lang', lang);
  langChangeCallbacks.forEach(function(fn) { fn(lang); });
}

function initLang() {
  // Called ONCE after all translations are merged
  var stored = localStorage.getItem('bb_lang');
  var lang = stored || (navigator.language.startsWith('de') ? 'de' : 'en');
  setLang(lang);
}

// Language change callback system (replaces monkey-patching)
var langChangeCallbacks = [];
function onLangChange(fn) { langChangeCallbacks.push(fn); }

// Do NOT auto-call setLang() on load — pages call initLang() explicitly
```

Each page's `DOMContentLoaded` handler calls `mergeAppTranslations()` then `initLang()`.

### 2. HTML Deduplication via JS Injection

**Problem:** Header HTML duplicated in 7 files, tab bar in 4 files. Any change requires editing every file.

**Fix:** `app.js` injects header and tab bar from JS:
```javascript
function renderAppHeader(logoTarget) {
  var header = document.createElement('header');
  // ... build header HTML with logo, theme toggle, lang switch
  document.body.prepend(header);
}

function renderTabBar(activeTab) {
  var nav = document.createElement('nav');
  nav.className = 'tab-bar';
  nav.setAttribute('role', 'navigation');
  nav.setAttribute('aria-label', 'Main');
  // ... build 3 tabs, mark active
  document.body.appendChild(nav);
}
```

Each page sets `<body data-active-tab="discover" data-header-target="discover.html">` and calls `renderAppHeader()` + `renderTabBar()`.

### 3. CSS Class Naming Convention

Consistent with existing codebase — flat, hyphenated, section-scoped:

```
Tab bar:    .tab-bar, .tab-item, .tab-icon, .tab-label, .tab-item.active
Top bar:    .app-header, .app-logo (reusing .header-inner pattern)
Wizard:     .wizard-progress, .wizard-step, .wizard-step.active, .wizard-step.completed
Forms:      .form-group, .form-input, .form-select, .form-textarea, .form-label
Tags:       .tag-pill, .tag-pill.selected
Cards:      .discover-card, .discover-tag, .discover-quote
Chat:       .chat-list, .chat-preview, .chat-bubble, .chat-bubble.self, .chat-input-bar
Profile:    .profile-form, .profile-tags
Filter:     .filter-bar, .filter-pill, .filter-pill.active
```

**Scoping rule:** Always scope `.active` with parent (`.tab-item.active`, `.filter-pill.active`) to avoid style leaks.

### 4. Security: URL Parameter Validation

All pages that read query params must whitelist-validate:
```javascript
// app.js — shared utility
function getParam(name, allowed) {
  var val = new URLSearchParams(window.location.search).get(name);
  if (!val) return null;
  if (allowed && allowed.indexOf(val) === -1) return null;
  return val;
}

// Usage:
var founderId = getParam('id', ['mika', 'lena', 'jonas', 'ayla', 'tom', 'priya']);
var chatWith = getParam('with', ['mika', 'lena', 'jonas', 'ayla', 'tom', 'priya']);
var mode = getParam('mode', ['setup']);
```

**Always use `textContent`** (never `innerHTML`) when rendering founder data into the DOM.

### 5. Performance: Single Animation Loop

Consolidate grain, cursor glow, and logo canvas into one `requestAnimationFrame` loop:
```javascript
// shared.js
var animationCallbacks = [];
function registerAnimation(fn) { animationCallbacks.push(fn); }

(function loop(ts) {
  for (var i = 0; i < animationCallbacks.length; i++) {
    animationCallbacks[i](ts);
  }
  requestAnimationFrame(loop);
})(0);
```

Each effect registers itself via `registerAnimation()`. The loop pauses when the tab is hidden (browser does this automatically with rAF).

### 6. Shared Data: Industry & Challenge Tags

Define once in `app.js`, reference from all pages:
```javascript
// Industry and challenge tag IDs — i18n keys use shared namespace
var INDUSTRIES = ['saas', 'ecommerce', 'marketplace', 'fintech', 'healthtech', 'edtech', 'devtools', 'agency', 'hardware', 'other'];
var CHALLENGE_TAGS = ['first-revenue', 'customer-acquisition', 'pricing', 'product-market-fit', 'first-hire', 'cashflow', 'marketing', 'sales', 'retention', 'scaling', 'tech', 'legal', 'work-life-balance', 'bootstrapping-vs-funding', 'go-to-market', 'partnerships'];
```

i18n keys use shared namespace: `industry.saas`, `tag.first-revenue` (not `apply.industry.saas`).

---

## Implementation Phases

### Phase 1: CSS/JS Extraction & Refactor

**Goal:** Extract shared styles and scripts from `index.html` into external files without any visual or behavioral change to the landing page.

#### Tasks

- [ ] Create `shared.css` with all shared design system styles from `index.html` (lines 11-871, excluding landing-page-specific selectors)
- [ ] Create `shared.js` with theme, i18n, cursor effects, grain, parallax, scroll reveal, header glass, logo canvas logic
- [ ] Keep landing-page-specific CSS (hero, manifesto, value, audience, CTA, footer) inline in `index.html` (or in a separate `landing.css`)
- [ ] Keep landing-page-specific JS (magnetic typography, arrow particles, setLang wrapper, waitlist form) inline in `index.html`
- [ ] Update `index.html` to link to `shared.css` and `shared.js`
- [ ] **Verify:** `index.html` renders identically in both themes, both languages, desktop and mobile
- [ ] **Verify:** All effects still work (cursor glow, trail, grain, parallax, magnetic type, arrow particles, header glass)

**Critical: The `translations` object must be extensible.** `shared.js` should initialize `window.translations = { en: {...}, de: {...} }` and app pages should merge additional keys into it via `Object.assign(window.translations.en, { ...appTranslations.en })`.

#### Risks
- Magnetic typography depends on `setLang` wrapper chain — must not break during extraction
- CSS specificity may change when moving from inline `<style>` to external file (test thoroughly)
- `getComputedStyle` calls for `--accent` in logo canvas must still work with external CSS

#### Research Insights (Phase 1)

**CSS extraction is safe — no FOUC risk:** External `<link rel="stylesheet">` in `<head>` is render-blocking by design. The browser won't paint until the stylesheet loads. Source order determines cascade for equal-specificity selectors — shared styles load first, page overrides second (inline `<style>`), preserving current behavior.

**Canvas timing fix:** `getComputedStyle()` may return wrong values if script runs before external CSS loads. Wrap canvas init (logo, grain) in `window.addEventListener('load', ...)` — not `DOMContentLoaded`.

**Script loading:** Use `defer` on both `shared.js` and `app.js` in `<head>`. Deferred scripts download in parallel with HTML parsing, execute in document order after DOM is parsed, and all complete before `DOMContentLoaded` fires. This is faster than scripts at end of `<body>`.

**Cache busting:** Use `?v=1.0.0` query strings. Bump manually when files change. Set long cache headers on `.css`/`.js`, short on `.html`.

**Font preloading:** Add `<link rel="preload" as="font" type="font/woff2" crossorigin>` for the 1-2 most critical fonts. The `crossorigin` attribute is mandatory even for self-hosted fonts. Use `font-display: swap` in `@font-face` to avoid FOIT.

**CSS custom properties work identically in external files.** Always use `getComputedStyle()` (not `element.style`) to read them from JS — `element.style.getPropertyValue()` returns empty for externally-defined variables.

### Phase 2: Shared App Components

**Goal:** Build reusable HTML/CSS/JS patterns used across multiple app pages.

#### Tasks

- [ ] **Bottom Tab Bar** (`app.css` + `app.js`)
  - 3 tabs: Discover, Chats, Profile
  - Icons (simple SVG inline) + text labels
  - Full viewport width, fixed at bottom
  - `safe-area-inset-bottom` padding for iOS
  - Active tab: Copper `#c45a3c` icon + label; Inactive: `var(--text-muted)`
  - Min height: 56px (48px tap target + 8px padding)
  - Highlight current page based on `data-active-tab` attribute on `<body>`

- [ ] **App Top Bar** (`app.css`)
  - Simplified version of landing page header
  - Logo (canvas noise sphere) + "bootstrapped" text
  - Theme toggle + Language switch
  - Same glass-on-scroll behavior
  - Logo links to `discover.html` (in app) or `index.html` (in onboarding)

- [ ] **Form Elements** (`app.css`)
  - Text inputs: dark bg, copper border on focus, `var(--sans)` font
  - Select dropdowns: same style, custom arrow
  - Multi-select tags: clickable pills, copper bg when selected, `var(--mono)` font
  - Textareas: same as inputs, resizable
  - Buttons: primary (copper bg, dark text), secondary (outlined)
  - All inputs support dark/light theme via CSS variables
  - `type="email"` on email fields for mobile keyboard

- [ ] **Wizard Progress Indicator** (`app.css`)
  - 3 numbered step indicators with connecting lines (not dots — numbers give stronger feedback)
  - Active step: copper outline + bold label, `aria-current="step"`
  - Completed step: copper filled + checkmark, label visible
  - Pending step: muted/gray, no interaction
  - Step labels below indicators: "Basics", "Business", "Motivation"
  - On mobile (< 400px): compact "Schritt 2 von 3" text + thin progress bar
  - Future steps NOT clickable (prevents skipping validation)
  - Responsive: works at 320px minimum

- [ ] **Card Component** (`app.css`)
  - Used in discover list
  - Challenge tag as header (mono font, copper)
  - Founder name + industry below
  - Subtle border, hover effect (slight lift)
  - Dark/light theme support

#### i18n Keys (Tab Bar + Top Bar)

```javascript
// app/app.js - merged into window.translations
const appTranslations = {
  en: {
    'tab.discover': 'Discover',
    'tab.chats': 'Chats',
    'tab.profile': 'Profile',
  },
  de: {
    'tab.discover': 'Entdecken',
    'tab.chats': 'Chats',
    'tab.profile': 'Profil',
  }
};
```

#### Research Insights (Phase 2)

**Tab bar implementation:**
- Use `position: fixed` (not `sticky`) — must be visible regardless of scroll position
- Height: `calc(56px + env(safe-area-inset-bottom, 0px))` with `padding-bottom: env(safe-area-inset-bottom, 0px)`
- Requires `<meta name="viewport" content="..., viewport-fit=cover">` for `env()` to work
- Active state: 3 differentiators — copper color + filled icon variant + bold label. Never rely on color alone.
- Labels always visible below icons — "Discover" is not universally iconifiable
- ARIA: Use `role="tablist"` + `role="tab"` pattern since this is SPA-style view switching. Since this is actually multi-page, use `<nav>` with `<a>` links instead and `aria-current="page"` on the active link.
- Hide tab bar when mobile keyboard is open: use Visual Viewport API (`window.visualViewport.height` vs `window.innerHeight`, threshold ~150px)

**Tab bar as `<nav>` (multi-page correction):**
Since each tab navigates to a different `.html` file, this is NOT an SPA tab pattern. Use semantic `<nav>` with links:
```html
<nav class="tab-bar" aria-label="Main navigation">
  <a href="discover.html" class="tab-item active" aria-current="page">
    <svg class="tab-icon" aria-hidden="true">...</svg>
    <span class="tab-label" data-i18n="tab.discover">Discover</span>
  </a>
  <!-- ... -->
</nav>
```

**Card hover:** Use `transform: translateY(-2px)` + subtle `box-shadow` increase on hover. Add `@media (hover: hover)` guard so touch devices don't get stuck hover states.

**Form inputs:** Use `caret-color: var(--accent)` for copper cursor. Focus ring: `outline: 2px solid var(--accent); outline-offset: 2px`.

### Phase 3: Onboarding Flow (apply, welcome, profile)

#### 3a: apply.html — Application Wizard

**URL scheme:** `app/apply.html#step1`, `#step2`, `#step3`, `#success`

**Layout:**
- App top bar (logo → index.html, theme/lang toggles)
- No bottom tab bar
- Wizard progress indicator (top of content area)
- Form content area (720px max-width container)
- Back/Next buttons at bottom of each step

**Step 1: Basics**
```
[Progress: ● ─ ○ ─ ○]
         Basics

First name              [________________]
Email                   [________________]

                              [Next →]
```

**Step 2: Business**
```
[Progress: ● ─ ● ─ ○]
        Business

Your product / company  [________________]
Industry               [▼ Select...      ]
Monthly revenue        [▼ Select...      ]

                   [← Back]    [Next →]
```

**Step 3: Motivation**
```
[Progress: ● ─ ● ─ ●]
       Motivation

Why this community?     [                ]
                        [________________]

Your biggest challenge  [                ]
right now               [________________]

                   [← Back]   [Submit →]
```

**Success State (inline, replaces form):**
```
[Progress: ● ─ ● ─ ●]

    ✓ Application received.

    We'll review your application and
    get back to you soon.

    (For this demo, you're already in.)

                        [Continue →]
```

**Wizard Implementation:**
- Steps are `<section>` elements with `role="group"` and `aria-labelledby`, shown/hidden via `hidden` attribute (not CSS `display: none`)
- Hash navigation: `location.hash` → `hashchange` event listener (use `addEventListener`, not inline)
- Hash validation: whitelist `['step1', 'step2', 'step3', 'success']` — invalid hashes default to `step1`
- On step transition: move focus to the step heading (`h2` with `tabindex="-1"`) and announce via `aria-live="polite"` region
- Browser back button navigates between steps (works automatically with hash changes)
- Form data stored in a JS object (not persisted — demo only)
- "Submit" triggers a brief animation (300ms crossfade) then shows success state
- "Continue" navigates to `welcome.html`
- Step transition animation: CSS crossfade (200-300ms) with `prefers-reduced-motion` guard. Consider View Transitions API with instant-switch fallback for modern browsers.
- Mobile keyboard: `scrollIntoView({ behavior: 'smooth', block: 'center' })` on input focus (300ms delay for keyboard animation)

**Validation (demo-level, "reward early, punish late" pattern):**
- Validate on "Next" click — show inline errors below fields, block navigation if invalid
- Clear errors on `input` event as user corrects (reward early)
- Do NOT validate on blur for the first pass (avoids premature errors while tabbing through)
- Do NOT disable the "Next" button — let users click and see what's wrong
- Email: basic format check (`@` + `.`)
- Error markup: `<span class="field-error" role="alert" hidden>` with `aria-describedby` on the input
- On field error: `aria-invalid="true"` + copper/red border (`var(--accent)`)
- "Back" button NEVER validates — always allow going back

**Industry Dropdown Options (EN/DE):**

| EN | DE |
|---|---|
| SaaS / Software | SaaS / Software |
| E-Commerce | E-Commerce |
| Marketplace | Marktplatz |
| FinTech | FinTech |
| HealthTech | HealthTech |
| EdTech | EdTech |
| Developer Tools | Developer Tools |
| Agency / Services | Agentur / Dienstleistung |
| Hardware / IoT | Hardware / IoT |
| Other | Sonstiges |

**Revenue Range Options (same in EN/DE):**
- Pre-Revenue
- 0-10k EUR/mo
- 10-50k EUR/mo
- 50-100k EUR/mo
- 100k+ EUR/mo

#### i18n Keys (apply.html)

```javascript
{
  en: {
    'apply.progress.1': 'Basics',
    'apply.progress.2': 'Business',
    'apply.progress.3': 'Motivation',
    'apply.step1.title': 'Tell us about yourself',
    'apply.field.name': 'First name',
    'apply.field.email': 'Email',
    'apply.step2.title': 'Your business',
    'apply.field.product': 'Your product / company',
    'apply.field.industry': 'Industry',
    'apply.field.revenue': 'Monthly revenue',
    'apply.step3.title': 'Your motivation',
    'apply.field.why': 'Why this community?',
    'apply.field.challenge': 'Your biggest challenge right now',
    'apply.btn.next': 'Next',
    'apply.btn.back': 'Back',
    'apply.btn.submit': 'Submit',
    'apply.success.title': 'Application received.',
    'apply.success.body': "We'll review your application and get back to you soon.",
    'apply.success.demo': "(For this demo, you're already in.)",
    'apply.btn.continue': 'Continue',
    'apply.industry.saas': 'SaaS / Software',
    'apply.industry.ecommerce': 'E-Commerce',
    'apply.industry.marketplace': 'Marketplace',
    'apply.industry.fintech': 'FinTech',
    'apply.industry.healthtech': 'HealthTech',
    'apply.industry.edtech': 'EdTech',
    'apply.industry.devtools': 'Developer Tools',
    'apply.industry.agency': 'Agency / Services',
    'apply.industry.hardware': 'Hardware / IoT',
    'apply.industry.other': 'Other',
  },
  de: {
    'apply.progress.1': 'Basics',
    'apply.progress.2': 'Business',
    'apply.progress.3': 'Motivation',
    'apply.step1.title': 'Erzähl uns von dir',
    'apply.field.name': 'Vorname',
    'apply.field.email': 'E-Mail',
    'apply.step2.title': 'Dein Business',
    'apply.field.product': 'Dein Produkt / Unternehmen',
    'apply.field.industry': 'Branche',
    'apply.field.revenue': 'Monatlicher Umsatz',
    'apply.step3.title': 'Deine Motivation',
    'apply.field.why': 'Warum diese Community?',
    'apply.field.challenge': 'Deine größte Herausforderung gerade',
    'apply.btn.next': 'Weiter',
    'apply.btn.back': 'Zurück',
    'apply.btn.submit': 'Absenden',
    'apply.success.title': 'Bewerbung eingegangen.',
    'apply.success.body': 'Wir prüfen deine Bewerbung und melden uns bald.',
    'apply.success.demo': '(In dieser Demo bist du direkt drin.)',
    'apply.btn.continue': 'Weiter',
    'apply.industry.saas': 'SaaS / Software',
    'apply.industry.ecommerce': 'E-Commerce',
    'apply.industry.marketplace': 'Marktplatz',
    'apply.industry.fintech': 'FinTech',
    'apply.industry.healthtech': 'HealthTech',
    'apply.industry.edtech': 'EdTech',
    'apply.industry.devtools': 'Developer Tools',
    'apply.industry.agency': 'Agentur / Dienstleistung',
    'apply.industry.hardware': 'Hardware / IoT',
    'apply.industry.other': 'Sonstiges',
  }
}
```

#### 3b: welcome.html — Acceptance Screen

**Layout:**
- App top bar (logo → index.html)
- No bottom tab bar
- Centered content, generous whitespace

**Content:**
```
    Welcome to bootstrapped, berlin.

    You're in. Here's what happens next:

    1. Set up your profile
    2. Discover founders facing similar challenges
    3. Connect over coffee

    No small talk. No pitching. Just founders
    helping founders.

                    [Get Started →]
```

"Get Started" links to `profile.html?mode=setup`.

#### 3c: profile.html — Profile Setup / View

**Dual-mode page:**
- **Setup mode** (`?mode=setup`): Part of onboarding, no tab bar, "Continue to Discover" button
- **View mode** (no param / accessed via tab bar): Bottom tab bar visible, "Save" button (non-functional in demo), edit fields pre-filled with dummy data

**Layout (Setup Mode):**
```
[Top Bar: Logo → index.html]

    Set up your profile

    First name           [________________]

    Industry             [▼ Select...      ]

    What are you working on?
    [Tag pills - multi-select from challenge tags list]
    ┌─────────────┐ ┌──────────────────┐ ┌─────────────┐
    │ First Revenue│ │ Customer Acquis. │ │ Pricing     │
    └─────────────┘ └──────────────────┘ └─────────────┘
    ┌──────────────┐ ┌─────────────┐ ┌─────────────────┐
    │ Product-Market│ │ First Hire  │ │ Cashflow        │
    └──────────────┘ └─────────────┘ └─────────────────┘
    ... (all 16 tags)

    Select the challenges you're currently facing.
    Other founders will find you through these.

                    [Continue to Discover →]
```

**Layout (View Mode):**
```
[Top Bar: Logo → discover.html]

    Your profile

    [Same form fields, pre-filled]

                        [Save]

[Tab Bar: Discover | Chats | ● Profile]
```

**Challenge Tags (Multi-Select):**
- All 16 tags displayed as pills
- Click to toggle selection (copper bg when selected)
- At least 1 tag required in setup mode
- Tags use `data-i18n` attributes for language switching

#### Missing i18n Keys (welcome, profile)

```javascript
{
  en: {
    // welcome.html
    'welcome.title': 'Welcome to bootstrapped, berlin.',
    'welcome.subtitle': "You're in. Here's what happens next:",
    'welcome.step1': 'Set up your profile',
    'welcome.step2': 'Discover founders facing similar challenges',
    'welcome.step3': 'Connect over coffee',
    'welcome.tagline': 'No small talk. No pitching. Just founders helping founders.',
    'welcome.btn': 'Get Started',

    // profile.html
    'profile.title.setup': 'Set up your profile',
    'profile.title.view': 'Your profile',
    'profile.field.name': 'First name',
    'profile.field.industry': 'Industry',
    'profile.tags.label': 'What are you working on?',
    'profile.tags.hint': 'Select the challenges you\'re currently facing. Other founders will find you through these.',
    'profile.btn.continue': 'Continue to Discover',
    'profile.btn.save': 'Save',

    // Shared industry labels (used on apply, profile, discover, challenge pages)
    'industry.saas': 'SaaS / Software',
    'industry.ecommerce': 'E-Commerce',
    'industry.marketplace': 'Marketplace',
    'industry.fintech': 'FinTech',
    'industry.healthtech': 'HealthTech',
    'industry.edtech': 'EdTech',
    'industry.devtools': 'Developer Tools',
    'industry.agency': 'Agency / Services',
    'industry.hardware': 'Hardware / IoT',
    'industry.other': 'Other',

    // Shared challenge tag labels
    'tag.first-revenue': 'First Revenue',
    'tag.customer-acquisition': 'Customer Acquisition',
    'tag.pricing': 'Pricing & Monetization',
    'tag.product-market-fit': 'Product-Market Fit',
    'tag.first-hire': 'First Hire / Team',
    'tag.cashflow': 'Cashflow / Runway',
    'tag.marketing': 'Marketing & Branding',
    'tag.sales': 'Sales Process',
    'tag.retention': 'Retention / Churn',
    'tag.scaling': 'Scaling Operations',
    'tag.tech': 'Tech / Infrastructure',
    'tag.legal': 'Legal & Compliance',
    'tag.work-life-balance': 'Work-Life Balance',
    'tag.bootstrapping-vs-funding': 'Bootstrapping vs. Funding',
    'tag.go-to-market': 'Go-to-Market',
    'tag.partnerships': 'Partnerships',
  },
  de: {
    'welcome.title': 'Willkommen bei bootstrapped, berlin.',
    'welcome.subtitle': 'Du bist drin. So geht es weiter:',
    'welcome.step1': 'Richte dein Profil ein',
    'welcome.step2': 'Entdecke Gründer mit ähnlichen Herausforderungen',
    'welcome.step3': 'Vernetzt euch beim Kaffee',
    'welcome.tagline': 'Kein Smalltalk. Kein Pitching. Nur Gründer, die Gründern helfen.',
    'welcome.btn': 'Los geht\'s',

    'profile.title.setup': 'Richte dein Profil ein',
    'profile.title.view': 'Dein Profil',
    'profile.field.name': 'Vorname',
    'profile.field.industry': 'Branche',
    'profile.tags.label': 'Woran arbeitest du?',
    'profile.tags.hint': 'Wähle die Herausforderungen, die dich gerade beschäftigen. Andere Gründer finden dich darüber.',
    'profile.btn.continue': 'Weiter zum Entdecken',
    'profile.btn.save': 'Speichern',

    'industry.saas': 'SaaS / Software',
    'industry.ecommerce': 'E-Commerce',
    'industry.marketplace': 'Marktplatz',
    'industry.fintech': 'FinTech',
    'industry.healthtech': 'HealthTech',
    'industry.edtech': 'EdTech',
    'industry.devtools': 'Developer Tools',
    'industry.agency': 'Agentur / Dienstleistung',
    'industry.hardware': 'Hardware / IoT',
    'industry.other': 'Sonstiges',

    'tag.first-revenue': 'Erster Umsatz',
    'tag.customer-acquisition': 'Kundengewinnung',
    'tag.pricing': 'Pricing & Monetarisierung',
    'tag.product-market-fit': 'Product-Market Fit',
    'tag.first-hire': 'Erste Einstellung / Team',
    'tag.cashflow': 'Cashflow / Runway',
    'tag.marketing': 'Marketing & Branding',
    'tag.sales': 'Vertriebsprozess',
    'tag.retention': 'Kundenbindung / Churn',
    'tag.scaling': 'Skalierung',
    'tag.tech': 'Tech / Infrastruktur',
    'tag.legal': 'Recht & Compliance',
    'tag.work-life-balance': 'Work-Life Balance',
    'tag.bootstrapping-vs-funding': 'Bootstrapping vs. Funding',
    'tag.go-to-market': 'Go-to-Market',
    'tag.partnerships': 'Partnerschaften',
  }
}
```

**Note:** The `apply.industry.*` keys in Phase 3a should be updated to use the shared `industry.*` namespace instead.

### Phase 4: App Screens (discover, challenge, chat)

#### 4a: discover.html — Challenge-Based Discovery

**Layout:**
```
[Top Bar: Logo → discover.html | Theme | Lang]

    Discover

    [Filter pills - horizontal scroll]
    [All] [First Revenue] [Pricing] [Hiring] [...]

    ┌─────────────────────────────────────────┐
    │  💬 Customer Acquisition                │
    │  Ayla · HealthTech                      │
    │  "How do I find my first 100 users      │
    │   without a marketing budget?"          │
    └─────────────────────────────────────────┘

    ┌─────────────────────────────────────────┐
    │  💬 Pricing & Monetization              │
    │  Jonas · Developer Tools                │
    │  "Freemium or paid from day one?        │
    │   I keep going back and forth."         │
    └─────────────────────────────────────────┘

    ... (6 cards total)

    ┌─────────────────────────────────────────┐
    │  No founders match your filters.        │
    │  [Clear filters]                        │
    └─────────────────────────────────────────┘
    (shown when filter results are empty)

[Tab Bar: ● Discover | Chats | Profile]
```

**Filter Logic:**
- "All" shows all 6 founders (default)
- Tag pills filter by challenge tag
- Multiple tags: show founders matching ANY selected tag (OR logic)
- Empty results: "No founders match your filters." + "Clear filters" link
- Filter state: not persisted (resets on page reload)

**Cards:**
- Click entire card → navigate to `challenge.html?id=<founder-id>`
- Show: primary challenge tag (mono, copper), name + industry, short quote
- Hover: subtle lift + border glow

#### Dummy Founder Data

```javascript
const founders = [
  {
    id: 'mika',
    name: 'Mika',
    industry: 'saas',
    tags: ['retention', 'pricing'],
    challenge: 'retention',
    quote: {
      en: "My MRR looks good but churn is killing me. 8% monthly — I need to fix this before I scale.",
      de: "Mein MRR sieht gut aus, aber der Churn bringt mich um. 8% monatlich — das muss ich lösen, bevor ich skaliere."
    },
    revenue: '10-50k'
  },
  {
    id: 'lena',
    name: 'Lena',
    industry: 'ecommerce',
    tags: ['first-hire', 'scaling'],
    challenge: 'first-hire',
    quote: {
      en: "I'm drowning in operations. Need to hire but terrified of making the wrong first hire.",
      de: "Ich versinke im Tagesgeschäft. Muss einstellen, aber hab Angst vor der falschen ersten Einstellung."
    },
    revenue: '10-50k'
  },
  {
    id: 'jonas',
    name: 'Jonas',
    industry: 'devtools',
    tags: ['pricing', 'go-to-market'],
    challenge: 'pricing',
    quote: {
      en: "Freemium or paid from day one? I keep going back and forth.",
      de: "Freemium oder direkt Paid? Ich schwanke ständig hin und her."
    },
    revenue: '0-10k'
  },
  {
    id: 'ayla',
    name: 'Ayla',
    industry: 'healthtech',
    tags: ['customer-acquisition', 'marketing'],
    challenge: 'customer-acquisition',
    quote: {
      en: "How do I find my first 100 users without a marketing budget?",
      de: "Wie finde ich meine ersten 100 Nutzer ohne Marketingbudget?"
    },
    revenue: '0-10k'
  },
  {
    id: 'tom',
    name: 'Tom',
    industry: 'marketplace',
    tags: ['product-market-fit', 'first-revenue'],
    challenge: 'product-market-fit',
    quote: {
      en: "Chicken-and-egg problem. No sellers without buyers, no buyers without sellers.",
      de: "Henne-Ei-Problem. Keine Verkäufer ohne Käufer, keine Käufer ohne Verkäufer."
    },
    revenue: 'pre-revenue'
  },
  {
    id: 'priya',
    name: 'Priya',
    industry: 'fintech',
    tags: ['legal', 'cashflow'],
    challenge: 'legal',
    quote: {
      en: "BaFin compliance is a nightmare. Every step forward feels like two steps back.",
      de: "BaFin-Compliance ist ein Albtraum. Jeder Schritt vorwärts fühlt sich an wie zwei zurück."
    },
    revenue: '50-100k'
  }
];
```

#### 4b: challenge.html — Challenge Detail

**URL:** `challenge.html?id=mika` (reads founder ID from query param)

**Layout:**
```
[Top Bar: Logo → discover.html | Theme | Lang]

    [← Back to Discover]

    ┌─────────────────────────────────────────┐
    │                                         │
    │  💬 Retention / Churn                   │
    │                                         │
    │  Mika · SaaS                            │
    │  Revenue: 10-50k EUR/mo                 │
    │                                         │
    │  "My MRR looks good but churn is        │
    │   killing me. 8% monthly — I need       │
    │   to fix this before I scale."          │
    │                                         │
    │  Also working on:                       │
    │  [Pricing] [Retention]                  │
    │                                         │
    │         [Let's connect →]               │
    │                                         │
    └─────────────────────────────────────────┘

[Tab Bar: ● Discover | Chats | Profile]
```

**"Let's connect" button:**
- Navigates to `chat.html?with=<founder-id>`
- Shows a pre-populated dummy conversation with that founder

**"Back to Discover" link:**
- Simple `<a href="discover.html">` (filter state resets — acceptable for demo)

#### 4c: chat.html — Chat Interface

**Two views within one page:**
1. **Chat List** (default, no `?with=` param)
2. **Conversation View** (`?with=jonas`)

**Chat List Layout:**
```
[Top Bar: Logo → discover.html | Theme | Lang]

    Chats

    ┌─────────────────────────────────────────┐
    │  Jonas · Developer Tools                │
    │  "Klar! Nächste Woche? Ich bin..."     │
    │  2 hours ago                            │
    └─────────────────────────────────────────┘

    ┌─────────────────────────────────────────┐
    │  Ayla · HealthTech                      │
    │  "Ja, lass uns das machen!"            │
    │  Yesterday                              │
    └─────────────────────────────────────────┘

[Tab Bar: Discover | ● Chats | Profile]
```

**Conversation View Layout:**
```
[Top Bar: Logo | Theme | Lang]

    [← Back to Chats]
    Jonas · Developer Tools

    ┌─────────────────────────────────────────┐
    │                                         │
    │  [You]  Hey! Ich hab gesehen du         │
    │         kämpfst auch mit Pricing.       │
    │         Was genau ist deine Situation?  │
    │                              14:32      │
    │                                         │
    │  [Jonas] Hey! Ja, ich hab ein Dev-Tool  │
    │          und weiß nicht ob Freemium     │
    │          oder direkt Paid. Du?          │
    │                              14:35      │
    │                                         │
    │  [You]  Ähnlich. SaaS, B2B. Hab Angst  │
    │         zu viel zu verlangen. Wollen    │
    │         wir uns auf einen Kaffee        │
    │         treffen?                        │
    │                              14:38      │
    │                                         │
    │  [Jonas] Klar! Nächste Woche? Ich bin   │
    │          flexibel.                      │
    │                              14:40      │
    │                                         │
    └─────────────────────────────────────────┘

    ┌─────────────────────────────────────────┐
    │ Type a message...          [Send]       │
    └─────────────────────────────────────────┘

[Tab Bar: Discover | ● Chats | Profile]
```

**Chat behavior (demo):**
- "Send" button and input are present but non-functional
- Clicking a chat list item navigates to `chat.html?with=<id>`
- If `?with=` param doesn't match a known conversation, show the chat list
- Pre-populate 2 dummy conversations (Jonas + Ayla)
- Message bubbles: "You" messages right-aligned (copper-tinted bg), other person left-aligned (card bg)

**Note on anonymous relay:** The chat header can show a subtle "Messages are anonymized" note in `var(--text-muted)` to communicate the privacy model.

#### Dummy Chat Data

```javascript
const chatConversations = {
  jonas: {
    founder: 'jonas',
    messages: [
      { from: 'you', text: { en: "Hey! I saw you're also struggling with pricing. What's your situation?", de: "Hey! Ich hab gesehen du kämpfst auch mit Pricing. Was genau ist deine Situation?" }, time: '14:32' },
      { from: 'jonas', text: { en: "Hey! Yeah, I have a dev tool and can't decide between freemium or paid. You?", de: "Hey! Ja, ich hab ein Dev-Tool und weiß nicht ob Freemium oder direkt Paid. Du?" }, time: '14:35' },
      { from: 'you', text: { en: "Similar. SaaS, B2B. Scared of charging too much. Want to grab a coffee?", de: "Ähnlich. SaaS, B2B. Hab Angst zu viel zu verlangen. Wollen wir uns auf einen Kaffee treffen?" }, time: '14:38' },
      { from: 'jonas', text: { en: "Sure! Next week? I'm flexible.", de: "Klar! Nächste Woche? Ich bin flexibel." }, time: '14:40' },
    ]
  },
  ayla: {
    founder: 'ayla',
    messages: [
      { from: 'you', text: { en: "Hey Ayla! Your challenge resonated with me. I'm also trying to find my first users.", de: "Hey Ayla! Deine Challenge hat mich angesprochen. Ich versuche auch meine ersten Nutzer zu finden." }, time: '10:15' },
      { from: 'ayla', text: { en: "Nice to meet you! What's your approach so far?", de: "Schön dich kennenzulernen! Was ist dein bisheriger Ansatz?" }, time: '10:22' },
      { from: 'you', text: { en: "Mostly cold outreach on LinkedIn. Not scaling well. You?", de: "Hauptsächlich Kaltakquise auf LinkedIn. Skaliert nicht gut. Du?" }, time: '10:25' },
      { from: 'ayla', text: { en: "Same! Let's brainstorm together. Coffee this week?", de: "Genauso! Lass uns zusammen brainstormen. Kaffee diese Woche?" }, time: '10:30' },
      { from: 'you', text: { en: "Absolutely, let's do it!", de: "Ja, lass uns das machen!" }, time: '10:32' },
    ]
  }
};
```

#### Missing i18n Keys (discover, challenge, chat)

```javascript
{
  en: {
    // discover.html
    'discover.title': 'Discover',
    'discover.filter.all': 'All',
    'discover.empty': 'No founders match your filters.',
    'discover.empty.clear': 'Clear filters',

    // challenge.html
    'challenge.back': 'Back to Discover',
    'challenge.also': 'Also working on:',
    'challenge.connect': "Let's connect",
    'challenge.revenue': 'Revenue',
    'challenge.request.title': 'Connection request sent',
    'challenge.request.body': 'Start a conversation to connect.',

    // chat.html
    'chat.title': 'Chats',
    'chat.back': 'Back to Chats',
    'chat.input': 'Type a message...',
    'chat.send': 'Send',
    'chat.anonymous': 'Messages are anonymized',
    'chat.time.hours': 'hours ago',
    'chat.time.yesterday': 'Yesterday',
    'chat.no-conversation': 'No conversation yet. Say hello!',
  },
  de: {
    'discover.title': 'Entdecken',
    'discover.filter.all': 'Alle',
    'discover.empty': 'Keine Gründer passen zu deinem Filter.',
    'discover.empty.clear': 'Filter zurücksetzen',

    'challenge.back': 'Zurück zum Entdecken',
    'challenge.also': 'Arbeitet auch an:',
    'challenge.connect': 'Vernetzen',
    'challenge.revenue': 'Umsatz',
    'challenge.request.title': 'Anfrage gesendet',
    'challenge.request.body': 'Starte eine Unterhaltung zum Vernetzen.',

    'chat.title': 'Chats',
    'chat.back': 'Zurück zu Chats',
    'chat.input': 'Nachricht eingeben...',
    'chat.send': 'Senden',
    'chat.anonymous': 'Nachrichten sind anonymisiert',
    'chat.time.hours': 'Stunden her',
    'chat.time.yesterday': 'Gestern',
    'chat.no-conversation': 'Noch keine Unterhaltung. Sag Hallo!',
  }
}
```

#### Research Insights (Phase 4)

**Founder data cleanup:**
- Remove redundant `challenge` field — use `tags[0]` as primary challenge
- Remove redundant `founder` field from `chatConversations` — object key is the ID
- Replace `from: 'you'` with `isSelf: true` boolean to avoid i18n collision

**"Let's connect" dead end fix:** 4 of 6 founders have no chat data. When `?with=` doesn't match a conversation, show a "Connection request sent" state instead of silently falling back to chat list:
```
┌─────────────────────────────────────────┐
│  ✓ Connection request sent              │
│                                         │
│  Tom · Marketplace                      │
│  Start a conversation to connect.       │
│                                         │
│  [________________] [Send]              │
└─────────────────────────────────────────┘
```

**Discover filtering:** Use single-tag filter (click one pill → filter). Simpler than multi-tag OR logic and more appropriate for 6 items. "All" pill is default and always shown.

**Card interaction:** Use `@media (hover: hover)` guard on card hover effects. On touch devices, `:hover` can stick after tap.

### Phase 5: Landing Page Integration & Polish

#### Tasks

- [ ] Update CTA button on `index.html` to link to `app/apply.html#step1`
- [ ] Add `prefers-reduced-motion` support across all pages (use "no-motion-first" CSS pattern):
  - **Disable completely:** cursor trail (`display: none`), parallax transforms (`transform: none`), scroll reveal translateY (content visible immediately)
  - **Show static frame:** grain overlay (`cancelAnimationFrame`, render one frame) or CSS `background-image` fallback
  - **Reduce, not remove:** theme transitions (reduce from 5s to ~200ms), cursor glow (remove lerp smoothing, instant position)
  - **Keep as-is:** opacity and color changes (not vestibular triggers)
  - CSS: wrap all motion in `@media (prefers-reduced-motion: no-preference)` — motion is opt-in, not opt-out
  - JS: use `window.matchMedia('(prefers-reduced-motion: reduce)')` + `addEventListener('change', ...)` for runtime OS preference changes
  - Guard cursor trail with `@media (hover: hover)` — should not render on touch-only devices regardless of motion preference
- [ ] Test all 6 pages in both themes (dark/light)
- [ ] Test all 6 pages in both languages (EN/DE)
- [ ] Test mobile layout at 375px, 390px, 414px widths
- [ ] Test iPad layout at 768px
- [ ] Verify `safe-area-inset-bottom` on tab bar (iOS notch devices)
- [ ] Verify form inputs scroll above mobile keyboard
- [ ] Test browser back button through entire onboarding flow
- [ ] Test hash navigation in apply.html wizard
- [ ] Verify cursor effects work on all pages (desktop)
- [ ] Verify grain overlay has `pointer-events: none` (doesn't block clicks)
- [ ] Check color contrast for Copper `#c45a3c` on both bg colors (WCAG AA)

---

## Acceptance Criteria

### Functional Requirements

- [ ] CTA on landing page navigates to `app/apply.html#step1`
- [ ] 3-step wizard with forward/backward navigation and hash-based URL
- [ ] Browser back button works through wizard steps
- [ ] Success state shows inline after wizard completion
- [ ] Welcome page with "Get Started" → Profile setup
- [ ] Profile page works in setup mode (onboarding) and view mode (tab bar)
- [ ] Multi-select challenge tags with visual toggle
- [ ] Discover page shows 6 founder cards with challenge-based content
- [ ] Filter pills filter discover cards by challenge tag (OR logic)
- [ ] Empty filter state shows "No founders match" message
- [ ] Challenge detail page shows founder info + "Let's connect"
- [ ] "Let's connect" navigates to chat with pre-populated conversation
- [ ] Chat list shows 2 dummy conversations
- [ ] Chat conversation view shows message bubbles with timestamps
- [ ] Bottom tab bar on all app screens (not on onboarding)
- [ ] Theme toggle works on all pages (dark/light, 5s transition)
- [ ] Language toggle works on all pages (EN/DE)
- [ ] All text has i18n support via `data-i18n` attributes

### Non-Functional Requirements

- [ ] All effects (cursor glow, grain, parallax) work on all pages
- [ ] `prefers-reduced-motion` disables animations
- [ ] Mobile-first responsive design (works at 320px minimum)
- [ ] `safe-area-inset-bottom` on tab bar
- [ ] `pointer-events: none` on grain overlay
- [ ] No build step required — open any `.html` file directly in browser
- [ ] Color contrast meets WCAG AA for text elements
- [ ] Tab bar touch targets >= 48px

### Accessibility Requirements (from research)

- [ ] Wizard: `aria-live="polite"` region announces step changes to screen readers
- [ ] Wizard: focus moves to step heading on transition (`tabindex="-1"` on headings)
- [ ] Forms: `aria-invalid="true"` + `aria-describedby` for field errors
- [ ] Tab bar: `<nav>` with `aria-label="Main navigation"` and `aria-current="page"` on active link
- [ ] Decorative elements: `aria-hidden="true"` on SVG icons, cursor glow, grain canvas
- [ ] Touch targets: minimum 48px on all interactive elements
- [ ] Copper `#c45a3c` contrast: passes 3:1 for UI components; for small text, ensure 14px bold minimum (qualifies as large text at 3:1) or lighten slightly

### Quality Gates

- [ ] `index.html` renders identically before and after CSS/JS extraction
- [ ] All pages work when accessed directly via URL (no routing gates)
- [ ] No console errors in Chrome, Safari, Firefox
- [ ] German text does not overflow or break layouts
- [ ] i18n: every visible text string has `data-i18n` attribute or bilingual data object
- [ ] `textContent` used everywhere — no `innerHTML` with dynamic data
- [ ] URL params validated against whitelist (no arbitrary strings rendered to DOM)

---

## File-by-File Implementation Checklist

| File | Phase | Lines (est.) | Dependencies |
|------|-------|-------------|-------------|
| `shared.css` | 1 | ~700 | None |
| `shared.js` | 1 | ~400 | `shared.css` |
| `index.html` (refactor) | 1 | ~400 (down from 1605) | `shared.css`, `shared.js` |
| `app/app.css` | 2 | ~300 | `shared.css` |
| `app/app.js` | 2-4 | ~500 | `shared.js` |
| `app/apply.html` | 3a | ~200 | `shared.*`, `app.*` |
| `app/welcome.html` | 3b | ~80 | `shared.*`, `app.*` |
| `app/profile.html` | 3c | ~150 | `shared.*`, `app.*` |
| `app/discover.html` | 4a | ~150 | `shared.*`, `app.*` |
| `app/challenge.html` | 4b | ~120 | `shared.*`, `app.*` |
| `app/chat.html` | 4c | ~180 | `shared.*`, `app.*` |

**Total new files:** 10 (4 shared + 6 pages)
**Estimated total new lines:** ~2,780

---

## References

### Internal

- Landing page: `index.html` (1605 lines — CSS lines 11-871, JS lines 1039-1602)
- Design constraints: `CLAUDE.md` (strict 2-color palette, effect specifications)
- Brainstorm: `docs/brainstorms/2026-03-08-community-platform-demo-brainstorm.md`
- Solution patterns: `docs/solutions/001-005` (theme flash fix, logo canvas, magnetic type, glass header, cursor trail pool)

### Design Constraints (from CLAUDE.md)

- Only Copper `#c45a3c` as accent — no other colors
- No scroll-based color shifting
- Grain must be very subtle (0.025/0.02 opacity)
- No section borders
- Button text: `var(--bg-primary)` not `#fff`
- Header content always visible; only glass bg fades on scroll
