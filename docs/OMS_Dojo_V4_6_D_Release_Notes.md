# ONEMANSHYO BETA V4_6_0 - RELEASE NOTES

**Release Date:** December 10, 2025  
**Version:** Beta V4_6_0  
**Type:** Workflow Consistency & Save Improvements

---

## WHAT'S NEW

### 🎯 Unified Save Workflow

**Consistent saves everywhere!**

All save paths now use the same container builder:
- Save button in CREATE tab
- Add to Library (+)
- Drag-drop audio to ORGANIZE

Your containers always have complete, consistent data.

---

### 🎵 Audio Swap in CREATE Tab

**Pick audio from your Library!**

New controls in AUDIO CONTROLS header:

| Button | Action |
|--------|--------|
| **Swap** | Show Library audio files |
| **X** | Remove loaded audio |

Click Swap to see audio files grouped by:
- **Unused** - Files not in any container (pick these first!)
- **In Use** - Files already referenced

---

### 🎨 Shader Picker Dropdown

**Pick shaders without leaving CREATE!**

The Name dropdown now shows:
- System presets (built-in)
- User presets (your /Presets folder)
- Library containers (all your saved work)

Select any shader to load its code immediately.

---

### 📐 Orientation in SHADER CONTROLS

**Set canvas shape upfront!**

Orientation moved from OUTPUT to SHADER CONTROLS section:
- Landscape (1920×1080)
- Portrait (1080×1920) 
- Square (1080×1080)

Changes auto-save to your project file.

---

### 📝 Smart Filename

**Auto-populated working title!**

The filename field now auto-fills:
- Load shader "TronTunnel" → filename: "TronTunnel"
- Load audio "BassTrack_128BPM.mp3" → filename: "BassTrack TronTunnel"

Used for both rendering and saving to Library.

---

### 🔄 Loop Defaults

**Your loop preferences persist!**

- Set Loop In/Out how you like
- Values save to project file
- Fresh audio uses your defaults
- Existing containers keep their values

Example: Always work with 32-bar loops? Set once, applies to all new audio.

---

### ✏️ Click-to-Edit Names

**Rename containers instantly!**

In ORGANIZE tab:
- Click container name → edit inline
- Press Enter → saved to disk
- No dialogs, no buttons

---

### 🎵 All Audio Filter

**Find containers with audio!**

New "All Audio" option in SET filters:
- ORGANIZE tab: Click "All Audio" card
- PERFORM tab: Select from SET dropdown

Shows only containers that have audio files attached.

---

### 🏷️ Tempo Column

**Standardized naming!**

- "BPM" column → now "Tempo"
- White text (was grey)
- Displays as "128 BPM"

Same data, clearer presentation.

---

## WORKFLOW IMPROVEMENTS

### Save Flow

**Before:**
1. Click Save
2. If no folder → switch tabs, set folder
3. Go back to CREATE
4. Click Save again

**After:**
1. Click Save
2. If no folder → set folder
3. Save completes automatically!

---

### Audio Selection

**Before:**
1. Use file picker
2. Navigate to folder
3. Find audio file
4. Select

**After:**
1. Click Swap
2. See Library audio files
3. Click to select

One click to see what's available!

---

## KEYBOARD SHORTCUTS

All V4_5_0 shortcuts unchanged. New behaviors:

| Action | How |
|--------|-----|
| Edit container name | Click name in list |
| Audio swap | Click Swap button |
| Shader swap | Click Name dropdown |

---

## COMPATIBILITY

✅ Fully compatible with V4_5_0  
✅ All .oms containers work  
✅ All .omsp projects work  
✅ New fields added gracefully

---

## MIGRATION NOTES

### Container Field Changes

| Field | Before | After |
|-------|--------|-------|
| Audio tempo | `audioBPM` | `audioTempo` |

Old containers still load fine - the app handles both field names.

### New Project Fields

Your .omsp file now includes:
- `orientation` - Canvas orientation
- `defaultLoopIn` - Default loop start
- `defaultLoopOut` - Default loop end

These add automatically on first save.

---

## PRODUCTION READINESS

**Overall: ~95%**

**100% Complete:**
- Unified container builder
- Audio/shader swapping
- Filename auto-population
- Loop defaults
- Name editing

**Remaining:**
- MIDI integration
- Advanced settings

---

## QUICK START

### New Audio Workflow

1. Open CREATE tab
2. Click **Swap** in AUDIO CONTROLS header
3. Select audio from dropdown
4. Tempo auto-detected, filename auto-filled
5. Click **Save** when ready

### New Shader Workflow

1. Click **Name** dropdown in SHADER CONTROLS
2. See all presets + Library shaders
3. Select one to load
4. Compile and preview immediately

---

## FEEDBACK

- Social: @itswessmithyo
- YouTube: @itswessmithyo
- Website: https://onemanshyo.com

---

**END OF RELEASE NOTES**

Version: Beta V4_6_0
Date: December 10, 2025
License: GPL-3.0

*"Create → Organize → Perform"*
