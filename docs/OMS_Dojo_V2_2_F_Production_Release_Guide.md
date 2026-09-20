# ONEMANSHYO Dojo V2_2 - Production Release Guide

**Version:** V2_2  
**Target Audience:** Content creators, streamers, producers  
**Purpose:** Production workflows and post-processing guide

---

## Table of Contents

- [Production Workflow Overview](#production-workflow-overview)
- [Audio Preparation](#audio-preparation)
- [Loop Finding & Timing](#loop-finding--timing)
- [Shader Development](#shader-development)
- [Video Rendering](#video-rendering)
- [FFmpeg Post-Processing](#ffmpeg-post-processing)
- [YouTube Live Streaming](#youtube-live-streaming)
- [Social Media Export](#social-media-export)
- [Quality Control](#quality-control)
- [Troubleshooting](#troubleshooting)

---

## Production Workflow Overview

### Standard Pipeline

```
1. AUDIO PREPARATION
   └─ Export from DAW (Ableton, FL Studio, etc.)
   
2. DOJO
   └─ Upload audio → BPM auto-detects
   └─ Find loop section (arrow keys)
   └─ Fine-tune timing (Nudge field)
   └─ Design shader (Code tab)
   └─ Test preview (HD canvas)
   └─ Render video (WebM output)
   
3. FFMPEG POST-PROCESSING
   └─ Convert WebM → MP4 (H.264)
   └─ Add metadata
   └─ Optimize for platform
   
4. DEPLOYMENT
   └─ YouTube live streaming
   └─ Social media posts
   └─ Archive/backup
```

### Time Estimate

**Typical 16-bar loop (64 beats):**
- Audio export: 30 seconds
- Loop finding: 2-5 minutes
- Shader design: 10-30 minutes
- Video render: 1-2 minutes
- FFmpeg processing: 30-60 seconds
- **Total:** 15-40 minutes

---

## Audio Preparation

### Export from DAW

**Recommended Settings:**
- Format: WAV or MP3
- Sample Rate: 44.1kHz or 48kHz
- Bit Depth: 16-bit or 24-bit
- Channels: Stereo

**Ableton Live Export:**
1. Set loop region (full song or section)
2. File → Export Audio/Video
3. Rendered Track → Master
4. Format: WAV (recommended) or MP3
5. Sample Rate: 48kHz
6. Bit Depth: 24-bit
7. Export

**Pro Tip:** Export full song, find loop section in Dojo. Allows flexibility for different loop lengths.

### Audio Levels

**Target Levels:**
- Peak: -3dB to -1dB (headroom for compression)
- RMS: -12dB to -8dB (good average loudness)
- True Peak: Below -1dBTP (prevents clipping)

**Why:** Properly leveled audio gives better:
- FFT frequency analysis
- Bass/mid/treble separation
- Audio reactivity
- Video encoding quality

---

## Loop Finding & Timing

### Step 1: Load Audio

1. Open OMS_Dojo_V2_2.html
2. Click "Upload Audio" or drag file onto waveform
3. **BPM auto-detects** → Check if correct (override if needed)
4. Audio jumps to Loop In beat (default: 64)

### Step 2: Find Loop Section

**Using Arrow Keys:**
- Focus Loop In or Loop Out field
- **Right Arrow:** Shift window +4 beats (1 bar) →
- **Left Arrow:** Shift window -4 beats (1 bar) ←
- **Up Arrow:** Shift window +1 beat →
- **Down Arrow:** Shift window -1 beat ←

**Both Loop In/Out move together** - maintains loop length!

**Finding Good Loops:**
- Start at beat 64 (past intro)
- Length: 64 beats (16 bars) typical
- Look for musical phrases (4, 8, 16 bars)
- Avoid mid-phrase or awkward cuts

### Step 3: Fine-Tune with Nudge

1. Found rough loop position? Good!
2. Click **Nudge field**
3. Use arrow keys for fine-tuning:
   - **Up:** +1ms
   - **Down:** -1ms
   - **Right:** +4ms
   - **Left:** -4ms
4. Or type value directly (-50ms typical range)

**When to Nudge:**
- Loop sounds perfect but starts **slightly early** → Nudge -30ms
- Loop sounds perfect but starts **slightly late** → Nudge +30ms
- Video render audio/visual **slightly off-sync** → Nudge ±10-50ms

### Pro Tips

**Waveform Visual Analysis:**
- Look for kick drum peaks (bass)
- Look for snare hits (mid)
- Look for hi-hat patterns (treble)
- Align Loop In to downbeat (strong beat)

**Testing Loops:**
- Enable Loop button (default ON)
- Press Play (Spacebar)
- Listen for smooth transition
- Adjust Loop In/Out or Nudge as needed

---

## Shader Development

### Workflow

1. **Switch to Code Tab**
2. **Edit WGSL fragment shader**
3. **Press [Compile]** → Preview updates
4. **Adjust modulators** (Controls tab)
5. **Repeat** until satisfied

### Using Auto-Detected BPM

**Shader Tempo vs Audio BPM:**
- **BPM field** (audio controls) = Audio tempo (for Loop In/OUT positioning)
- **Shader Tempo** (controls tab) = Visual timing (can differ!)

**Example:**
- Audio BPM: 127.5 (auto-detected)
- Shader Tempo: 127.5 (match for sync) OR 63.75 (half-time) OR 255 (double-time)

**Pro Tip:** Match Shader Tempo to Audio BPM initially, then experiment with multiples for different visual feels.

### Testing Audio Reactivity

**Modulator Gains:**
- **Default:** 0.1 (subtle reactivity)
- **Visible:** 0.5-1.0 (moderate)
- **Extreme:** 2.0-5.0 (intense)

**Frequency Bands:**
- **Mod A × Bass:** Kicks, bass guitars, low synths
- **Mod B × Mid:** Vocals, guitars, instruments
- **Mod C × Treble:** Hi-hats, cymbals, high notes

### Shader Management

**Archive Your Work:**
1. Switch to Code tab
2. Click **[Copy Code]** → Clipboard
3. Click **[Export .txt]** → Download `OMS_{name}_V2.2.0.txt`

**Import Later:**
- Drag .txt file onto code editor
- Or paste from clipboard

---

## Video Rendering

### Pre-Render Checklist

**Controls Tab:**
- [ ] **Orientation:** Landscape (YouTube) / Portrait (TikTok) / Square (Instagram)
- [ ] **Resolution:** 1080p (standard) / 1440p / 2160p (4K)
- [ ] **FPS:** 30 (standard) / 60 (smooth) / 24 (film)
- [ ] **Filename:** Descriptive name (e.g., `TronTunnel_Pink-Blue_127bpm`)

**Audio Controls:**
- [ ] **Loop In/Out:** Correct range
- [ ] **Nudge:** Fine-tuned if needed
- [ ] **BPM:** Verified

**Shader:**
- [ ] **Compiled:** Press [C] if changed
- [ ] **Preview looks good:** Check HD canvas

### Render Process

1. **Switch to Controls Tab**
2. **Set all parameters** (orientation, resolution, FPS, filename)
3. **Press [R] Render Video**
4. **Wait** (~1-2 minutes for 64-beat loop @ 1080p30)
5. **Video downloads** as WebM

**Render Time Estimates:**
- 1080p @ 30fps, 32s: ~1 minute
- 1080p @ 60fps, 32s: ~2 minutes
- 2160p @ 30fps, 32s: ~3-4 minutes
- 2160p @ 60fps, 32s: ~6-8 minutes

### Filename Output

```
OMS_{yourname}_{res}p_{fps}fps_{dur}s_{beats}beats_{L|P|S}.webm

Example:
  OMS_TronTunnel_Pink-Blue_1080p_30fps_32s_128beats_L.webm
  
Where:
  L = Landscape (16:9)
  P = Portrait (9:16)
  S = Square (1:1)
```

---

## FFmpeg Post-Processing

### Why FFmpeg?

**WebM → MP4 Conversion:**
- YouTube live streaming requires MP4 (H.264)
- Social media platforms prefer MP4
- Better compatibility across devices

### Basic Conversion

```bash
# WebM to MP4 (H.264, AAC audio)
ffmpeg -i OMS_myshader_1080p_30fps_32s_128beats_L.webm \
       -c:v libx264 -preset medium -crf 18 \
       -c:a aac -b:a 192k \
       output.mp4
```

**Parameters Explained:**
- `-c:v libx264` = H.264 video codec
- `-preset medium` = Encoding speed/quality balance
- `-crf 18` = Quality (18 = visually lossless, 23 = good, lower = better)
- `-c:a aac` = AAC audio codec
- `-b:a 192k` = Audio bitrate (192kbps = high quality)

### YouTube Optimized

```bash
# YouTube recommended settings
ffmpeg -i input.webm \
       -c:v libx264 -preset slow -crf 18 \
       -pix_fmt yuv420p \
       -c:a aac -b:a 192k \
       -movflags +faststart \
       output.mp4
```

**Additional Parameters:**
- `-preset slow` = Better compression (slower encode)
- `-pix_fmt yuv420p` = Compatible pixel format
- `-movflags +faststart` = Web-optimized (metadata at start)

### Social Media Specs

**Instagram Feed (Square):**
```bash
ffmpeg -i input.webm \
       -c:v libx264 -preset medium -crf 20 \
       -vf "scale=1080:1080" \
       -c:a aac -b:a 128k \
       -t 60 \
       output.mp4
```

**Instagram Reels / TikTok (Portrait):**
```bash
ffmpeg -i input.webm \
       -c:v libx264 -preset medium -crf 20 \
       -vf "scale=1080:1920" \
       -c:a aac -b:a 128k \
       output.mp4
```

**YouTube Shorts (Portrait):**
```bash
ffmpeg -i input.webm \
       -c:v libx264 -preset medium -crf 18 \
       -vf "scale=1080:1920" \
       -c:a aac -b:a 192k \
       -t 60 \
       output.mp4
```

### Metadata Addition

```bash
# Add metadata
ffmpeg -i input.webm \
       -c:v libx264 -preset medium -crf 18 \
       -c:a aac -b:a 192k \
       -metadata title="My Shader Title" \
       -metadata artist="OneManShYo" \
       -metadata comment="Created with ONEMANSHYO Dojo V2_2" \
       output.mp4
```

---

## YouTube Live Streaming

### Setup

1. **YouTube Studio** → **Go Live**
2. **Create Stream** → Select "Streaming Software"
3. Copy **Stream Key**

### OBS Studio Configuration

**Video Settings:**
- Base Resolution: 1920x1080
- Output Resolution: 1920x1080
- FPS: 30 or 60

**Encoder Settings:**
- Encoder: NVENC (GPU) or x264 (CPU)
- Rate Control: CBR
- Bitrate: 6000 Kbps (1080p30) or 9000 Kbps (1080p60)
- Keyframe Interval: 2 seconds

**Audio Settings:**
- Sample Rate: 48kHz
- Bitrate: 160 Kbps

### Looping Video in OBS

**Method 1: Media Source (Simple)**
1. Add Source → Media Source
2. Select converted MP4 file
3. Check "Loop"
4. Check "Restart playback when source becomes active"

**Method 2: VLC Source (Advanced)**
1. Add Source → VLC Video Source
2. Add MP4 to playlist
3. Check "Loop Playlist"
4. Allows multiple videos in rotation

### Stream Quality Tips

**Pre-Stream Checklist:**
- [ ] Test stream (unlisted) before going live
- [ ] Check audio sync in stream output
- [ ] Verify no dropped frames (OBS stats)
- [ ] Monitor CPU/GPU usage

**Troubleshooting:**
- **Dropped frames** → Lower bitrate or resolution
- **Audio desync** → Use Nudge field in Dojo, re-render
- **Choppy video** → Increase keyframe interval or lower FPS

---

## Social Media Export

### Platform Requirements

**Instagram Feed:**
- Aspect: 1:1 (Square)
- Resolution: 1080x1080
- Duration: Max 60 seconds
- Format: MP4 (H.264)

**Instagram Reels:**
- Aspect: 9:16 (Portrait)
- Resolution: 1080x1920
- Duration: Max 90 seconds
- Format: MP4 (H.264)

**TikTok:**
- Aspect: 9:16 (Portrait)
- Resolution: 1080x1920
- Duration: Max 10 minutes
- Format: MP4 (H.264)

**YouTube Shorts:**
- Aspect: 9:16 (Portrait)
- Resolution: 1080x1920
- Duration: Max 60 seconds
- Format: MP4 (H.264)

**YouTube Videos:**
- Aspect: 16:9 (Landscape)
- Resolution: 1920x1080 or higher
- Duration: Unlimited
- Format: MP4 (H.264)

### Batch Processing

**Convert Multiple Files:**
```bash
# Batch convert all WebM files
for file in *.webm; do
    ffmpeg -i "$file" \
           -c:v libx264 -preset medium -crf 18 \
           -c:a aac -b:a 192k \
           "${file%.webm}.mp4"
done
```

---

## Quality Control

### Visual Inspection

**Check for:**
- [ ] Audio/visual sync (use Nudge if off)
- [ ] No dropped frames
- [ ] Smooth animation (60fps if needed)
- [ ] No artifacts or compression issues
- [ ] Proper loop point (seamless transition)

### Audio Quality

**Check for:**
- [ ] No clipping or distortion
- [ ] Balanced levels (not too loud/quiet)
- [ ] Proper stereo field
- [ ] Clean loop transition

### Export Verification

**Before Publishing:**
1. Watch full video start to finish
2. Check first/last frames (loop point)
3. Verify metadata (title, artist)
4. Test on target platform (private/unlisted upload)

---

## Troubleshooting

### Common Issues

**Problem:** BPM detection incorrect  
**Solution:** Type correct BPM manually in BPM field

**Problem:** Loop doesn't sound smooth  
**Solution:** Adjust Loop In/Out by 1 beat (arrow keys) or use Nudge field

**Problem:** Video render audio/visual desync  
**Solution:** Use Nudge field (±10-50ms typical), re-render

**Problem:** Shader preview looks different than render  
**Solution:** Check resolution/orientation settings match intended output

**Problem:** FFmpeg conversion fails  
**Solution:** Check FFmpeg installed, verify input file exists, check command syntax

**Problem:** YouTube stream audio desync  
**Solution:** Use Nudge in Dojo, re-export, ensure OBS audio delay is 0ms

**Problem:** Social media rejects video  
**Solution:** Verify format (MP4), resolution, and duration requirements for platform

### Performance Issues

**Problem:** Render takes too long  
**Solution:**
- Lower resolution (2160p → 1080p)
- Lower FPS (60 → 30)
- Simplify shader code

**Problem:** Browser crashes during render  
**Solution:**
- Close other tabs
- Increase browser memory limit
- Render shorter sections

---

## Recommended Workflow

### For YouTube Live Streaming

```
1. Prepare Audio (Ableton) → Export WAV
2. Dojo → Upload → BPM detects → Find loop → Design shader
3. Render 1080p30 Landscape WebM
4. FFmpeg → Convert to MP4 (H.264, YouTube optimized)
5. OBS Studio → Loop video → Stream to YouTube
```

### For Social Media Posts

```
1. Prepare Audio → Export MP3/WAV
2. Dojo → Upload → Find loop → Design shader
3. Render appropriate orientation:
   - Instagram Feed: 1080p Square
   - Reels/TikTok: 1080p Portrait
   - YouTube Shorts: 1080p Portrait
4. FFmpeg → Convert to MP4 (platform optimized)
5. Upload directly to platform
```

### For Content Archive

```
1. Create multiple versions:
   - 2160p60 Landscape (YouTube videos)
   - 1080p30 Portrait (social media)
   - 1080p30 Square (Instagram feed)
2. Convert all to MP4
3. Add metadata
4. Organize by project/date
5. Backup to cloud storage
```

---

**Last Updated:** November 2025  
**Version:** V2_2  
**Status:** Production Release

---

*For technical details, see C_Developer_Documentation.md*  
*For user guide, see D_Release_Summary.md*  
*For complete changelog, see B_Complete_Changelog.md*
