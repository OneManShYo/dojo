# ONEMANSHYO BETA V4_4_0 - RELEASE NOTES

**Release Date:** December 6, 2025  
**Version:** Beta V4_4_0  
**Type:** Major UI Restructure + Workflow Enhancement

---

## WHAT'S NEW

### 📐 Library Tab Two-Panel Layout

**Professional DAW-style organization!**

The Library tab is now split into two panels:
- **Left panel:** Your Sets with shader preview
- **Right panel:** Container list with audio controls

```
+------------------+------------------------------------------------+
|      SETS        |  CONTAINERS                                    |
|                  |  Load  Update  Export  Add Current         X   |
|  All             |  Name ▼                      Audio  BPM        |
|  Presets    [●]  |------------------------------------------------|
|------------------|  [🎨] ● My Shader               Y    128        |
|  Shader Output   |  [🎨]   Another Container       N     —         |
|  [live preview]  |  Waveform + Audio Controls below               |
+------------------+------------------------------------------------+
```

---

### 🎭 Sets System

**Filter containers by category!**

- **Presets:** Built-in system presets (works without folder!)
- **All:** Your containers from the Library folder

Click a Set to filter. Selection saves automatically.

---

### 📥 Unified Drag/Drop

**Drop files anywhere!**

No more hunting for the right button:
- **Audio files** → Creates container (Library tab) or loads audio (other tabs)
- **Shader files** → Loads into editor and compiles
- **ZIP files** → Imports container (Library tab only)

Full-screen overlay guides you when dragging.

---

### 🖼️ Shader Thumbnails

**See your shaders at a glance!**

Every container card now shows a live 32x18 thumbnail preview of its shader output.

---

### 🎬 Shader-Only Containers

**Launch shaders without audio!**

Containers without audio files now work in the LIVE grid:
- Visual overlay shows playing state
- Compositor renders shader properly
- Perfect for silent visual effects

---

### ⌨️ Shift+Enter: Shader Only

**Fast shader auditioning!**

- **Enter:** Full load (replaces everything)
- **Shift+Enter:** Load shader only, keep your audio

Workflow: Load your audio track, then Shift+Enter through shaders to find the perfect match.

---

### 🎵 Audio Workflow Improvements

**Better control over your audio boundaries!**

**Audio Start/End Pinning:**
- Click label → Reset to default
- Shift+click → Pin to current playhead position

**Loop Visual:**
- Cyan brackets show Loop In/Out
- Toggle Loop OFF → brackets hidden
- Cleaner waveform display

**Navigation:**
- Arrow keys navigate full file (not clamped to loop)
- Spacebar plays from clicked position

---

### 🎨 Tab Reorganization

**Tabs grouped by function!**

New order: **C P W | L LIVE | S U A**

| Color | Tabs | Meaning |
|-------|------|---------|
| Cyan | C P W | Creation (no folder needed) |
| Magenta | L LIVE | Library/Performance |
| Yellow | S U A | Utilities |

**Controls tab** is now the default — see your shader immediately!

---

### 💾 User Presets Folder

**Save your shaders permanently!**

- W tab now has a **Save** button
- Presets saved to `/Presets/` folder
- Appear in P tab USER section
- Survive browser restarts

---

## UI IMPROVEMENTS

### Library Container Cards

- **Shader thumbnail** (32x18 preview)
- **Audio column** showing Y/N
- **Three sortable columns:** Name, Audio, BPM
- **Active indicator** (●) shows loaded container

### Button Labels

Full text instead of single letters:
- **Load** | **Update** | **Export** | **Add Current** | **X**

### Drop Overlay

Professional drop feedback:
- Dark overlay with cyan icon
- "Drop files here to add" message
- Only shows for external file drops

---

## KEYBOARD SHORTCUTS

### Library Tab (New)
| Key | Action |
|-----|--------|
| ↑/↓ | Navigate container list |
| Enter | Load container (full) |
| Shift+Enter | Load shader only |

### Audio (Unchanged)
| Key | Action |
|-----|--------|
| Space | Play/pause |
| ←/→ | Navigate by beat |

### Everything Else (Unchanged from V4.3.0_A)

---

## COMPATIBILITY

✅ Fully compatible with V4.3.0_A  
✅ Fully compatible with V4.2.x  
✅ All .oms containers work  
✅ All .omsp projects work

### Data Migration

Your existing containers work immediately. Project files (.omsp) gain a new `selectedSetId` field on first save.

---

## FOLDER STRUCTURE (Updated)

```
Your Folder/
  ├── onemanshyo.omsp   ← Now includes selectedSetId
  ├── Library/          ← Your containers + audio
  ├── Presets/          ← User shader presets (NEW!)
  └── Exports/          ← ZIP exports
```

---

## PRODUCTION READINESS

**Overall: ~92%**

**Rock Solid:**
- Two-panel Library layout (100%)
- Sets system (100%)
- Unified drag/drop (100%)
- Shader thumbnails (100%)
- Audio workflow (100%)

**Inherited from V4.3.0_A:**
- Folder-based storage (100%)
- LIVE grid (100%)
- Audio engine (100%)

---

## WHAT'S NEXT

**Path to V1.0:**
1. MIDI control integration
2. Additional effect shaders
3. Final polish
4. Ship it!

---

**FEEDBACK WELCOME**

- Social: @itswessmithyo
- YouTube: @itswessmithyo
- Website: https://itswessmithyo.com

---

**END OF RELEASE NOTES**

Version: Beta V4_4_0
Date: December 6, 2025
License: GPL-3.0

*"Create → Organize → Perform"*
