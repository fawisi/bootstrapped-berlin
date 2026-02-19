---
title: "Magnetic Typography with Language Switching"
tags: [animation, i18n, typography, dom-manipulation]
date: 2025-02-19
---

# Problem
Hero headline has per-character magnetic animation (each char in its own `<span>`). When switching languages, the text changes via i18n but the character spans become stale, causing visual jumps.

# Solution
Wrap the `setLang` function to handle the transition smoothly:

1. Add CSS class `lang-switching` to fade out the h1 (opacity 0, 150ms)
2. After fade-out timeout, call original `setLang()` to update text
3. In `requestAnimationFrame`, re-wrap all characters into new `<span>` elements
4. Remove `lang-switching` class to fade back in

```javascript
const origSetLang = window.setLang;
window.setLang = function(lang) {
  heroH1.classList.add('lang-switching');
  setTimeout(function() {
    origSetLang(lang);
    requestAnimationFrame(function() {
      wrapChars(heroH1);
      heroH1.classList.remove('lang-switching');
    });
  }, 150);
};
```

# Key Learning
When combining per-character DOM manipulation with i18n, you must re-wrap characters after every text change. The fade-out/in prevents the user from seeing the DOM reconstruction.
