# assignment1-exercise4
## Description
This is your first ‘free-style’ exercise in this course. 
Here is a code of a 3d game that I created. Make it “your own”.
You decide what to do here in this exercise. No instructions
from me. 
Feel free to change anything, add/remove anything etc. It doesn’t need to be even
similar to my game if you want major changes. Again, this is completely up to you. Also here
we will have a competition between all of your games so make it cool and fun.
Publish it on Netlify drop when done.

## Feature TODO List

Build in order (easiest → most complex). Commit after each checked-off feature works.

### 1) Core Game Flow
- [x] `Start` button initialises game; state resets cleanly on restart
- [x] Death (fall / enemy) shows `GAME OVER` + `Play again` button
- [x] No broken state after multiple start → die → restart cycles

### 2) Scoring System (speed rewarded)
- [x] Live score displayed during gameplay
- [x] Score increases with upward progress (platform reached)
- [x] Time-based bonus: faster climbs yield higher score

### 3) Leaderboard (Top 10)
- [x] Top 10 scores saved in `localStorage`
- [x] Leaderboard rendered on game over screen
- [x] Scores persist after page refresh

### 4) Platform Labels + Layout Rules
- [x] Platforms divisible by 10 show a visible number label (`10`, `20`, …)
- [x] Some platforms are wider than default
- [x] No two platforms overlap or touch
- [x] **Bonus**: Random platforms marked with +50 bonus labels award points when landed on

### 5) Falling Platforms
- [ ] Standing on a platform > 4 seconds makes it fall
- [ ] Visual warning before fall (shake / color change)

### 6) Moving Platforms (6%)
- [ ] 6% of platforms move up and down
- [ ] Speed and amplitude tuned for fair gameplay

### 7) Enemy Spawning (5%, weighted types)
- [ ] 5% of platforms have an enemy
- [ ] Type1 50% — static
- [ ] Type2 40% — side-to-side patrol
- [ ] Type3 10% — static shooter
- [ ] Touching an enemy kills the player

### 8) Enemy Behaviors
- [ ] Type1: stands still
- [ ] Type2: moves left ↔ right within platform bounds
- [ ] Type3: shoots bullets toward the player
- [ ] All three types are visually distinct

### 9) Powerups
- [ ] Star powerup: 6 s invincibility, kills enemies on touch
- [ ] Spring on 4% of platforms: one super-high bounce
- [ ] Visual indicators (glow / icon) for active powerup timer

### 10) Visual Upgrade (image assets)
- [ ] Background image
- [ ] Player sprite / texture
- [ ] Enemy sprites (one per type)
- [ ] Platform texture

### 11) Audio Upgrade
- [ ] Background music loop
- [ ] SFX: jump, hit, collect, enemy shot, game over, button click
- [ ] Volume balanced; no sound spam on rapid events

### 12) Mobile Support
- [ ] Responsive canvas + UI scaling
- [ ] On-screen touch controls (joystick / buttons)
- [ ] Verified in Chrome DevTools device emulation

### 13) Camera Control Mode (MediaPipe)
- [ ] `Start with camera` button on start screen
- [ ] MediaPipe hand-tracking drives player movement
- [ ] Gestures tuned until fun and stable

### 14) Three Unique Features
- [ ] Feature A: _(describe here)_
- [ ] Feature B: _(describe here)_
- [ ] Feature C: _(describe here)_

---

## Automated Testing Strategy

Since this is a single-file browser game with no build step, use an in-game **debug / self-test module** that runs automatically and logs results to the console.

### How it works

Add a global `GameDebug` object (gated behind a `?debug=1` URL param so it never runs in production). It hooks into game events, simulates actions, and asserts expected outcomes.

### Test categories

| # | Category | What it checks | Method |
|---|----------|---------------|--------|
| 1 | **Spawn audit** | Enemy ratio ≈ 50/40/10 %; enemy count ≈ 5 % of platforms; spring ≈ 4 %; moving platforms ≈ 6 % | After `createCourse()`, iterate arrays, count types, log pass/fail |
| 2 | **Platform overlap** | No two platform bounding boxes intersect | Pairwise `Box3.intersectsBox` check on all platforms |
| 3 | **Platform labels** | Every 10th platform has a visible text label | Check `platformLabels` array length matches expected count |
| 4 | **Falling platform timer** | Standing > 4 s triggers fall | Simulate: set `platform.standTime = 4.1`, call `update()`, assert platform is falling |
| 5 | **Score speed bonus** | Reaching platform N faster gives higher score | Run two timed simulations with different elapsed times, compare scores |
| 6 | **Leaderboard persistence** | Scores survive page reload | Write mock scores to `localStorage`, read back, assert match |
| 7 | **Star powerup** | Player survives enemy contact during star | Set `starActive = true`, trigger enemy collision, assert player alive |
| 8 | **Spring bounce** | Stepping on spring produces higher jump velocity | Trigger spring collision, assert `ballVelocity.y > jumpForce` |
| 9 | **Game over flow** | Death shows game over; restart resets all state | Trigger death, assert screen visible; call restart, assert score/lives reset |
| 10 | **Mobile touch** | Touch event listeners are registered | Assert `ontouchstart` handlers exist on control elements |

### Implementation sketch

```js
// Add to end of index.html <script>, inside init()
if (new URLSearchParams(location.search).get('debug') === '1') {
  window.GameDebug = {
    results: [],
    assert(name, condition) {
      const status = condition ? 'PASS' : 'FAIL';
      this.results.push({ name, status });
      console[condition ? 'log' : 'error'](`[TEST ${status}] ${name}`);
    },
    runAll() {
      // 1 — Spawn audit
      const totalPlatforms = platforms.length;
      const enemyCount = enemies.length;
      this.assert('Enemy count ≈ 5%', Math.abs(enemyCount / totalPlatforms - 0.05) < 0.03);

      const t1 = enemies.filter(e => e.type === 1).length;
      const t2 = enemies.filter(e => e.type === 2).length;
      const t3 = enemies.filter(e => e.type === 3).length;
      this.assert('Type1 ≈ 50%', Math.abs(t1 / enemyCount - 0.5) < 0.15);
      this.assert('Type2 ≈ 40%', Math.abs(t2 / enemyCount - 0.4) < 0.15);
      this.assert('Type3 ≈ 10%', Math.abs(t3 / enemyCount - 0.1) < 0.1);

      // 2 — Platform overlap
      let overlapFound = false;
      for (let i = 0; i < platforms.length; i++) {
        for (let j = i + 1; j < platforms.length; j++) {
          if (platforms[i].box.intersectsBox(platforms[j].box)) overlapFound = true;
        }
      }
      this.assert('No platform overlaps', !overlapFound);

      // 3 — Leaderboard persistence
      const key = '__test_lb';
      localStorage.setItem(key, JSON.stringify([100, 200]));
      const read = JSON.parse(localStorage.getItem(key));
      this.assert('localStorage read/write', read.length === 2);
      localStorage.removeItem(key);

      // Summary
      const passed = this.results.filter(r => r.status === 'PASS').length;
      console.log(`\n=== ${passed}/${this.results.length} tests passed ===`);
    }
  };
  // Run after course is built
  setTimeout(() => GameDebug.runAll(), 500);
}
```

### Running the tests

Open the game with `?debug=1` appended to the URL:
```
http://localhost:5500/index.html?debug=1
```
Open DevTools → Console to see `[TEST PASS]` / `[TEST FAIL]` lines and a summary.

## Suggested Commit Sequence
1. `feat: core start and game over flow`
2. `feat: scoring with speed bonus`
3. `feat: leaderboard top 10`
4. `feat: platform labels widths and non-overlap`
5. `feat: falling platforms after 4s`
6. `feat: vertical moving platforms`
7. `feat: enemy spawn system with weighted types`
8. `feat: enemy movement and shooter bullets`
9. `feat: star invincibility and spring boost`
10. `feat: texture and image asset pass`
11. `feat: music and sound effects`
12. `feat: responsive mobile controls`
13. `feat: mediapipe camera control mode`
14. `feat: three unique gameplay additions`
15. `test: add GameDebug self-test module`
