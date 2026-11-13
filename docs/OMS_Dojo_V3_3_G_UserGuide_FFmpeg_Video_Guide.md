# OMS SHADER BAKER V3.3.0 - FFMPEG VIDEO EXPORT GUIDE

**Purpose:** Convert OMS renders to Apple ProRes 422 for Ableton/EboSuite  
**Version:** V3.3.0  
**Date:** November 2025

---

## OVERVIEW

OMS Shader Baker exports shaders as WebM video files. For professional use in Ableton Live + EboSuite, you need to convert these to Apple ProRes 422 format using FFmpeg.

**Workflow:**
1. OMS RENDER Tab → .webm file
2. FFmpeg → .mov file (ProRes 422)
3. Ableton + EboSuite → Load video track

---

## FFMPEG INSTALLATION

### macOS

**Using Homebrew:**
```bash
# Install Homebrew (if not installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install FFmpeg
brew install ffmpeg
```

**Verify Installation:**
```bash
ffmpeg -version
```

### Windows

**Using Chocolatey:**
```powershell
# Install Chocolatey (run as Administrator)
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# Install FFmpeg
choco install ffmpeg
```

**Manual Installation:**
1. Download from https://ffmpeg.org/download.html
2. Extract to C:\ffmpeg
3. Add C:\ffmpeg\bin to PATH

**Verify:**
```cmd
ffmpeg -version
```

### Linux (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install ffmpeg
```

---

## CONVERSION COMMANDS

### Basic Conversion (ProRes 422)

```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 2 -pix_fmt yuv422p10le output.mov
```

**Breakdown:**
- `-i input.webm` - Input WebM file
- `-c:v prores_ks` - ProRes encoder (Kostya's implementation)
- `-profile:v 2` - ProRes 422 (standard quality)
- `-pix_fmt yuv422p10le` - 10-bit YUV 4:2:2 color
- `output.mov` - Output MOV file

### Quality Profiles

**ProRes 422 Proxy (smallest file, faster):**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 0 -pix_fmt yuv422p10le output.mov
```

**ProRes 422 LT (good balance):**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 1 -pix_fmt yuv422p10le output.mov
```

**ProRes 422 (recommended):**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 2 -pix_fmt yuv422p10le output.mov
```

**ProRes 422 HQ (highest quality, large files):**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 3 -pix_fmt yuv422p10le output.mov
```

### Batch Conversion

**macOS/Linux:**
```bash
#!/bin/bash
for file in *.webm; do
    ffmpeg -i "$file" -c:v prores_ks -profile:v 2 -pix_fmt yuv422p10le "${file%.webm}.mov"
done
```

**Windows (PowerShell):**
```powershell
Get-ChildItem *.webm | ForEach-Object {
    ffmpeg -i $_.FullName -c:v prores_ks -profile:v 2 -pix_fmt yuv422p10le ($_.BaseName + ".mov")
}
```

---

## ABLETON LIVE + EBOSUITE INTEGRATION

### Setup

1. **Install EboSuite** (https://ebosuite.com)
   - EboSuite is required for video playback in Ableton
   - Trial version available
   - Purchase for full features

2. **Create Video Track:**
   - Ableton Live → Add MIDI Track
   - Add EboSuite → VideoPlayer device
   - Load ProRes .mov file

3. **Sync with Audio:**
   - Drag audio file to audio track
   - Video and audio auto-align
   - Warp both to project tempo

### Performance Tips

**File Organization:**
```
MyProject/
  ├── Audio/
  │   └── Track1.mp3
  └── Video/
      └── Track1_ProRes.mov
```

**Naming Convention:**
- Match video to audio filename
- Easy to identify pairs
- E.g., "MyTrack.mp3" + "MyTrack_ProRes.mov"

**Resolution:**
- 1920×1080 (Full HD) - Good balance
- 3840×2160 (4K) - High quality, large files
- 1280×720 (HD) - Lower quality, smaller files

**Frame Rate:**
- 60fps - Smooth, recommended
- 30fps - Standard, smaller files
- Match your export FPS to project needs

---

## WORKFLOW EXAMPLE

### Complete Workflow: Shader to Ableton

**Step 1: Create Shader (OMS)**
```
CONTROLS Tab → Write shader
AUDIO Tab → Load audio, set BPM
LIBRARY Tab → Save as MyShader.oms
```

**Step 2: Render Video (OMS)**
```
RENDER Tab → Settings:
  - Resolution: 1920×1080
  - FPS: 60
  - Start: 0.0
  - End: 120.0 (2 minutes)
Start Render → Download MyShader.webm
```

**Step 3: Convert to ProRes (FFmpeg)**
```bash
ffmpeg -i MyShader.webm -c:v prores_ks -profile:v 2 -pix_fmt yuv422p10le MyShader_ProRes.mov
```

**Step 4: Load in Ableton**
```
Ableton Live:
1. Create MIDI track
2. Add EboSuite VideoPlayer
3. Load MyShader_ProRes.mov
4. Audio track: Load matching audio
5. Sync and perform!
```

---

## TROUBLESHOOTING

### "ffmpeg: command not found"

**Solution:**
- FFmpeg not installed or not in PATH
- Reinstall FFmpeg
- Add FFmpeg to system PATH

### "Unknown encoder 'prores_ks'"

**Solution:**
- Your FFmpeg build doesn't include ProRes encoder
- Reinstall FFmpeg with full codecs
- macOS: `brew reinstall ffmpeg`

### Large File Sizes

**Issue:** ProRes files are MUCH larger than WebM

**Comparison:**
- WebM (1080p60, 2min): ~50MB
- ProRes 422 (same): ~5GB

**Solutions:**
- Use ProRes Proxy (profile 0) for tests
- Use ProRes 422 (profile 2) for performance
- Only convert final versions
- Delete WebM after conversion if needed

### Choppy Playback in Ableton

**Causes:**
- File too large for system
- Wrong codec/profile
- Insufficient RAM/CPU

**Solutions:**
- Lower resolution (1280×720)
- Use ProRes Proxy (profile 0)
- Increase Ableton buffer size
- Close other applications

---

## ADVANCED OPTIONS

### Custom Quality Settings

**Higher Bitrate:**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 2 -qscale:v 0 -pix_fmt yuv422p10le output.mov
```

**With Audio (if WebM has audio):**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 2 -c:a pcm_s16le -pix_fmt yuv422p10le output.mov
```

**Specific Frame Rate:**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 2 -r 60 -pix_fmt yuv422p10le output.mov
```

### Trim Video

**Start at 5 seconds, duration 30 seconds:**
```bash
ffmpeg -ss 5 -i input.webm -t 30 -c:v prores_ks -profile:v 2 -pix_fmt yuv422p10le output.mov
```

### Resize Video

**1080p to 720p:**
```bash
ffmpeg -i input.webm -vf scale=1280:720 -c:v prores_ks -profile:v 2 -pix_fmt yuv422p10le output.mov
```

---

## PERFORMANCE OPTIMIZATION

### Multi-Core Encoding

FFmpeg uses all CPU cores by default, but you can optimize:

```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 2 -threads 8 -pix_fmt yuv422p10le output.mov
```

**Note:** ProRes encoding is CPU-intensive. Expect:
- 1080p60, 2 minutes: ~2-5 minutes encoding on modern CPU
- 4K60, 2 minutes: ~10-20 minutes encoding

### GPU Acceleration

ProRes encoding is CPU-only (no GPU acceleration available).

---

## STORAGE CONSIDERATIONS

### File Size Estimates

**ProRes 422 (profile 2):**
- 1080p60: ~2.5GB per minute
- 4K60: ~10GB per minute

**ProRes Proxy (profile 0):**
- 1080p60: ~500MB per minute
- 4K60: ~2GB per minute

**Recommendations:**
- For performance: Use ProRes 422
- For storage: Use ProRes Proxy
- For archive: Keep original WebM files

---

## QUICK REFERENCE

### Most Common Command

```bash
ffmpeg -i MyShader.webm -c:v prores_ks -profile:v 2 -pix_fmt yuv422p10le MyShader_ProRes.mov
```

**Copy-paste template:**
Replace "MyShader" with your filename, run in terminal.

### Profile Quick Reference

| Profile | Name | Quality | File Size | Use Case |
|---------|------|---------|-----------|----------|
| 0 | Proxy | Low | Smallest | Testing, backup |
| 1 | LT | Medium-Low | Small | Rough edits |
| 2 | 422 | Medium | Medium | **Performance** |
| 3 | HQ | High | Large | Mastering, export |

---

## ALTERNATIVE TOOLS

### GUI Applications

**HandBrake:**
- Free, open-source
- GUI interface
- Limited ProRes support

**Adobe Media Encoder:**
- Professional encoding
- Full ProRes support
- Part of Adobe CC

**Compressor (macOS):**
- Built into Final Cut Pro
- Native ProRes encoding
- Batch processing

**Note:** FFmpeg command-line is fastest for batch processing.

---

## RESOURCES

**FFmpeg Documentation:**
- Official: https://ffmpeg.org/documentation.html
- ProRes: https://trac.ffmpeg.org/wiki/Encode/VFX

**EboSuite:**
- Website: https://ebosuite.com
- Documentation: https://ebosuite.com/documentation
- Tutorials: YouTube "EboSuite tutorials"

**Ableton Live:**
- Official docs: https://www.ableton.com/en/manual/

---

**END OF FFMPEG VIDEO GUIDE**

**Recommended Workflow:**
1. OMS → Render WebM
2. FFmpeg → Convert to ProRes 422
3. Ableton + EboSuite → Perform!

**Standard Command:**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 2 -pix_fmt yuv422p10le output.mov
```
