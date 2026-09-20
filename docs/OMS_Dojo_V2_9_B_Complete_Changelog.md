# OMS DOJO V2_9 - COMPLETE CHANGELOG

**Version:** V2_9  
**Release Date:** November 2025  
**Iterations:** 22 (A through V, reusing letters after Z)  
**Base:** V2.8.0-N production release

---

## OVERVIEW

V2_9 implements LIBRARY-first architecture, requiring LIBRARY folder for all saves. Adds professional file management, keyboard-first navigation, and improved UX. Removes BroadcastChannel and pitch control features (moved to future LIVE tab).

**Three Major Feature Sets:**
1. **LIBRARY-First Architecture** (L, V) - Required folder for persistent storage
2. **Auto-Save Workflow** (K, T) - DAW-style .oms file management
3. **Keyboard Navigation** (P-S) - Professional keyboard-first UI

---

## ITERATION A: LIBRARY BROADCASTING

**Purpose:** Add library broadcasting via BroadcastChannel for DojoLive integration

**Features Added:**
- broadcastLibrary() function reads complete container array from localStorage
- Broadcasts via 'oms-live-channel'
- Message type: 'library_update'
- Includes full container data (shader code, control states, audio filename, metadata)
- Called on page load + 5-second heartbeat + after library changes

**Technical:**
- Lines added: ~30
- Message format: { type: 'library_update', id: 'sb-abc123', containers: [...], timestamp: ... }

**When Library Broadcasted:**
1. Page load - initial broadcast
2. 5-second heartbeat - continuous updates
3. After library changes (add/remove/import)

---

## ITERATION B: TAB REORDERING

**Purpose:** Reorganize tabs for better workflow

**Features Changed:**
- Tab order BEFORE: Controls, Code, Presets, Library
- Tab order AFTER: Controls, Presets, Library, Code
- Code tab moved to end (advanced users)

**Rationale:**
- Controls: most-used (default)
- Presets: quick exploration
- Library: project management
- Code: advanced editing (least-used)

**Technical:**
- Lines changed: ~4 (tab button order)
- No functional changes, just visual

---

## ITERATION C: UPDATE CONTAINER & AUDIO LOADING

**Purpose:** Add update functionality and fix audio loading from LIBRARY

**Features Added:**
- "U" button added to container cards: L | U | E | X
- updateContainer() function replaces existing container with current editor state
- Keeps same ID, name, created timestamp
- Adds updated timestamp
- currentLoadedContainerId tracks which container is loaded

**Audio Loading Fixed:**
- loadContainer() automatically calls getAudioFromLibrary(audioFilename)
- Audio loads from LIBRARY folder
- Shows error if audio missing but still loads shader/states
- Graceful degradation (like Ableton "Find Missing File")

**Container Format Updated:**
```json
{
  "created": "2025-11-10T12:00:00.000Z",
  "updated": "2025-11-11T18:45:00.000Z"  // NEW: Optional timestamp
}
```

**Technical:**
- Lines added: ~50
- Functions: updateContainer(), tracking variable

---

## ITERATION D: SIMPLIFIED STATE DISPLAY

**Purpose:** Clean up Control States info box formatting

**Features Changed:**
- Control States info box simplified to comma-separated format
- BEFORE: Columnar layout with multiple spans (wrapping issues)
- AFTER: Single-line format per modulator

**Visual Result:**
```
BEFORE:
Mod A:    1/4        Sine       0.1   30-50Hz

AFTER:
Mod A: 1/4, Sine, 0.1, 30-50Hz
```

**Technical:**
- Lines changed: ~20 (HTML template)
- Cleaner, more readable

---

## ITERATION E: EDITABLE NAMES & BUTTON ALIGNMENT

**Purpose:** Enable name editing and fix layout issues with long names

**Features Added:**
- Click container name to edit (prompt dialog)
- Long names truncated with ellipsis
- Full name shown in tooltip on hover
- Buttons (L U E X) always aligned

**Technical:**
- Lines added: ~40
- Functions: editContainerName(), renameContainer()
- CSS: text-overflow ellipsis, flexbox layout

---

## ITERATION F: FIX AUDIO PERSISTENCE

**Purpose:** CRITICAL FIX - prevent audio info from being cleared on update

**Bug Fixed:**
- updateContainer() was clearing audioFilename when no audio loaded in editor
- Now preserves existing audio info if no audio currently loaded
- Critical for DojoLive library broadcasting

**Logic:**
```javascript
if (audioLoaded && currentAudioFile) {
    // Use editor audio
} else if (existingContainer.audioFilename) {
    // Preserve existing audio  // ← THIS WAS MISSING
}
```

**Technical:**
- Lines changed: ~10
- Essential fix for audio workflow

---

## ITERATION G: INLINE EDIT

**Purpose:** Replace prompt dialog with inline editing

**Features Added:**
- Click name → becomes input field
- Text auto-selected
- Enter to save, Escape to cancel
- Blur to save (click away)
- Professional UX (no browser dialogs)

**Input Styling:**
- Cyan border (#0ff)
- Dark background
- Matches app theme

**Technical:**
- Lines changed: ~50
- Removed prompt(), added inline input

---

## ITERATION H: FIX AUDIO LOAD

**Purpose:** Fix audio loading when switching between containers

**Features Added:**
- loadContainer() now calls removeAudio() first
- Clears existing audio before loading new container
- Sets currentAudioFile = null
- Ensures clean state transitions

**Workflow Fixed:**
1. Load container A with audio → audio loads
2. Click L on container B with different audio → container A audio cleared, B audio loads
3. No audio conflicts

**Technical:**
- Lines added: ~5
- Better state management

---

## ITERATION I: DIRTY TRACKING

**Purpose:** Track unsaved changes and warn user

**Features Added:**
- isDirty boolean flag tracks unsaved changes
- Tracks shader code changes (input event)
- Tracks control changes (markAsCustomState)
- Magenta warning when loading with unsaved changes
- Non-blocking (warns but allows loading)

**Dirty State Clearing:**
- loadContainer() → isDirty = false
- updateContainer() → isDirty = false
- confirmAddToLibrary() → isDirty = false
- loadPreset() → isDirty = false

**Audio Debug Logging:**
- Comprehensive console logging throughout loadContainer()
- Logs container name, audio filename, LIBRARY status
- Helps diagnose audio loading issues

**Technical:**
- Lines added: ~40
- Variable: isDirty
- Event listeners added

---

## ITERATION J: FIX INFO FORMAT

**Purpose:** Fix Control States info box formatting (again)

**Features Changed:**
- Simplified from span-based layout to clean comma-separated format
- No more line wrapping issues
- Single template string per line

**Technical:**
- Lines changed: ~10
- Final formatting fix

---

## ITERATION K: AUTO-SAVE OMS

**Purpose:** Implement DAW-style auto-save workflow

**Features Added:**
- Auto-save .oms files to LIBRARY folder on Add/Update
- No manual export dialogs
- .oms filename matches container name (or audio filename)
- Auto-delete on Remove
- Removed "E" (Export) button from Library cards

**DAW Model:**
```
Ableton: MySet.als lives WITH audio files
OMS: track01.oms lives WITH track01.mp3
```

**Technical:**
- Lines added: ~100
- Functions: saveContainerToOMS()
- Async file operations

---

## ITERATION L: LIBRARY FIRST

**Purpose:** Enforce LIBRARY folder requirement for all saves

**Features Added:**
- checkLibrarySet() helper function
- Blocks ALL saves without LIBRARY folder
- "Save Control State" enforces LIBRARY
- "Add Current" enforces LIBRARY
- Control States save as {Name}_ControlState.json in LIBRARY

**UI Text Updates:**
- "Setup Audio Library" → "Set Library Folder"
- "Change Audio Library" → "Change Library"
- Info box title: "Audio Library" → "Library"

**Two Modes:**
1. **LIBRARY Set (Persistent):** All saves to LIBRARY, survives sessions
2. **LIBRARY Not Set (Temp Session):** Work + render only, nothing saves

**Technical:**
- Lines added: ~80
- checkLibrarySet() function
- Control States → LIBRARY folder

---

## ITERATION M: ACTIVE INDICATOR

**Purpose:** Add visual indicators for active and selection states

**Features Added:**
- Cyan dot (●) appears left of L button when container is active
- Persistent - stays visible until different container loaded
- Uses currentLoadedContainerId to track
- Cyan border on hover/focus

**Technical:**
- Lines added: ~30
- CSS: hover/focus-within states
- Active indicator HTML

---

## ITERATION N: NAV AND NAME

**Purpose:** Fix cyan dot visibility, add keyboard navigation, improve auto-naming

**Features Added:**
- Cyan dot changed from bullet (●) to filled circle (●)
- Size increased: 16px → 20px
- Arrow Up/Down navigates through library cards
- Enter key loads selected container
- Cards have tabindex="0" for accessibility

**Auto-Suggested Name:**
- Format: "AudioName_ShaderName"
- If audio loaded: removes extension, adds underscore, appends shader name
- If no audio: just shader name

**Technical:**
- Lines added: ~60
- Keyboard event handlers
- Auto-naming logic

---

## ITERATION O: DOT FIX

**Purpose:** Reduce cyan dot size and confirm refresh calls

**Features Changed:**
- Cyan dot reduced from 20px to 10px (better balance)
- Confirmed refreshLibraryList() called after loadContainer()
- Confirmed refreshLibraryList() called after updateContainer()

**Technical:**
- Lines changed: ~2
- Size adjustment only

---

## ITERATION P: KEYBOARD FIRST

**Purpose:** Implement full keyboard-first navigation

**Features Added:**
- Auto-focus first card on Library tab switch
- 50ms delay ensures cards rendered first
- Hold arrow key → rapidly scroll through entries
- scrollIntoView with smooth behavior
- Works with 100+ entries

**Enter Key Behavior:**
- Enter on card → Load container
- Tab to button (L/U/X) → Enter activates button
- Checked: e.target === card to only load when card focused

**Technical:**
- Lines added: ~40
- Auto-focus on tab switch
- Smooth scrolling

---

## ITERATION Q: ENTER FIX

**Purpose:** Fix Enter key behavior for loading and maintaining focus

**Features Added:**
- loadContainer() re-focuses card after refreshLibraryList()
- 100ms setTimeout ensures HTML regeneration completes
- updateContainer() also re-focuses card
- Maintains keyboard navigation flow

**Workflow:**
1. Arrow Down → Enter → Container loads
2. Cyan dot appears
3. Card remains focused
4. Continue with Arrow Down/Up immediately

**Technical:**
- Lines added: ~20
- Re-focus logic after refresh

---

## ITERATION R: LOAD BUTTON

**Purpose:** Fix L button not responding to Enter key

**Features Added:**
- Explicit button click handling
- Checks if target.tagName === 'BUTTON'
- Explicitly triggers click event on button
- Ensures onclick handlers fire reliably

**Technical:**
- Lines changed: ~10
- Button Enter key handler

---

## ITERATION S: BUTTON FIX

**Purpose:** Fix L button visual feedback when Enter pressed

**Features Added:**
- Dedicated keydown listeners directly to each button
- Buttons handle their own Enter keypresses
- Simplified card listener (arrow keys only)
- button.click() fires onclick handler properly

**Technical:**
- Lines added: ~30
- Dedicated button event listeners
- Cleaner separation of concerns

---

## ITERATION T: LIBRARY OMS FIX

**Purpose:** CRITICAL FIX - Container name = .oms filename (1:1 match)

**Bug Fixed:**
- BEFORE: Container name "MyShader", .oms filename "audio_track.oms" (confusing)
- AFTER: Container name "MyShader", .oms filename "MyShader.oms" (1:1 match)
- Audio filename stored in .oms JSON (preserved)

**Pattern:**
```
Container name → .oms filename (direct match)
- Save:   "MyShader" → "MyShader.oms"
- Load:   "MyShader" → read "MyShader.oms"
- Rename: "MyShader" → "MyShader_v2" → rename .oms file
- Delete: "MyShader" → delete "MyShader.oms"
```

**loadContainer() Fixed:**
- If LIBRARY set: reads ${container.name}.oms from LIBRARY
- Parses JSON, uses .oms data
- Falls back to localStorage if .oms not found
- Loads audio using audioFilename from .oms

**Why This Pattern:**
- Matches Ableton .asd pattern (companion files)
- Container name always findable
- Audio filename stays original (preserved in JSON)
- Clean renaming (delete old, save new)

**Technical:**
- Lines changed: ~100
- Critical fix for entire ecosystem

---

## ITERATION U: APPLY STATE FIX

**Purpose:** CRITICAL FIX - Shader code and audio not loading via L button

**Bugs Fixed:**
- applyControlState() tried to access audioElement.src when audioElement was null (TypeError)
- Execution stopped, never reached compileShader() or audio loading
- updateContainer() removed audio because L button failed to load it

**Fixes:**
- Safe null check: `const hasAudio = audioElement && !!audioElement.src;`
- Wrapped applyControlState() in try-catch
- Ensures compileShader() always runs
- Ensures audio loading always runs

**Technical:**
- Lines changed: ~30
- Safe null checks
- Error handling

---

## ITERATION V: LIBRARY REQUIRED

**Purpose:** Enforce LIBRARY folder requirement, save Control States as .oms

**Features Added:**
- checkLibrarySet(autoOpen) with auto-open parameter
- Auto-opens folder picker if LIBRARY not set
- "Add Current" enforces LIBRARY (auto-open picker)
- "Save State" enforces LIBRARY (auto-open picker)
- Control States save as {Name}_ControlState.oms (not .json)
- type: "controlState" distinguishes from containers

**Two Modes Enforced:**
1. **LIBRARY Set (Persistent):** All saves to LIBRARY as .oms files
2. **LIBRARY Not Set (Temp Session):** Nothing saves, work + render only

**Control State .oms Structure:**
```json
{
  "id": "1762908543210",
  "name": "QuickTest",
  "type": "controlState",
  "controlStates": { ... },
  "created": "2025-11-12T01:23:45.678Z"
}
```

**Filename Convention:**
- Format: {Name}_ControlState.oms
- Example: "QuickTest_ControlState.oms"
- Distinguishes from container .oms files

**LIBRARY Contents:**
```
/LIBRARY
  ├── Bass_Tunnel.oms (container)
  ├── Lava_Lamp.oms (container)
  ├── QuickTest_ControlState.oms (control state)
  ├── HD_Settings_ControlState.oms (control state)
  ├── track01.mp3 (audio file)
  └── track02.mp3 (audio file)
```

**Technical:**
- Lines changed: ~150
- Auto-open folder picker
- Control States as .oms files
- Library requirement enforced

---

## FEATURE SUMMARY BY CATEGORY

### LIBRARY Architecture
- LIBRARY folder REQUIRED for all saves
- Two modes: Persistent (LIBRARY set) vs Temp Session (no LIBRARY)
- Auto-save .oms files on Add/Update
- Container name = .oms filename (1:1 match)
- Audio filename stored in .oms JSON
- Control States as .oms files
- Auto-open folder picker when needed
- Pattern matches Ableton .asd files

### Keyboard Navigation
- Auto-focus first card on Library tab switch
- Arrow Up/Down navigate rapidly (smooth scrolling)
- Enter key loads container
- Tab to buttons (L/U/X), Enter activates
- Re-focus after load/update maintains flow
- Works with 100+ entries

### UI/UX Improvements
- Active container indicator (cyan dot)
- Inline editable names (no prompts)
- Dirty state tracking (warns on unsaved changes)
- Simplified info box formatting
- Button alignment fixed (long names truncate)
- Tab reordering (Controls, Presets, Library, Code)
- Professional keyboard-first workflow

### Audio & File Management
- Audio loads automatically from LIBRARY
- Audio references preserved on update
- Safe null checks prevent crashes
- Graceful degradation if audio missing
- Clean state management
- Consistent file naming

### Removed Features
- BroadcastChannel integration (moved to LIVE tab)
- Audio pitch control (moved to LIVE tab)
- Library broadcasting (moved to LIVE tab)

---

## TECHNICAL METRICS

**File Size:** 266KB → 272KB (+6KB, +2%)  
**Functions Added:** ~20  
**Lines of Code Added:** ~800  
**Iterations:** 22 (A-V)  
**Development Time:** ~2 weeks  
**Browser APIs Used:** WebGPU, Web Audio, File System Access, IndexedDB

---

## BACKWARD COMPATIBILITY

**V2.8.0 Features:** Most preserved  
**Removed:** BroadcastChannel, pitch control (will return in LIVE tab)  
**Container Format:** Enhanced with .oms files  
**Control States:** Now .oms files instead of localStorage  
**Breaking Changes:** LIBRARY folder now required for saves

---

## BROWSER REQUIREMENTS

**Full Support:**
- Chrome 88+ (all features)
- Safari 15.2+ (all features)
- Edge 88+ (all features)

**Partial Support:**
- Firefox (WebGPU only, no File System Access API)

**Required APIs:**
- WebGPU (shader rendering)
- Web Audio API (FFT analysis)
- File System Access API (LIBRARY folder)
- IndexedDB (permission storage)

---

## KNOWN LIMITATIONS

**LIBRARY Folder:**
- File System Access API required (Chrome/Safari only)
- Permission may expire after long inactivity
- User can re-grant if needed

**No localStorage Saves:**
- All user data in LIBRARY folder
- Temp session mode if no LIBRARY
- Close browser = lose everything (temp mode)

---

## NEXT STEPS (V2.10.0+)

**LIVE Tab Integration:**
- Add 5th LIVE tab to Dojo
- Performance grid (4 tracks × 8 scenes)
- Mixer section (4 tracks + master)
- Real-time shader rendering (4 simultaneous)
- Real-time audio mixing
- Visual compositing
- BroadcastChannel restored (for discovery)
- Pitch control restored (for tempo sync)

**Future Enhancements:**
- Container thumbnail previews
- Sort/filter containers
- Batch operations
- Subfolder support
- MIDI control integration

---

**END OF COMPLETE CHANGELOG**
