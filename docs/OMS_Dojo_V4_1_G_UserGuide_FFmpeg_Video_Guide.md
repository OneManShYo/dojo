# FFMPEG VIDEO EXPORT GUIDE - OMS SHADER BAKER V4.1.0

**Purpose:** Export shaders as ProRes 422 video for Ableton Live + EboSuite  
**Version:** V4.1.0  
**Date:** November 27, 2025

---

## QUICK START

### The Workflow

```
OMS Shader Baker → WebM/MP4 → FFmpeg → ProRes 422 → Ableton Live + EboSuite
     (render)        (export)   (convert)   (import)      (perform live)
```

---

## WHAT'S NEW IN V4.1.0

**Perfect Grid Alignment:**
- Exported videos now render with perfect beat timing
- Timeline reactivity is pre-baked accurately
- Bar-aligned exports work flawlessly in Ableton

**Visibility Testing:**
- Use M key to verify alignment before rendering
- Track grids match Master Grid exactly
- Confident exports every time

---

## STEP 1: INSTALL FFMPEG

### macOS (Homebrew)

```bash
brew install ffmpeg
```

### Windows (Download)

1. Visit: https://ffmpeg.org/download.html
2. Download Windows build
3. Extract to `C:\ffmpeg`
4. Add to PATH: `C:\ffmpeg\bin`

### Linux (apt)

```bash
sudo apt update
sudo apt install ffmpeg
```

**Verify Installation:**
```bash
ffmpeg -version
```

---

## STEP 2: RENDER VIDEO IN OMS SHADER BAKER

### V4.1.0 Preparation

1. **Controls Tab Setup:**
   - Load your audio file
   - Set BPM accurately
   - Set Audio Start (Bar 1 position)
   - Click S button (snap to bars)
   - Verify Last Bar field shows count

2. **Verify Alignment (NEW!):**
   - Go to LIVE tab
   - Load container
   - Press M to show Master Grid
   - Confirm track grid aligns perfectly
   - Return to Controls tab

3. **Render Settings:**
   - Resolution: 1920×1080 (recommended)
   - FPS: 60 (smooth) or 30 (smaller files)
   - Duration: Automatically from loop points

### Rendering Process

1. **Click "Render Video" Button**
2. **Watch progress bar**
3. **Video saves to Downloads**
   - Format: WebM (Chrome) or MP4 (Safari)
   - Filename: `shader_output_[timestamp].webm`
   - SILENT video (audio in Ableton)

**V4.1.0 Advantage:** Grid-perfect timing baked into the video!

---

## STEP 3: CONVERT TO PRORES 422

### Standard Conversion

```bash
ffmpeg -i shader_output.webm \
       -c:v prores_ks \
       -profile:v 3 \
       -vendor apl0 \
       -pix_fmt yuv422p10le \
       output.mov
```

### V4.1.0 Bar-Aligned Export

Since you used the S button, your video is exactly N bars:

```bash
# Example: 164 bars at 120 BPM = 328 seconds
ffmpeg -i shader_164bars.webm \
       -c:v prores_ks \
       -profile:v 3 \
       -vendor apl0 \
       -pix_fmt yuv422p10le \
       shader_164bars_prores.mov
```

### ProRes Profiles

**Profile 2 - Standard** (~147 Mbps)
```bash
-profile:v 2  # Good for practice/testing
```

**Profile 3 - HQ** (~220 Mbps)
```bash
-profile:v 3  # Recommended for performance
```

---

## STEP 4: IMPORT TO ABLETON LIVE

### Preparation

**Required:**
- Ableton Live 11+
- EboSuite plugin
- ProRes video from Step 3
- Original audio file

### Import Process

1. **Create Ableton Project:**
   - Set project BPM to match video
   - Create audio track for music
   - Create video track (EboSuite)

2. **Import Files:**
   - Drag ProRes video to video track
   - Drag audio to audio track
   - Align Bar 1 positions

3. **V4.1.0 Sync Advantage:**
   - Video has grid-perfect timing
   - Perfect sync with audio track
   - Bar boundaries align perfectly
   - No manual warping needed

### Warp Settings

**Since V4.1.0 uses grid-aligned exports:**
- Enable Warp on video clip
- Set correct BPM
- Warp markers auto-align to bars
- Perfect sync guaranteed

---

## V4.1.0 BATCH WORKFLOW

### Multiple Shaders for One Track

```bash
#!/bin/bash
# Convert all shaders for a set

for file in shader_*bars.webm; do
    ffmpeg -i "$file" \
           -c:v prores_ks \
           -profile:v 3 \
           -vendor apl0 \
           -pix_fmt yuv422p10le \
           "${file%.webm}_prores.mov"
done
```

### Organizing by Bar Count

**V4.1.0 S Button Advantage:**
- All exports have verified bar counts
- Name files with bar count: `shader_intro_64bars.webm`
- Easy to organize in Ableton: 64-bar sections, 128-bar drops, etc.

---

## OPTIMIZED SETTINGS

### For Timeline Reactivity

**High Quality (Performance):**
```bash
ffmpeg -i shader_timeline.webm \
       -c:v prores_ks \
       -profile:v 3 \
       -vendor apl0 \
       -pix_fmt yuv422p10le \
       -r 60 \
       shader_timeline_hq.mov
```

**Standard Quality (Practice):**
```bash
ffmpeg -i shader_timeline.webm \
       -c:v prores_ks \
       -profile:v 2 \
       -r 30 \
       shader_timeline_std.mov
```

### File Size Estimates

**With V4.1.0 Timeline Reactivity:**
- 164 bars @ 120 BPM = 328 seconds
- ProRes 422 HQ @ 1080p60: ~7.2 GB
- ProRes 422 Std @ 1080p30: ~2.4 GB

---

## TROUBLESHOOTING

### Timeline Reactivity Not Visible

**Problem:** Exported video doesn't show audio reactivity

**Solution:**
1. Re-render with timeline data:
   - Load audio in Controls
   - Click "Add to Library" (analyzes FFT)
   - Then render video
2. Check modulators in shader code (uModA, uModB, uModC)
3. Verify timeline arrays in container

### Bar Alignment Issues

**Problem:** Video doesn't align with Ableton bars

**Solution:**
1. Use S button before rendering
2. Check Last Bar field shows count
3. Name file with bar count
4. Set exact BPM in Ableton
5. **NEW:** Press M in LIVE tab to verify before rendering

### Large File Sizes

**With timeline reactivity, files may be larger:**
```bash
# Reduce size with H.264 for storage
ffmpeg -i shader_timeline.mov \
       -c:v libx264 \
       -preset slow \
       -crf 18 \
       shader_timeline_h264.mp4
       
# Convert back to ProRes when needed
```

---

## V4.1.0 BEST PRACTICES

### Preparation Checklist

✅ Load final audio file  
✅ Set accurate BPM  
✅ Click S button (snap to bars)  
✅ Verify Last Bar count  
✅ "Add to Library" (timeline analysis)  
✅ **NEW:** Press M to verify alignment  
✅ Test in preview  
✅ Render at target resolution  

### Naming Convention

Include bar count in filename:
- `bass_drop_128bars.mov`
- `intro_64bars.mov`
- `buildup_32bars.mov`

### Ableton Organization

**V4.1.0 Bar-Based Structure:**
- Intro: 64 bars
- Buildup: 32 bars
- Drop: 128 bars
- Breakdown: 64 bars
- All perfectly aligned!

---

## ADVANCED V4.1.0 TECHNIQUES

### Concatenate Bar-Aligned Sections

```bash
# Create list of sections in order
cat > filelist.txt << EOF
file 'intro_64bars.mov'
file 'buildup_32bars.mov'
file 'drop_128bars.mov'
EOF

# Concatenate
ffmpeg -f concat -safe 0 -i filelist.txt -c copy full_track.mov
```

### Extract Specific Bar Ranges

Since V4.1.0 exports are bar-aligned:

```bash
# Extract bars 65-128 (at 120 BPM, bar = 2 seconds)
# Start: 64 bars × 2 sec = 128 seconds
# Duration: 64 bars × 2 sec = 128 seconds

ffmpeg -i full_track.mov \
       -ss 128 \
       -t 128 \
       -c:v prores_ks \
       -profile:v 3 \
       bars_65to128.mov
```

---

## QUICK COMMAND REFERENCE

### Essential Commands

**Standard ProRes Conversion:**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 3 output.mov
```

**With Frame Rate Control:**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 3 -r 60 output.mov
```

**Batch Convert All:**
```bash
for f in *.webm; do ffmpeg -i "$f" -c:v prores_ks -profile:v 3 "${f%.webm}.mov"; done
```

---

## COMPLETE V4.1.0 WORKFLOW

1. **Create Shader** with audio reactivity
2. **Load Audio** and set BPM
3. **Click S Button** to snap to bars
4. **Add to Library** for timeline analysis
5. **Press M** to verify grid alignment (NEW!)
6. **Render Video** with baked reactivity
7. **Convert to ProRes** via FFmpeg
8. **Import to Ableton** with EboSuite
9. **Perform Live** with perfect sync!

**Result:** Professional audio-visual performance with grid-perfect timing!

---

## RESOURCES

### FFmpeg
- Official: https://ffmpeg.org/
- ProRes Guide: https://trac.ffmpeg.org/wiki/Encode/VFX

### EboSuite
- Website: https://ebosuite.rocks/
- Ableton Integration Guide

### OMS Shader Baker
- Website: https://itswessmithyo.com
- YouTube: @itswessmithyo

---

**END OF FFMPEG VIDEO EXPORT GUIDE**

Version: V4.1.0
Date: November 27, 2025
License: GPL-3.0
