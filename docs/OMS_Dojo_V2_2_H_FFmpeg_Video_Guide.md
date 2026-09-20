# ONEMANSHYO Dojo V2_2 - FFmpeg Video Guide

**Version:** V2_2  
**Purpose:** Post-processing ONEMANSHYO Dojo outputs with FFmpeg  
**Target Audience:** Content creators, streamers, video producers

---

## Overview

ONEMANSHYO Dojo exports WebM files (VP8 codec). While WebM works great for many platforms, FFmpeg provides powerful post-processing for:
- Converting to H.264 (better compatibility)
- Adjusting bitrate and quality
- Trimming and concatenating clips
- Adding overlays and effects
- Preparing for live streaming

---

## Installing FFmpeg

### Windows
```bash
# Using Chocolatey
choco install ffmpeg

# Or download from: https://ffmpeg.org/download.html
```

### macOS
```bash
# Using Homebrew
brew install ffmpeg
```

### Linux
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install ffmpeg

# Fedora
sudo dnf install ffmpeg

# Arch
sudo pacman -S ffmpeg
```

### Verify Installation
```bash
ffmpeg -version
```

---

## Basic Conversions

### WebM to MP4 (H.264)

**Standard quality:**
```bash
ffmpeg -i OMS_myshader_1080p_30fps_32s_128beats_L.webm \
  -c:v libx264 -crf 23 \
  -c:a aac -b:a 128k \
  output.mp4
```

**High quality:**
```bash
ffmpeg -i input.webm \
  -c:v libx264 -preset slow -crf 18 \
  -c:a aac -b:a 192k \
  output_hq.mp4
```

**Fast encode (lower quality):**
```bash
ffmpeg -i input.webm \
  -c:v libx264 -preset veryfast -crf 28 \
  -c:a aac -b:a 96k \
  output_fast.mp4
```

**Parameters explained:**
- `-c:v libx264`: H.264 video codec
- `-crf 23`: Constant quality (lower = better, 18-28 typical)
- `-preset slow`: Encoding speed (slower = better compression)
- `-c:a aac`: AAC audio codec
- `-b:a 128k`: Audio bitrate

---

## Social Media Formats

### Instagram Feed (Square 1:1)

**From Square WebM:**
```bash
ffmpeg -i OMS_myshader_1080p_30fps_32s_128beats_S.webm \
  -c:v libx264 -preset medium -crf 23 \
  -c:a aac -b:a 128k \
  -vf scale=1080:1080 \
  instagram_feed.mp4
```

### Instagram Reels / TikTok (Portrait 9:16)

**From Portrait WebM:**
```bash
ffmpeg -i OMS_myshader_1080p_30fps_32s_128beats_P.webm \
  -c:v libx264 -preset medium -crf 23 \
  -c:a aac -b:a 128k \
  instagram_reels.mp4
```

### YouTube Standard (Landscape 16:9)

**Direct upload (keep as WebM):**
- YouTube accepts WebM directly
- No conversion needed

**Or convert to H.264:**
```bash
ffmpeg -i OMS_myshader_1080p_30fps_300s_720beats_L.webm \
  -c:v libx264 -preset slow -crf 18 \
  -c:a aac -b:a 192k \
  youtube_video.mp4
```

### YouTube Shorts (Portrait 9:16)

**Same as Instagram Reels:**
```bash
ffmpeg -i OMS_myshader_1080p_30fps_32s_128beats_P.webm \
  -c:v libx264 -preset medium -crf 23 \
  -c:a aac -b:a 128k \
  youtube_shorts.mp4
```

---

## Live Streaming

### Prepare for OBS

**Convert to streaming-friendly format:**
```bash
ffmpeg -i OMS_myshader_1080p_30fps_240s_480beats_L.webm \
  -c:v libx264 -preset veryfast -tune zerolatency \
  -b:v 6000k -maxrate 6000k -bufsize 12000k \
  -c:a aac -b:a 128k -ar 44100 \
  streaming_loop.mp4
```

**Parameters:**
- `-preset veryfast`: Fast encoding
- `-tune zerolatency`: Low latency
- `-b:v 6000k`: 6 Mbps bitrate (YouTube recommended)
- `-maxrate`/`-bufsize`: Prevent spikes

### Create Looping Content

**Seamless loop (requires seamless source):**
```bash
ffmpeg -stream_loop -1 -i input.webm \
  -c:v libx264 -preset veryfast \
  -t 01:00:00 \
  loop_1hour.mp4
```

---

## Editing Operations

### Trim Video

**Trim from start:**
```bash
# Skip first 10 seconds
ffmpeg -i input.webm -ss 00:00:10 -c copy output.webm
```

**Extract segment:**
```bash
# From 10s to 40s (30 second clip)
ffmpeg -i input.webm -ss 00:00:10 -t 00:00:30 -c copy output.webm
```

**Accurate trim (re-encode):**
```bash
ffmpeg -i input.webm -ss 00:00:10 -t 00:00:30 \
  -c:v libx264 -crf 23 -c:a aac \
  output.mp4
```

### Concatenate Videos

**Create file list:**
```bash
# list.txt
file 'clip1.webm'
file 'clip2.webm'
file 'clip3.webm'
```

**Concatenate:**
```bash
ffmpeg -f concat -safe 0 -i list.txt \
  -c copy \
  concatenated.webm
```

**Or with re-encoding:**
```bash
ffmpeg -f concat -safe 0 -i list.txt \
  -c:v libx264 -crf 23 -c:a aac \
  concatenated.mp4
```

### Change Speed

**Speed up (2x faster):**
```bash
ffmpeg -i input.webm \
  -filter:v "setpts=0.5*PTS" \
  -filter:a "atempo=2.0" \
  output_fast.webm
```

**Slow down (0.5x speed):**
```bash
ffmpeg -i input.webm \
  -filter:v "setpts=2.0*PTS" \
  -filter:a "atempo=0.5" \
  output_slow.webm
```

---

## Visual Effects

### Add Text Overlay

**Simple text:**
```bash
ffmpeg -i input.webm \
  -vf "drawtext=text='My Shader':x=10:y=10:fontsize=48:fontcolor=white:box=1:boxcolor=black@0.5" \
  output_text.webm
```

**Centered text:**
```bash
ffmpeg -i input.webm \
  -vf "drawtext=text='DOJO':x=(w-text_w)/2:y=(h-text_h)/2:fontsize=72:fontcolor=white" \
  output_centered.webm
```

### Add Logo/Watermark

**Top-right corner:**
```bash
ffmpeg -i input.webm -i logo.png \
  -filter_complex "[0:v][1:v]overlay=W-w-10:10" \
  output_logo.webm
```

**Bottom-right corner:**
```bash
ffmpeg -i input.webm -i logo.png \
  -filter_complex "[0:v][1:v]overlay=W-w-10:H-h-10" \
  output_logo.webm
```

### Fade In/Out

**Fade in (2 seconds):**
```bash
ffmpeg -i input.webm \
  -vf "fade=in:0:60" \
  output_fadein.webm
```

**Fade out (2 seconds at end):**
```bash
ffmpeg -i input.webm \
  -vf "fade=out:st=30:d=2" \
  output_fadeout.webm
```

**Both:**
```bash
ffmpeg -i input.webm \
  -vf "fade=in:0:60,fade=out:st=30:d=60" \
  output_fades.webm
```

---

## Quality Settings

### CRF Values (Constant Rate Factor)

| CRF | Quality | Use Case | File Size |
|-----|---------|----------|-----------|
| 18 | Very High | Archival, high-end | Large |
| 20 | High | YouTube, professional | Medium-Large |
| 23 | Good | YouTube, general use | Medium |
| 26 | Acceptable | Social media, web | Small |
| 28 | Lower | Quick tests, previews | Very Small |

**Recommendation:** CRF 23 for most use cases

### Presets (Encoding Speed)

| Preset | Speed | Quality | Use Case |
|--------|-------|---------|----------|
| ultrafast | Fastest | Lowest | Testing only |
| veryfast | Very Fast | Lower | Live streaming |
| fast | Fast | Good | Quick encodes |
| medium | Medium | Good | General use |
| slow | Slow | Better | Production |
| veryslow | Very Slow | Best | Archival |

**Recommendation:** `medium` for most, `slow` for high quality

---

## Batch Processing

### Convert all WebM files in folder

**Windows (PowerShell):**
```powershell
Get-ChildItem *.webm | ForEach-Object {
    ffmpeg -i $_.Name -c:v libx264 -crf 23 -c:a aac "$($_.BaseName).mp4"
}
```

**macOS/Linux (Bash):**
```bash
for file in *.webm; do
    ffmpeg -i "$file" -c:v libx264 -crf 23 -c:a aac "${file%.webm}.mp4"
done
```

### Process with consistent settings

**Create bash script (process.sh):**
```bash
#!/bin/bash
for file in *.webm; do
    ffmpeg -i "$file" \
      -c:v libx264 -preset medium -crf 23 \
      -c:a aac -b:a 128k \
      "processed/${file%.webm}.mp4"
done
```

---

## Advanced Techniques

### Two-Pass Encoding (Best Quality)

**Pass 1:**
```bash
ffmpeg -i input.webm \
  -c:v libx264 -preset slow -b:v 5000k \
  -pass 1 -f mp4 /dev/null
```

**Pass 2:**
```bash
ffmpeg -i input.webm \
  -c:v libx264 -preset slow -b:v 5000k \
  -pass 2 \
  -c:a aac -b:a 192k \
  output.mp4
```

### Extract Audio Only

**To MP3:**
```bash
ffmpeg -i input.webm -vn -c:a libmp3lame -b:a 192k audio.mp3
```

**To WAV:**
```bash
ffmpeg -i input.webm -vn -c:a pcm_s16le audio.wav
```

### Replace Audio

**Swap audio track:**
```bash
ffmpeg -i video.webm -i new_audio.mp3 \
  -c:v copy -c:a aac \
  -map 0:v:0 -map 1:a:0 \
  output.webm
```

---

## Troubleshooting

### "Unknown encoder" error

**Problem:** libx264 not found

**Solution:**
```bash
# Reinstall FFmpeg with libx264 support
# or use native encoder:
ffmpeg -i input.webm -c:v h264 output.mp4
```

### Audio/Video out of sync

**Problem:** A/V desync after processing

**Solutions:**
1. Use `-c copy` to avoid re-encoding
2. Add `-vsync 1` flag
3. Use `-async 1` for audio sync

### File size too large

**Solutions:**
1. Increase CRF value (23 â†’ 26)
2. Lower bitrate (`-b:v 3000k`)
3. Use faster preset
4. Reduce resolution

### Slow encoding

**Solutions:**
1. Use faster preset (`-preset veryfast`)
2. Lower resolution if acceptable
3. Use hardware encoding (see below)

---

## Hardware Acceleration

### NVIDIA (NVENC)

```bash
ffmpeg -i input.webm \
  -c:v h264_nvenc -preset fast -crf 23 \
  -c:a aac \
  output.mp4
```

### AMD (AMF)

```bash
ffmpeg -i input.webm \
  -c:v h264_amf -quality balanced -rc cqp -qp 23 \
  -c:a aac \
  output.mp4
```

### Intel (QuickSync)

```bash
ffmpeg -i input.webm \
  -c:v h264_qsv -preset medium -global_quality 23 \
  -c:a aac \
  output.mp4
```

---

## Common Workflows

### For Instagram Feed

```bash
# 1. Convert to H.264
ffmpeg -i OMS_myshader_1080p_30fps_32s_128beats_S.webm \
  -c:v libx264 -preset medium -crf 23 \
  -c:a aac -b:a 128k \
  -vf scale=1080:1080 \
  instagram.mp4

# 2. Upload to Instagram
```

### For YouTube Live Streaming

```bash
# 1. Convert to streaming format
ffmpeg -i OMS_myshader_1080p_30fps_240s_480beats_L.webm \
  -c:v libx264 -preset veryfast -tune zerolatency \
  -b:v 6000k -maxrate 6000k -bufsize 12000k \
  -c:a aac -b:a 128k \
  stream_loop.mp4

# 2. Add to OBS as Media Source
# 3. Set to loop
# 4. Stream to YouTube
```

### For Content Archive

```bash
# High-quality archival version
ffmpeg -i OMS_myshader_2160p_60fps_300s_720beats_L.webm \
  -c:v libx264 -preset veryslow -crf 18 \
  -c:a aac -b:a 256k \
  archive_master.mp4
```

---

## Resources

**FFmpeg Documentation:**
- Official docs: https://ffmpeg.org/documentation.html
- Wiki: https://trac.ffmpeg.org/wiki

**Useful Links:**
- H.264 encoding guide: https://trac.ffmpeg.org/wiki/Encode/H.264
- Filters documentation: https://ffmpeg.org/ffmpeg-filters.html

---

**Last Updated:** November 2025  
**Version:** V2_2  
**For:** Post-processing ONEMANSHYO Dojo outputs

---

*For complete workflows, see UserGuide_All.md*  
*For production tips, see E_Production_Release_Guide.md*
