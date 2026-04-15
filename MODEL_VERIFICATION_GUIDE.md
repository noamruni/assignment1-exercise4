# Model Verification Guide

## Overview
This document explains the model loading and rendering verification system implemented for the jellyfish pillar decorations.

## What Changed

### 1. **Promise-Based Model Loading**
The `loadJellyModel()` function now returns a Promise, ensuring the model is fully loaded before the course is created.

```javascript
// Old: Non-blocking, model may not load in time
loadJellyModel('assets/models/jellyfish.glb');
createCourse(); // May run before model loads!

// New: Blocking, model guaranteed loaded
await loadJellyModel('assets/models/jellyfish.glb');
createCourse(); // Now safe to create pillars
```

### 2. **Model Verification System**
A new `ModelVerification` object tracks:
- **Loaded models**: Which models loaded successfully and when
- **Rendered models**: How many instances of each model were rendered

```javascript
const ModelVerification = {
    loadedModels: {},      // { 'jellyfish': { success: true, timestamp: 1234567890 } }
    renderedModels: {},    // { 'jellyfish-pillars': { instanceCount: 50, timestamp: 1234567890 } }
    markLoaded(name, success, details),
    markRendered(name, count),
    getReport(),
    logReport()
}
```

### 3. **Pillar Creation**
- All 50 decorative pillars now use the jellyfish model (no cylinder fallback)
- Cylinder fallback removed entirely
- Random scale variation (0.8x to 1.6x) for visual diversity

## How to Verify

### Console Commands (Browser DevTools)

**Check full verification report:**
```javascript
verifyModels()
```

**Get report as JSON:**
```javascript
getModelReport()
```

**Expected output:**
```
=== MODEL VERIFICATION REPORT ===
┌─────────────────────────────────────────────────────────────┐
│ (index)   │ success │ timestamp        │ source / instanceCount │
├─────────────────────────────────────────────────────────────┤
│ jellyfish │ true    │ 1723456789123    │ assets/models/...     │
└─────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────┐
│ (index)           │ instanceCount │ timestamp         │
├──────────────────────────────────────────────────────────┤
│ jellyfish-pillars │ 50           │ 1723456789150     │
└──────────────────────────────────────────────────────────┘
Summary: Loaded: 1, Rendered: 1
```

### Console Logs to Check

**On page load**, you should see:
```
[MODEL VERIFY] jellyfish loaded: true {source: "assets/models/jellyfish.glb"}
[GLTF] jelly loaded successfully
[MODEL VERIFY] jellyfish-pillars rendered: 50 instances
```

### Visual Verification

1. Open http://localhost:5500/index.html
2. Look for jellyfish models scattered around the course (not gray cylinders)
3. They should have varying scales and positions
4. They should cast shadows and be illuminated by lights

## Troubleshooting

### Jellyfishes not appearing?

**Issue:** Still seeing cylinders or nothing
**Debug steps:**
```javascript
// Check if model loaded
verifyModels()

// Check if template exists
console.log(jellyTemplate ? 'Template loaded' : 'Template missing')

// Check instance count
console.log(jellyInstances.length)
```

### Model failed to load?

**Issue:** Console shows `jellyfish loaded: false`
**Possible causes:**
- File not found at `assets/models/jellyfish.glb`
- Network error (check Network tab in DevTools)
- CORS issue (if serving from different domain)

**Fix:**
```bash
# Verify file exists
ls -lh assets/models/jellyfish.glb

# Check file is readable
file assets/models/jellyfish.glb
```

### Performance issues?

**Issue:** Game runs slow with 50 jellyfishes
**Debug:**
```javascript
// Check instance count
console.log(`Jelly instances: ${jellyInstances.length}`)

// Check scene object count
console.log(`Scene children: ${scene.children.length}`)

// Reduce number if needed - edit createCourse()
for (let i = 0; i < 25; i++) { // Change 50 to 25
```

## Implementation Details

### Files Modified
- `index.html`: Lines 107-189 (verification system), 351-443 (async init), 958-968 (pillar creation)

### Key Functions
- `ModelVerification.markLoaded(name, success, details)` - Log model load status
- `ModelVerification.markRendered(name, count)` - Log rendering stats
- `ModelVerification.logReport()` - Print verification report to console
- `loadJellyModel(url)` - Promise-based model loader
- `spawnJelly(x, y, z, scale)` - Create jellyfish instance

### Exposed to Window (for debugging)
- `window.verifyModels()` - Print full report
- `window.getModelReport()` - Get report as JavaScript object

## Testing in Debug Mode

Run with `?debug=1` URL parameter:
```
http://localhost:5500/index.html?debug=1
```

The debug module will run automated tests. Check browser console for test results including model verification.
