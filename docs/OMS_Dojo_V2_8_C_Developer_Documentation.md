# OMS DOJO V2_8-N - DEVELOPER DOCUMENTATION

**Version:** V2_8-N  
**Last Updated:** November 2025  
**Purpose:** Technical architecture and API reference

---

## ARCHITECTURE OVERVIEW

### Three Major Systems

**1. Core Shader System (V2.7.0 base)**
- WebGPU rendering engine
- WGSL fragment shader compilation
- FFT audio analysis (32768 samples)
- Control States management
- Preset system

**2. BroadcastChannel Communication (V2_8 A-C)**
- Instance discovery and identification
- Canvas snapshot broadcasting
- Heartbeat keep-alive
- Foundation for DojoLive integration

**3. Container Library (V2_8 F-N)**
- File System Access API for audio library
- localStorage for container metadata
- IndexedDB for permission storage
- Export/import .oms format

---

## BROADCASTCHANNEL PROTOCOL

### Channel Configuration

```javascript
const liveChannel = new BroadcastChannel('oms-live-channel');
```

**Channel Name:** `'oms-live-channel'`  
**Scope:** Same origin (localhost or HTTPS)  
**Lifetime:** Tab session

### Instance Identification

**ID Generation:**
```javascript
const instanceId = 'sb-' + Math.random().toString(36).substring(2, 11);
// Example: 'sb-k7f2x9p3q'
```

**Format:** 9-character alphanumeric suffix  
**Purpose:** Unique identifier for DojoLive track assignment

### Message Types

**1. Announce Message**
```javascript
{
  type: 'announce',
  id: 'sb-abc123',
  name: 'Bass Tunnel v3',  // from first comment line
  timestamp: 1699564800000
}
```

**Sent When:**
- Page load (immediate)
- Shader compile (successful)
- Heartbeat (every 5 seconds)

**2. Snapshot Message**
```javascript
{
  type: 'snapshot',
  id: 'sb-abc123',
  imageData: 'data:image/jpeg;base64,/9j/4AAQSkZJRg...',
  timestamp: 1699564800000
}
```

**Sent When:**
- Page load (500ms delay)
- Shader compile (100ms delay)

**Image Specs:**
- Format: JPEG
- Quality: 80% (0.8)
- Encoding: Base64
- Size: ~50-100KB
- Resolution: Canvas size (typically 800×600)

### Code Implementation

**Initialization:**
```javascript
// Create channel
const liveChannel = new BroadcastChannel('oms-live-channel');
let isBroadcasting = true;

// Generate ID
const instanceId = 'sb-' + Math.random().toString(36).substring(2, 11);

// Setup listener
liveChannel.onmessage = (event) => {
  const msg = event.data;
  if (msg.target === instanceId) {
    // Handle control messages (future)
  }
};

// Cleanup
window.addEventListener('beforeunload', () => {
  if (liveChannel) liveChannel.close();
});
```

**Announce Function:**
```javascript
function announcePresence() {
  if (!isBroadcasting || !liveChannel) return;
  
  const shaderName = getShaderNameFromCode();
  liveChannel.postMessage({
    type: 'announce',
    id: instanceId,
    name: shaderName,
    timestamp: Date.now()
  });
}
```

**Snapshot Function:**
```javascript
function broadcastSnapshot() {
  if (!canvas || !isBroadcasting || !liveChannel) return;
  
  try {
    const imageData = canvas.toDataURL('image/jpeg', 0.8);
    liveChannel.postMessage({
      type: 'snapshot',
      id: instanceId,
      imageData: imageData,
      timestamp: Date.now()
    });
  } catch (err) {
    console.warn('Snapshot failed:', err);
  }
}
```

**Heartbeat:**
```javascript
function startHeartbeat() {
  setInterval(() => {
    announcePresence();
  }, 5000);
}
```

---

## FILE SYSTEM ACCESS API

### Purpose

Replace localStorage-embedded audio with filesystem references. Enables scaling to 10,000+ tracks like professional DJ software.

### Workflow

**1. Setup Audio Library (One-Time):**
```javascript
async function setupAudioLibrary() {
  try {
    // Request directory picker
    const handle = await window.showDirectoryPicker({
      mode: 'read',
      startIn: 'music'
    });
    
    // Store handle in IndexedDB
    await storeAudioLibraryHandle(handle);
    
    // Update UI
    audioLibraryHandle = handle;
    document.getElementById('audioLibraryInfo').style.display = 'block';
    document.getElementById('audioLibraryFolderName').textContent = handle.name;
    
  } catch (err) {
    if (err.name !== 'AbortError') {
      console.error('Setup failed:', err);
    }
  }
}
```

**2. Store Handle in IndexedDB:**
```javascript
async function storeAudioLibraryHandle(handle) {
  const db = await openLibraryDB();
  const tx = db.transaction(['settings'], 'readwrite');
  const store = tx.objectStore('settings');
  await store.put(handle, 'audioLibraryHandle');
}
```

**3. Load Handle on Page Load:**
```javascript
async function loadAudioLibraryHandle() {
  try {
    const db = await openLibraryDB();
    const tx = db.transaction(['settings'], 'readonly');
    const store = tx.objectStore('settings');
    const handle = await store.get('audioLibraryHandle');
    
    if (handle) {
      // Verify permission still valid
      if ((await handle.queryPermission({ mode: 'read' })) === 'granted') {
        audioLibraryHandle = handle;
        // Update UI
      }
    }
  } catch (err) {
    console.error('Load handle failed:', err);
  }
}
```

**4. Read Audio File:**
```javascript
async function getAudioFromLibrary(filename) {
  if (!audioLibraryHandle) {
    throw new Error('No audio library configured');
  }
  
  try {
    const fileHandle = await audioLibraryHandle.getFileHandle(filename);
    const file = await fileHandle.getFile();
    return file;
  } catch (err) {
    throw new Error(`Audio not found: ${filename}`);
  }
}
```

### IndexedDB Schema

**Database:** `'OMSDojo'`  
**Version:** `1`

**Object Store:** `'settings'`  
**Key:** `'audioLibraryHandle'`  
**Value:** `DirectoryHandle` object

```javascript
function openLibraryDB() {
  return new Promise((resolve, reject) => {
    const request = indexedDB.open('OMSDojo', 1);
    
    request.onupgradeneeded = (event) => {
      const db = event.target.result;
      if (!db.objectStoreNames.contains('settings')) {
        db.createObjectStore('settings');
      }
    };
    
    request.onsuccess = () => resolve(request.result);
    request.onerror = () => reject(request.error);
  });
}
```

### Browser Support

**Supported:**
- Chrome 86+ (stable since 2020)
- Edge 86+
- Safari 15.2+ (stable since 2023)
- Opera 72+
- Brave 1.17+

**Not Supported:**
- Firefox (no plans to implement)

**Fallback Strategy:**
- Show error: "Use Chrome/Safari for audio library feature"
- Manual file loading still works

---

## CONTAINER SYSTEM

### Container Structure

```javascript
{
  // Identity
  id: '1731277200000',           // timestamp
  name: 'Bass Tunnel v3',        // user-defined
  created: '2025-11-10T...',     // ISO timestamp
  
  // Shader
  shaderCode: '// Bass tunnel...\n...',  // WGSL code
  
  // Control States
  controlStates: {
    shaderTempo: 125,
    loopLength: 16,
    renderDuration: 30000,
    modulatorA: { subdivision: 16, waveform: 'triangle', gain: 0.5 },
    modulatorB: { subdivision: 32, waveform: 'sine', gain: 0.3 },
    modulatorC: { subdivision: 8, waveform: 'saw', gain: 0.7 },
    freq1A: 30,
    freq2A: 50,
    freq1B: 50,
    freq2B: 75,
    freq1C: 100,
    freq2C: 500,
    audioTempo: 125,
    audioPitch: 0,
    outputFilename: 'shader',
    outputResolution: 1920,
    outputFPS: 30,
    outputFormat: 'webm'
  },
  
  // Audio (File System Access API mode)
  audioFilename: 'bass_loop_128.mp3',
  audioBPM: 128.5,
  hasAudio: true,
  
  // Audio (Legacy mode - backward compatible)
  audioData: 'data:audio/mpeg;base64,...',  // optional
}
```

### Storage Strategy

**Metadata:** localStorage (`'omsLibrary'` key)
```javascript
{
  containers: [
    { id, name, created, shaderCode, controlStates, audioFilename, audioBPM, hasAudio },
    // ... more containers
  ]
}
```

**Audio Files:** User's filesystem (via File System Access API)

**Permissions:** IndexedDB (`'audioLibraryHandle'` key)

### API Functions

**Get All Containers:**
```javascript
function getLibraryContainers() {
  try {
    const data = localStorage.getItem('omsLibrary');
    return data ? JSON.parse(data).containers : [];
  } catch (err) {
    console.error('Failed to load library:', err);
    return [];
  }
}
```

**Save Containers:**
```javascript
function saveLibraryContainers(containers) {
  try {
    localStorage.setItem('omsLibrary', JSON.stringify({ containers }));
    return true;
  } catch (err) {
    console.error('Failed to save library:', err);
    return false;
  }
}
```

**Add Container:**
```javascript
async function confirmAddToLibrary() {
  const name = document.getElementById('containerNameInput').value.trim();
  const shaderCode = document.getElementById('shaderCode').value;
  const controlStates = getCurrentControlState();
  
  // Audio handling
  let audioFilename = null;
  let audioBPM = null;
  let hasAudio = false;
  
  if (audioElement && audioElement.src && currentAudioFile) {
    audioFilename = currentAudioFile.name;
    audioBPM = audioTempo;
    hasAudio = true;
  }
  
  const container = {
    id: Date.now().toString(),
    name: name,
    shaderCode: shaderCode,
    controlStates: controlStates,
    audioFilename: audioFilename,
    audioBPM: audioBPM,
    hasAudio: hasAudio,
    created: new Date().toISOString()
  };
  
  const containers = getLibraryContainers();
  containers.push(container);
  saveLibraryContainers(containers);
  refreshLibraryList();
}
```

**Load Container:**
```javascript
async function loadContainer(containerId) {
  const containers = getLibraryContainers();
  const container = containers.find(c => c.id === containerId);
  
  if (!container) return;
  
  // Load shader
  document.getElementById('shaderCode').value = container.shaderCode;
  
  // Apply states
  applyControlState(container.controlStates);
  
  // Load audio (File System Access API)
  if (container.audioFilename && audioLibraryHandle) {
    try {
      const file = await getAudioFromLibrary(container.audioFilename);
      await loadAudioFile(file);
    } catch (err) {
      showStatus(`Audio not found: ${container.audioFilename}`, 'error');
    }
  }
  // Fallback: Load audio (legacy embedded)
  else if (container.audioData) {
    const response = await fetch(container.audioData);
    const blob = await response.blob();
    const file = new File([blob], 'audio.mp3', { type: 'audio/mpeg' });
    await loadAudioFile(file);
  }
  
  // Compile
  compileShader();
}
```

---

## .OMS FILE FORMAT

### Structure

```javascript
{
  "format": "oms-container",
  "version": "1.0",
  "container": {
    // Full container object (with audioData embedded)
  }
}
```

### Export

```javascript
function exportContainer(containerId) {
  const containers = getLibraryContainers();
  const container = containers.find(c => c.id === containerId);
  
  if (!container) return;
  
  // For export: embed audio as base64 (portability)
  const exportContainer = { ...container };
  
  if (container.audioFilename && audioLibraryHandle) {
    // Read file and convert to base64
    const file = await getAudioFromLibrary(container.audioFilename);
    const reader = new FileReader();
    reader.onload = (e) => {
      exportContainer.audioData = e.target.result;
      delete exportContainer.audioFilename;
      
      // Create .oms file
      const omsData = {
        format: 'oms-container',
        version: '1.0',
        container: exportContainer
      };
      
      const blob = new Blob([JSON.stringify(omsData, null, 2)], 
                           { type: 'application/json' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = `${container.name.replace(/[^a-z0-9]/gi, '_')}.oms`;
      a.click();
    };
    reader.readAsDataURL(file);
  }
}
```

### Import

```javascript
async function handleOmsFileImport(event) {
  const file = event.target.files[0];
  if (!file) return;
  
  const reader = new FileReader();
  reader.onload = (e) => {
    try {
      const data = JSON.parse(e.target.result);
      
      // Validate format
      if (data.format !== 'oms-container' || !data.container) {
        showStatus('Invalid .oms file format', 'error');
        return;
      }
      
      // Generate new ID and timestamp
      const container = {
        ...data.container,
        id: Date.now().toString(),
        created: new Date().toISOString()
      };
      
      // Add to library
      const containers = getLibraryContainers();
      containers.push(container);
      saveLibraryContainers(containers);
      refreshLibraryList();
      
      showStatus(`Imported: ${container.name}`, 'success');
      
    } catch (err) {
      showStatus('Failed to import .oms file', 'error');
    }
  };
  reader.readAsText(file);
}
```

---

## AUDIO PITCH CONTROL

### Implementation

```javascript
let audioPitch = 0;  // ±16%

function applyPitchToAudio() {
  if (!audioElement || !audioElement.src) return;
  
  // Calculate playback rate
  const playbackRate = 1.0 + (audioPitch / 100.0);
  audioElement.playbackRate = playbackRate;
  
  // Update BPM display
  const adjustedBPM = audioTempo * playbackRate;
  document.getElementById('audioTempoField').value = adjustedBPM.toFixed(2);
}
```

### Arrow Key Navigation

```javascript
document.getElementById('pitchField').addEventListener('keydown', (e) => {
  if (e.key === 'ArrowUp') {
    e.preventDefault();
    audioPitch = Math.min(16, audioPitch + 0.1);
  } else if (e.key === 'ArrowDown') {
    e.preventDefault();
    audioPitch = Math.max(-16, audioPitch - 0.1);
  } else if (e.key === 'ArrowRight') {
    e.preventDefault();
    audioPitch = Math.min(16, audioPitch + 1.0);
  } else if (e.key === 'ArrowLeft') {
    e.preventDefault();
    audioPitch = Math.max(-16, audioPitch - 1.0);
  } else {
    return;
  }
  
  pitchField.value = audioPitch.toFixed(2);
  applyPitchToAudio();
  markAsCustomState();
});
```

### Control State Integration

```javascript
function getCurrentControlState() {
  return {
    // ... other states
    audioPitch: audioPitch,
  };
}

function applyControlState(state) {
  // ... other states
  
  audioPitch = state.audioPitch || 0;
  document.getElementById('pitchField').value = audioPitch.toFixed(2);
  applyPitchToAudio();
}
```

---

## PERFORMANCE CONSIDERATIONS

### BroadcastChannel
- Minimal overhead (~1KB messages)
- 5-second heartbeat (low CPU)
- JPEG snapshots optimized (80% quality, ~50-100KB)

### File System Access API
- Read operations only (no writes)
- Fast local disk access (milliseconds)
- Permission cached in IndexedDB

### localStorage
- Containers without audio: ~5KB each
- 100+ containers fit comfortably in 5-10MB limit
- Audio files externalized (no quota issues)

### Memory
- Single WebGPU context (efficient)
- Audio buffer reused (not duplicated)
- Canvas snapshots generated on-demand (not stored)

---

## BROWSER COMPATIBILITY MATRIX

| Feature | Chrome | Safari | Edge | Firefox |
|---------|--------|--------|------|---------|
| WebGPU | ✓ 88+ | ✓ 15+ | ✓ 88+ | ✓ Nightly |
| Web Audio | ✓ | ✓ | ✓ | ✓ |
| BroadcastChannel | ✓ | ✓ | ✓ | ✓ |
| File System Access | ✓ 86+ | ✓ 15.2+ | ✓ 86+ | ✗ |
| IndexedDB | ✓ | ✓ | ✓ | ✓ |

**Recommended:** Chrome 88+ or Safari 15+ for full functionality

---

## ERROR HANDLING

**Missing Audio Library:**
```javascript
if (!audioLibraryHandle) {
  showStatus('Setup Audio Library first', 'error');
  return;
}
```

**File Not Found:**
```javascript
try {
  const file = await getAudioFromLibrary(filename);
} catch (err) {
  showStatus(`Audio not found: ${filename}`, 'error');
  // Continue loading shader/states without audio
}
```

**Permission Denied:**
```javascript
const permission = await handle.queryPermission({ mode: 'read' });
if (permission !== 'granted') {
  const newPermission = await handle.requestPermission({ mode: 'read' });
  if (newPermission !== 'granted') {
    showStatus('Permission denied', 'error');
    return;
  }
}
```

**localStorage Quota:**
```javascript
try {
  localStorage.setItem('omsLibrary', JSON.stringify({ containers }));
} catch (err) {
  if (err.name === 'QuotaExceededError') {
    showStatus('Storage full - remove old containers', 'error');
  }
}
```

---

## TESTING GUIDELINES

### BroadcastChannel Testing
1. Open 2+ Dojo tabs
2. Each gets unique ID
3. Open DojoLive
4. Verify all instances discovered
5. Assign instances to tracks
6. Compile shaders, verify snapshots

### File System Access API Testing
1. Click Setup Audio Library
2. Pick folder with audio files
3. Create container with audio
4. Close browser, reopen
5. Verify folder permission persists
6. Load container, verify audio loads

### Container Export/Import Testing
1. Create container with audio
2. Export as .oms file
3. Remove container from library
4. Import .oms file
5. Verify all data intact

### Cross-Browser Testing
- Chrome: Full functionality
- Safari: Full functionality
- Firefox: WebGPU only (warn about File System Access)

---

**END OF DEVELOPER DOCUMENTATION**
