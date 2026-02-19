---
title: "Glass Header with Content Always Visible, Background Fading on Scroll"
tags: [css, header, glass-morphism, scroll, ux]
date: 2025-02-19
---

# Problem
User wanted header content (logo, buttons) always visible, but the glass background effect should only appear when scrolling begins.

# Solution
Use a `::before` pseudo-element for the glass background, separate from the content:

```css
.header-inner {
  position: relative;
  /* Content always visible */
}

.header-inner::before {
  content: '';
  position: absolute;
  inset: 0;
  background: var(--header-bg);
  backdrop-filter: blur(24px) saturate(1.4);
  border: 1px solid var(--header-border);
  box-shadow: 0 2px 16px rgba(0,0,0,0.06);
  z-index: -1;
  opacity: 0;
  transition: opacity 0.5s ease;
}

header.scrolled .header-inner::before {
  opacity: 1;
}
```

JS adds `scrolled` class when `scrollY > 20`.

# Key Learning
Separate visual decoration (glass effect) from content using pseudo-elements. This lets you animate the decoration independently while content stays permanently accessible.
