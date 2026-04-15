# Aircraft Player Implementation Guide

## Overview
The ball player has been replaced with the aircraft 3D model from `assets/models/aircraft.glb`. The aircraft now serves as the player character that navigates the obstacle course.

## What Changed

### 1. **Aircraft Model Loading**
- New `loadAircraftModel()` function loads the aircraft GLTF model asynchronously
- Integrated with the existing model verification system
- Loaded during game initialization before the player is created

### 2. **Player Creation**
- `createBall()` now clones the aircraft model instead of creating a sphere
- Aircraft is scaled to 0.015 to match the game world scale
- Fallback to sphere if aircraft fails to load

### 3. **Physics & Collision**
- **No changes to physics** - Uses same velocity/gravity/friction system
- **No changes to collisions** - Uses generic bounding box detection
- **Rotation system** - Aircraft naturally banks and tilts with movement
- **Jumping** - Works exactly the same way

## How to Test

### 1. Open the Game
```
http://localhost:5500/index.html
```

### 2. Check Console Logs
```
[MODEL VERIFY] aircraft loaded: true
[GLTF] aircraft loaded successfully
[PLAYER] Aircraft player created
[MODEL VERIFY] aircraft-player rendered: 1 instances
```

### 3. Verify Model Loading
```javascript
// In browser console:
verifyModels()

// Should show:
// aircraft: true
// aircraft-player: 1 instance
```

### 4. Test Gameplay

**Controls:**
- **WASD / Arrow Keys** - Move aircraft
- **Space** - Jump
- **Collect** - Fly into golden octahedrons for points
- **Avoid** - Hit obstacles to lose lives

**Visual Checks:**
- Aircraft should be visible instead of red sphere
- Aircraft should be appropriately sized for the platforms
- Aircraft should rotate/bank when moving
- Aircraft should cast shadows
- Aircraft should be illuminated by scene lights

## Performance Considerations

### Model Size
- **Aircraft:** 41MB (GLTF compressed)
- **Load time:** ~2-5 seconds depending on connection
- **Rendering:** Same as any 3D model, minimal performance impact

### Scaling
- **Current scale:** 0.015x (aircraft is very large, needs significant scaling down)
- **Adjust if needed:** Edit line in `createBall()`:
  ```javascript
  ball.scale.setScalar(0.015); // Change this value to tune
  ```

## Troubleshooting

### Aircraft not appearing?

**Check 1: Model loaded?**
```javascript
console.log(aircraftTemplate ? 'Loaded' : 'Not loaded');
```

**Check 2: File exists?**
```bash
ls -lh assets/models/aircraft.glb
```

**Check 3: Scale too small/large?**
- Edit `ball.scale.setScalar(0.015)` to different value
- Try: 0.01, 0.02, 0.03, etc.

### Slow loading?

- Aircraft model is 41MB (large)
- Consider reducing model complexity or compressing GLTF
- Add loading bar if needed

### Aircraft orientation wrong?

- Edit rotation: `ball.rotation.y = 0;`
- Try: `Math.PI`, `-Math.PI/2`, etc.

## Gameplay Adjustments

### Rotation/Banking Feel

**Current (rolling motion):**
```javascript
ball.rotation.x -= ballVelocity.z * delta * 2;
ball.rotation.z += ballVelocity.x * delta * 2;
```

**Alternative (aircraft banking):**
```javascript
ball.rotation.x -= ballVelocity.z * delta * 1;  // Reduced pitch
ball.rotation.z = -ballVelocity.x * 0.15;      // Side banking
```

**To test:** Edit lines 1295-1296 in index.html

### Camera Perspective

**Current (close follow):**
```javascript
const cameraTargetY = ball.position.y + 8;
const cameraTargetZ = ball.position.z + 12;
```

**Alternative (cinematic back view):**
```javascript
const cameraTargetY = ball.position.y + 6;
const cameraTargetZ = ball.position.z + 20; // Further back
```

**To test:** Edit lines 1546-1548 in index.html

## Implementation Details

### Files Modified
- `index.html`: Aircraft loading and player creation

### Key Functions
- `loadAircraftModel(url)` - Loads aircraft GLTF model
- `createBall()` - Creates aircraft player instead of sphere

### Model System Integration
- Uses existing `ModelVerification` for load tracking
- Integrated with `SkeletonUtils.clone()` for safe model cloning
- Shadow casting and lighting enabled automatically

## Next Steps

### Optional Enhancements
1. **Adjust scale** for perfect fit to platforms
2. **Tune rotation** for desired banking feel
3. **Adjust camera** for preferred perspective
4. **Add engine sound** for immersion
5. **Particle effects** for flight/boost visuals

### Possible Issues to Monitor
1. Collision feel with elongated aircraft hitbox
2. Platform landing detection (AABB vs sphere)
3. Performance on lower-end devices (41MB model)

## Verification Command

```javascript
// Full verification report
verifyModels()

// Expected output should include:
// Aircraft loaded: true
// Aircraft-player rendered: 1
```

That's it! The aircraft is now your player character. Fly through the obstacle course! 🛩️
