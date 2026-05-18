# Prompt v1 — Initial Implementation

**Issued:** 2026-05-18
**Reviewer:** Claude (will review after submission)

---

## Your Task

### Step 1 — Copy assets

Copy the files listed in `SPEC.md > Asset Manifest` from the Kenney Pirate Pack into the `assets/` folder.

Source root:
```
G:\MyDownloads\Kenney Game Assets All-in-1 3.4.0 (Windows)\2D assets\Pirate Pack\PNG\Default size\
```

### Step 2 — Implement `index.html`

Build the complete game in a single `index.html` file.
Follow every rule in `DEVGUIDE.md`. Key constraints:
- Vanilla JS + Canvas only, no external libraries
- Use `class` for all entities
- All movement/timers must use `deltaTime`
- Circle collision (not AABB)
- Image load failure must fall back to a colored rectangle

### Step 3 — Self-verify

Before committing, check every item in `DEVGUIDE.md > What the Reviewer Will Check` (10 items).
Include your verification results in the commit message.

### Step 4 — Push

```
git add assets/ index.html
git commit -m "feat: implement playable ad game

Reviewer checklist:
1. HP scaling ✅
2. deltaTime ✅
..."
git push
```

Then notify the reviewer (Claude) with the commit hash.
