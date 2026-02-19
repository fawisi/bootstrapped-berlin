---
title: "Canvas Noise Logo with Transparent Background"
tags: [canvas, logo, transparency, theming]
date: 2025-02-19
---

# Problem
Canvas-based noise logo showed colored border artifacts when switching between dark/light themes, because anti-aliased edge pixels used the theme background color.

# Solution
Remove all anti-alias edge logic. Use a binary approach:
- Outside the sphere radius: `alpha = 0` (fully transparent)
- Inside sphere, no particle: `alpha = 0` (fully transparent)
- Inside sphere, particle: `alpha = 255` (fully opaque)

No intermediate alpha values at edges. This eliminates any theme-dependent border artifacts.

# Key Learning
When canvas elements need to work across theme switches, avoid using theme-dependent colors for any partially-transparent pixels. Use fully transparent (alpha 0) or fully opaque (alpha 255) only.
