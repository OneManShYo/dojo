# OMS SHADER BAKER V3.8.0 - USER GUIDE

**Version:** V3.8.0  
**Date:** November 17, 2025

---

## QUICK START

1. Open `OMS_ShaderBaker_V3_8_0.html`
2. Settings tab (S) → "Set Application Folder"
3. Settings → GREETING → Type custom greeting
4. Controls tab (C) → Create shader
5. Save as .oms
6. LIVE tab → Drag to grid
7. Pop Out → Second monitor
8. Space to play!

---

## WHAT'S NEW IN V3.8.0

### Ergonomic Controls

**Opacity:** `←→` arrows (no Shift!)  
**Volume:** `↑↓` arrows  
**Zoom:** `[` `]` brackets + trackpad

### Custom Greeting

Settings → GREETING → Type yours!

### About Tab

Check out credits, bio, tech specs

### 5-Column Grid

30px wider cells for better visibility

---

## INTERFACE

### Tabs

**C (Controls):** Shader creation  
**P (Presets):** Library  
**L (Library):** .oms containers *  
**W (Code):** WGSL editor  
**S (Settings):** Config *  
**A (About):** Info ← NEW!  
**LIVE:** Performance *

* = Requires folder

### Header

**Left:** ONEMANSHYO + version  
**Right:** Your custom greeting

---

## SHADER CREATION

### Controls Tab

Sliders for instant preview

### Code Tab

Edit WGSL directly

**Modulators:**
- uModA: Bass (30-500 Hz)
- uModB: Mids (500-2000 Hz)  
- uModC: Treble (2000-20000 Hz)

---

## LIVE PERFORMANCE

### Grid Layout

5 columns × 8 rows  
Tracks 1-4 + Master/Scenes

### Loading

Drag .oms from Library → Grid

### Launching

**Scene:** Click number or Enter  
**Cell:** Double-click or Enter  
**Quantized:** Bar-aligned

### Mixer

**Per-Track:**
- Volume knob (audio)
- Opacity knob (shader)
- M/S buttons (both domains)
- Level meters

**Master:**
- Volume (2x larger)
- M/S buttons
- Tempo control

---

## ERGONOMIC CONTROLS

### Right Hand

```
       [↑]        Volume
 [←] [↓] [→]     Opacity ←/→
```

No Shift needed!

### Left Hand

```
     [W]         Up
 [A] [S] [D]    Left/Down/Right
   [Space]      Play/Pause
```

---

## DUAL-DOMAIN MIXER

**Shader Domain:**
- Opacity knob
- M/S (purple/cyan)
- `←→` keys

**Audio Domain:**
- Volume fader
- M/S (standard)
- `↑↓` keys

**Use Cases:**
- Visual build without audio change
- Audio drop with visual hold
- Solo one shader, play all audio

---

## ZOOM CONTROLS

**Trackpad:** Pinch/spread  
**Brackets:** `[` `]` (10px steps)  
**Both work anytime!**

---

## POP-OUT WINDOW

1. Click "Pop Out ↗"
2. Drag to projector
3. Resize as needed
4. Optional: Always on Top (Settings)

**Shows:** Four-track composite, 60fps

---

## KEYBOARD SHORTCUTS

**Transport:**
- `Space`: Play/pause
- `Esc`: Emergency stop

**Track Selection:**
- `1-4`: Tracks
- `5`: Master
- `Tab/Shift+Tab`: Cycle

**Navigation:**
- `WASD`: Grid navigation
- `Enter`: Launch

**Audio:**
- `↑↓`: Volume
- `,`: Mute
- `.`: Solo

**Shader:** (NEW!)
- `←→`: Opacity
- `Shift+,`: Mute
- `Shift+.`: Solo

**Zoom:** (NEW!)
- `[` `]`: Discrete
- Trackpad: Continuous

**Cell:**
- `Delete`: Clear

---

## SETTINGS

### APPLICATION
- Set folder
- Canvas size

### PROJECT STATES
- Save/load .omsp
- default.omsp auto-saves

### GREETING (NEW!)
- Custom header text
- 50 char max

### POP-OUT
- Window size
- Always on Top

---

## ABOUT TAB

**Identity:** Version, creator  
**ONEMANSHYO:** Bio, achievements  
**Technical:** WebGPU, WGSL, GPL-3.0  
**Tools:** Claude, Web APIs  
**Inspired By:** 27 people/companies/communities

---

## FILE MANAGEMENT

### .oms Containers

**Create:**
1. Controls → shader
2. Load audio (optional)
3. "Save as .oms"
4. Library tab shows it

**Use:**
1. Drag to LIVE grid
2. Launch!

### .omsp Projects

**Save:**
1. Settings → PROJECT STATES
2. "Save Project"
3. Name it

**Load:**
1. Settings → PROJECT STATES
2. Select dropdown
3. "Load Project"

---

## TIPS & TRICKS

### Performance

**Prep:**
- Create shaders beforehand
- Organize Library
- Build grid layout
- Save .omsp

**During:**
- Keyboard only
- Pop-out to projector
- Trust muscle memory
- Esc = emergency

### Workflow

**Shaders:**
- Start simple
- Test incrementally
- Save often
- Name clearly

**Audio:**
- Isolate frequencies
- Test with real music
- Subtle > crazy

### Troubleshooting

**Won't compile:** Check console (F12)  
**Not reactive:** Check FFT ranges  
**Slow:** Lower resolution  
**Overlay stuck:** Press Esc

---

## CHEAT SHEET

```
Right Hand:        Left Hand:
    [↑]                 [W]
[←][↓][→]          [A][S][D]
                    [Space]

Track: [1][2][3][4][5]
Zoom: [ ]
M/S: , .  (Shift+ for shader)
```

---

**END OF USER GUIDE**

Version: V3.8.0  
License: GPL-3.0  
Website: https://itswessmithyo.com
