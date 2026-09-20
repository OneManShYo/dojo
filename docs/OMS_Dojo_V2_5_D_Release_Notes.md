# ONEMANSHYO Dojo V2_5 - Release Notes

## Executive Summary

ONEMANSHYO Dojo V2_5 is a major stability and usability release focused on fixing critical audio/visual sync issues, optimizing file size, and improving the render workflow.

### Release Highlights

**File Optimization (90% Size Reduction)**
- Reduced from 1.4MB to 142KB through cleanup of corrupted UTF-8 data
- Removed embedded changelog bloat (now in external docs)
- Prevents token limit crashes in Claude development chats

**Fixed Audio/Visual Sync**
- Render function now correctly uses Audio Tempo (not Shader Tempo)
- Bar.beat format properly converted to beats
- Visual timing syncs with Loop In/Out positions

**Accurate Rendering**
- Removed Math.ceil rounding that caused overshoot
- Removed +1 second buffer causing extra length
- Renders stop precisely at Loop Out position

**Perfect Loop Playback**
- Fixed loop-back calculation using correct Audio Tempo
- Added 0.15 beat threshold to prevent overshoot
- Tight looping within In/Out brackets

**Enhanced UX**
- Render Duration field for no-audio mode (MM:SS.mmm format)
- Loop button visual feedback (cyan when active)
- Render progress in status bar (canvas stays visible)
- Simplified filename format
- "Audio Tempo" terminology (industry standard)

---

## What's New in V2_5

### Major Features

**1. Render Duration Controls**
For rendering shader animations without audio:
- MM:SS.mmm time format (e.g., "00:30.000" for 30 seconds)
- Left-to-right parsing (typing "5" = 5 minutes)
- Arrow key controls: Up/Down ±0.1s, Left/Right ±1.0s
- Consistent with Audio Start field behavior

**2. Visual Feedback**
- Loop button displays cyan when active (OMS brand color)
- Render progress moved to status bar
- Canvas remains visible during rendering

**3. Terminology Updates**
- Changed "BPM" to "Audio Tempo" throughout UI
- Matches industry standards (Ableton Live, Logic Pro)
- Clear distinction from "Shader Tempo"

**4. Simplified Filenames**
Removed duration and beat count from filenames:
- Old: `OMS_filename_1080p_30fps_30s_64beats_P.webm`
- New: `OMS_filename_1080p_30fps_P.webm`

### Bug Fixes

**Critical Audio Bugs:**
- Render using wrong tempo variable (Shader Tempo vs Audio Tempo)
- Bar.beat format incorrectly parsed with parseInt()
- Loop playback jumping to wrong position
- Render duration calculation rounding errors

**File Issues:**
- 1.2MB of corrupted UTF-8 comment blocks
- 100KB of corrupted dropdown separators
- 505KB of embedded changelog bloat

---

## System Requirements

### Browser Support

**Required:**
- Chrome 113+ (recommended)
- Edge 113+
- Safari Technology Preview

**Not Supported:**
- Firefox (WebGPU not stable yet)
- Mobile browsers (limited WebGPU support)
- Older browser versions

### Hardware Requirements

**Minimum:**
- GPU with WebGPU support
- 4GB RAM
- 2GB free disk space (for video exports)

**Recommended:**
- Modern GPU (NVIDIA GTX 1060 or better, AMD equivalent)
- 8GB+ RAM
- 10GB+ free disk space
- SSD for faster video export

### Audio Format Support

**Supported:**
- MP3 (most common)
- WAV (uncompressed)
- OGG Vorbis
- M4A/AAC

**Requirements:**
- 4/4 time signature music
- Known tempo for beat positioning

---

## Production Workflows

### Workflow 1: No Audio Mode (Shader Animation Only)

**Use Case:** Pure visual content for backgrounds, loops, social media

**Steps:**
1. Create shader with Claude (see E_Claude_ShaderCreation_Guide.md)
2. Paste shader code into Code tab
3. Click [Compile] to test
4. Adjust Shader Tempo (default: 120)
5. Set Render Duration in MM:SS.mmm format
   - Social media: 00:15.000 (15s) or 00:30.000 (30s)
   - Background loops: 01:00.000 (60s) or longer
6. Select resolution and frame rate
7. Enter custom filename
8. Click [R] to render
9. Video downloads automatically

**Best Practices:**
- Test shader at lower resolution first (720p)
- Keep Shader Tempo reasonable (60-180 range)
- Watch canvas during render to verify visual output
- Use Portrait orientation for social media (TikTok, Instagram)

### Workflow 2: Audio Mode (Music Visualizer)

**Use Case:** Music videos, audio-reactive content, live performance

**Steps:**
1. Create shader with Claude
2. Load audio file using [Upload Audio]
3. Enter correct Audio Tempo manually
4. Use Audio Start field to locate first downbeat
5. Set Loop In/Out brackets using arrow keys
   - Loop In: start of desired section
   - Loop Out: end of desired section
6. Enable Loop button to audition
   - Button displays cyan when active
   - Audio plays from Loop In to Loop Out and repeats
7. Adjust modulators (Mod A/B/C) to taste
8. Select resolution and frame rate
9. Enter custom filename
10. Click [R] to render
11. Video with audio downloads automatically

**Best Practices:**
- Use decimal Audio Tempo precision (e.g., 126.53)
- Audition loop before rendering (enable Loop button)
- Watch for cyan Loop button to confirm loop is active
- Start with shorter loops (16-32 beats) for testing
- Use arrow keys for precise bracket positioning

### Workflow 3: Live Performance Integration

**Use Case:** VJ sets, live streaming, Ableton Live sessions

**Steps:**
1. Render multiple shader variations without audio
   - Use No Audio Mode workflow
   - Create 10-20 different visual loops
   - Vary colors, tempo, modulator settings
2. Export at consistent resolution (1080p recommended)
3. Export at consistent frame rate (30fps recommended)
4. Import videos into Ableton Live
5. Use ebosuite for video clip launching
6. Control with Midifighter Twister or Yaeltex controller

**Best Practices:**
- Render consistent durations (all 30s or all 60s)
- Name files systematically (color, tempo, style)
- Test all clips load properly in Ableton
- Have backup clips ready
- Render at performance resolution (don't scale up live)

### Workflow 4: Social Media Content

**Use Case:** TikTok, Instagram Reels, YouTube Shorts

**Platform Specs:**
- TikTok: 1080x1920 (Portrait), 15-60 seconds
- Instagram Reels: 1080x1920 (Portrait), 15-90 seconds
- YouTube Shorts: 1080x1920 (Portrait), up to 60 seconds

**Steps:**
1. Download audio from target platform
2. Load audio into Dojo
3. Enter correct Audio Tempo
4. Set Loop In/Out to platform duration limit
5. Use Portrait orientation (1080p)
6. Use 30 FPS (mobile-friendly)
7. Render with audio
8. Optional: Post-process with FFmpeg (see UserGuide_FFmpeg_Video_Guide.md)
9. Upload to platform

**Best Practices:**
- Match audio exactly to platform's audio track
- Keep visuals engaging but not overwhelming
- Test render on actual phone before posting
- Consider text overlay space in composition
- Respect platform aspect ratio exactly

---

## Post-Processing with FFmpeg

For advanced video manipulation, see: **UserGuide_FFmpeg_Video_Guide.md**

**Common Operations:**
- Convert WebM to MP4 for wider compatibility
- Add text overlays or watermarks
- Concatenate multiple renders
- Adjust audio levels
- Extract frames for thumbnails

---

## Troubleshooting

### WebGPU Not Available

**Symptoms:** "WebGPU not supported" message in status bar

**Solutions:**
1. Update browser to latest version
2. Enable WebGPU in chrome://flags
3. Check chrome://gpu for GPU compatibility
4. Try different browser (Chrome vs Edge vs Safari)

### Shader Won't Compile

**Symptoms:** Error message after clicking [Compile]

**Solutions:**
1. Check browser console for WGSL error details
2. Verify uniform bindings match available uniforms
3. Test with default shader first
4. Ask Claude to debug shader code

### Audio Won't Load

**Symptoms:** File upload appears to work but no audio plays

**Solutions:**
1. Check file format (MP3, WAV, OGG supported)
2. Verify file isn't corrupted (play in media player)
3. Try different audio file
4. Check browser console for errors

### Loop Playback Issues

**Symptoms:** Audio doesn't loop or loops at wrong position

**Solutions:**
1. Verify Loop button displays cyan (active state)
2. Check Audio Tempo value is correct
3. Verify Loop In/Out brackets are set
4. Use arrow keys to fine-tune Loop Out position
5. If loop jumps to wrong spot, verify Audio Tempo is accurate

### Render Duration Mismatch

**Symptoms:** Video length doesn't match expected duration

**Solutions:**
1. **With audio:** Check Loop In/Out bracket positions
2. **With audio:** Verify Audio Tempo is correct (not Shader Tempo)
3. **No audio:** Check Render Duration field value
4. Verify no audio is loaded if using No Audio Mode

### Video Quality Issues

**Symptoms:** Video looks pixelated or low quality

**Solutions:**
1. Increase resolution (1440p or 2160p)
2. Increase frame rate (60 FPS for smoother motion)
3. Check shader isn't overly complex
4. Simplify shader if GPU is struggling
5. Consider post-processing with FFmpeg for quality boost

---

## Version Migration Notes

### Upgrading from V2.4.0

**Breaking Changes:** None

**New Features Available:**
- Render Duration field (no audio mode)
- Visual loop button feedback
- Simplified filename format

**Behavior Changes:**
- Filenames no longer include duration/beat metadata
- Loop button displays cyan when active
- Render progress shows in status bar (not overlay)

**No Action Required:** All V2.4.0 workflows continue to work identically.

### Upgrading from V2.3.0

**Breaking Changes:** None

**Fixed Issues:**
- Audio rendering now uses correct Audio Tempo
- Loop playback positions correctly
- Renders stop at exact Loop Out position

**Recommended Actions:**
- Re-test audio-based renders for accuracy
- Verify loop playback behaves as expected
- Enjoy the improved workflow!

---

## Known Limitations

**Browser Limitations:**
- WebGPU support required (not available in all browsers)
- Video recording limited by browser memory
- Very long renders (>2 minutes) may cause browser slowdown

**Audio Limitations:**
- Requires 4/4 time signature music
- Manual tempo entry required (no auto-detection)
- Bar.beat display assumes 4 beats per bar

**Shader Limitations:**
- Complex shaders may not achieve 60 FPS at 4K
- Shader compilation errors not always descriptive
- Limited to fragment shaders only (no vertex shader control)

---

## Future Development

**Planned Features:**
- Additional modulator waveforms
- Custom FFT band configuration
- Multiple simultaneous shader layers
- Real-time shader parameter recording

**Community Requests:**
- MP4 export format (browser support pending)
- Vertex shader support
- MIDI controller integration
- Shader preset library

---

## Support & Contact

**Documentation:**
- Z_Project_Instructions.md - Comprehensive project context
- UserGuide_All.md - Complete user documentation
- C_Developer_Documentation.md - Technical reference
- E_Claude_ShaderCreation_Guide.md - Shader patterns

**Creator:**
Wes Smith / OneManShYo Streaming Project

---

**Last Updated:** November 2025  
**Release Date:** November 2025  
**Version:** V2_5
