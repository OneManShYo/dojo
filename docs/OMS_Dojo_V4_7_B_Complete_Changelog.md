# ONEMANSHYO BETA V4_7_0 - COMPLETE CHANGELOG

**Version:** Beta V4_7_0  
**Release Date:** December 12, 2025  
**Base Version:** V4_6_0  
**Development Iterations:** 22 sub-iterations (V4_7_01 through V4_7_22)

---

## EXECUTIVE SUMMARY

V4_7_0 is a PERFORM tab polish release that establishes the Message Bar as a CORTEX-level system. All user feedback now flows through a single `showMessage()` function with semantic color coding. The PERFORM tab receives comprehensive keyboard and mouse feedback, making every control discoverable and every action confirmable.

**Key Achievement:** Central communication system with 44 PERFORM controls providing instant visual feedback.

---

## MAJOR FEATURES

### 1. Message Bar System (V4_7_21-22) - CORTEX

✅ **Architectural Refactor**
- `shaderStatus` → `messageBar` (element ID)
- `.status` → `.message-bar` (CSS class)
- `showStatus()` → `showMessage()` (function)
- `'success'` → `'state'` (type value)

✅ **Color-Coded Messages**
| Type | Color | Use |
|------|-------|-----|
| `'info'` | Cyan #00d4ff | Help text, shortcuts |
| `'state'` | Orange #ffa500 | Current values, confirmations |
| `'error'` | Magenta #ff006e | Failures, problems |

✅ **Single Entry Point**
```javascript
// CORTEX - the ONLY way to communicate transient info to user
showMessage(text, type)
```

### 2. PERFORM Tab Controls Feedback (V4_7_18-22)

✅ **Keyboard Feedback** (28 shortcuts)
- Track selection (1-5): "Track N selected"
- Arrow keys: Shows current value after change
- Mute/Solo: Shows new state with key hint
- Grid visibility: Shows ON/OFF state
- Zoom: Shows current level
- Navigation: Shows current cell position

✅ **Mouse Hover Feedback** (16 controls)
- Knobs show keyboard shortcut hints
- Buttons show action descriptions
- Mouseenter → info message
- Mouseleave → clears message

### 3. User-Created Sets (V4_7_09-11)

✅ **Custom Container Collections**
- Create sets in ORGANIZE tab
- Add containers to sets
- Filter by set in PERFORM tab

✅ **Set Management**
- Rename/delete sets
- Containers can belong to multiple sets
- Sets persist in project file

### 4. Auto-Play Library (V4_7_08)

✅ **Toggle in ORGANIZE Toolbar**
- Click container → auto-play audio preview
- Toggle ON/OFF in toolbar
- Preference saved to project

### 5. Non-Blocking Save (V4_7_02)

✅ **Background FFT Analysis**
- Save workflow doesn't freeze UI
- Progress indicator during analysis
- User can continue working

---

## ITERATION HISTORY

### V4_7_01-05: ORGANIZE Tab Polish

**V4_7_01:** Info drawer rename fix
- Click-to-edit with immediate filesystem save
- Fixed: Name changes now persist correctly

**V4_7_02:** Non-blocking save with background FFT
- analyzeAudioTimeline() runs async
- UI remains responsive during save

**V4_7_03:** Centralized container naming
- Single source for container name generation
- Consistent naming across all save paths

**V4_7_04:** Info drawer keyboard shortcuts
- Cmd+L load, Cmd+U update
- Enter saves changes

**V4_7_05:** Tempo handling hierarchy
- Container tempo > Project tempo > Default
- Consistent tempo source of truth

### V4_7_06-10: Library Features

**V4_7_06:** Audio end time restoration
- Fixed: audioEndTime now restores on container load
- Was defaulting to 0 instead of stored value

**V4_7_07:** Audio positioning after restoration
- Waveform positions correctly after restore
- Bar 1 aligns at playhead

**V4_7_08:** Auto-play library feature
- Toggle button in ORGANIZE toolbar
- Plays audio preview on container select

**V4_7_09:** User-created sets system
- Create custom sets
- Add/remove containers from sets

**V4_7_10:** Sets system refinement
- Set persistence in .omsp
- UI polish for set cards

### V4_7_11-15: Delete & Filter System

**V4_7_11:** Unified keyboard delete
- Delete key works across tabs
- Consistent confirmation pattern

**V4_7_12:** Safe audio file handling
- Protected when in use by containers
- Clear warning before delete

**V4_7_13:** Auto-select next after delete
- Cursor moves to next item
- No orphan selection state

**V4_7_14:** Hide Swap/X on PERFORM
- Buttons hidden when PERFORM tab active
- Cleaner performance interface

**V4_7_15:** Unified library filter architecture
- Single filter system across ORGANIZE/PERFORM
- Consistent container filtering

### V4_7_16-18: PERFORM Keyboard Fixes

**V4_7_16:** Shader M/S keyboard shortcuts fix
- Shift+, and Shift+. now work correctly
- Fixed: Was checking wrong modifier key

**V4_7_17:** PERFORM tab controls audit
- Documented all 44 controls
- Identified missing feedback

**V4_7_18:** Status bar feedback for mute/solo
- All M/S keys show state change
- Format: "Track N Domain: State (key)"

### V4_7_19-22: Message Bar System

**V4_7_19:** Comprehensive status bar feedback
- ALL keyboard actions show feedback
- Track selection, arrows, zoom, navigation

**V4_7_20:** Knob hover messages
- Removed slow browser title tooltips
- All hovers use instant message bar

**V4_7_21:** Color-coded status system
- Cyan = info (help/shortcuts)
- Orange = state (values/confirmations)
- Magenta = error (failures)

**V4_7_22:** Message Bar refactor (CORTEX)
- Renamed: shaderStatus → messageBar
- Renamed: showStatus() → showMessage()
- Renamed: 'success' → 'state'
- Fixed: Shift+W conflict with WASD
- Added: Master Shader Solo hover

---

## NEW FUNCTIONS

### showMessage(text, type)

**Purpose:** Central message bar API - CORTEX level.

**Location:** ~Line 5766

**Parameters:**
- `text` (string): Message to display
- `type` (string): 'info' | 'state' | 'error'

**Usage:**
```javascript
showMessage('Audio Volume (↑/↓)', 'info')    // Cyan - help
showMessage('Track 1 Shader: 80%', 'state')  // Orange - value
showMessage('Container not found', 'error')  // Magenta - error
```

---

## CSS CHANGES

### Message Bar Styles

```css
/* Base class */
.message-bar {
    width: 100%;
    height: 30px;
    background: linear-gradient(180deg, #222, #2a2a2a);
    color: #00d4ff;  /* Default cyan */
    border-top: 1px solid #444;
    text-align: center;
}

/* Type modifiers */
.message-bar.info { color: #00d4ff; }   /* Cyan */
.message-bar.state { color: #ffa500; }  /* Orange */
.message-bar.error { color: #ff006e; }  /* Magenta */
```

---

## BUG FIXES

| Issue | Cause | Fix |
|-------|-------|-----|
| Shift+W not working | WASD handler catching W first | Added `&& !e.shiftKey` check |
| Master Shader Solo no hover | Missing event listener | Added mouseenter/mouseleave handlers |
| Info drawer rename not saving | Missing filesystem call | Added immediate renameContainer() |
| Audio end time resetting | Not restored from container | Fixed applyControlState() |
| Delete removing audio in use | No protection check | Added container reference check |

---

## STATISTICS

- **Total Sub-iterations:** 22
- **Lines Changed:** ~800+
- **Functions Added:** 1 (showMessage replaces showStatus)
- **Function Calls Updated:** 245
- **Controls with Feedback:** 44
- **New UI Features:** 3 (sets, auto-play, message colors)

---

## PRODUCTION READINESS

**Overall: ~96%**

**100% Complete:**
- Message bar system
- PERFORM tab controls
- Color-coded feedback
- User-created sets
- Auto-play library
- Unified filters

**Inherited from V4_6_0 (100%):**
- Unified container builder
- Tempo standardization
- Shader/audio swap
- Click-to-edit names

**Remaining:**
- MIDI integration
- PERFORM EQ controls
- Advanced ADMIN settings

---

**END OF COMPLETE CHANGELOG**

Version: Beta V4_7_0
Date: December 12, 2025
Total Development: 22 sub-iterations
License: GPL-3.0
