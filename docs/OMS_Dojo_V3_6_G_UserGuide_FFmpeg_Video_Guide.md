# FFMPEG VIDEO EXPORT GUIDE - OMS SHADER BAKER V3.6.0

**Purpose:** Export shaders as ProRes 422 video for Ableton Live + EboSuite  
**Version:** V3.6.0  
**Date:** November 16, 2025

---

## QUICK START

### The Workflow

```
OMS Shader Baker → WebM/MP4 → FFmpeg → ProRes 422 → Ableton Live + EboSuite
     (render)        (export)   (convert)   (import)      (perform live)
```

**Why This Workflow:**
- OMS renders shaders as silent video
- FFmpeg converts to ProRes 422 (professional codec)
- Ableton Live plays video in sync with audio
- EboSuite provides video playback in Ableton
- Live performance with audio-reactive visuals

---

## STEP 1: INSTALL FFMPEG

### macOS (Homebrew)

```bash
brew install ffmpeg
```

**Verify Installation:**
```bash
ffmpeg -version
```

### Windows (Chocolatey)

```powershell
choco install ffmpeg
```

**OR Download Binary:**
1. Visit: https://ffmpeg.org/download.html
2. Download Windows build
3. Extract to `C:\ffmpeg`
4. Add to PATH: `C:\ffmpeg\bin`

**Verify Installation:**
```cmd
ffmpeg -version
```

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

### Settings

1. **Open Controls Tab (C)**
2. **Set Resolution:**
   - 1920×1080 (Full HD, most common)
   - 2560×1440 (2K, higher quality)
   - 3840×2160 (4K, maximum quality, slow render)

3. **Load Audio File** (if using audio reactivity)
   - MP3, WAV, FLAC supported
   - Set BPM accurately
   - Set loop points if needed

4. **Set Render Duration:**
   - Match audio length
   - Or set custom duration for looping shader

### Rendering

1. **Click "Render Video" Button**
2. **Wait for Render** (real-time or faster depending on GPU)
3. **Video Saves to Downloads** folder
   - Format: WebM or MP4 (browser dependent)
   - Filename: `shader_output_[timestamp].webm`

**Render Times:**
- 1920×1080, 60 FPS, 3 minutes: ~3-5 minutes (real-time)
- 2560×1440, 60 FPS, 3 minutes: ~5-8 minutes
- 3840×2160, 60 FPS, 3 minutes: ~10-15 minutes

**Note:** Render is SILENT (no audio). Audio added in Ableton Live.

---

## STEP 3: CONVERT TO PRORES 422

### Why ProRes 422?

**Benefits:**
- Professional codec used in video production
- High quality with reasonable file size
- Frame-accurate seeking
- Widely supported in DAWs
- Ableton Live + EboSuite prefer ProRes

**ProRes 422 Specs:**
- Codec: Apple ProRes 422
- Container: MOV (QuickTime)
- Color Space: YUV 4:2:2 (10-bit)
- Bitrate: ~147 Mbps @ 1080p60

### Basic Conversion Command

```bash
ffmpeg -i shader_output.webm \
       -c:v prores_ks \
       -profile:v 3 \
       -vendor apl0 \
       -pix_fmt yuv422p10le \
       output.mov
```

**Breakdown:**
- `-i shader_output.webm`: Input file
- `-c:v prores_ks`: Use ProRes encoder (Kostya Shishkov variant, best quality)
- `-profile:v 3`: ProRes 422 profile (Standard)
- `-vendor apl0`: Apple vendor code (ensures compatibility)
- `-pix_fmt yuv422p10le`: 10-bit 4:2:2 chroma (professional)
- `output.mov`: Output filename

### ProRes Profiles

**Choose Based on Quality Needs:**

**Profile 0 - ProRes 422 Proxy** (~45 Mbps)
```bash
-profile:v 0
```
- Lowest quality
- Smallest files
- Editing proxy only

**Profile 1 - ProRes 422 LT** (~102 Mbps)
```bash
-profile:v 1
```
- Lower quality
- Good for rough edits

**Profile 2 - ProRes 422** (~147 Mbps)
```bash
-profile:v 2
```
- Standard quality
- **Recommended for most uses**

**Profile 3 - ProRes 422 HQ** (~220 Mbps)
```bash
-profile:v 3
```
- Higher quality
- **Recommended for final output**
- Larger files

**Profile 4 - ProRes 4444** (~330 Mbps)
```bash
-profile:v 4
```
- Highest quality
- Alpha channel support
- Massive files

**For Shader Videos:** Profile 2 (Standard) or 3 (HQ) recommended.

### Resolution-Specific Commands

**1920×1080 (Full HD):**
```bash
ffmpeg -i shader_1080p.webm \
       -c:v prores_ks \
       -profile:v 3 \
       -vendor apl0 \
       -pix_fmt yuv422p10le \
       shader_1080p_prores.mov
```

**2560×1440 (2K):**
```bash
ffmpeg -i shader_2k.webm \
       -c:v prores_ks \
       -profile:v 3 \
       -vendor apl0 \
       -pix_fmt yuv422p10le \
       shader_2k_prores.mov
```

**3840×2160 (4K):**
```bash
ffmpeg -i shader_4k.webm \
       -c:v prores_ks \
       -profile:v 3 \
       -vendor apl0 \
       -pix_fmt yuv422p10le \
       shader_4k_prores.mov
```

### Frame Rate Options

**Preserve Original Frame Rate (Default):**
```bash
ffmpeg -i shader_output.webm \
       -c:v prores_ks \
       -profile:v 3 \
       output.mov
```

**Force 60 FPS:**
```bash
ffmpeg -i shader_output.webm \
       -c:v prores_ks \
       -profile:v 3 \
       -r 60 \
       output.mov
```

**Force 30 FPS (Smaller Files):**
```bash
ffmpeg -i shader_output.webm \
       -c:v prores_ks \
       -profile:v 3 \
       -r 30 \
       output.mov
```

**Note:** 60 FPS looks smoother but doubles file size. 30 FPS acceptable for most uses.

---

## STEP 4: IMPORT TO ABLETON LIVE + EBOSUITE

### Prerequisites

**Required:**
- Ableton Live 11+ (Standard or Suite)
- EboSuite plugin (https://ebosuite.rocks/)
- ProRes video file from Step 3

### Import Workflow

**Method 1: Drag & Drop**
1. Open Ableton Live project
2. Locate ProRes video file
3. Drag video onto audio track
4. EboSuite video track appears
5. Video plays in sync with audio

**Method 2: EboSuite Video Player**
1. Add EboSuite Video Player device
2. Click "Load Video" button
3. Select ProRes video file
4. Video loads into player
5. Route audio for sync

### Synchronization

**Timeline Sync:**
- Video starts at clip start marker
- Plays at project tempo (if warped)
- Loops automatically (if loop enabled)
- Stop/start follows playback

**Tempo Matching:**
- Warp video clip to project tempo
- Use Ableton's warp markers
- Match video BPM to audio BPM
- Perfect sync for audio-reactive content

### Performance Setup

**Typical Routing:**
```
Track 1: Audio (kick, bass)
Track 2: EboSuite Video (shader reactive to Track 1)
Track 3: Audio (synths)
Track 4: EboSuite Video (shader reactive to Track 3)
Master: Output to projector/screen
```

**Controls:**
- Volume controls affect audio (not video)
- Use EboSuite opacity for video mixing
- Use EboSuite effects for visual FX
- Map MIDI controllers for live control

---

## ADVANCED TECHNIQUES

### Batch Conversion

**Convert Multiple Files:**

**Bash Script (macOS/Linux):**
```bash
#!/bin/bash
for file in *.webm; do
    ffmpeg -i "$file" \
           -c:v prores_ks \
           -profile:v 3 \
           -vendor apl0 \
           -pix_fmt yuv422p10le \
           "${file%.webm}_prores.mov"
done
```

**PowerShell Script (Windows):**
```powershell
Get-ChildItem *.webm | ForEach-Object {
    $output = $_.BaseName + "_prores.mov"
    ffmpeg -i $_.Name `
           -c:v prores_ks `
           -profile:v 3 `
           -vendor apl0 `
           -pix_fmt yuv422p10le `
           $output
}
```

### Quality Comparison

**Test Different Profiles:**
```bash
# Profile 2 (Standard)
ffmpeg -i shader.webm -c:v prores_ks -profile:v 2 shader_422.mov

# Profile 3 (HQ)
ffmpeg -i shader.webm -c:v prores_ks -profile:v 3 shader_422hq.mov

# Compare file sizes
ls -lh shader_*.mov
```

**Typical Results:**
- 3-minute 1080p60 video
- Profile 2: ~3.5 GB
- Profile 3: ~5.0 GB

**Recommendation:** Use Profile 3 (HQ) for final performances, Profile 2 for practice/testing.

### Trimming Video

**Extract Specific Section:**
```bash
ffmpeg -i shader_output.webm \
       -ss 00:00:30 \
       -t 00:02:00 \
       -c:v prores_ks \
       -profile:v 3 \
       shader_trimmed.mov
```

- `-ss 00:00:30`: Start at 30 seconds
- `-t 00:02:00`: Duration of 2 minutes
- Result: 30s to 2m30s section

### Concatenating Videos

**Join Multiple Shader Videos:**

Create `filelist.txt`:
```
file 'shader1_prores.mov'
file 'shader2_prores.mov'
file 'shader3_prores.mov'
```

Convert:
```bash
ffmpeg -f concat \
       -safe 0 \
       -i filelist.txt \
       -c copy \
       shader_combined.mov
```

**Note:** All videos must have same codec, resolution, frame rate.

### Adding Silent Audio Track

**If Ableton requires audio:**
```bash
ffmpeg -i shader_output.webm \
       -f lavfi \
       -i anullsrc=channel_layout=stereo:sample_rate=48000 \
       -c:v prores_ks \
       -profile:v 3 \
       -c:a pcm_s16le \
       -shortest \
       shader_with_audio.mov
```

- Adds silent stereo audio track
- 48kHz sample rate (professional standard)
- PCM uncompressed audio

---

## TROUBLESHOOTING

### FFmpeg Not Found

**Symptom:** `ffmpeg: command not found`

**Solutions:**
- Install FFmpeg (see Step 1)
- Add FFmpeg to PATH
- Use full path: `/usr/local/bin/ffmpeg -i ...`
- Restart terminal after installation

### Conversion Too Slow

**Symptom:** Takes hours to convert short video

**Causes:**
- High resolution (4K)
- Complex codec settings
- Slow CPU

**Solutions:**
- Use GPU encoding (if available):
  ```bash
  ffmpeg -i shader.webm \
         -c:v h264_videotoolbox \
         -profile:v high \
         -b:v 20M \
         shader.mp4
  ```
  (Then convert H.264 to ProRes if needed)
- Lower resolution for testing
- Use Profile 2 instead of 3
- Close other apps during conversion

### File Size Too Large

**Symptom:** 3-minute video is 10+ GB

**Causes:**
- Profile 3 (HQ) or 4 (4444)
- High resolution (4K)
- High frame rate (60 FPS)

**Solutions:**
- Use Profile 2 (Standard) instead of 3:
  ```bash
  -profile:v 2
  ```
- Lower resolution:
  ```bash
  -s 1920x1080
  ```
- Lower frame rate:
  ```bash
  -r 30
  ```
- Use H.264 for storage, convert to ProRes when needed

### Ableton Won't Import Video

**Symptom:** Video file doesn't show up or won't load

**Causes:**
- Wrong codec
- Missing EboSuite
- File corrupted

**Solutions:**
- Verify ProRes encoding: `ffmpeg -i video.mov`
- Install/update EboSuite
- Try different FFmpeg command
- Test with small sample video first

### Video/Audio Out of Sync

**Symptom:** Video plays at wrong speed or drifts

**Causes:**
- Wrong BPM in OMS Shader Baker
- Wrong warp settings in Ableton
- Frame rate mismatch

**Solutions:**
- Verify BPM is correct in shader export
- Warp video clip in Ableton
- Match video frame rate to project frame rate
- Use beat grid for alignment

---

## OPTIMIZATION TIPS

### File Size vs Quality

**Best Balance:**
- 1920×1080 resolution
- 60 FPS frame rate
- ProRes 422 HQ (Profile 3)
- Result: ~5GB per 3 minutes

**For Storage/Backup:**
- 1920×1080 resolution
- 30 FPS frame rate
- H.264 high profile (~100MB per 3 minutes)
- Convert to ProRes when needed for performance

### Workflow Efficiency

**Recommended Workflow:**
1. Create and test shader at low resolution (720p)
2. Export test video at low quality (Profile 0)
3. Verify looks good in Ableton
4. Re-render at full resolution (1080p/4K)
5. Convert to ProRes 422 HQ (Profile 3)
6. Import final version for performance

**Time Saved:** Test at low quality first, avoid wasting time on full-quality renders that don't work.

---

## COMMAND REFERENCE

### Standard Conversion
```bash
ffmpeg -i shader.webm -c:v prores_ks -profile:v 3 -vendor apl0 -pix_fmt yuv422p10le output.mov
```

### High Quality (Profile 3)
```bash
ffmpeg -i shader.webm -c:v prores_ks -profile:v 3 -vendor apl0 -pix_fmt yuv422p10le output_hq.mov
```

### Lower Quality for Testing (Profile 2)
```bash
ffmpeg -i shader.webm -c:v prores_ks -profile:v 2 -vendor apl0 -pix_fmt yuv422p10le output_std.mov
```

### With Frame Rate Control
```bash
ffmpeg -i shader.webm -c:v prores_ks -profile:v 3 -r 60 output_60fps.mov
```

### With Resolution Scaling
```bash
ffmpeg -i shader_4k.webm -c:v prores_ks -profile:v 3 -s 1920x1080 output_1080p.mov
```

### Batch Convert All WebM Files
```bash
for file in *.webm; do ffmpeg -i "$file" -c:v prores_ks -profile:v 3 "${file%.webm}.mov"; done
```

---

## RESOURCES

### FFmpeg Documentation
- Official: https://ffmpeg.org/documentation.html
- ProRes Encoding: https://trac.ffmpeg.org/wiki/Encode/VFX

### EboSuite
- Website: https://ebosuite.rocks/
- Documentation: https://ebosuite.rocks/documentation/
- Video Tutorials: YouTube "EboSuite tutorials"

### Ableton Live
- User Manual: https://www.ableton.com/en/manual/
- Video Sync: Ableton Live + EboSuite workflow

---

## SUMMARY

**The Complete Workflow:**

1. **OMS Shader Baker:** Create audio-reactive shader
2. **Render:** Export as WebM/MP4 (silent video)
3. **FFmpeg:** Convert to ProRes 422 HQ
4. **Ableton Live:** Import video with EboSuite
5. **Add Audio:** Sync with music track
6. **Perform:** Live DJ set with reactive visuals

**Key Commands:**
```bash
# Install FFmpeg
brew install ffmpeg  # macOS

# Convert to ProRes
ffmpeg -i shader.webm -c:v prores_ks -profile:v 3 output.mov

# Import to Ableton
# Drag output.mov onto audio track
```

**Result:** Professional audio-visual performance setup!

---

**END OF FFMPEG VIDEO EXPORT GUIDE**

Version: V3.6.0  
Date: November 16, 2025  
License: GPL-3.0  
Website: https://onemanshyo.com
