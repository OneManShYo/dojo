# OMS Shader Baker Live V1.0.0 - Developer Documentation

**Version:** V1.0.0  
**Date:** November 2025  
**File:** ShaderBakerLive_V1_0_0.html (906 lines, ~37KB)

---

## Architecture Overview

### Single-File Application
- Pure HTML + CSS + JavaScript (no external dependencies)
- Self-contained, portable
- Runs entirely client-side
- No server required

### Core Systems
1. **UI Layer** - Performance grid, mixer controls, status messaging
2. **Communication Layer** - BroadcastChannel for inter-tab messaging
3. **Instance Management** - Discovery, tracking, assignment
4. **Visual Display** - Canvas snapshot integration

---

## File Structure

```
Lines 1-10: Version header comments
Lines 11-13: HTML doctype + head
Lines 14-400: <style> CSS (all styling)
Lines 401-500: <body> HTML structure
Lines 501-906: <script> JavaScript (all logic)
```

### HTML Structure

```html
<body>
  <div class="header">
    <!-- OMS Shader Baker Live branding -->
  </div>
  
  <div class="main-container">
    <div class="performance-view">
      <!-- Grid headers (fixed) -->
      <div class="grid-headers" id="gridHeaders"></div>
      
      <!-- Scrollable scenes -->
      <div class="scenes-scroll-container">
        <div class="scenes-grid" id="scenesGrid"></div>
      </div>
    </div>
    
    <!-- Mixer section (fixed) -->
    <div class="mixer-section">
      <div class="mixer-spacer"></div>
      <div id="mixer1" class="track-controls"></div>
      <div id="mixer2" class="track-controls"></div>
      <div id="mixer3" class="track-controls"></div>
      <div id="mixer4" class="track-controls"></div>
    </div>
    
    <!-- Waveform viewing -->
    <div class="waveform-section" id="waveformSection"></div>
  </div>
  
  <span id="liveStatus" class="status">Ready</span>
</body>
```

---

## CSS Architecture

### Layout System
- **Fixed width:** 1200px (non-responsive hardware paradigm)
- **Flexbox:** Main container, mixer layout
- **CSS Grid:** Performance grid, mixer section
- **Aspect ratio:** 4:3 landscape clip cells

### Key Classes

**Grid:**
- `.grid-headers` - Fixed track headers
- `.scenes-grid` - Scrollable scene rows
- `.clip-slot` - Individual clip cells (4:3 aspect)

**Mixer:**
- `.mixer-section` - Fixed bottom mixer (140px height)
- `.track-controls` - Per-track mixer column
- `.rotary-knob` - Canvas-based volume control
- `.meter-bar` - Vertical L/R audio meters

**Status:**
- `.status` - Footer messaging (cyan border/text)
- `.status.error` - Error state (magenta)
- `.status.info` - Info state (orange)

---

## JavaScript Architecture

### Global State

```javascript
// Instance discovery
const discoveredInstances = new Map(); 
// Format: instanceId -> {id, name, lastSeen, snapshot}

// Track assignments
const trackAssignments = {}; 
// Format: {1: 'sb-abc123', 2: 'sb-def456', ...}

// Volume state
const trackVolumes = [75, 75, 75, 75]; // Per track (0-100)

// BroadcastChannel
let liveChannel = null;
```

### Initialization Flow

```javascript
window.addEventListener('DOMContentLoaded', () => {
  1. initHeaders();        // Generate track headers
  2. initScenes();         // Generate 8 scene rows
  3. initMixer();          // Setup mixer controls
  4. initWaveforms();      // Setup waveform canvases
  5. initBroadcastChannel(); // Start listening
  6. Console log ready message
});
```

### Key Functions

**Grid Generation:**
- `initHeaders()` - Creates corner + 4 track headers
- `initScenes()` - Creates 8 scenes × 5 columns (40 cells)
- Track headers include instance dropdown

**Mixer:**
- `initMixer()` - Populates 4 track mixer controls
- `drawRotaryKnob(canvas, value)` - Renders knob visual
- Mouse event handlers for knob drag interaction

**BroadcastChannel:**
- `initBroadcastChannel()` - Creates 'oms-live-channel'
- `liveChannel.onmessage` - Handles announcements + snapshots
- `updateInstanceDropdowns()` - Populates track assignment dropdowns
- `assignInstanceToTrack(trackIndex, instanceId)` - Maps instance to track
- `updateGridSnapshots()` - Displays canvas images in grid

**Scene Management:**
- `launchScene(sceneIndex)` - Highlights active scene, updates status
- Scene trigger buttons call this function

---

## BroadcastChannel Protocol

### Channel Name
```javascript
const liveChannel = new BroadcastChannel('oms-live-channel');
```

### Message Types

**1. Announce (Received from Shader Baker)**
```javascript
{
  type: 'announce',
  id: 'sb-abc123',          // Unique instance ID
  name: 'Bass Tunnel v3',   // Shader name
  timestamp: 1699564800000
}
```

**Purpose:** Instance discovery, heartbeat  
**Frequency:** On load + every 5 seconds  
**Action:** Store instance, update dropdowns

**2. Snapshot (Received from Shader Baker)**
```javascript
{
  type: 'snapshot',
  id: 'sb-abc123',
  imageData: 'data:image/jpeg;base64,...', // Base64 JPEG
  timestamp: 1699564800000
}
```

**Purpose:** Canvas visual preview  
**Frequency:** On shader compile  
**Action:** Store snapshot, update grid backgrounds

### Instance Management

**Discovery:**
1. Shader Baker announces → stored in `discoveredInstances` Map
2. Track dropdowns populated with available instances
3. User selects instance from dropdown
4. `assignInstanceToTrack()` called

**Heartbeat:**
- 30-second timeout for stale instances
- `cleanupStaleInstances()` runs every 5 seconds
- Removes instances not seen in 30+ seconds

**Storage:**
```javascript
discoveredInstances.set(instanceId, {
  id: 'sb-abc123',
  name: 'Bass Tunnel v3',
  lastSeen: Date.now(),
  snapshot: 'data:image/jpeg;base64,...'
});
```

---

## Canvas Snapshot Integration

### Display Logic

```javascript
function updateGridSnapshots() {
  const clipSlots = document.querySelectorAll('.clip-slot');
  
  clipSlots.forEach(slot => {
    const trackIndex = parseInt(slot.dataset.track);
    const instanceId = trackAssignments[trackIndex + 1];
    
    if (instanceId) {
      const instance = discoveredInstances.get(instanceId);
      if (instance?.snapshot) {
        slot.style.backgroundImage = `url(${instance.snapshot})`;
        slot.style.backgroundSize = 'cover';
        slot.style.backgroundPosition = 'center';
        slot.textContent = '';
        slot.classList.add('has-snapshot');
      }
    } else {
      slot.style.backgroundImage = '';
      slot.textContent = 'Empty';
      slot.classList.remove('has-snapshot');
    }
  });
}
```

**When Called:**
- After track assignment change
- When snapshot message received
- Updates all cells in assigned track columns

---

## Rotary Knob Rendering

### Canvas Drawing

```javascript
function drawRotaryKnob(canvas, value) {
  const ctx = canvas.getContext('2d');
  const centerX = 30, centerY = 30, radius = 25;
  
  // Clear canvas
  ctx.clearRect(0, 0, 60, 60);
  
  // Background circle
  ctx.beginPath();
  ctx.arc(centerX, centerY, radius, 0, Math.PI * 2);
  ctx.fillStyle = '#333';
  ctx.fill();
  ctx.strokeStyle = '#555';
  ctx.lineWidth = 2;
  ctx.stroke();
  
  // Value arc (270° range)
  const startAngle = (135 * Math.PI) / 180;
  const angleRange = (270 * Math.PI) / 180;
  const currentAngle = startAngle + (value / 100) * angleRange;
  
  ctx.beginPath();
  ctx.arc(centerX, centerY, radius - 3, startAngle, currentAngle);
  ctx.strokeStyle = '#00d4ff'; // Cyan
  ctx.lineWidth = 4;
  ctx.stroke();
  
  // Center indicator line
  ctx.save();
  ctx.translate(centerX, centerY);
  ctx.rotate(currentAngle);
  ctx.beginPath();
  ctx.moveTo(0, -radius + 8);
  ctx.lineTo(0, -radius + 18);
  ctx.strokeStyle = '#fff';
  ctx.lineWidth = 2;
  ctx.stroke();
  ctx.restore();
}
```

### Mouse Interaction

```javascript
canvas.addEventListener('mousedown', (e) => {
  isDragging = true;
  startY = e.clientY;
  startValue = trackVolumes[trackIndex];
});

document.addEventListener('mousemove', (e) => {
  if (!isDragging) return;
  
  const deltaY = startY - e.clientY; // Inverted (up = increase)
  const newValue = Math.max(0, Math.min(100, startValue + deltaY * 0.5));
  
  trackVolumes[trackIndex] = newValue;
  drawRotaryKnob(canvas, newValue);
  updateVolumeLabel(trackIndex, newValue);
});

document.addEventListener('mouseup', () => {
  isDragging = false;
});
```

---

## Extension Points

### Adding New Message Types

```javascript
liveChannel.onmessage = (event) => {
  const msg = event.data;
  
  switch(msg.type) {
    case 'announce':
      // Handle instance announcement
      break;
    case 'snapshot':
      // Handle canvas snapshot
      break;
    case 'YOUR_NEW_TYPE':
      // Add your handler here
      break;
  }
};
```

### Adding Track Controls

```javascript
function initMixer() {
  for (let i = 0; i < 4; i++) {
    const mixer = document.getElementById(`mixer${i + 1}`);
    
    mixer.innerHTML = `
      <!-- Add new controls here -->
      <button class="your-button">New Feature</button>
    `;
    
    // Add event listeners
  }
}
```

### Customizing Grid Size

```javascript
// Change number of scenes
const NUM_SCENES = 16; // Was 8

// Update initScenes() to generate more rows
```

---

## Browser Compatibility

**Required Features:**
- BroadcastChannel API
- CSS Grid
- CSS aspect-ratio
- Canvas 2D context
- ES6 JavaScript

**Tested Browsers:**
- Chrome 88+ ✓
- Safari 15+ ✓
- Firefox 89+ ✓
- Edge 88+ ✓

---

## Performance Notes

### File Size
- HTML: ~37KB
- Single file (no external resources)
- Fast load time
- Works offline

### Memory Usage
- Lightweight (no video streaming)
- Static canvas snapshots only
- ~100KB per instance snapshot
- Typical usage: 4 instances = ~400KB

### Rendering
- CSS Grid hardware accelerated
- Canvas knobs: 60fps capable
- Background images: GPU accelerated
- No performance concerns

---

## Debugging

### Console Logging

```javascript
// Initialization
"OMS Shader Baker Live V1.0.0 initialized"

// Instance discovery
"Instance discovered: sb-abc123 - Bass Tunnel v3"
"Track 1 assigned to instance: sb-abc123"

// Snapshots
"Snapshot received from: sb-abc123"

// Scene launching
"Scene 1 launched"
```

### Common Issues

**Instances not discovered:**
- Check both tabs open (Live + Baker)
- Verify Shader Baker V2.8.0+ (has broadcasting)
- Check browser console for errors

**Snapshots not displaying:**
- Verify track assignment (dropdown selected)
- Check console for "Snapshot received" message
- Verify Shader Baker compiled shader

---

## Code Style

- Vanilla JavaScript (no frameworks)
- camelCase for variables/functions
- Comments for major sections
- Consistent indentation (4 spaces)
- Descriptive variable names

---

**End of Developer Documentation**
