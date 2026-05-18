# Battle Ship — Game Specification

## Overview

A top-down pirate sea battle survival game, delivered as a single `index.html` playable ad demo.
The player controls a pirate ship that grows stronger by absorbing defeated enemies' HP.

---

## Tech Stack

- **Single file:** `index.html` (all JS/CSS inline, no external files except assets)
- **Renderer:** HTML5 Canvas 2D
- **Language:** Vanilla JavaScript (ES6+), no libraries or CDN links
- **Assets:** PNG sprites from Kenney Pirate Pack (see Asset Manifest below)
- **Must work:** open `index.html` directly in Chrome (file:// protocol)

---

## Asset Manifest

Copy from `G:\MyDownloads\Kenney Game Assets All-in-1 3.4.0 (Windows)\2D assets\Pirate Pack\PNG\Default size\`

| Game use | Source file | Destination |
|---|---|---|
| Player ship | `Ships\ship (1).png` | `assets/ship_player.png` |
| Enemy ship type 1 | `Ships\ship (3).png` | `assets/ship_enemy1.png` |
| Enemy ship type 2 | `Ships\ship (5).png` | `assets/ship_enemy2.png` |
| Enemy ship type 3 | `Ships\ship (7).png` | `assets/ship_enemy3.png` |
| Cannonball | `Ship parts\cannonBall.png` | `assets/cannonball.png` |
| Explosion frame 1 | `Effects\explosion1.png` | `assets/explosion1.png` |
| Explosion frame 2 | `Effects\explosion2.png` | `assets/explosion2.png` |
| Explosion frame 3 | `Effects\explosion3.png` | `assets/explosion3.png` |

If any image fails to load, substitute with a colored rectangle so the game still runs.

---

## Canvas & Layout

- Canvas resolution: **800 × 600**
- Scale to window: `canvas.style.maxWidth = "100%"`, centered on page
- Background: draw each frame — dark blue (`#0a2a5e`) to teal (`#1a6b8a`) vertical gradient
- HUD (top of canvas): survive timer (left), ships sunk count (right)

---

## Core Mechanic — HP & Attack Scaling

```
Player.initialHP  = 10
Player.hp         = Player.initialHP        (current HP, decreases when hit)
Player.maxHP      = Player.initialHP        (tracks total absorbed HP for scaling)
Player.atk        = Player.maxHP / 10

When player kills an enemy:
  Player.maxHP += Enemy.initialHP
  Player.hp    += Enemy.initialHP           (heal by that amount too)
  Player.atk    = Player.maxHP / 10
```

Enemy stats on spawn:
```
Enemy.initialHP = Player.maxHP * random(0.7, 1.3)   // rounded to integer
Enemy.hp        = Enemy.initialHP
Enemy.atk       = Enemy.initialHP / 10
Enemy.speed     = PLAYER_SPEED * random(0.7, 0.9)
```

This creates a snowball curve: sinking bigger ships makes the player stronger and attracts even tougher enemies.

---

## Player

| Property | Value |
|---|---|
| Sprite | `assets/ship_player.png` |
| Size on canvas | 64 × 64 px |
| Collision radius | 28 px |
| Move speed | 180 px/s |
| Controls | WASD or Arrow keys |
| Sprite rotation | face direction of movement; default facing = up |

**Auto-fire:**
- Each frame: find the nearest living enemy
- If distance ≤ 200 px AND fire cooldown expired (1000 ms): spawn a cannonball aimed at that enemy's current position
- Cannonball travels in a straight line (not homing)

**HP display:**
- Draw current HP as a white number centered on the ship sprite
- Text: bold 14px, white fill, 1px black stroke (for readability on any background)

**When hit:**
- Deduct `enemy.atk` from `player.hp`
- Flash ship sprite red for 300 ms
- If `player.hp ≤ 0`: trigger GAME OVER

---

## Enemies

**Spawning:**
- Spawn outside the canvas by 60 px on a randomly chosen edge (top / bottom / left / right)
- Position along that edge: random within canvas bounds
- Initial spawn interval: **4000 ms**
- Every 30 s of game time, interval decreases: 4000 → 3000 → 2500 → 2000 ms (minimum)
- No cap on simultaneous enemies on screen

**Behavior (per frame):**
1. Rotate sprite to face player
2. Move toward player at `enemy.speed`
3. If distance to player ≤ 200 px AND fire cooldown expired (2000 ms): fire cannonball at player

**HP display:**
- Same as player: white bold number centered on sprite, black outline
- Also draw a thin HP bar above the sprite:
  - Width 40 px, height 5 px
  - Red background, green fill proportional to `hp / initialHP`

**On death:**
- Play 3-frame explosion animation at enemy position (120 ms per frame)
- Transfer `enemy.initialHP` to player (see Core Mechanic)
- Remove enemy from scene

---

## Cannonballs

| Property | Value |
|---|---|
| Sprite | `assets/cannonball.png` (or 6 px filled yellow circle as fallback) |
| Draw size | 16 × 16 px |
| Speed | 350 px/s |
| Collision radius | 6 px |
| Lifetime | Remove when off-canvas OR on hit |

On hit: remove cannonball, apply `shooter.atk` damage to target, trigger hit flash.

---

## Explosion Animation

- Frames: `explosion1.png` → `explosion2.png` → `explosion3.png`
- Duration per frame: 120 ms
- Draw size: 80 × 80 px, centered on impact point
- Remove after all 3 frames play

---

## Game States

```
LOADING → TITLE → PLAYING → GAME_OVER
```

### LOADING
- Show "Loading…" centered on canvas
- Wait until all images are loaded, then switch to TITLE

### TITLE
- Dark semi-transparent overlay on canvas background
- Text (centered):
  - Line 1: `BATTLE SHIP` — large, gold, bold
  - Line 2: `Pirate Sea Battle` — medium, white
  - Line 3: `WASD to sail · auto-cannon fires` — small, light gray
  - Line 4: `Click or press SPACE to start` — blinking (toggle 600 ms)

### PLAYING
- Full game loop active
- HUD top-left: `⏱ 00:47`
- HUD top-right: `⚓ 12 sunk`
- Player HP bar displayed under player ship

### GAME_OVER
- Pause all game logic
- Semi-transparent dark overlay
- Text:
  - `SHIP SUNK` — large, red
  - `Survived: 01:23` — medium, white
  - `Ships sunk: 17` — medium, white
- Button: `[ Play Again ]` — restarts PLAYING state from scratch
- Button: `[ Play Full Game ]` — href="#" placeholder

---

## HUD Details

- Font: system monospace or sans-serif
- All HUD text has a 1 px black shadow for legibility on ocean background
- Player HP displayed as a bar below the player sprite (50 px wide, 6 px tall, red/green)

---

## Code Structure (inside `<script>`)

Organize code into clearly named sections with a single-line section comment:

```
// Config
// State
// Assets
// Input
// Entities — Player
// Entities — Enemy
// Entities — Cannonball
// Entities — Explosion
// Spawning
// Collision
// Update
// Draw
// Screens
// Game loop
// Init
```

Target: 400–550 lines. Prioritize correctness and readability over cleverness.
