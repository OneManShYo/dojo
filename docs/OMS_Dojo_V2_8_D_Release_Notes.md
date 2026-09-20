# OMS DOJO V2_8-N - RELEASE GUIDE

**Version:** V2_8-N  
**Release Date:** November 2025  
**Type:** Major Feature Release

---

## WHAT'S NEW

V2_8 adds three major feature sets while preserving all V2.7.0 functionality:

1. **BroadcastChannel Integration** - Communicate with DojoLive for multi-instance performance workflows
2. **Audio Pitch Control** - DJ-style tempo sync (±16%) for matching tracks to session tempo
3. **Container Library System** - Professional audio library management scaling to 10,000+ tracks

---

## THREE-MINUTE QUICK START

### For Existing Users (V2.7.0)

Everything you know still works. New features are additive.

**Try BroadcastChannel (with DojoLive):**
1. Open DojoLive V1.0.0-L
2. Open Dojo V2_8-N
3. DojoLive auto-discovers instance
4. Assign to track, compile shader → see snapshot

**Try Pitch Control:**
1. Load audio track
2. Find Pitch field (after Audio Tempo)
3. Press Up/Down arrows (fine tune)
4. Watch BPM display update in real-time

**Try Container Library:**
1. Click Library tab
2. Click "Setup Audio Library" (pick folder once)
3. Put audio files in that folder
4. Click "Add Current" to save shader+audio
5. Load containers instantly

### For New Users

**Basic Workflow:**
1. Open OMS_Dojo_V2_8_N.html in Chrome/Safari
2. Paste shader code (or load preset from Presets tab)
3. Click Compile
4. Optional: Load audio file for reactive visuals
5. Click Render to export video

**Advanced Workflow with Library:**
1. Library tab → Setup Audio Library (one-time)
2. Add audio files to library folder
3. Create shader in Code tab
4. Load audio from library folder
5. Configure reactivity (Controls tab)
6. Library tab → Add Current (saves everything)
7. Next time: Load container → instant recall

---

## BROADCASTCHANNEL FEATURES

### What It Does

Enables communication between Dojo instances and DojoLive for multi-track performance mixing.

### How It Works

**Instance Discovery:**
- Each Dojo tab gets unique ID: `sb-abc123`
- Announces presence to DojoLive
- 5-second heartbeat keeps connection alive

**Canvas Snapshots:**
- Broadcasts visual preview on compile
- DojoLive displays in grid
- Auto-snapshot on page load (no compile needed)

**Shader Names:**
- Extracted from first line: `// Bass Tunnel v3`
- Displayed in DojoLive dropdowns
- Makes instance selection easy

### Usage Example

```
Setup:
1. Open DojoLive (V1.0.0-L or later)
2. Open 3 Dojo tabs
3. Each tab gets unique ID
4. DojoLive discovers all 3

Workflow:
1. Tab 1: Compile kick shader → snapshot appears in grid
2. Tab 2: Compile bass shader → snapshot appears in grid  
3. Tab 3: Compile lead shader → snapshot appears in grid
4. DojoLive: Assign instances to tracks
5. Build your set!
```

### Future Integration (V2.9.0+)

- Receive FFT/tempo from DojoLive
- Update modulators from external audio
- Scene launch commands
- Synchronized multi-instance performance

---

## AUDIO PITCH CONTROL

### What It Does

Adjust audio playback speed/pitch (±16%) for tempo matching between tracks with different native BPMs.

### How It Works

**Pitch Field:**
- Located after Audio Tempo field
- Range: -16% to +16%
- Arrow key navigation:
  - Up/Down: ±0.1% (fine tune)
  - Left/Right: ±1.0% (coarse tune)

**Real-Time Feedback:**
- BPM display updates with adjusted tempo
- Duration display adjusts (faster = shorter)
- Saved with Control States

**Web Audio API:**
- Changes playback rate (speed + pitch)
- Like vinyl/CDJ pitch control
- NOT time-stretching

### Usage Example

```
Scenario: Session tempo is 128 BPM

Track 1: 124 BPM native
- Set pitch: +3.23%
- Plays at: 128 BPM ✓

Track 2: 126 BPM native
- Set pitch: +1.59%
- Plays at: 128 BPM ✓

Track 3: 130 BPM native
- Set pitch: -1.54%
- Plays at: 128 BPM ✓

All tracks sync!
```

### Formula

```
Adjusted BPM = Native BPM × (1 + Pitch / 100)

Example: 124 BPM × 1.0323 = 128 BPM
```

### Use Cases

**Standalone:**
- Prep work: Match BPMs before exporting
- Save as Control State for specific tempo

**Future (with DojoLive):**
- DojoLive sends pitch commands
- Dojo applies automatically
- Master tempo drives all instances

---

## CONTAINER LIBRARY SYSTEM

### What It Does

Store complete shader+audio+state packages with professional audio library management. Scales to 10,000+ tracks like Rekordbox/Serato.

### How It Works

**Audio Library Folder (One-Time Setup):**
1. Library tab → "Setup Audio Library"
2. Pick folder (e.g., `~/Music/OMS/`)
3. Browser stores permission (persists forever)
4. Put audio files in that folder

**Container Structure:**
- Shader code (WGSL)
- Control States (21 values)
- Audio filename reference (not embedded)
- Metadata (name, BPM, created date)

**Storage:**
- Metadata: localStorage (~5KB per container)
- Audio: Your filesystem (unlimited)
- Permissions: IndexedDB (persistent)

### Usage

**Daily Workflow:**
1. Create shader
2. Load audio from library folder (auto-discovery)
3. Configure reactivity
4. Library tab → "Add Current"
5. Container saved (~5KB)

**Load Container:**
1. Library tab
2. Click "L" button on container
3. Shader + audio + states load instantly
4. Ready to work

**Export/Import:**
- Export: Click "E" → downloads .oms file
- Import: Click "Import" → pick .oms file
- Share with community or backup

### Before vs After

**Before (V2.7.0):**
- Audio embedded in Control States (4MB per)
- localStorage limit: 2-3 states max
- QuotaExceededError

**After (V2_8):**
- Audio referenced by filename (5KB per)
- localStorage: 100+ containers easily
- Scales to 10,000+ tracks

### .oms File Format

Portable, shareable container format:
```json
{
  "format": "oms-container",
  "version": "1.0",
  "container": {
    "name": "Bass Tunnel v3",
    "shaderCode": "...",
    "controlStates": {...},
    "audioData": "...",  // embedded for portability
    "audioBPM": 128
  }
}
```

Export embeds audio for sharing. Import adds to library.

### Button Reference

- **L** - Load container (shader+audio+states)
- **E** - Export as .oms file
- **X** - Remove from library

---

## MIGRATION FROM V2.7.0

### Backward Compatibility

**All V2.7.0 features work identically:**
- WebGPU shader rendering
- FFT audio analysis
- Frequency mapping
- Control States
- Preset system
- Video export

**No Breaking Changes:**
- Existing Control States load correctly
- Existing presets load correctly
- Same keyboard shortcuts
- Same UI layout (+ new tabs)

### What's New in UI

**New Fields:**
- Pitch field (after Audio Tempo)
- Container library actions

**New Tabs:**
- Library tab (4th tab)

**New Buttons:**
- Setup Audio Library
- Add Current, Import

### Data Migration

**Control States:**
- Old states work without changes
- New states include audioPitch field
- Automatic backward compatibility

**Audio:**
- Old workflow: Load audio file (still works)
- New workflow: Audio library folder + filename reference
- Choose what fits your workflow

---

## BROWSER REQUIREMENTS

### Full Support

**Chrome 88+**
- All features working
- Recommended for best experience

**Safari 15+**
- All features working
- File System Access API since 15.2

**Edge 88+**
- All features working
- Chromium-based

### Partial Support

**Firefox**
- WebGPU: ✓ (Nightly)
- BroadcastChannel: ✓
- File System Access API: ✗ (not supported)

**Recommendation:** Use Chrome or Safari for full functionality.

### Required APIs

- WebGPU (shader rendering)
- Web Audio API (FFT, pitch control)
- BroadcastChannel API (Live integration)
- File System Access API (audio library)
- IndexedDB (permission storage)

---

## COMMON WORKFLOWS

### Solo Creator Workflow

**Prep Phase:**
1. Create 5-10 shader variations
2. Setup Audio Library folder
3. Add audio tracks to folder
4. Create containers (shader+audio combos)
5. Export containers as .oms files (backup)

**Performance Phase:**
1. Open DojoLive
2. Open Dojo instances (load containers)
3. Assign instances to tracks
4. Mix audio + visuals live
5. Record performance

### Producer Workflow

**Creation:**
1. Write shader in Code tab
2. Load audio for preview
3. Tune frequency mapping
4. Export video (WebM)
5. Convert to ProRes (FFmpeg)
6. Import to Ableton/EboSuite

**Library Management:**
1. Organize audio files in library folder
2. Save containers for quick recall
3. Export .oms files for backup
4. Share .oms files with collaborators

### Community Workflow

**Share Your Work:**
1. Create amazing shader+audio combo
2. Library tab → Export container
3. Upload .oms file to Discord/GitHub
4. Others import and enjoy

**Use Others' Work:**
1. Download .oms file from community
2. Library tab → Import
3. Container appears in library
4. Load and customize

---

## TROUBLESHOOTING

### "Audio Library Not Configured"

**Solution:** Click "Setup Audio Library", pick folder.

**Why:** File System Access API requires one-time permission.

### "Audio not found: filename.mp3"

**Solution:** Add audio file to library folder.

**Why:** Container references file that doesn't exist.

**Like:** Ableton's "Find Missing File" workflow.

### "Storage full - remove old containers"

**Cause:** localStorage quota exceeded (~5-10MB).

**Solution:** Remove unused containers, or export and delete.

**Note:** Audio files don't count toward quota (external).

### Firefox - File System Access API Error

**Cause:** Firefox doesn't support File System Access API.

**Solution:** Use Chrome or Safari.

**Workaround:** Manual file loading still works.

### BroadcastChannel Not Working

**Check:**
- Using HTTPS or localhost?
- DojoLive open in same browser?
- Browser supports BroadcastChannel? (all modern browsers)

---

## PERFORMANCE TIPS

### Container Library
- Keep audio files organized in subfolders
- Use descriptive filenames
- Backup .oms files regularly
- Export before clearing library

### BroadcastChannel
- Close unused Dojo tabs
- Limit to 4-8 instances max
- Snapshots are lightweight (~50-100KB)

### Rendering
- Test with Quick Test preset first
- Render at target resolution (don't upscale)
- Close other tabs during render

---

## WHAT'S NEXT (V2.9.0+)

### DojoLive Integration
- Receive FFT/tempo from Live
- Update modulators from external audio
- Scene launch commands
- Remote preset loading

### Library Enhancements
- Container thumbnail previews
- Sort/filter containers
- Batch operations
- Subfolder support

### Future Separation (V3.0.0)
- Remove audio features from Dojo
- Pure visual player
- Receives all data from OMS Session

---

## GETTING HELP

**Documentation:**
- UserGuide_All.md - Comprehensive usage guide
- Developer_Documentation.md - Technical reference
- FFmpeg_Video_Guide.md - Video conversion

**Website:** https://www.itswessmithyo.com/

**Community:** (Coming soon - Discord/Reddit)

---

**Welcome to V2_8! Happy shader creating!**
