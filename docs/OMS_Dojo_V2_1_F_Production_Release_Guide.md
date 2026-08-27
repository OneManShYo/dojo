# ONEMANSHYO Dojo V2_1 - Production Release Guide

**Version:** V2_1  
**Target Audience:** Content creators, streamers, video producers  
**Purpose:** Production workflow from creation to deployment

---

## Table of Contents

- [Production Workflow](#production-workflow)
- [Quality Settings](#quality-settings)
- [Social Media Specs](#social-media-specs)
- [YouTube Live Streaming](#youtube-live-streaming)
- [FFmpeg Post-Processing](#ffmpeg-post-processing)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

---

## Production Workflow

### Standard Workflow

```
1. PREPARE AUDIO
   ↓
2. CREATE/ADJUST SHADER
   ↓
3. CONFIGURE PARAMETERS
   ↓
4. PREVIEW & TEST
   ↓
5. RENDER VIDEO
   ↓
6. POST-PROCESS (FFmpeg)
   ↓
7. UPLOAD/STREAM
```

### Detailed Steps

**1. Prepare Audio**
- Export audio from your DAW (Ableton, FL Studio, etc.)
- Recommended: WAV or high-quality MP3
- Ensure proper levels (not clipping)
- Check duration matches your needs

**2. Create/Adjust Shader**
- Switch to "Code" tab
- Write or modify WGSL fragment shader
- Use available uniforms (time, modA/B/C, resolution)
- Press [C] to compile and preview

**3. Configure Parameters (Controls Tab)**
- **Orientation:** Landscape, Portrait, or Square
- **Shader Tempo:** Match audio BPM or set visual tempo
- **Modulators:** Adjust subdivisions and waveforms
- **Gains:** Set audio reactivity strength
- **IN/OUT Beats:** Define render range

**4. Preview & Test**
- Watch full loop in real-time
- Check for glitches or errors
- Verify audio sync
- Adjust parameters as needed
- Use transport controls (▶ ⏹) for quick testing

**5. Render Video**
- Set **Resolution:** 720p (fast), 1080p (standard), 1440p/2160p (high quality)
- Set **FPS:** 30 (standard), 60 (smooth motion)
- Enter **Filename:** Descriptive name (auto-generates full filename)
- Press [R] to render
- **Critical:** Don't switch tabs or minimize browser during render!

**6. Post-Process (FFmpeg)**
- See `UserGuide_FFmpeg_Video_Guide.md` for details
- Convert to H.264 for compatibility
- Adjust bitrate if needed
- Add branding/overlays (optional)

**7. Upload/Stream**
- Upload to YouTube, Instagram, TikTok, etc.
- Use for live streaming backgrounds
- Share on social media

---

## Quality Settings

### Resolution Guide

| Resolution | Use Case | Render Time | File Size |
|------------|----------|-------------|-----------|
| 720p | Quick tests, previews | Fast (~2-5 min) | Small (~50MB) |
| 1080p | Standard quality, streaming | Medium (~5-10 min) | Medium (~150MB) |
| 1440p | High quality, YouTube | Slow (~10-20 min) | Large (~300MB) |
| 2160p | Maximum quality, archive | Very slow (~20-40 min) | Very large (~600MB) |

*Note: Times are for ~60 second videos on modern hardware*

### FPS Guide

| FPS | Use Case | Motion Quality | File Size |
|-----|----------|----------------|-----------|
| 24 | Cinematic look | Good | Smallest |
| 30 | Standard video | Good | Medium |
| 60 | Smooth motion, gaming | Excellent | Largest |

**Recommendation:** 30 FPS for most use cases (good balance of quality and size)

### Bitrate

**WebM Export:** 8 Mbps (fixed)  
**FFmpeg Re-encode:** Adjustable (see FFmpeg guide)

---

## Social Media Specs

### Instagram Feed (Square Posts)

**Settings:**
- **Orientation:** Square (1:1)
- **Resolution:** 1080p (1080x1080)
- **FPS:** 30
- **Duration:** Up to 60 seconds
- **Format:** Convert to H.264 with FFmpeg

**Workflow:**
```bash
ffmpeg -i OMS_myshader_1080p_30fps_32s_128beats_S.webm \
  -c:v libx264 -preset medium -crf 23 \
  -c:a aac -b:a 128k \
  -vf scale=1080:1080 \
  instagram_feed.mp4
```

### Instagram Reels / TikTok

**Settings:**
- **Orientation:** Portrait (9:16)
- **Resolution:** 1080p (607x1080)
- **FPS:** 30
- **Duration:** 15-60 seconds
- **Format:** Convert to H.264 with FFmpeg

**Workflow:**
```bash
ffmpeg -i OMS_myshader_1080p_30fps_32s_128beats_P.webm \
  -c:v libx264 -preset medium -crf 23 \
  -c:a aac -b:a 128k \
  reels_tiktok.mp4
```

### YouTube Standard Videos

**Settings:**
- **Orientation:** Landscape (16:9)
- **Resolution:** 1080p or 1440p recommended
- **FPS:** 30 or 60
- **Duration:** Any length
- **Format:** H.264 or keep as WebM

**Direct Upload:**
- YouTube accepts WebM directly
- No FFmpeg conversion required (unless adding effects)

**For Best Quality:**
```bash
ffmpeg -i OMS_myshader_1080p_30fps_300s_720beats_L.webm \
  -c:v libx264 -preset slow -crf 18 \
  -c:a aac -b:a 192k \
  youtube_video.mp4
```

### YouTube Shorts

**Settings:**
- **Orientation:** Portrait (9:16)
- **Resolution:** 1080p (607x1080)
- **FPS:** 30
- **Duration:** Under 60 seconds
- **Format:** H.264

**Workflow:** Same as Instagram Reels

---

## YouTube Live Streaming

### Overview

ONEMANSHYO Dojo is designed for creating background visuals for YouTube live streaming. The typical workflow:

1. Create audio-reactive visuals in Dojo
2. Render multiple clips
3. Process with FFmpeg for streaming compatibility
4. Use in OBS or streaming software
5. Stream to YouTube

### Render Settings for Streaming

**Recommended:**
- **Resolution:** 1080p (streaming standard)
- **FPS:** 30 (streaming standard)
- **Orientation:** Landscape (16:9)
- **Bitrate:** 8 Mbps (WebM export)

### FFmpeg Processing for Streaming

**Convert to streaming-friendly format:**
```bash
ffmpeg -i OMS_myshader_1080p_30fps_240s_480beats_L.webm \
  -c:v libx264 -preset veryfast -tune zerolatency \
  -b:v 6000k -maxrate 6000k -bufsize 12000k \
  -c:a aac -b:a 128k -ar 44100 \
  -f flv \
  streaming_output.flv
```

**Parameters Explained:**
- `-preset veryfast`: Optimized for real-time encoding
- `-tune zerolatency`: Reduces encoding delay
- `-b:v 6000k`: 6 Mbps video bitrate (YouTube recommended)
- `-maxrate`/`-bufsize`: Prevent bitrate spikes
- `-f flv`: FLV format for streaming software

### OBS Integration

1. Add video as **Media Source** in OBS
2. Set to loop if needed
3. Combine with other sources (camera, audio, overlays)
4. Stream to YouTube at 1080p30

---

## FFmpeg Post-Processing

See `UserGuide_FFmpeg_Video_Guide.md` for complete FFmpeg workflows.

### Common Tasks

**Basic Conversion (WebM → MP4):**
```bash
ffmpeg -i input.webm -c:v libx264 -crf 23 -c:a aac output.mp4
```

**Trim Video:**
```bash
ffmpeg -i input.webm -ss 00:00:10 -t 00:00:30 -c copy output.webm
```

**Add Text Overlay:**
```bash
ffmpeg -i input.webm \
  -vf "drawtext=text='My Shader':x=10:y=10:fontsize=24:fontcolor=white" \
  output.webm
```

**Concatenate Multiple Clips:**
```bash
# Create file list
echo "file 'clip1.webm'" > list.txt
echo "file 'clip2.webm'" >> list.txt

# Concatenate
ffmpeg -f concat -safe 0 -i list.txt -c copy output.webm
```

---

## Best Practices

### Before Rendering

**✓ Checklist:**
- [ ] Audio uploaded and waveform visible
- [ ] Shader compiled without errors
- [ ] Parameters set for desired visual style
- [ ] IN/OUT beats define correct range
- [ ] Preview looks good (watch full loop)
- [ ] Filename is descriptive
- [ ] Resolution/FPS appropriate for use case

### During Rendering

**✓ Do:**
- Let the render complete without interruption
- Keep browser tab active and visible
- Ensure computer doesn't go to sleep
- Close unnecessary programs to free resources

**✗ Don't:**
- Switch browser tabs (may pause render)
- Minimize browser window
- Start other heavy tasks
- Put computer to sleep

### After Rendering

**✓ Verify:**
- Video downloaded successfully
- Audio/video are in sync
- Duration matches expectations
- Visual quality is acceptable
- No glitches or errors
- Filename is correct

### Quality vs. Speed Trade-offs

**Fast Rendering:**
- 720p resolution
- 30 FPS
- Shorter duration
- Simpler shaders

**High Quality:**
- 1440p or 2160p
- 60 FPS
- Complex shaders
- Post-process with FFmpeg

---

## Troubleshooting

### Video Won't Render

**Problem:** Clicking [R] does nothing

**Solutions:**
1. Ensure audio is loaded (if using audio)
2. Check IN/OUT beats are valid (OUT > IN)
3. Verify shader compiled without errors
4. Check browser console for errors (F12)
5. Refresh page and try again

### Audio/Video Out of Sync

**Problem:** Rendered video has audio/video desync

**Solutions:**
1. Don't switch tabs during render
2. Use fixed FPS (not variable)
3. Ensure audio started at IN beat (automatic)
4. Try shorter render duration
5. Close other programs during render

### Poor Video Quality

**Problem:** Exported video looks pixelated or low quality

**Solutions:**
1. Increase resolution (720p → 1080p → 1440p)
2. Re-encode with FFmpeg at higher bitrate
3. Use CRF 18-23 for H.264 encoding
4. Ensure source shader is high quality
5. Check if platform compresses uploads

### Large File Sizes

**Problem:** Video files are too large

**Solutions:**
1. Lower resolution if acceptable
2. Use 30 FPS instead of 60 FPS
3. Re-encode with FFmpeg at lower bitrate
4. Trim unnecessary frames
5. Use more efficient codec (H.264 vs WebM)

### Browser Crashes During Render

**Problem:** Browser crashes or freezes

**Solutions:**
1. Close other browser tabs
2. Close unnecessary programs
3. Lower resolution (try 720p)
4. Update GPU drivers
5. Try different browser (Chrome/Edge)
6. Ensure adequate RAM available

---

## Performance Tips

### Hardware Optimization

**GPU:**
- Update drivers to latest version
- Close GPU-intensive programs
- Ensure adequate cooling
- Monitor GPU temperature

**RAM:**
- Close unnecessary programs
- Clear browser cache
- Restart browser before big renders
- Upgrade RAM if frequently running out

**CPU:**
- Close background processes
- Disable unnecessary startup programs
- Monitor CPU temperature
- Consider render settings if overheating

### Software Optimization

**Browser:**
- Use Chrome or Edge (best WebGPU support)
- Clear cache and cookies
- Disable unnecessary extensions
- Update to latest version

**Shader Code:**
- Avoid expensive operations
- Use efficient algorithms
- Test on smaller canvas first
- Profile performance in real-time

---

## Production Checklist

### Pre-Production

- [ ] Audio track prepared (WAV/MP3)
- [ ] Shader concept/design decided
- [ ] Target platform known (YouTube, Instagram, etc.)
- [ ] Duration planned (beats/bars)
- [ ] BPM determined

### Production

- [ ] Audio uploaded successfully
- [ ] Shader written and tested
- [ ] Parameters tuned for desired look
- [ ] IN/OUT range set correctly
- [ ] Preview looks good
- [ ] Ready to render

### Render

- [ ] Resolution set appropriately
- [ ] FPS set (30 recommended)
- [ ] Filename entered
- [ ] Browser window active
- [ ] Computer won't sleep
- [ ] Render completed successfully

### Post-Production

- [ ] Video downloaded and verified
- [ ] Audio/video sync checked
- [ ] Quality acceptable
- [ ] FFmpeg processing (if needed)
- [ ] Final file ready for upload

### Deployment

- [ ] Video uploaded to platform
- [ ] Metadata added (title, description, tags)
- [ ] Thumbnail created
- [ ] Published/shared
- [ ] Tested on target platform

---

## Conclusion

ONEMANSHYO Dojo V2_1 is a powerful tool for creating audio-reactive visuals for professional use. With the new 1920x1080 canvas, you get a perfect preview of what your 1080p exports will look like - no surprises, better workflow.

**Key Points:**
- Use appropriate quality settings for your use case
- Don't switch tabs during rendering
- Post-process with FFmpeg for maximum compatibility
- Test on small clips before long renders
- Keep browser and GPU drivers updated

**For More Information:**
- Complete workflows: `UserGuide_All.md`
- FFmpeg details: `UserGuide_FFmpeg_Video_Guide.md`
- Technical docs: `C_Developer_Documentation.md`

---

**Last Updated:** November 2025  
**Version:** V2_1  
**Status:** Production Release
