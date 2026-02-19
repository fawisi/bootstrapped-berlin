---
title: "Performant Cursor Trail Using Object Pool Pattern"
tags: [animation, performance, cursor, dom-pool]
date: 2025-02-19
---

# Problem
Creating/destroying DOM elements per cursor movement is expensive. Need a smooth, broad cursor trail that doesn't impact performance.

# Solution
Pre-create a pool of DOM elements and cycle through them:

```javascript
const POOL_SIZE = 12;
const trailPool = [];
let trailIndex = 0;

// Create pool once
for (let i = 0; i < POOL_SIZE; i++) {
  const el = document.createElement('div');
  el.className = 'cursor-trail';
  el.style.opacity = '0';
  document.body.appendChild(el);
  trailPool.push(el);
}

function spawnTrailGhost(x, y) {
  const el = trailPool[trailIndex % POOL_SIZE];
  trailIndex++;
  el.style.transition = 'none';
  el.style.left = x + 'px';
  el.style.top = y + 'px';
  el.style.opacity = '0.7';
  void el.offsetWidth; // force reflow
  el.style.transition = 'opacity 1.2s ease-out';
  el.style.opacity = '0';
}
```

Spawn based on distance moved (not time), so static cursor = no new ghosts.

# Key Learning
The `void el.offsetWidth` trick forces a browser reflow between removing and re-adding transitions, allowing instant repositioning followed by animated fade-out. Without it, the browser batches both style changes and the transition won't fire.
