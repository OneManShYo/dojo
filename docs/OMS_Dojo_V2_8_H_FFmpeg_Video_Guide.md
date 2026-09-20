# OMS DOJO - FFMPEG VIDEO CONVERSION GUIDE

**Version:** V2.7.0  
**Purpose:** Convert WebM exports to ProRes 422 for Ableton Live + EboSuite  
**Audience:** DJ/VJ users preparing videos for live performance

---

## TABLE OF CONTENTS

1. [Why ProRes 422?](#why-prores-422)
2. [Installing FFmpeg](#installing-ffmpeg)
3. [Basic Conversion](#basic-conversion)
4. [Batch Processing](#batch-processing)
5. [Advanced Options](#advanced-options)
6. [Troubleshooting](#troubleshooting)

---

## WHY PRORES 422?

### The Problem

ONEMANSHYO Dojo exports videos as WebM (VP8 codec). While this format works fine for web playback, EboSuite (Max for Live video plugin) performs best with ProRes 422.

**WebM Issues in EboSuite:**
- Occasional frame drops
- Higher CPU usage
- Playback timing inconsistencies
- Not optimized for real-time performance

**ProRes 422 Benefits:**
- Smooth playback in EboSuite
- Lower CPU overhead
- Frame-accurate timing
- Industry standard for video production
- Optimized for real-time decoding

### When to Convert

**Always convert for:**
- Live DJ performances
- EboSuite video playback
- Critical timing applications
- Professional sets

**Optional for:**
- Testing/preview (WebM works fine)
- YouTube/social media uploads (WebM fine)
- Archival purposes (keep both formats)

---

## INSTALLING FFMPEG

### macOS

**Option 1: Homebrew (Recommended)**

1. Install Homebrew (if not already):
   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

2. Install FFmpeg:
   ```bash
   brew install ffmpeg
   ```

3. Verify installation:
   ```bash
   ffmpeg -version
   ```

**Option 2: Download Binary**

1. Go to https://ffmpeg.org/download.html
2. Download macOS build
3. Extract to `/usr/local/bin`
4. Add to PATH if needed

### Windows

**Option 1: Chocolatey**

1. Install Chocolatey: https://chocolatey.org/install
2. Run in PowerShell (Admin):
   ```powershell
   choco install ffmpeg
   ```

**Option 2: Manual Install**

1. Download from https://ffmpeg.org/download.html
2. Extract to `C:\ffmpeg`
3. Add `C:\ffmpeg\bin` to PATH:
   - System Properties → Environment Variables
   - Edit PATH variable
   - Add FFmpeg bin directory

4. Verify in Command Prompt:
   ```cmd
   ffmpeg -version
   ```

### Linux

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install ffmpeg
```

**Fedora:**
```bash
sudo dnf install ffmpeg
```

**Arch:**
```bash
sudo pacman -S ffmpeg
```

---

## BASIC CONVERSION

### Single File Conversion

**Command Structure:**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 3 -c:a pcm_s16le output.mov
```

**Breakdown:**
- `-i input.webm` - Input file
- `-c:v prores_ks` - ProRes encoder (high quality)
- `-profile:v 3` - ProRes 422 profile
- `-c:a pcm_s16le` - Uncompressed audio (if present)
- `output.mov` - Output file

**Example:**
```bash
ffmpeg -i OMS_Dojo_20251109_143022.webm -c:v prores_ks -profile:v 3 -c:a pcm_s16le Drop_Section_ProRes.mov
```

### Silent Videos (No Audio)

If your WebM has no audio track (shader-only mode), omit audio codec:

```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 3 output.mov
```

### Verify Conversion

**Check output file:**
```bash
ffmpeg -i output.mov
```

**Look for:**
- Video codec: prores (prores422)
- Resolution: Matches input
- Frame rate: Matches input
- Duration: Matches input

---

## BATCH PROCESSING

### Convert All WebM Files in Folder

**macOS/Linux:**

Create script `convert_all.sh`:
```bash
#!/bin/bash
for file in *.webm; do
    filename="${file%.webm}"
    ffmpeg -i "$file" -c:v prores_ks -profile:v 3 -c:a pcm_s16le "${filename}_ProRes.mov"
done
```

**Make executable and run:**
```bash
chmod +x convert_all.sh
./convert_all.sh
```

**Windows (PowerShell):**

Create script `convert_all.ps1`:
```powershell
Get-ChildItem -Filter *.webm | ForEach-Object {
    $outputName = $_.BaseName + "_ProRes.mov"
    ffmpeg -i $_.Name -c:v prores_ks -profile:v 3 -c:a pcm_s16le $outputName
}
```

**Run:**
```powershell
.\convert_all.ps1
```

### Organized Output Folder

**macOS/Linux:**
```bash
#!/bin/bash
mkdir -p ProRes
for file in *.webm; do
    filename="${file%.webm}"
    ffmpeg -i "$file" -c:v prores_ks -profile:v 3 -c:a pcm_s16le "ProRes/${filename}.mov"
done
```

**Windows (PowerShell):**
```powershell
New-Item -ItemType Directory -Force -Path ProRes
Get-ChildItem -Filter *.webm | ForEach-Object {
    $outputName = "ProRes\" + $_.BaseName + ".mov"
    ffmpeg -i $_.Name -c:v prores_ks -profile:v 3 -c:a pcm_s16le $outputName
}
```

---

## ADVANCED OPTIONS

### ProRes Profiles

**Available Profiles:**
- `0` - ProRes 422 Proxy (smallest, lower quality)
- `1` - ProRes 422 LT (light, good quality)
- `2` - ProRes 422 (standard, recommended)
- `3` - ProRes 422 HQ (high quality, larger files)
- `4` - ProRes 4444 (with alpha, largest)

**Default (profile 3) is best for EboSuite.**

**If file size is critical:**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 2 output.mov
```

### Resolution Scaling

**Downscale to 1080p (if exported 4K):**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 3 -vf scale=1920:1080 output.mov
```

**Maintain aspect ratio:**
```bash
# Portrait: scale to 1080 width
ffmpeg -i input.webm -c:v prores_ks -profile:v 3 -vf scale=1080:-1 output.mov

# Landscape: scale to 1920 width
ffmpeg -i input.webm -c:v prores_ks -profile:v 3 -vf scale=1920:-1 output.mov
```

### Frame Rate Conversion

**Convert 24fps to 30fps:**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 3 -r 30 output.mov
```

**Convert 60fps to 30fps:**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 3 -r 30 output.mov
```

**Note:** Not recommended - render at target FPS in Dojo instead.

### Audio Options

**Remove audio entirely:**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 3 -an output.mov
```

**Keep original audio codec:**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 3 -c:a copy output.mov
```

**Convert to AAC (if needed):**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 3 -c:a aac -b:a 320k output.mov
```

### Quality Settings

**Highest quality (larger files):**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 4 -qscale:v 9 output.mov
```

**Faster encoding (lower quality):**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 1 output.mov
```

---

## BATCH CONVERSION WITH METADATA

### Preserve Creation Date

**macOS/Linux:**
```bash
#!/bin/bash
for file in *.webm; do
    filename="${file%.webm}"
    timestamp=$(stat -f "%Sm" -t "%Y%m%d_%H%M%S" "$file")
    ffmpeg -i "$file" -c:v prores_ks -profile:v 3 -c:a pcm_s16le "${filename}_ProRes.mov"
    touch -r "$file" "${filename}_ProRes.mov"
done
```

### Add Custom Metadata

```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 3 \
  -metadata title="Track Name - Drop Section" \
  -metadata artist="DJ Name" \
  -metadata comment="ONEMANSHYO Dojo V2.7.0" \
  output.mov
```

---

## FILE SIZE REFERENCE

**WebM (VP8) vs ProRes 422:**

**1080p 30fps 60sec:**
- WebM: ~50MB
- ProRes 422: ~400MB (8x larger)
- ProRes 422 HQ: ~600MB (12x larger)

**4K 60fps 30sec:**
- WebM: ~80MB
- ProRes 422: ~800MB (10x larger)
- ProRes 422 HQ: ~1.2GB (15x larger)

**Storage Requirements (DJ Set):**
- 10 tracks × 2 minutes each
- ProRes 422: ~13GB total
- ProRes 422 HQ: ~20GB total

**Recommendation:** Use ProRes 422 (profile 3), not HQ, for good balance.

---

## WORKFLOW INTEGRATION

### Complete DJ Set Preparation

**1. Render in Dojo:**
- Select "HD Render" or "4K Final" Control State
- Render all track videos (WebM format)

**2. Batch Convert to ProRes:**
```bash
# In folder with all WebM files
for file in *.webm; do
    ffmpeg -i "$file" -c:v prores_ks -profile:v 3 -an "${file%.webm}_ProRes.mov"
done
```

**3. Import to Ableton:**
- Open Ableton Live session
- Load EboSuite on video channels
- Drag ProRes files into EboSuite slots
- Delete or archive WebM originals

**4. Performance:**
- Trigger videos with MIDI controller
- Mix with DJ audio tracks
- ProRes ensures smooth playback

### File Organization

**Recommended Structure:**
```
DJ_Set_2025/
├── WebM_Source/
│   ├── Track1_Drop.webm
│   ├── Track2_Breakdown.webm
│   └── Track3_Drop.webm
├── ProRes_Finals/
│   ├── Track1_Drop_ProRes.mov
│   ├── Track2_Breakdown_ProRes.mov
│   └── Track3_Drop_ProRes.mov
└── Ableton_Session/
    └── Set.als
```

---

## TROUBLESHOOTING

### "Command not found: ffmpeg"

**Solution:**
- Verify installation: `ffmpeg -version`
- Check PATH environment variable
- Reinstall FFmpeg
- Restart terminal/command prompt

### "Unknown encoder 'prores_ks'"

**Solution:**
- Use older encoder: `-c:v prores` instead of `-c:v prores_ks`
- Update FFmpeg: `brew upgrade ffmpeg` (macOS)
- Or use: `-c:v prores_aw` (Apple ProRes encoder)

### Output File Huge

**Cause:** ProRes is uncompressed/lightly compressed.

**Solutions:**
- Use ProRes 422 LT (profile 1): `-profile:v 1`
- Downscale resolution: `-vf scale=1920:1080`
- Accept larger file size (necessary for performance)

### Conversion Very Slow

**Cause:** High resolution + high quality.

**Solutions:**
- Use faster profile: `-profile:v 2`
- Enable hardware acceleration (if available)
- Close other applications
- Be patient (4K conversion takes time)

### Audio Out of Sync

**Cause:** Frame rate mismatch or codec issues.

**Solution:**
- Use PCM audio: `-c:a pcm_s16le`
- Or remove audio entirely: `-an`
- Verify source video timing in Dojo

### "Invalid profile 3"

**Cause:** Old FFmpeg version.

**Solution:**
- Update FFmpeg
- Or use profile 2 instead: `-profile:v 2`

### EboSuite Still Drops Frames

**Cause:** Computer performance, not video format.

**Solutions:**
- Lower resolution (1080p instead of 4K)
- Lower FPS (30fps instead of 60fps)
- Reduce shader complexity
- Close other applications
- Check EboSuite buffer settings

---

## ALTERNATIVE TOOLS

### Compressor (macOS)

**Built-in video conversion:**
1. Open Compressor app
2. Drag WebM file
3. Select "ProRes 422" preset
4. Click Start Batch
5. Wait for conversion

**Pros:** GUI, easy to use  
**Cons:** Slower than FFmpeg, less control

### HandBrake

**Free GUI tool:**
- Download: https://handbrake.fr/
- Limited ProRes support
- Better for other formats (H.264, H.265)

**Not recommended for ProRes - use FFmpeg instead.**

### Adobe Media Encoder

**Professional tool:**
- Part of Adobe Creative Cloud
- Excellent ProRes support
- Batch processing
- GPU acceleration

**Pros:** Professional features, fast  
**Cons:** Requires subscription

---

## PERFORMANCE TIPS

### Faster Conversions

**1. Use Hardware Acceleration (if available):**
```bash
# macOS (VideoToolbox)
ffmpeg -hwaccel videotoolbox -i input.webm -c:v prores_ks -profile:v 3 output.mov

# Windows (NVIDIA)
ffmpeg -hwaccel cuda -i input.webm -c:v prores_ks -profile:v 3 output.mov
```

**2. Use Multiple Threads:**
```bash
ffmpeg -threads 8 -i input.webm -c:v prores_ks -profile:v 3 output.mov
```

**3. Convert Multiple Files in Parallel:**
```bash
# macOS/Linux
parallel -j 4 ffmpeg -i {} -c:v prores_ks -profile:v 3 {.}_ProRes.mov ::: *.webm
```

**Note:** Requires GNU Parallel: `brew install parallel`

### Storage Optimization

**After conversion:**
1. Test ProRes file in EboSuite
2. If working, delete WebM source
3. Archive ProRes to external drive
4. Keep only active set videos on SSD

**Backup Strategy:**
- Keep WebM sources archived (small)
- Regenerate ProRes if needed
- ProRes files are large but temporary

---

## QUICK REFERENCE

### Most Common Command

**Silent video (shader-only mode):**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 3 output.mov
```

**Video with audio:**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 3 -c:a pcm_s16le output.mov
```

**Batch convert all in folder:**
```bash
for file in *.webm; do ffmpeg -i "$file" -c:v prores_ks -profile:v 3 "${file%.webm}.mov"; done
```

### Profile Quick Reference

- `1` - ProRes 422 LT (smaller files)
- `2` - ProRes 422 (balanced)
- `3` - ProRes 422 HQ (best quality, recommended)

### Resolution Scaling

```bash
# 1080p landscape
ffmpeg -i input.webm -c:v prores_ks -profile:v 3 -vf scale=1920:1080 output.mov

# 1080p portrait
ffmpeg -i input.webm -c:v prores_ks -profile:v 3 -vf scale=1080:1920 output.mov
```

---

## ADDITIONAL RESOURCES

**FFmpeg Documentation:**
https://ffmpeg.org/documentation.html

**ProRes Specifications:**
https://support.apple.com/en-us/HT202410

**EboSuite Manual:**
https://ebosuite.com/manual/

**ONEMANSHYO Dojo:**
https://www.itswessmithyo.com/

---

## APPENDIX: CODEC COMPARISON

### ProRes 422 vs Other Formats

**ProRes 422:**
- Codec: Apple ProRes 422
- Container: .mov
- Compression: Intra-frame (each frame independent)
- Use case: Real-time editing/playback
- CPU overhead: Low
- Quality: Very high

**H.264:**
- Codec: AVC/H.264
- Container: .mp4, .mov
- Compression: Inter-frame (temporal compression)
- Use case: Streaming, web, archival
- CPU overhead: High (decoding)
- Quality: Good (at high bitrates)

**VP8/WebM:**
- Codec: VP8
- Container: .webm
- Compression: Inter-frame
- Use case: Web video, YouTube
- CPU overhead: High
- Quality: Good

**Why ProRes for EboSuite:**
- Intra-frame = easier decoding
- Lower CPU usage
- Frame-accurate seeking
- No temporal artifacts
- Real-time performance

---

**END OF FFMPEG VIDEO CONVERSION GUIDE**
