# ONEMANSHYO Dojo V2_6 - Release Notes

## Executive Summary

**Version:** 2.6.0  
**Release Date:** November 2025  
**Status:** Production Release  
**File Size:** ~199KB (single HTML file)  
**Iterations:** 20 (A through U, skipping K)

### What's New in V2_6

**Shader Presets System:**
Complete preset library with visual thumbnails, drag-and-drop import, and keyboard navigation. Build performance video libraries quickly with 20-30 shader imports.

**Real-Time Render Progress:**
Live elapsed time and frame counters during video rendering. Essential for timing modulator tweaks without audio reference.

**Intelligent Filename Defaults:**
Automatic shader name parsing from code comments. Meaningful filenames without manual entry.

**UX Polish:**
Consistent 400px panel width, no auto-tab-switching, one-click preset deletion, and drag-over visual feedback.

---

## Quick Start

### Installation

1. **Download:** Extract ZIP to desired location
2. **Open:** Double-click `OMS_Dojo_V2_6.html`
3. **Browser:** Chrome 113+ or Edge 113+ (WebGPU support required)
4. **Ready:** Start creating shader visualizations immediately

**No installation, no build process, no dependencies.**

### First Shader in 2 Minutes

1. Click **Presets** tab
2. Click any thumbnail (e.g., "Tron Tunnel")
3. Shader loads and compiles automatically
4. Preview shows real-time animation
5. Click **Render** button
6. Video downloads as WebM file

**Done!** You've created your first audio-reactive shader video.

---

## Core Features Overview

### 1. Shader Module (Visual Timing)

**Controls Tab - Rows 1-5:**

**Row 1: Timing Controls**
- Shader Tempo: BPM for visual animation timing
- Loop Length: Duration in bars (4/4 time)
- Render Duration: MM:SS.mmm format (shader-only mode)

**Rows 2-4: Modulators**
- Mod A/B/C: Audio-reactive controls
- Subdivision: Note values (1/4, 1/8, 1/16, 1/32)
- Waveform: Sine, Triangle, Square, Sawtooth
- Gain: 0.0-1.0 intensity

**Row 5: Output Settings**
- Resolution: 720p/1080p/1440p/2160p
- FPS: 24/30/60
- Orientation: Portrait/Landscape/Square
- Filename: Auto-parsed or manual entry

### 2. Audio Module (Optional)

**Audio Controls Strip:**
- Upload audio track (.mp3, .wav, .m4a)
- Manual tempo entry (Audio Tempo field)
- Loop In/Out brackets (bar.beat format)
- Arrow key navigation (beat-based positioning)
- Loop button (cyan when active)

### 3. Presets System

**Library Section (5 Curated Shaders):**
1. Tron Tunnel - Pink/blue radial tunnel
2. Tron Grid - Retrowave sun with grid floor
3. MetaBalls - Lava lamp organic blobs
4. Flower Power - Geometric circle pattern
5. Retro Fish Farm - Animated grid with dots

**User Presets Section:**
- Import multiple .txt/.wgsl shader files
- Drag-and-drop support
- Visual thumbnail grid
- LocalStorage persistence
- Keyboard navigation (Tab/Enter)
- One-click deletion

### 4. Code Editor

**Code Tab:**
- WGSL fragment shader editor
- Syntax-aware (basic)
- Compile button with error feedback
- Export .txt functionality
- Version tracking in filename

---

## Production Workflows

### Workflow 1: Shader-Only Mode (No Audio)

**Use Case:** Creating visual loops for live performance

1. Click **Code** tab
2. Write or paste WGSL shader code
3. Click **Compile** to preview
4. Adjust **Shader Tempo** (e.g., 120 BPM)
5. Set **Render Duration** (e.g., 00:30 for 30 seconds)
6. Tweak **Modulators** A/B/C for visual variety
7. Click **Render**
8. Watch progress: "Rendering 00:15/00:30 | Frame 450/900"
9. Video downloads as silent WebM file
10. Load into EboSuite/Resolume for live DJ set

**Result:** Silent shader loop at specified tempo, ready for Ableton Live integration.

### Workflow 2: Audio-Reactive Mode

**Use Case:** Creating videos synced to specific music tracks

1. Upload audio track (drag onto waveform or click "Upload Audio")
2. Enter correct BPM in **Audio Tempo** field
3. Set **Audio Start** (MM:SS.mmm) to first downbeat
4. Use arrow keys to set **Loop In** bracket
5. Use arrow keys to set **Loop Out** bracket
6. Enable **Loop** button (turns cyan) to audition
7. Tweak **Modulators** to match music energy
8. Click **Render**
9. Video includes audio from Loop In to Loop Out
10. Perfect sync for social media posts

**Result:** Video with audio, visuals synced to musical structure.

### Workflow 3: Performance Library Building

**Use Case:** Preparing 20-30 shader videos for live DJ set

1. Click **Presets** tab
2. Drag 20-30 .txt shader files onto **User Presets** grid
3. Visual thumbnails generate automatically
4. Tab through cards to browse visuals
5. Press Enter on card → shader loads and compiles
6. Adjust settings if needed → Click Render
7. Repeat for next shader (Tab → Enter → Render)
8. Build complete video library in 30-60 minutes

**Result:** Full performance video library with diverse visual styles.

### Workflow 4: Shader Creation with Claude

**Use Case:** Designing custom shaders conversationally

1. Chat with Claude: "Create a shader with rotating hexagons"
2. Claude generates WGSL shader code
3. Copy/paste into **Code** tab
4. Click **Compile** to preview
5. Iterate with Claude: "Make them pulse to bass"
6. Test modulator assignments
7. Export as .txt for archiving
8. Add to User Presets for future use

**Result:** Custom shader tailored to your creative vision.

---

## Technical Requirements

### Browser Compatibility

**Supported:**
- Chrome 113+ (Windows/Mac/Linux)
- Edge 113+ (Windows/Mac/Linux)
- Opera 99+ (Windows/Mac/Linux)

**Not Supported:**
- Firefox (WebGPU in development)
- Safari (WebGPU limited support)
- Mobile browsers (performance limitations)

### Hardware Requirements

**Minimum:**
- Modern GPU (Intel UHD 620 or equivalent)
- 4GB RAM
- Dual-core CPU

**Recommended:**
- Dedicated GPU (NVIDIA/AMD/Intel Arc)
- 8GB+ RAM
- Quad-core+ CPU

**For Best Performance:**
- NVIDIA RTX or AMD RX series
- 16GB+ RAM
- Modern CPU (Intel 10th gen+ or AMD Ryzen 3000+)

### Storage

- HTML File: ~199KB
- User Presets: ~1-5MB (LocalStorage, varies by shader count)
- Rendered Videos: ~10-100MB per video (depends on duration/resolution)

---

## Known Limitations

### WebGPU Support
- Firefox users: Must wait for WebGPU stable release
- Safari users: Limited support, may have issues

### Video Export
- Format: WebM/VP8 only (use FFmpeg to convert)
- No H.264 export (browser limitations)
- Large files for high-res/long duration (use compression)

### Audio Support
- Manual BPM entry required (no automatic detection)
- Manual beat alignment needed (no automatic beatgrid)
- Some formats may not load (browser codec support)

### LocalStorage
- User Presets limited by browser (~5-10MB typical)
- Clearing browser data = presets lost
- No cloud sync (local only)

---

## Troubleshooting

### Black/Blank Thumbnails

**Problem:** Preset thumbnails show black or blank screens

**Solution:**
1. Check browser console for WebGPU errors
2. Update GPU drivers
3. Try different browser (Chrome vs Edge)
4. Reduce preset count (resource issue)
5. Clear browser cache and reload

### Video Won't Render

**Problem:** Clicking Render does nothing or shows error

**Solution:**
1. Check Render Duration field has valid time (MM:SS.mmm)
2. Ensure shader compiles without errors
3. Try lower resolution (1080p instead of 2160p)
4. Try lower FPS (30 instead of 60)
5. Close other GPU-heavy applications
6. Restart browser

### Audio Won't Load

**Problem:** Dragging audio file shows error

**Solution:**
1. Check file format (.mp3/.wav/.m4a supported)
2. Try different audio file
3. Check file isn't corrupted
4. Use VLC to convert to .mp3 if needed
5. Ensure file size < 100MB

### Loop Playback Issues

**Problem:** Audio doesn't loop back to Loop In

**Solution:**
1. Verify Audio Tempo matches track BPM
2. Check Loop In/Out brackets are valid (e.g., 1.1 to 5.1)
3. Ensure Loop button is enabled (cyan)
4. Try wider loop range (4 bars instead of 1 bar)
5. Set Audio Start to first downbeat

---

## File Management

### Video Files

**Format:** WebM (VP8 codec)  
**Naming:** `OMS_[name]_[res]p_[fps]fps_[orient].webm`  
**Example:** `OMS_TronTunnel_1080p_30fps_P.webm`

**Conversion to H.264 (for compatibility):**
```bash
ffmpeg -i OMS_TronTunnel_1080p_30fps_P.webm -c:v libx264 -preset medium -crf 23 TronTunnel_1080p.mp4
```

### Shader Files

**Format:** Plain text (.txt or .wgsl)  
**Naming:** `OMS_[name]_Shader_V2.6.0.txt`  
**Example:** `OMS_TronTunnel_Shader_V2.6.0.txt`

**Content:** WGSL fragment shader code with OMS comment header

### Archiving Projects

**Recommended Structure:**
```
MyProject/
├── shaders/
│   ├── TronTunnel_Shader_V2.6.0.txt
│   ├── MetaBalls_Shader_V2.6.0.txt
│   └── Custom_Shader_V2.6.0.txt
├── videos/
│   ├── TronTunnel_1080p_30fps_P.webm
│   ├── MetaBalls_1440p_30fps_S.webm
│   └── Custom_1080p_60fps_L.webm
└── audio/
    └── track.mp3
```

---

## Performance Tips

### Optimizing Shader Complexity

**Reduce Shader Load:**
- Avoid excessive loops in fragment shader
- Use cheaper math operations (avoid pow/exp when possible)
- Simplify ray marching (reduce steps)
- Lower resolution for testing (720p vs 2160p)

### Optimizing Render Times

**Faster Rendering:**
- Lower resolution (1080p → 720p) = 2x faster
- Lower FPS (60 → 30) = 2x faster
- Shorter duration (60s → 30s) = 2x faster
- Close other applications
- Disable browser extensions

### Managing Memory

**Reduce Memory Usage:**
- Limit User Presets to 20-30 shaders
- Clear unused presets regularly
- Close other browser tabs
- Restart browser between long renders

---

## Integration with Live Performance

### Ableton Live + EboSuite Workflow

1. **Render Videos:** Use Dojo to create 30-60 second silent loops
2. **Import to Ableton:** Drag WebM files into EboSuite video channels
3. **Tempo Sync:** Set Ableton session tempo to match Shader Tempo
4. **Beat Matching:** Videos loop perfectly at correct BPM
5. **Live Mixing:** Use Midifighter Twister to trigger video clips
6. **Audio Mixing:** Audio channels separate (DJ mixing as normal)
7. **Visual Effects:** EboSuite provides additional video FX layer

**Result:** Full audio-visual DJ set with real-time mixing control.

### Resolume Arena Workflow

Similar to EboSuite but with Resolume's interface and FX suite.

---

## Best Practices

### Shader Development

1. **Start Simple:** Basic patterns first, complexity later
2. **Test Modulators:** Verify A/B/C respond correctly
3. **Save Increments:** Export .txt after each successful iteration
4. **Name Clearly:** Use descriptive shader names in comments
5. **Document Settings:** Note good modulator assignments in comments

### Video Library Management

1. **Organize by Style:** Group similar visuals together
2. **Consistent Naming:** Use meaningful filenames
3. **Version Control:** Keep shader .txt files with videos
4. **Test Loop Points:** Verify seamless looping before performance
5. **Backup Everything:** Keep copies of shader files and videos

### Performance Prep

1. **Build Early:** Don't wait until day before show
2. **Test Hardware:** Verify laptop can handle render load
3. **Render Backups:** Have extra videos in case of issues
4. **Organize Clips:** Group by energy level/style for easy access
5. **Practice Mixing:** Rehearse visual transitions with music

---

## Getting Help

### Resources

- **Documentation:** Complete guides included in deliverables
- **Website:** https://www.itswessmithyo.com/
- **Community:** (Discord/forum links if available)

### Reporting Issues

When reporting problems:
1. Browser version (Chrome/Edge + number)
2. Operating system (Windows/Mac/Linux + version)
3. GPU model (check in browser://gpu)
4. Steps to reproduce issue
5. Screenshot/error message if applicable

---

## Deployment Checklist

### Pre-Release Verification

- [ ] All 20 iterations tested
- [ ] WebGPU initialization works
- [ ] All 5 library presets render correctly
- [ ] User preset import/delete functional
- [ ] Drag-and-drop working
- [ ] Video export produces valid WebM files
- [ ] Audio sync accurate
- [ ] Loop playback reliable
- [ ] Filename parsing correct
- [ ] Progress counters accurate

### Distribution

- [ ] ZIP file created with all 9 deliverables
- [ ] HTML file verified (opens in browser)
- [ ] Documentation complete
- [ ] Version numbers consistent
- [ ] README/license included (if applicable)

---

## Version Roadmap

### Completed (V2_6)
✅ Presets system with visual thumbnails  
✅ User preset import and management  
✅ Real-time render progress tracking  
✅ Intelligent filename parsing  
✅ Drag-and-drop import  
✅ One-click deletion  
✅ Keyboard navigation  

### Potential Future Features
- Preset search/filter functionality
- Drag-to-reorder presets
- Preset tags/categories
- Batch export selected presets
- Custom thumbnail time points
- Preset sharing via URLs
- Real-time Ableton Link integration
- Native Mac app with Syphon output

**Note:** Future features depend on community feedback and use cases.

---

**Release Guide Version:** V2_6  
**Last Updated:** November 2025  
**Target Audience:** Users deploying or supporting the application

**Ready for production use!**
