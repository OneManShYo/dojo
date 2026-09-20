# OMS DOJO V2_8-N - COMPLETE CHANGELOG

**Version:** V2_8-N  
**Release Date:** November 2025  
**Iterations:** 14 (A through N)  
**Base:** V2.7.0 production release

---

## OVERVIEW

V2_8 adds BroadcastChannel communication for DojoLive integration, audio pitch control, and a professional container library system using File System Access API. All V2.7.0 features preserved.

**Three Major Feature Sets:**
1. **BroadcastChannel Integration** (A-C) - Live instance discovery and canvas snapshots
2. **Audio Pitch Control** (D-E) - DJ-style tempo sync (±16%)
3. **Container Library System** (F-N) - Professional audio library management

---

## ITERATION A: BROADCASTCHANNEL FOUNDATION

**Purpose:** Add communication layer for DojoLive integration

**Features Added:**
- BroadcastChannel 'oms-live-channel' created
- Unique instance ID: 'sb-XXXXXXXXX' format (9 chars)
- Instance announcements on page load and shader compile
- Shader name extraction from first comment line
- 5-second heartbeat keep-alive
- Message listener for future control commands
- Cleanup on page unload

**Technical:**
- Lines added: ~60
- Functions: announcePresence(), getShaderNameFromCode(), startHeartbeat()
- Variables: instanceId, liveChannel, isBroadcasting

**Message Format:**
```javascript
{
  type: 'announce',
  id: 'sb-abc123',
  name: 'Bass Tunnel v3',
  timestamp: 1699564800000
}
```

---

## ITERATION B: CANVAS SNAPSHOTS

**Purpose:** Send visual previews to DojoLive grid

**Features Added:**
- broadcastSnapshot() function captures canvas as JPEG
- 80% quality, base64 encoded (~50-100KB)
- Triggered after successful shader compile
- 100ms delay ensures render() completes first frame

**Technical:**
- Lines added: ~30
- Function: broadcastSnapshot()
- Uses canvas.toDataURL('image/jpeg', 0.8)

**Message Format:**
```javascript
{
  type: 'snapshot',
  id: 'sb-abc123',
  imageData: 'data:image/jpeg;base64,...',
  timestamp: 1699564800000
}
```

---

## ITERATION C: AUTO-SNAPSHOT

**Purpose:** Immediate visual feedback on page load

**Features Added:**
- Auto-snapshot 500ms after page load
- No manual compile needed for initial preview
- DojoLive sees visual immediately

**Technical:**
- Lines added: ~10
- setTimeout(broadcastSnapshot, 500) on init

---

## ITERATION D: PITCH CONTROL

**Purpose:** Enable tempo syncing between tracks with different BPMs

**Features Added:**
- Pitch adjustment field (±16% range)
- Real-time playback rate control via Web Audio API
- Arrow key navigation (Up/Down ±0.1%, Left/Right ±1.0%)
- BPM display updates with adjusted tempo
- Pitch saved/restored with Control States
- Disabled when no audio loaded (cyan theme)

**Technical:**
- Lines added: ~100
- Variable: audioPitch
- Function: applyPitchToAudio()
- Formula: playbackRate = 1.0 + (audioPitch / 100.0)
- Example: +3.23% → 124 BPM becomes 128 BPM

**Use Case:**
- Live session: 128 BPM master tempo
- Track 1: 124 BPM native → +3.23% pitch
- Track 2: 126 BPM native → +1.59% pitch
- All tracks sync to 128 BPM

---

## ITERATION E: DURATION FIX

**Purpose:** Accurate time display when pitch adjusted

**Bug Fixed:**
- Total duration display now accounts for playback rate
- Faster pitch = shorter duration shown
- Slower pitch = longer duration shown

**Technical:**
- Lines changed: 3
- Old: `audioElement.duration`
- New: `audioElement.duration / audioElement.playbackRate`
- Example: 198 seconds at 1.069x = 185 seconds (3:05)

---

## ITERATION F: LIBRARY TAB (MVP)

**Purpose:** Store complete shader+audio+state packages

**Features Added:**
- 4th tab: Library (after Controls/Code/Presets)
- Container structure: shader + audio + states
- localStorage storage
- Add Current, Export, Import, Load, Remove functions
- Audio embedded as base64 (MVP approach)

**Container Format:**
```javascript
{
  id: 'timestamp',
  name: 'Bass Tunnel v3',
  shaderCode: '...',
  controlStates: {...},
  audioData: 'data:audio/mpeg;base64,...',
  audioBPM: 124.5,
  hasAudio: true,
  created: '2025-11-10T...'
}
```

**Technical:**
- Lines added: ~200
- Functions: 8 library management functions
- Storage: localStorage ('omsLibrary' key)
- Limit: 2-5 containers with audio (~5-10MB)

---

## ITERATION G: EXPORT/IMPORT

**Purpose:** Enable container sharing and backup

**Features Added:**
- Export button per container (downloads .oms file)
- Import button triggers file picker (.oms only)
- .oms file format defined (JSON wrapper)
- Format validation on import
- New ID/timestamp generated on import

**.oms File Format:**
```javascript
{
  "format": "oms-container",
  "version": "1.0",
  "container": {
    "id": "...",
    "name": "...",
    // ... full container data
  }
}
```

**Bug Fixed:**
- ReferenceError: shaderTextarea → getElementById('shaderCode')
- Container list button styling (removed inline styles)

**Technical:**
- Lines added: ~100
- Functions: exportContainer(), handleOmsFileImport()
- File naming: ContainerName.oms (sanitized)

---

## ITERATION H: INLINE INPUT

**Purpose:** Replace browser prompt() with integrated UI

**Features Added:**
- Inline input row below action buttons
- Pre-filled with shader name
- ✓/✗ buttons for confirm/cancel
- Enter key confirms, Escape cancels
- Matches Control States UX pattern

**Bug Investigation:**
- Added extensive debug logging for "not saving to UI" issue
- Console tracks full workflow (removed in iteration I)

**Technical:**
- Lines added: ~50
- Functions: confirmAddToLibrary(), cancelAddToLibrary()
- Keyboard handlers added

---

## ITERATION I: CLEANUP

**Purpose:** Remove debug logging, fix UI header

**Changes:**
- UI header: V2.8.0_F → V2.8.0_I (correct version)
- Removed all debug console.log statements
- Export filename already correct (container name only)

**Technical:**
- Lines removed: ~15 (debug logging)
- Clean production code

---

## ITERATION J: MATCH UI

**Purpose:** Library tab styling matches Controls tab

**Styling Fixed:**
- Container cards: background, padding, border, font-size
- Before: #2a2a2a, 15px, 1px solid #444, 14px
- After: rgba(255,255,255,0.03), 12px 15px, 1px solid #333, 13px
- Perfect match with Control State info boxes

**Technical:**
- Lines changed: ~5 (inline styles in refreshLibraryList)
- Visual consistency across all tabs

---

## ITERATION K: FIX STRUCTURE

**Purpose:** Use proper controls-section hierarchy

**Structure Changed:**
- Added controls-section wrapper (padding, border-bottom)
- Added controls wrapper (flex column)
- Changed header: h3 → shader-header class (uppercase)
- Buttons: custom flex → control-row/control-group
- Matches Controls tab structure exactly

**Technical:**
- Lines changed: ~30 (HTML structure)
- Using existing CSS classes (no new styles)

---

## ITERATION L: FILE SYSTEM ACCESS API

**Purpose:** Solve localStorage quota, scale to 10,000+ tracks

**Features Added:**
- File System Access API for audio library folder
- User grants one-time folder permission
- Audio referenced by filename (not embedded)
- IndexedDB stores directory handle (persists across sessions)
- Containers now ~5KB (from 4MB)
- localStorage: 100+ containers easily

**Container Format Changed:**
```javascript
{
  // OLD:
  audioData: 'data:audio/mpeg;base64,...',  // 4MB
  
  // NEW:
  audioFilename: 'bass_loop.mp3',           // 5KB
}
```

**Workflow:**
1. Setup Audio Library (one-time)
2. Pick folder (e.g., ~/Music/OMS/)
3. Put audio files in folder
4. Containers reference files by name
5. Load container → auto-loads from folder

**Technical:**
- Lines added: ~200
- Functions: setupAudioLibrary(), getAudioFromLibrary(), etc.
- IndexedDB: 'OMSDojo' database, 'settings' store
- Browser support: Chrome/Safari only (no Firefox)

**Limitations:**
- Firefox not supported (use Chrome/Safari)
- Missing files show error with filename
- Like Ableton "Find Missing File" workflow

---

## ITERATION M: COMPACT UI

**Purpose:** Reorganize Library tab for better workflow

**UI Reorganized:**
- Top: Add Current / Import (daily use)
- Middle: Container list (daily use)
- Bottom: Setup Audio Library (one-time)
- Added info box for audio library folder (like Control States)

**Before:** Setup button at top (prime real estate)  
**After:** Setup button at bottom (out of the way)

**Technical:**
- Lines moved: ~50 (HTML reorganization)
- Info box: .control-state-info class (reused)
- Max-height adjusted: calc(100vh - 480px)

---

## ITERATION N: SINGLE LETTERS

**Purpose:** Compact container action buttons

**Buttons Changed:**
- Load → L (title="Load .oms container")
- Export → E (title="Export .oms container")
- Remove → X (title="Remove .oms container")
- 60% space savings (~250px → ~100px)

**Technical:**
- Lines changed: 3 (button text)
- Tooltips: HTML title attribute
- More room for container names

---

## FEATURE SUMMARY BY CATEGORY

### BroadcastChannel Features
- Instance announcements with unique IDs
- Shader name broadcasting
- 5-second heartbeat keep-alive
- Canvas snapshots (JPEG base64)
- Auto-snapshot on page load
- Foundation for Live integration

### Audio Features
- Pitch control (±16% range)
- Real-time playback rate adjustment
- Arrow key navigation (fine/coarse)
- BPM display with pitch adjustment
- Duration display accounts for pitch
- Pitch saved with Control States

### Library System
- Container storage (shader+audio+states)
- Export/import .oms files
- File System Access API integration
- Audio library folder management
- IndexedDB for permissions
- Scales to 10,000+ tracks
- Professional DJ software paradigm

### UI/UX Improvements
- Inline input (no prompts)
- Compact single-letter buttons
- Consistent styling across tabs
- Proper controls-section hierarchy
- Info boxes for status display
- Clean, professional aesthetic

---

## TECHNICAL METRICS

**File Size:** 215KB → 266KB (+51KB, +24%)  
**Functions Added:** ~25  
**Lines of Code Added:** ~800  
**Browser APIs Used:** 5 (WebGPU, Web Audio, BroadcastChannel, File System Access, IndexedDB)  
**Iterations:** 14 (A-N)  
**Development Time:** ~2 weeks

---

## BACKWARD COMPATIBILITY

**V2.7.0 Features:** All preserved, 100% compatible  
**Old Containers:** Still load (audioData embedded)  
**New Containers:** Use filename references  
**Export .oms:** Still embeds audio for portability  
**Standalone Mode:** Works without BroadcastChannel  
**No Breaking Changes:** Guaranteed

---

## BROWSER REQUIREMENTS

**Full Support:**
- Chrome 88+ (all features)
- Safari 15+ (all features)
- Edge 88+ (all features)

**Partial Support:**
- Firefox (WebGPU only, no File System Access API)

**Required APIs:**
- WebGPU (shader rendering)
- Web Audio API (FFT, pitch control)
- BroadcastChannel API (Live integration)
- File System Access API (audio library)
- IndexedDB (permission storage)

---

## KNOWN LIMITATIONS

**File System Access API:**
- Chrome/Safari only (no Firefox support)
- Permission may expire after long inactivity
- User can re-grant if needed

**localStorage:**
- 5-10MB limit (still relevant for containers without audio)
- ~100 containers max before quota issues
- Upgrade to IndexedDB for more (future)

**BroadcastChannel:**
- Same-origin policy (localhost or HTTPS)
- Tab-based (not cross-device)
- Manual instance assignment in Live

---

## NEXT STEPS (V2.9.0+)

**DojoLive Integration:**
- Receive FFT/tempo from Live
- Update modulators from external audio
- Scene launch commands
- Remote preset loading

**Library Enhancements:**
- Container thumbnail previews
- Sort/filter containers
- Batch operations
- Duplicate detection
- Subfolder support

**Future Separation (V3.0.0):**
- Remove all audio features from Dojo
- Pure visual player
- Receives all data from OMS Session
- Continuous play mode

---

**END OF COMPLETE CHANGELOG**
