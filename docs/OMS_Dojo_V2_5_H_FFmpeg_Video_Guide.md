# ONEMANSHYO Dojo - FFmpeg Video Processing Guide

## Overview

This guide covers post-processing ONEMANSHYO Dojo videos with FFmpeg for format conversion, optimization, and advanced manipulation.

---

## Installing FFmpeg

### Windows
Download from: https://ffmpeg.org/download.html
Add to PATH or use full path to ffmpeg.exe

### macOS
```bash
brew install ffmpeg
```

### Linux
```bash
sudo apt install ffmpeg  # Debian/Ubuntu
sudo dnf install ffmpeg  # Fedora
```

---

## Common Operations

### 1. Convert WebM to MP4

**Basic Conversion (Re-encode):**
```bash
ffmpeg -i input.webm -c:v libx264 -crf 23 -c:a aac -b:a 192k output.mp4
```

**Fast Conversion (Copy streams if compatible):**
```bash
ffmpeg -i input.webm -c copy output.mp4
```

**High Quality:**
```bash
ffmpeg -i input.webm -c:v libx264 -crf 18 -preset slow -c:a aac -b:a 256k output.mp4
```

### 2. Adjust Video Quality

**Lower File Size (Higher CRF):**
```bash
ffmpeg -i input.webm -c:v libx264 -crf 28 -c:a aac -b:a 128k output.mp4
```
*CRF range: 18 (high quality) to 28 (lower quality, smaller size)*

**Maximum Quality:**
```bash
ffmpeg -i input.webm -c:v libx264 -crf 15 -preset veryslow -c:a aac -b:a 320k output.mp4
```

### 3. Change Resolution

**Downscale to 1080p:**
```bash
ffmpeg -i input.webm -vf scale=1920:1080 -c:v libx264 -crf 23 output.mp4
```

**Maintain Aspect Ratio:**
```bash
ffmpeg -i input.webm -vf scale=1920:-1 -c:v libx264 -crf 23 output.mp4
```

**Upscale to 4K (not recommended):**
```bash
ffmpeg -i input.webm -vf scale=3840:2160 -c:v libx264 -crf 20 output.mp4
```

### 4. Change Frame Rate

**Convert to 30 FPS:**
```bash
ffmpeg -i input.webm -r 30 -c:v libx264 -crf 23 output.mp4
```

**Convert to 24 FPS (cinematic):**
```bash
ffmpeg -i input.webm -r 24 -c:v libx264 -crf 23 output.mp4
```

### 5. Adjust Audio

**Change Audio Volume (+6dB):**
```bash
ffmpeg -i input.webm -c:v copy -af "volume=6dB" -c:a aac output.mp4
```

**Change Audio Bitrate:**
```bash
ffmpeg -i input.webm -c:v copy -c:a aac -b:a 256k output.mp4
```

**Remove Audio:**
```bash
ffmpeg -i input.webm -c:v copy -an output.mp4
```

**Replace Audio:**
```bash
ffmpeg -i input.webm -i new_audio.mp3 -c:v copy -c:a aac -map 0:v:0 -map 1:a:0 output.mp4
```

### 6. Trim Video

**Trim to Specific Duration:**
```bash
ffmpeg -i input.webm -ss 00:00:05 -to 00:00:35 -c copy output.webm
```
*Starts at 5 seconds, ends at 35 seconds*

**Cut First 3 Seconds:**
```bash
ffmpeg -i input.webm -ss 00:00:03 -c copy output.webm
```

### 7. Concatenate Multiple Videos

**Create input list file (filelist.txt):**
```
file 'video1.webm'
file 'video2.webm'
file 'video3.webm'
```

**Concatenate:**
```bash
ffmpeg -f concat -safe 0 -i filelist.txt -c copy output.webm
```

### 8. Add Text Overlay

**Simple Text:**
```bash
ffmpeg -i input.webm -vf "drawtext=text='OMS':fontsize=48:fontcolor=white:x=10:y=10" -c:a copy output.mp4
```

**Centered Text:**
```bash
ffmpeg -i input.webm -vf "drawtext=text='OneManShYo':fontsize=64:fontcolor=white:x=(w-text_w)/2:y=(h-text_h)/2" -c:a copy output.mp4
```

### 9. Add Watermark Image

```bash
ffmpeg -i input.webm -i watermark.png -filter_complex "overlay=10:10" -c:a copy output.mp4
```

**Positioned watermark (bottom-right):**
```bash
ffmpeg -i input.webm -i watermark.png -filter_complex "overlay=W-w-10:H-h-10" -c:a copy output.mp4
```

### 10. Create GIF

**High Quality GIF:**
```bash
ffmpeg -i input.webm -vf "fps=15,scale=640:-1:flags=lanczos" -c:v gif output.gif
```

**Optimized GIF (with palette):**
```bash
ffmpeg -i input.webm -vf "fps=15,scale=640:-1:flags=lanczos,palettegen" palette.png
ffmpeg -i input.webm -i palette.png -filter_complex "fps=15,scale=640:-1:flags=lanczos[x];[x][1:v]paletteuse" output.gif
```

---

## Platform-Specific Optimization

### Instagram Reels / TikTok

**Portrait 1080x1920, 30 FPS, High Quality:**
```bash
ffmpeg -i input.webm -vf scale=1080:1920 -r 30 -c:v libx264 -crf 20 -preset slow -c:a aac -b:a 192k output.mp4
```

### YouTube

**1080p Upload:**
```bash
ffmpeg -i input.webm -c:v libx264 -crf 18 -preset slow -pix_fmt yuv420p -c:a aac -b:a 256k output.mp4
```

**4K Upload:**
```bash
ffmpeg -i input.webm -c:v libx264 -crf 17 -preset slow -pix_fmt yuv420p -c:a aac -b:a 320k output.mp4
```

### Twitter/X

**Maximum compatibility:**
```bash
ffmpeg -i input.webm -c:v libx264 -crf 23 -preset medium -pix_fmt yuv420p -c:a aac -b:a 192k -movflags +faststart output.mp4
```

---

## Batch Processing

### Convert All WebM Files in Directory

**Windows (PowerShell):**
```powershell
Get-ChildItem *.webm | ForEach-Object {
    ffmpeg -i $_.Name -c:v libx264 -crf 23 -c:a aac ("$($_.BaseName).mp4")
}
```

**macOS / Linux (Bash):**
```bash
for file in *.webm; do
    ffmpeg -i "$file" -c:v libx264 -crf 23 -c:a aac "${file%.webm}.mp4"
done
```

---

## Ableton Live Integration

### Prepare Videos for Ableton

**Consistent Format (No Audio):**
```bash
ffmpeg -i input.webm -an -c:v libx264 -crf 20 -preset medium -pix_fmt yuv420p output.mp4
```

**Batch Convert for Live Performance:**
```bash
# Create consistent format for all clips
for file in *.webm; do
    ffmpeg -i "$file" -an -c:v libx264 -crf 20 -r 30 -s 1920x1080 -preset fast -pix_fmt yuv420p "${file%.webm}_live.mp4"
done
```

---

## Quality vs File Size Reference

| CRF Value | Quality | Use Case |
|-----------|---------|----------|
| 15-18 | Excellent | Archival, professional work |
| 20-23 | Very Good | Standard delivery, YouTube |
| 24-26 | Good | Social media, streaming |
| 27-30 | Acceptable | Low bandwidth, mobile |
| 31+ | Poor | Not recommended |

---

## Common FFmpeg Flags Explained

**Video Encoding:**
- `-c:v libx264` - Use H.264 codec
- `-crf 23` - Quality (lower = better, 18-28 typical)
- `-preset slow` - Encoding speed (slow = better quality)
- `-r 30` - Frame rate
- `-pix_fmt yuv420p` - Pixel format (most compatible)

**Audio Encoding:**
- `-c:a aac` - Use AAC audio codec
- `-b:a 192k` - Audio bitrate (128k-320k typical)
- `-an` - Remove audio entirely

**Other:**
- `-c copy` - Copy stream without re-encoding (fast)
- `-movflags +faststart` - Optimize for web streaming
- `-ss` - Start time
- `-to` - End time
- `-t` - Duration

---

## Troubleshooting

**Problem: "Unknown encoder 'libx264'"**
Solution: Install FFmpeg with H.264 support (most builds include it)

**Problem: Output file is huge**
Solution: Increase CRF value (23-28) or lower bitrate

**Problem: Video looks pixelated**
Solution: Decrease CRF value (18-20) or increase bitrate

**Problem: Audio/video out of sync**
Solution: Re-encode both streams (don't use -c copy)

**Problem: Colors look washed out**
Solution: Add `-pix_fmt yuv420p` flag

---

## Advanced Techniques

### Two-Pass Encoding (Highest Quality)

**Pass 1:**
```bash
ffmpeg -i input.webm -c:v libx264 -b:v 5000k -pass 1 -f mp4 /dev/null
```

**Pass 2:**
```bash
ffmpeg -i input.webm -c:v libx264 -b:v 5000k -pass 2 -c:a aac output.mp4
```

### Hardware Acceleration (NVIDIA)

```bash
ffmpeg -hwaccel cuda -i input.webm -c:v h264_nvenc -preset slow -crf 20 output.mp4
```

### Create Contact Sheet (Thumbnail Grid)

```bash
ffmpeg -i input.webm -vf "select='not(mod(n,30))',scale=320:180,tile=4x3" -frames:v 1 contact_sheet.png
```

---

## Resources

**FFmpeg Documentation:**
https://ffmpeg.org/documentation.html

**FFmpeg Wiki:**
https://trac.ffmpeg.org/wiki

**Codec Comparison:**
https://trac.ffmpeg.org/wiki/Encode/H.264

---

**Last Updated:** November 2025  
**Version:** V2_5  
**For:** ONEMANSHYO Dojo post-processing
