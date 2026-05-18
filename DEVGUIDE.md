# Development Guide — Battle Ship

This project is reviewed by a separate AI (Claude). Follow these standards exactly.

---

## Non-Negotiables

1. **Single file only.** All code in `index.html`. No `.js` or `.css` files.
2. **Zero external dependencies.** No CDN links, no npm, no imports.
3. **Must run via `file://`** — no fetch(), no ES modules (which require a server).
4. **No build step.** The delivered `index.html` must work as-is.

---

## JavaScript Standards

- ES6+ syntax is fine: `const`, `let`, arrow functions, template literals, destructuring
- **No `var`**
- Use `class` for game entities (Player, Enemy, Cannonball, Explosion)
- Game loop via `requestAnimationFrame`; pass `deltaTime` in seconds to all update functions
- All magic numbers go in a `// Config` block at the top as named `const`s

```js
// Good
const PLAYER_SPEED = 180;
const FIRE_COOLDOWN_MS = 1000;
const SPAWN_INTERVAL_INITIAL = 4000;

// Bad
if (dist < 200) { ... }   // unexplained literal
```

---

## Entity Structure

Each entity class must have:
- `update(dt)` — moves, fires, ticks timers
- `draw(ctx)` — renders sprite + overlaid text/bars
- `isDead()` — returns boolean

Example skeleton:

```js
class Enemy {
  constructor(x, y, playerMaxHP) { ... }
  update(dt) { ... }
  draw(ctx) { ... }
  isDead() { return this.hp <= 0; }
}
```

---

## Rendering Rules

- Clear + redraw background every frame (no dirty-rect optimization needed)
- Draw order: background → cannonballs → enemies → player → explosions → HUD → screen overlays
- Sprite rotation: use `ctx.save() / ctx.translate() / ctx.rotate() / ctx.restore()` pattern
- HP number on ship: always drawn AFTER the sprite, so it's on top
- All text: set `ctx.textAlign` and `ctx.textBaseline` explicitly before drawing

---

## Image Loading

```js
function loadImage(src) {
  return new Promise((resolve) => {
    const img = new Image();
    img.onload = () => resolve(img);
    img.onerror = () => resolve(null);   // null = use fallback shape
    img.src = src;
  });
}
```

Wait for all images with `Promise.all(...)` before starting the game.
If `img` is null, draw a colored rectangle of the same size as a fallback.

---

## Collision Detection

Use circle-circle:

```js
function circlesOverlap(ax, ay, ar, bx, by, br) {
  const dx = ax - bx, dy = ay - by;
  return dx*dx + dy*dy < (ar + br) * (ar + br);
}
```

Do not use AABB (rectangle collision) — ships are rotated.

---

## deltaTime Usage

All movement and timers must be frame-rate independent:

```js
// Correct
this.x += Math.cos(angle) * this.speed * dt;
this.fireCooldown -= dt * 1000;   // dt in seconds, cooldown in ms

// Wrong
this.x += 3;   // breaks at non-60fps
```

---

## What the Reviewer (Claude) Will Check

1. All SPEC.md mechanics are implemented correctly (HP scaling, attack formula, spawn interval steps)
2. `deltaTime` used everywhere — no frame-rate-dependent values
3. Circle collision used (not AABB)
4. HP number visible on every ship sprite
5. Explosion animation plays on enemy death
6. Image load failure handled gracefully (fallback shapes)
7. Game restarts cleanly (no lingering state from previous run)
8. No external URLs or resources
9. Code structure matches the section layout in SPEC.md
10. No commented-out dead code in final submission
