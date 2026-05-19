# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step. Open `index.html` directly in a browser — double-click the file or drag it into a browser window. Phaser 3 is loaded from CDN (`cdn.jsdelivr.net`), so an internet connection is required on first load (cached afterwards).

## Git workflow

**Commit and push after every meaningful change** — never leave work uncommitted. This ensures we can always revert to a known-good state.

Commit message format:
- `feat: <what was added>` — new feature or gameplay element
- `fix: <what was broken>` — bug fix
- `docs: <what was documented>` — CLAUDE.md or comments
- `refactor: <what changed>` — code restructure without behavior change

```powershell
cd "E:\AI-Projekte\ClaudeTest"
git add index.html        # or CLAUDE.md, etc.
git commit -m "feat: add boss enemy with charge attack"
git push
```

When to commit:
- After adding a new feature (enemy type, power-up, level, mechanic)
- After fixing a bug
- Before starting a large refactor (snapshot of working state)
- After a refactor is confirmed working

Remote: https://github.com/Eurekon538/retro-assault  
Push with: `git push` (branch `master` already tracks `origin/master`)

## Architecture

Everything lives in `index.html` as a single `<script>` block, split into 9 labeled sections:

| Section | What it does |
|---|---|
| 1 – Constants & Level Data | `GW/GH`, `COLORS`, `ECFG` (enemy configs), `PUCFG` (power-up configs), `LEVELS` array |
| 2 – Sprite Generation | Pure-graphics functions (`genPlayer`, `genEnemies`, `genBullet`, etc.) — draw to `Graphics`, call `generateTexture`, then manually register frames with `texture.add(i, 0, x, 0, w, h)` |
| 3 – BootScene | Calls all `gen*` functions then starts `MenuScene` |
| 4 – MenuScene | Title screen with floating enemy sprites and high-score read from `localStorage` |
| 5 – GameScene | Core gameplay: spawner queue, player physics, bullet/enemy/powerup groups, collision handlers, AI, effect timers |
| 6 – UIScene | Runs **in parallel** with GameScene; reads all state from `this.registry` (never from GameScene directly) |
| 7 – LevelCompleteScene | Between-level screen; passes `{level, score, playerHP}` to next `GameScene` |
| 8 – GameOverScene | Win/lose screen; reads/writes high score in `localStorage` key `raHigh` |
| 9 – Launch | `new Phaser.Game(...)` config |

## Key invariants

- **Groups**: always use `group.create(x, y, key)` for bullets, enemies, and power-ups — never `physics.add.image` + `group.add()`. The latter resets the physics body and kills velocity.
- **Player rotation**: gun barrel points RIGHT at `rotation = 0`. Aim is set with `setRotation(Phaser.Math.Angle.Between(player.x, player.y, ptr.x, ptr.y))` — no angular offset.
- **Power-ups**: use `physics.add.staticGroup()` so overlap works without velocity interference.
- **UIScene ↔ GameScene communication**: via `this.registry` only. UIScene calls `this.scene.get('GameScene').time.now` only for the power-up countdown timer.
- **Adding a level**: append an entry to `LEVELS`. Enemy wave is built from `{t: type, n: count}` objects; `ivl` is spawn interval in ms.
- **Adding an enemy type**: add to `ECFG`, add colors to `COLORS`, handle the shape in `genEnemies`, add an `ew_<type>` animation in `GameScene.create()`.
