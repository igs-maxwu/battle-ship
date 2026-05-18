# Prompt v3 — Feedback Round 2

**Issued:** 2026-05-18
**Reviewer:** Claude

---

## 3 changes needed in `index.html`

### 1. Fix hit flash — offscreen canvas tint (no red bounding box)

`source-atop` against an already-painted canvas tints the full rectangle, not just the sprite pixels.
Fix: use a dedicated offscreen canvas as an intermediate buffer.

**Add to State section (top of script):**
```js
const flashCanvas = document.createElement('canvas');
flashCanvas.width = 256;
flashCanvas.height = 256;
const flashCtx = flashCanvas.getContext('2d');
```

**Add a helper function (in Draw section):**
```js
function drawSprite(ctx, img, size, flashColor) {
  if (!img) return false;
  if (!flashColor) {
    ctx.drawImage(img, -size / 2, -size / 2, size, size);
    return true;
  }
  // Tint: draw to offscreen canvas first, then composite
  const offset = (256 - size) / 2;
  flashCtx.clearRect(0, 0, 256, 256);
  flashCtx.drawImage(img, offset, offset, size, size);
  flashCtx.globalCompositeOperation = 'source-atop';
  flashCtx.fillStyle = flashColor;
  flashCtx.fillRect(0, 0, 256, 256);
  flashCtx.globalCompositeOperation = 'source-over';
  ctx.drawImage(flashCanvas, -size / 2, -size / 2, size, size);
  return true;
}
```

**Update Player.draw() — replace the img draw block with:**
```js
const tint = this.flashTimer > 0 ? 'rgba(255, 60, 60, 0.55)' : null;
if (!drawSprite(ctx, img, this.drawSize, tint)) {
  ctx.fillStyle = this.flashTimer > 0 ? '#ff4444' : '#44aaff';
  ctx.fillRect(-this.drawSize / 2, -this.drawSize / 2, this.drawSize, this.drawSize);
}
```

**Update Enemy.draw() — same pattern:**
```js
const tint = this.flashTimer > 0 ? 'rgba(255, 60, 60, 0.55)' : null;
if (!drawSprite(ctx, img, PLAYER_SIZE, tint)) {
  ctx.fillStyle = this.flashTimer > 0 ? '#ff4444' : '#cc3333';
  ctx.fillRect(-PLAYER_SIZE / 2, -PLAYER_SIZE / 2, PLAYER_SIZE, PLAYER_SIZE);
}
```

---

### 2. Player grows larger as HP increases

The player's visual size and collision radius scale with `maxHP`.

**Add to Player constructor:**
```js
this.drawSize = PLAYER_SIZE;
this.collisionR = PLAYER_COLLISION_R;
```

**Add a method to Player class:**
```js
updateScale() {
  const scale = Math.min(Math.pow(this.maxHP / this.initialHP, 0.25), 3.0);
  this.drawSize = Math.round(PLAYER_SIZE * scale);
  this.collisionR = Math.round(PLAYER_COLLISION_R * scale);
}
```

**Call `player.updateScale()` in `checkCollisions()` every time `player.maxHP` changes** (right after setting `player.atk`).

**Update Player.draw():** replace all hardcoded `PLAYER_SIZE` with `this.drawSize` in the drawImage and fillRect calls. The HP bar position also uses `this.drawSize`:
```js
drawHPBar(ctx, this.x, this.y + this.drawSize / 2 + 8, 50, 6, this.hp, this.maxHP);
```

**Update `checkCollisions()`:** replace `PLAYER_COLLISION_R` with `player.collisionR` in the enemy-cannonball-hits-player check:
```js
if (circlesOverlap(b.x, b.y, CANNONBALL_COLLISION_R, player.x, player.y, player.collisionR)) {
```

---

### 3. Enemy speed — fixed at 80% of player speed

In `Enemy` constructor, replace the random speed calculation:
```js
// Before:
this.speed = PLAYER_SPEED * (0.7 + Math.random() * 0.2);

// After:
this.speed = PLAYER_SPEED * 0.8;
```

---

Complete all 3 changes, push, and report using the format in DEVGUIDE.md.
