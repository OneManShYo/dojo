# ONEMANSHYO Dojo - FFmpeg Video Guide

## Overview

ONEMANSHYO Dojo exports videos as WebM (VP8 codec). While this format works great for web playback, you may need to convert to H.264/MP4 for compatibility with:

- Social media platforms (Instagram, TikTok, Twitter)
- Video editing software (Premiere, Final Cut, DaVinci)
- Mobile devices (iOS, Android)
- Live performance software (EboSuite, Resolume)

This guide covers FFmpeg video processing for ONEMANSHYO Dojo outputs.

---

## Installing FFmpeg

### Windows

**Option 1: Chocolatey (Recommended)**
```bash
choco install ffmpeg
```

**Option 2: Manual**
1. Download from https://ffmpeg.org/download.html
2. Extract to `C:\ffmpeg`
3. Add `C:\ffmpeg\bin` to system PATH
4. Restart terminal

### macOS

**Homebrew (Recommended)**
```bash
brew install ffmpeg
```

### Linux

**Ubuntu/Debian**
```bash
sudo apt update
sudo apt install ffmpeg
```

**Fedora/RHEL**
```bash
sudo dnf install ffmpeg
```

**Verify Installation:**
```bash
ffmpeg -version
```

---

## Basic Conversions

### WebM → H.264/MP4 (Standard)

**Balanced quality and file size:**
```bash
ffmpeg -i input.webm -c:v libx264 -preset medium -crf 23 -c:a aac -b:a 192k output.mp4
```

**Parameters explained:**
- `-i input.webm`: Input file
- `-c:v libx264`: H.264 video codec
- `-preset medium`: Encoding speed (ultrafast|fast|medium|slow|veryslow)
- `-crf 23`: Quality (0-51, lower=better, 23=good default)
- `-c:a aac`: AAC audio codec
- `-b:a 192k`: Audio bitrate
- `output.mp4`: Output file

### WebM → H.264/MP4 (High Quality)

**For archival or professional use:**
```bash
ffmpeg -i input.webm -c:v libx264 -preset slow -crf 18 -c:a aac -b:a 320k output.mp4
```

**Better quality:**
- `-preset slow`: More encoding time, better compression
- `-crf 18`: Higher quality (visually lossless)
- `-b:a 320k`: Higher audio bitrate

### WebM → H.264/MP4 (Fast Encode)

**For quick previews:**
```bash
ffmpeg -i input.webm -c:v libx264 -preset veryfast -crf 28 -c:a aac -b:a 128k output.mp4
```

**Faster but lower quality:**
- `-preset veryfast`: Minimal encoding time
- `-crf 28`: Lower quality (still acceptable)
- `-b:a 128k`: Lower audio bitrate

---

## Social Media Optimizations

### Instagram/TikTok (Portrait 1080×1920)

```bash
ffmpeg -i input.webm -c:v libx264 -preset medium -crf 23 \
  -vf "scale=1080:1920:force_original_aspect_ratio=decrease,pad=1080:1920:(ow-iw)/2:(oh-ih)/2" \
  -c:a aac -b:a 192k -movflags +faststart output.mp4
```

**Ensures:**
- Portrait aspect ratio (9:16)
- Proper scaling/padding
- Fast start for streaming

### Instagram/TikTok (Square 1080×1080)

```bash
ffmpeg -i input.webm -c:v libx264 -preset medium -crf 23 \
  -vf "scale=1080:1080:force_original_aspect_ratio=decrease,pad=1080:1080:(ow-iw)/2:(oh-ih)/2" \
  -c:a aac -b:a 192k -movflags +faststart output.mp4
```

### YouTube (Landscape 1920×1080)

```bash
ffmpeg -i input.webm -c:v libx264 -preset slow -crf 20 \
  -vf "scale=1920:1080:force_original_aspect_ratio=decrease" \
  -c:a aac -b:a 320k -movflags +faststart output.mp4
```

**Higher quality for YouTube:**
- `-crf 20`: Better quality
- `-b:a 320k`: Higher audio bitrate

### Twitter (1280×720, File Size Limit)

```bash
ffmpeg -i input.webm -c:v libx264 -preset medium -crf 25 \
  -vf "scale=1280:720:force_original_aspect_ratio=decrease" \
  -c:a aac -b:a 128k -movflags +faststart -fs 512M output.mp4
```

**Twitter requirements:**
- `-fs 512M`: Max file size (512MB limit)
- Lower resolution if needed

---

## Resolution Scaling

### Upscale (720p → 1080p)

```bash
ffmpeg -i input.webm -vf "scale=1920:1080" -c:v libx264 -preset medium -crf 23 output.mp4
```

**Note:** Upscaling doesn't add detail, just makes file larger.

### Downscale (1080p → 720p)

```bash
ffmpeg -i input.webm -vf "scale=1280:720" -c:v libx264 -preset medium -crf 23 output.mp4
```

**Benefits:** Smaller file size, faster upload/playback

### Maintain Aspect Ratio

```bash
ffmpeg -i input.webm -vf "scale=-1:720" -c:v libx264 -preset medium -crf 23 output.mp4
```

**`-1` auto-calculates width to maintain aspect ratio**

---

## Frame Rate Adjustments

### Change FPS (30fps → 60fps)

```bash
ffmpeg -i input.webm -filter:v "fps=60" -c:v libx264 -preset medium -crf 23 output.mp4
```

**Note:** Doubling FPS creates interpolated frames (may look choppy)

### Change FPS (60fps → 30fps)

```bash
ffmpeg -i input.webm -filter:v "fps=30" -c:v libx264 -preset medium -crf 23 output.mp4
```

**Benefits:** Smaller file size, maintains smooth playback

### Change FPS with Motion Interpolation

```bash
ffmpeg -i input.webm -filter:v "minterpolate='mi_mode=mci:mc_mode=aobmc:vsbmc=1:fps=60'" \
  -c:v libx264 -preset slow -crf 20 output.mp4
```

**Creates smoother 60fps from 30fps (slow encoding)**

---

## Audio Processing

### Remove Audio (Silent Video)

```bash
ffmpeg -i input.webm -an -c:v libx264 -preset medium -crf 23 output.mp4
```

**`-an` removes audio track**

### Replace Audio

```bash
ffmpeg -i input.webm -i newaudio.mp3 -c:v copy -c:a aac -b:a 192k -map 0:v:0 -map 1:a:0 output.mp4
```

**Maps:**
- Video from input.webm
- Audio from newaudio.mp3

### Adjust Audio Volume

```bash
ffmpeg -i input.webm -c:v libx264 -preset medium -crf 23 -af "volume=1.5" output.mp4
```

**`volume=1.5` increases by 50% (0.5 = half volume)**

---

## Batch Processing

### Convert All WebM Files in Directory

**Windows (PowerShell):**
```powershell
Get-ChildItem *.webm | ForEach-Object {
    ffmpeg -i $_.Name -c:v libx264 -preset medium -crf 23 -c:a aac -b:a 192k ($_.BaseName + ".mp4")
}
```

**macOS/Linux (Bash):**
```bash
for file in *.webm; do
    ffmpeg -i "$file" -c:v libx264 -preset medium -crf 23 -c:a aac -b:a 192k "${file%.webm}.mp4"
done
```

### Batch Convert with Custom Settings

```bash
#!/bin/bash
for file in *.webm; do
    output="${file%.webm}_1080p.mp4"
    ffmpeg -i "$file" \
        -vf "scale=1920:1080:force_original_aspect_ratio=decrease" \
        -c:v libx264 -preset medium -crf 23 \
        -c:a aac -b:a 192k \
        -movflags +faststart \
        "$output"
    echo "Converted: $file → $output"
done
```

---

## Compression & File Size

### High Compression (Smaller File)

```bash
ffmpeg -i input.webm -c:v libx264 -preset veryslow -crf 28 -c:a aac -b:a 96k output.mp4
```

**Trade-offs:**
- Much slower encoding
- Smaller file size
- Lower quality

### Two-Pass Encoding (Best Quality/Size Ratio)

**Pass 1:**
```bash
ffmpeg -i input.webm -c:v libx264 -preset slow -b:v 5M -pass 1 -an -f mp4 NUL
```

**Pass 2:**
```bash
ffmpeg -i input.webm -c:v libx264 -preset slow -b:v 5M -pass 2 -c:a aac -b:a 192k output.mp4
```

**Best for:**
- Consistent file sizes
- Maximum quality at target bitrate

### Target File Size

```bash
# Calculate bitrate for 50MB target
# Formula: (target_size_MB * 8192) / duration_seconds - audio_bitrate
# Example: 30-second video, 50MB target, 192k audio
# (50 * 8192) / 30 - 192 = ~13472 kbps video bitrate

ffmpeg -i input.webm -c:v libx264 -b:v 13472k -c:a aac -b:a 192k output.mp4
```

---

## Advanced Filters

### Add Fade In/Out

```bash
ffmpeg -i input.webm \
  -vf "fade=in:0:30,fade=out:870:30" \
  -c:v libx264 -preset medium -crf 23 output.mp4
```

**Parameters:**
- `fade=in:0:30`: Fade in at frame 0 for 30 frames
- `fade=out:870:30`: Fade out at frame 870 for 30 frames

### Add Text Overlay

```bash
ffmpeg -i input.webm \
  -vf "drawtext=text='ONEMANSHYO Dojo':fontsize=48:fontcolor=white:x=(w-text_w)/2:y=h-th-20" \
  -c:v libx264 -preset medium -crf 23 output.mp4
```

**Creates centered text at bottom**

### Concatenate Videos

**Create list.txt:**
```
file 'video1.webm'
file 'video2.webm'
file 'video3.webm'
```

**Concatenate:**
```bash
ffmpeg -f concat -safe 0 -i list.txt -c copy output.mp4
```

---

## Preset Commands (Quick Reference)

### Standard Web Export
```bash
ffmpeg -i input.webm -c:v libx264 -preset medium -crf 23 -c:a aac -b:a 192k output.mp4
```

### High Quality Archive
```bash
ffmpeg -i input.webm -c:v libx264 -preset slow -crf 18 -c:a aac -b:a 320k output.mp4
```

### Social Media (Portrait)
```bash
ffmpeg -i input.webm -c:v libx264 -preset medium -crf 23 \
  -vf "scale=1080:1920:force_original_aspect_ratio=decrease" \
  -c:a aac -b:a 192k -movflags +faststart output.mp4
```

### Social Media (Square)
```bash
ffmpeg -i input.webm -c:v libx264 -preset medium -crf 23 \
  -vf "scale=1080:1080:force_original_aspect_ratio=decrease" \
  -c:a aac -b:a 192k -movflags +faststart output.mp4
```

### Quick Preview (Fast Encode)
```bash
ffmpeg -i input.webm -c:v libx264 -preset veryfast -crf 28 output.mp4
```

---

## EboSuite / Resolume Workflows

### Optimal Settings for Live Performance

```bash
ffmpeg -i input.webm \
  -c:v libx264 -preset medium -crf 20 \
  -vf "scale=-2:1080" \
  -pix_fmt yuv420p \
  -movflags +faststart \
  -an \
  output.mp4
```

**Why these settings:**
- `-crf 20`: High quality for projection
- `-pix_fmt yuv420p`: Maximum compatibility
- `-movflags +faststart`: Fast loading in software
- `-an`: Remove audio (not needed, saves space)

### Batch Convert for Performance Library

```bash
#!/bin/bash
for file in *.webm; do
    output="${file%.webm}_performance.mp4"
    ffmpeg -i "$file" \
        -c:v libx264 -preset medium -crf 20 \
        -pix_fmt yuv420p \
        -movflags +faststart \
        -an \
        "$output"
done
```

---

## Troubleshooting

### "Unknown encoder 'libx264'"

**Problem:** H.264 codec not installed

**Solution:**
```bash
# macOS
brew reinstall ffmpeg --with-x264

# Linux
sudo apt install libx264-dev
```

### Output Video Stutters

**Problem:** Frame rate mismatch

**Solution:** Match output FPS to source:
```bash
ffmpeg -i input.webm -r 30 -c:v libx264 -preset medium -crf 23 output.mp4
```

### Audio Out of Sync

**Problem:** Audio/video drift

**Solution:** Use `-async 1` flag:
```bash
ffmpeg -i input.webm -async 1 -c:v libx264 -preset medium -crf 23 output.mp4
```

### File Size Too Large

**Problem:** Output larger than expected

**Solutions:**
1. Increase CRF value (23 → 28)
2. Lower resolution (1080p → 720p)
3. Reduce audio bitrate (192k → 128k)
4. Use slower preset for better compression

---

## CRF Quality Guide

**CRF (Constant Rate Factor) Scale:**
- **0-17:** Visually lossless (huge files)
- **18-23:** High quality (recommended)
- **23-28:** Good quality (default range)
- **28-35:** Lower quality (smaller files)
- **35-51:** Poor quality (not recommended)

**Recommendations:**
- **Archival:** CRF 18
- **General use:** CRF 23
- **Web upload:** CRF 25
- **Social media:** CRF 26-28

---

## Preset Speed vs Quality

**Preset Speed Scale:**
- **ultrafast:** Fastest encode, largest file
- **veryfast:** Very fast, large file
- **fast:** Fast, good for previews
- **medium:** Balanced (default)
- **slow:** Better compression, smaller file
- **veryslow:** Best compression, very slow
- **placebo:** Negligible improvement, extremely slow

**Recommendations:**
- **Quick preview:** veryfast
- **Most cases:** medium
- **Archival:** slow
- **Maximum compression:** veryslow

---

## File Organization

### Suggested Structure

```
MyProject/
├── source/           # Original WebM from Dojo
│   ├── shader1.webm
│   └── shader2.webm
├── converted/        # H.264 MP4 versions
│   ├── shader1.mp4
│   └── shader2.mp4
├── social/           # Optimized for platforms
│   ├── shader1_ig.mp4
│   └── shader1_yt.mp4
└── scripts/          # Batch conversion scripts
    └── convert_all.sh
```

---

## Additional Resources

**FFmpeg Documentation:**
- Official: https://ffmpeg.org/documentation.html
- Wiki: https://trac.ffmpeg.org/wiki

**Video Codec Guides:**
- H.264: https://trac.ffmpeg.org/wiki/Encode/H.264
- VP9: https://trac.ffmpeg.org/wiki/Encode/VP9

**Community:**
- FFmpeg Reddit: r/ffmpeg
- Video Stack Exchange: video.stackexchange.com

---

**FFmpeg Guide Version:** V2_6  
**Last Updated:** November 2025  
**Target Audience:** Users converting ONEMANSHYO Dojo videos

**Convert with confidence!**
