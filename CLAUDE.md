# bootstrapped, berlin

## Project Overview
Landing page / waitlist for a curated network for bootstrapped founders in Berlin. Single static HTML file, no build step, no framework.

## Tech Stack
- **Single file**: `index.html` contains all HTML, CSS, and JS inline
- **No dependencies**: No npm, no build tools, no bundler
- **Fonts**: Google Fonts — JetBrains Mono (mono/headlines), Lora (serif/sublines), Inter (body/sans)
- **No framework**: Vanilla JS, CSS custom properties for theming

## Architecture & Key Systems

### Theming (Dark/Light Mode)
- CSS custom properties in `:root` (dark) and `html[data-theme="light"]` (light)
- 5-second crossfade transition via `cubic-bezier(0.25, 0, 0.25, 1)` on all color-bearing elements
- Auto-detection via `prefers-color-scheme`, stored in `localStorage` key `bb_theme`
- Instant load (no flash): `transition: none` before applying, restored via `requestAnimationFrame`
- Theme toggle in header (moon/sun icon)

### i18n (EN/DE)
- `translations` object with all text keyed by `data-i18n` attributes
- Auto-detection via `navigator.language`, stored in `localStorage` key `bb_lang`
- Language switch buttons in header
- **Important**: Hero headline uses magnetic typography (char wrapping). When changing language, the `setLang` function is wrapped to fade-out, update text, re-wrap chars, then fade-in. Do NOT modify `setLang` without preserving this wrapper chain.

### Color System — STRICT 2-COLOR PALETTE
- **Background**: Warm neutrals only (`#0a0a0a` dark / `#f5f0eb` light spectrum)
- **Accent**: Copper `#c45a3c` — ONE accent color everywhere
- **All effects** (cursor glow, atmospheric sections, particles, logo) use the same copper accent at varying opacities
- **No scroll-based color shift** — accent stays constant. This was explicitly removed by the user.
- The user explicitly does NOT want blue, gold, amber, or any other accent colors

### Interactive Effects (Desktop)
1. **Cursor Glow**: 600px radial gradient following cursor with lerp smoothing (0.06)
2. **Cursor Trail**: Pool of 12 glow ghosts (350px) that spawn every 30px of movement and fade over 1.2s
3. **Magnetic Typography**: Hero H1 characters wrapped in `<span class="magnetic-char">`, attracted toward cursor within 300px radius
4. **Parallax Headlines**: `.parallax-headline` elements shift -15px based on scroll position
5. **Atmospheric Section Glows**: Each section has a `.section-atmosphere` div with copper radial gradient, faded in/out via IntersectionObserver

### Interactive Effects (Mobile/Touch)
- Cursor glow uses scroll-position-based sine waves instead of cursor tracking
- No trail, no magnetic typography
- Parallax reduced to -8px

### Header (Glass Pill)
- Logo, text, and buttons always visible
- Glass background (blur + shadow) fades in via `::before` pseudo-element when `scrollY > 20px`
- CSS class `header.scrolled` toggles the glass effect
- Floats 12px from top, rounded 14px pill shape

### Logo (Canvas)
- 56x56 canvas rendered at 28x28 CSS (2x retina)
- Noise sphere: particles in accent color, dense at center, transparent at edges
- Animated at ~12fps via requestAnimationFrame
- Reads `--accent` CSS variable live

### Grain Overlay
- 256x256 canvas, `mix-blend-mode: overlay`, animated at ~12fps
- Very subtle: `--grain-opacity: 0.025` dark / `0.02` light
- The user explicitly said grain should be very fine and subtle. Previous coarser grain (128px) was rejected.

### Arrow Particles
- 20 small copper particles falling through the hero arrow
- Spawn every 120ms, start above arrow, fall downward with slight horizontal drift
- Accelerating ease via `cubic-bezier(0.4, 0, 1, 1)`

## Design Decisions & User Preferences
- **Dark, techie, serious** — not playful, not startup-y
- **Analog/organic feel** — grain, noise, handcraft textures
- **Hormozi Value Equation** as content framework (Dream Outcome + Perceived Likelihood / Time Delay + Effort)
- **Core emotional hook**: "You're not alone anymore"
- **Manifesto card** is a light-colored card (inverted) on the dark page, with hatch pattern overlay
- **No section borders** between sections — only footer has a top border
- **Generous whitespace**: sections have 140px padding, CTA has 160px
- **"Entscheidung"** (not "Wahl") for German hero translation — user explicitly preferred this word

## Section Structure
1. Hero (full viewport)
2. Problem ("the reality")
3. Manifesto (light card, "what we believe")
4. Value ("what you get" — 4 items)
5. Audience ("is this you?" — checklist + exclusion note)
6. CTA (waitlist email form)
7. Footer

## Next Steps (User's Priority)
- **Copywriting** — the user wants to focus on improving the copy/text on the page
- Wording and text need refinement (user mentioned this early on)
- Consider the Hormozi framework when rewriting copy

## Common Pitfalls
- Don't add new colors — stick to copper + neutrals
- Don't add scroll-based color shifting — it was explicitly removed
- Don't make grain coarser or more visible — user rejected noticeable grain twice
- Don't add border lines between sections
- Don't break the magnetic typography ↔ setLang wrapper chain
- The button text color is `var(--bg-primary)` (not `#fff`) so it works in both themes
- Header content is always visible; only the glass background fades on scroll
