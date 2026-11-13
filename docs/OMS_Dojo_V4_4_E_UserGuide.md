# ONEMANSHYO BETA V4_4_0 - USER GUIDE

**Version:** Beta V4_4_0  
**Date:** December 6, 2025

---

## QUICK START

1. Open `OMS_Beta_V4_4_0.html`
2. You're on Controls tab — shader is already running!
3. To save work: Settings tab (S) → "Set Application Folder"
4. Library tab (L) → Drag audio file anywhere → Container created!
5. LIVE tab → Drag container to grid → Click to select → Enter to launch!

---

## WHAT'S NEW IN V4_4_0

### 📐 Two-Panel Library

**Professional DAW-style layout!**

Library tab now shows:
- **Left panel:** Sets list + shader preview
- **Right panel:** Containers + audio controls

### 🎭 Sets System

**Filter your containers!**

- **Presets:** Built-in shaders (no folder needed!)
- **All:** Your saved containers

### 📥 Drag Anywhere

**Drop files anywhere on the app!**

- Audio → Creates container (Library) or loads (other tabs)
- Shader → Loads into editor
- ZIP → Imports container

### 🖼️ Shader Thumbnails

**Visual preview on every card!**

Each container shows a mini shader preview.

### ⌨️ Shift+Enter

**Fast shader auditioning!**

- Enter = full load
- Shift+Enter = shader only (keeps your audio)

---

## INTERFACE

### Tabs

| Tab | Name | Color | Folder Needed |
|-----|------|-------|---------------|
| **C** | Controls | Cyan | No |
| **P** | Presets | Cyan | No |
| **W** | WGSL Code | Cyan | No |
| **L** | Library | Magenta | Yes |
| **LIVE** | Performance | Magenta | Yes |
| **S** | Settings | Yellow | Yes |
| **U** | Utilities | Yellow | Yes |
| **A** | About | Yellow | No |

**Color Guide:**
- **Cyan:** Creation tools — work without a folder
- **Magenta:** Library/Performance — need folder for saving
- **Yellow:** Utilities and settings

---

## LIBRARY TAB

### Layout

```
+------------------+------------------------------------------------+
|      SETS        |  CONTAINERS                                    |
|                  |  Load  Update  Export  Add Current         X   |
|  All             |  Name ▼                      Audio  BPM        |
|  Presets    [●]  |------------------------------------------------|
|------------------|  [🎨] ● My Shader               Y    128        |
|  Shader Output   |  [🎨]   Cool Track              Y    124        |
|  [live preview]  |------------------------------------------------|
|                  |  [Waveform display area]                       |
|                  |  [Audio controls strip]                        |
+------------------+------------------------------------------------+
```

### Sets (Left Panel)

**Presets:**
- Built-in system shaders
- Works immediately (no folder needed!)
- Great for exploring

**All:**
- Your saved containers from Library folder
- Requires folder to be set

Click a set to filter the container list.

### Containers (Right Panel)

**Columns (click to sort):**
- **Name:** Container name (A-Z / Z-A)
- **Audio:** Has audio file (Y/N)
- **BPM:** Tempo (low-high / high-low)

**Indicators:**
- **●** = Currently loaded container
- **[🎨]** = Shader thumbnail

### Buttons

| Button | Action |
|--------|--------|
| **Load** | Load selected container |
| **Update** | Save current state to selected container |
| **Export** | Export as .oms.zip file |
| **Add Current** | Create new container from current state |
| **X** | Delete selected container + audio |

---

## ADDING CONTENT

### Method 1: Drag Audio (Fastest!)

1. Go to Library tab
2. Drag mp3/wav file anywhere on the app
3. Dark overlay appears with "Drop files here"
4. Drop!
5. Container created with:
   - Audio filename as name
   - BPM from filename (if present)
   - Current shader settings

### Method 2: Add Current

1. Set up your shader on Controls tab
2. Load audio (drag to Controls tab)
3. Go to Library tab
4. Click **Add Current**
5. Type a name
6. Done!

### Method 3: Import ZIP

1. Go to Library tab
2. Drag .oms.zip file onto app
3. Container + audio extracted

---

## AUDIO CONTROLS

### Waveform Section

- **Click** anywhere to set playhead position
- **Spacebar** to play/pause from that position
- **Arrow keys** to navigate beat by beat

### Audio Boundaries (Green Lines)

**Audio Start:**
- Click label → Reset to 0:00
- Shift+click → Pin to playhead position

**Audio End:**
- Click label → Reset to file end
- Shift+click → Pin to playhead position

### Loop (Cyan Brackets)

- **Loop In/Out** fields set the loop region
- **Loop toggle** shows/hides cyan brackets
- When OFF, playback starts at Audio Start

### Waveform Colors

| Color | Meaning |
|-------|---------|
| Magenta | Playhead (current position) |
| Green | Audio Start/End boundaries |
| Cyan | Loop In/Out markers |

---

## LIVE PERFORMANCE

### Loading Containers

**Drag & Drop:**
- Drag from Library left panel
- Drop on any grid cell
- Thumbnail shows shader preview

### Launching

**Scene Launch:**
1. Click scene button (row header) → Selects scene
2. Press Enter → Launches all tracks in scene

**Single Track:**
- Double-click cell → Launches that track

### Shader-Only Containers

Containers without audio now work!
- Launch like normal
- Shader renders in compositor
- Status shows "(shader only)"

### Set Dropdown

Top of left panel — filters which containers show in the list.

---

## KEYBOARD SHORTCUTS

### Library Tab
| Key | Action |
|-----|--------|
| ↑/↓ | Navigate container list |
| Enter | Load container (full) |
| Shift+Enter | Load shader only (keep audio) |
| Delete | Remove selected container |

### Audio
| Key | Action |
|-----|--------|
| Space | Play/pause |
| ←/→ | Navigate by beat |

### Navigation
| Key | Action |
|-----|--------|
| W/A/S/D | Navigate grid |
| 1-4 | Select tracks |
| Tab | Cycle tracks |

### Transport
| Key | Action |
|-----|--------|
| Space | Play/pause |
| Enter | Launch selected |
| Esc | Stop all |

### Mixer
| Key | Action |
|-----|--------|
| ↑/↓ | Volume up/down |
| ←/→ | Opacity up/down |
| ,/. | Audio mute/solo |
| Shift+,/. | Shader mute/solo |

### Visibility
| Key | Action |
|-----|--------|
| Shift+M | Master Grid |
| Shift+T | Track Grid |
| Shift+W | Waveform |

---

## WORKFLOW TIPS

### Shader Auditioning

**Find the perfect shader for your track:**
1. Load a container with your audio (Enter)
2. Arrow down to another container
3. Press Shift+Enter (loads shader, keeps audio)
4. Repeat until you find the match
5. Save as new container

### Organizing Library

**By BPM (DJ workflow):**
- Click BPM column header
- Group similar tempos
- Easy to find compatible tracks

**By Audio (production):**
- Click Audio column header
- See which containers need audio
- Focus on incomplete work

### Using Presets

**Quick exploration:**
1. Click "Presets" set (no folder needed!)
2. Browse built-in shaders
3. Find one you like
4. Add your audio
5. Save to your Library

---

## USER PRESETS

### Saving Your Shaders

1. Create/modify shader on W (WGSL) tab
2. Click **Save** button
3. Enter a name
4. Saved to /Presets/ folder

### Using Saved Presets

1. Go to P (Presets) tab
2. USER section shows your presets
3. Click to load
4. X button to delete

---

## FOLDER STRUCTURE

```
Your Folder/
  ├── onemanshyo.omsp   ← Session state (auto-saved)
  ├── Library/          ← Containers + audio files
  ├── Presets/          ← User shader presets
  └── Exports/          ← ZIP exports
```

### What Goes Where

| File Type | Location | Purpose |
|-----------|----------|---------|
| .oms | Library/ | Container files |
| Audio (.mp3, .wav) | Library/ | Audio files |
| .wgsl | Presets/ | User shader presets |
| .omsp | Root | Session state |
| .oms.zip | Exports/ | Export packages |

---

## TROUBLESHOOTING

### Library Empty

**Check:**
- Application folder set? (Settings tab)
- "All" set selected? (Presets might be empty)
- .oms files in Library/?

### Can't Create Container

**Make sure:**
- Library tab is active
- Application folder is set
- Valid audio file type (mp3, wav, etc.)

### Shader-Only Not Working

**Check:**
- Container in grid cell
- Cell clicked/selected
- Enter pressed to launch

### Audio Not Playing

**Check:**
- Audio file exists in Library/
- Loop enabled or playhead at Audio Start
- Browser audio enabled

### Drop Overlay Not Showing

**Make sure:**
- Dragging external file (not internal card)
- File is on top of app window

---

## MIGRATING FROM V4.3.0_A

### What Changed

1. **Library layout** — Now two panels
2. **Sets system** — New filtering
3. **Drag/drop** — Unified (buttons removed)
4. **Tab order** — C P W | L LIVE | S U A
5. **Default tab** — Controls (was Library)

### Your Data

✅ All containers work immediately  
✅ All projects load normally  
✅ New features available right away

---

## SUPPORT

**Documentation:**
- This user guide
- Developer documentation
- Complete changelog
- Release notes

**Community:**
- Social: @itswessmithyo
- YouTube: @itswessmithyo
- Website: https://itswessmithyo.com

---

**END OF USER GUIDE**

Version: Beta V4_4_0
Date: December 6, 2025
License: GPL-3.0

*"Create → Organize → Perform"*
