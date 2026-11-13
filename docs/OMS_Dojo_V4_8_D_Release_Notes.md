# ONEMANSHYO BETA V4_8_0 - RELEASE NOTES

**Release Date:** December 19, 2025  
**Version:** Beta V4_8_0  
**Type:** Modulator System Overhaul

---

## WHAT'S NEW

### 🎛️ Modulator Knobs

**Visual controls for all modulator settings!**

The CREATE tab reactivity section now has 9 rotary knobs:

| Knob | Range | Purpose |
|------|-------|---------|
| Gain (×3) | 0-1 | Amplitude multiplier |
| Freq1 (×3) | 20-24kHz | FFT range start |
| Freq2 (×3) | 20-24kHz | FFT range end |

- Drag up/down to adjust
- Purple color matches PERFORM tab
- Freq knobs use log scale (more control in bass frequencies)
- Freq knobs disabled when no audio loaded

---

### 🎹 Reactivity Keyboard Mode

**Control modulators without a mouse!**

| Key | Action |
|-----|--------|
| `Shift+R` | Enter/exit reactivity mode |
| `←` `→` | Navigate columns |
| `↑` `↓` | Navigate rows |
| `Shift+←` | Decrease value |
| `Shift+→` | Increase value |
| `Escape` | Exit mode |

**Progressive Acceleration:** Hold the key longer = faster changes!

Cyan selection ring shows current control. Perfect for dialing in settings during performance.

---

### 🔧 Independent Shader Uniforms

**Granular control for shader authors!**

New uniforms available in your shaders:

```wgsl
// Beat position (tempo-synced)
uniforms.beat

// Modulator A components
uniforms.shaderModAWave   // Waveform output (0-1)
uniforms.shaderModAGain   // Gain value (0-1)
uniforms.shaderModAFFT    // FFT level (0-1)

// Same for B and C
uniforms.shaderModBWave, uniforms.shaderModBGain, uniforms.shaderModBFFT
uniforms.shaderModCWave, uniforms.shaderModCGain, uniforms.shaderModCFFT
```

**Legacy still works!** Old shaders using `modA/B/C` are fully compatible.

---

### 📂 WGSL Preset Support

**Drop shader files directly!**

The /Presets/ folder now accepts both:
- `.oms` containers (full package)
- `.wgsl` files (raw shader code)

Copy shader code from Claude chats, save as .wgsl, and it appears in the shader dropdown!

---

### 🎵 Automatic BPM Detection

**ID3 tags read automatically!**

Drop an MP3 with embedded BPM:
- ID3v2 TBPM tag detected
- Audio Tempo auto-populates
- No more manual entry for tagged files

Falls back to filename parsing if no ID3 tag.

---

### 📦 Custom Shader Dropdown

**Better shader selection!**

The shader name dropdown now features:
- Shader thumbnails (32×18 preview)
- Keyboard navigation
- Grouped sections (Current, System, User, Library)
- Click outside to close

---

## KEYBOARD QUICK REFERENCE

### Reactivity Mode (Shift+R to enable)

| Key | Action |
|-----|--------|
| `←` `→` | Column: Subdiv → Waveform → Gain → Freq1 → Freq2 |
| `↑` `↓` | Row: A ↔ B ↔ C |
| `Shift+←` | Decrease value |
| `Shift+→` | Increase value |
| `Escape` | Exit mode |

### Grid Navigation (CREATE tab)

| Key | Action |
|-----|--------|
| `Shift+<` | Jump to previous grid line (4 bars) |
| `Shift+>` | Jump to next grid line (4 bars) |

---

## NEW WAVEFORM OPTIONS

Two new modulator waveform types:

| Type | Output | Use Case |
|------|--------|----------|
| None | 0 | Disable modulator |
| Static | 1 | Gain controls directly (no oscillation) |

---

## BUG FIXES

- ✅ Shift+Arrow no longer conflicts with playhead scrub
- ✅ Container names preserved when updating existing containers
- ✅ Gain keyboard range matches knob range (0-1)
- ✅ Zero gain values no longer wrap to default
- ✅ .wgsl files appear in shader dropdown after adding

---

## WORKFLOW TIPS

### Using Independent Uniforms

Instead of this:
```wgsl
let intensity = uniforms.modA;  // Combined value
```

Try this:
```wgsl
let wave = uniforms.shaderModAWave;     // Just the oscillator
let audio = uniforms.shaderModAFFT;     // Just the audio level
let intensity = wave * audio;            // Custom combination
```

### Quick Gain Adjustment

1. Press `Shift+R` to enter reactivity mode
2. Navigate to gain knob with arrows
3. Hold `Shift+→` to increase (accelerates!)
4. Press `Escape` when done

### Beat-Synced Effects

Use `uniforms.beat` for tempo-locked animations:
```wgsl
let pulse = fract(uniforms.beat);        // 0→1 each beat
let bar = fract(uniforms.beat / 4.0);    // 0→1 each bar
```

---

## COMPATIBILITY

✅ Fully compatible with V4_7_0  
✅ All .oms containers work  
✅ All .omsp projects work  
✅ Old shaders work unchanged  
✅ New uniforms default to sensible values

---

## WHAT'S NEXT

Coming in V4_9_0:
- **MIDI Integration** - Map controllers to modulators
- **PERFORM EQ** - Kill/adjust frequency bands live
- **Visual knob containers** - Grouped knob displays

---

## FEEDBACK

- Social: @itswessmithyo
- YouTube: @itswessmithyo
- Website: https://onemanshyo.com

---

**END OF RELEASE NOTES**

Version: Beta V4_8_0
Date: December 19, 2025
License: GPL-3.0

*"Independent control, unified performance"*
