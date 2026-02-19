---
title: "Dark/Light Theme Switch Without Flash on Load"
tags: [css, theming, dark-mode, localStorage, performance]
date: 2025-02-19
---

# Problem
When loading a page with a stored theme preference, there's a visible flash of the default theme before JS applies the stored theme.

# Solution
1. Set `transition: none` on `<html>` and `<body>` before applying the theme
2. Apply the theme via `data-theme` attribute
3. Restore transitions in a `requestAnimationFrame` callback

```javascript
function applyTheme(theme, instant) {
  const html = document.documentElement;
  if (instant) {
    html.style.transition = 'none';
    document.body.style.transition = 'none';
    void html.offsetHeight; // force reflow
  }
  html.setAttribute('data-theme', theme);
  if (instant) {
    requestAnimationFrame(function() {
      html.style.transition = '';
      document.body.style.transition = '';
    });
  }
}
```

Call with `instant: true` on page load, `instant: false` on user toggle.
