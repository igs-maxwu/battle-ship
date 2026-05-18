# Prompt v2 — Feedback Round 1

**Issued:** 2026-05-18
**Reviewer:** Claude

---

## 3 changes needed in `index.html`

### 1. Camera follow — player always at screen center

Remove the two boundary-clamping lines in `Player.update()`:
```js
// DELETE these two lines:
this.x = Math.max(PLAYER_SIZE / 2, Math.min(CANVAS_W - PLAYER_SIZE / 2, this.x));
this.y = Math.max(PLAYER_SIZE / 2, Math.min(CANVAS_H - PLAYER_SIZE / 2, this.y));
```

In the game loop draw section, wrap all world-object drawing in a camera transform:
```js
// Draw background first (no transform)
drawBackground(ctx);

// Apply camera transform
ctx.save();
ctx.translate(Math.round(CANVAS_W / 2 - player.x), Math.round(CANVAS_H / 2 - player.y));

// Draw world objects at their world coords
for (const b of cannonballs) b.draw(ctx);
for (const e of enemies) e.draw(ctx);
player.draw(ctx);
for (const ex of explosions) ex.draw(ctx);

ctx.restore();

// Draw HUD last (no transform)
drawHUD(ctx);
```

Apply the same pattern to the GAME_OVER state draw block.

Update `spawnEnemy()` to spawn at viewport edges relative to player:
```js
function spawnEnemy() {
  const edge = Math.floor(Math.random() * 4);
  let x, y;
  const vL = player.x - CANVAS_W / 2 - SPAWN_OFFSET;
  const vR = player.x + CANVAS_W / 2 + SPAWN_OFFSET;
  const vT = player.y - CANVAS_H / 2 - SPAWN_OFFSET;
  const vB = player.y + CANVAS_H / 2 + SPAWN_OFFSET;
  switch (edge) {
    case 0: x = vL + Math.random() * (vR - vL); y = vT; break;
    case 1: x = vL + Math.random() * (vR - vL); y = vB; break;
    case 2: x = vL; y = vT + Math.random() * (vB - vT); break;
    case 3: x = vR; y = vT + Math.random() * (vB - vT); break;
  }
  enemies.push(new Enemy(x, y, player.maxHP));
}
```

### 2. Full heal on kill

In `checkCollisions()`, change the kill reward block:
```js
// Before:
player.maxHP += e.initialHP;
player.hp += e.initialHP;
player.atk = player.maxHP / 10;

// After:
player.maxHP += e.initialHP;
player.hp = player.maxHP;     // full heal to new max
player.atk = player.maxHP / 10;
```

### 3. Hit flash — tint sprite pixels only, no red rectangle

In both `Player.draw()` and `Enemy.draw()`, replace the red `fillRect` overlay with `source-atop` so the tint applies only to visible sprite pixels, not the transparent bounding box.

Pattern to use (must be inside the `ctx.save()/translate()/rotate()` block):
```js
ctx.drawImage(img, -PLAYER_SIZE / 2, -PLAYER_SIZE / 2, PLAYER_SIZE, PLAYER_SIZE);
if (this.flashTimer > 0) {
  ctx.globalCompositeOperation = 'source-atop';
  ctx.fillStyle = 'rgba(255, 60, 60, 0.55)';
  ctx.fillRect(-PLAYER_SIZE / 2, -PLAYER_SIZE / 2, PLAYER_SIZE, PLAYER_SIZE);
  ctx.globalCompositeOperation = 'source-over';
}
```

For the fallback (no image) branch — just change `fillStyle` to red when flashing, no compositing needed:
```js
} else {
  ctx.fillStyle = this.flashTimer > 0 ? '#ff4444' : '#44aaff'; // player
  // or '#cc3333' for enemy default
  ctx.fillRect(-PLAYER_SIZE / 2, -PLAYER_SIZE / 2, PLAYER_SIZE, PLAYER_SIZE);
}
```

---

Complete all 3 changes, push, and report using the format in DEVGUIDE.md.
