# ONEMANSHYO BETA V4_5_0 - RELEASE NOTES

**Release Date:** December 10, 2025  
**Version:** Beta V4_5_0  
**Type:** Major Workflow Simplification

---

## WHAT'S NEW

### 🎯 4-Tab Workflow

**Simplified navigation!**

The interface is now organized into 4 focused tabs:

| Tab | Color | Purpose |
|-----|-------|---------|
| **CREATE** | Cyan | Make shaders |
| **ORGANIZE** | Yellow | Manage library |
| **PERFORM** | Magenta | DJ with 4 tracks |
| **ADMIN** | Grey | Settings & info |

The natural workflow: **Create → Organize → Perform → Admin**

---

### 💻 Code View Toggle

**Code editing built into CREATE!**

Click the **[Code]** button in the CREATE tab header to toggle between:
- **Controls view:** Timing, Reactivity, OUTPUT sections
- **Code view:** Full shader editor with Compile/Copy/Save/Export

No more switching tabs to edit shader code!

---

### ⚙️ Unified ADMIN Panel

**Everything in one place!**

The ADMIN tab consolidates:
- **Settings:** Folder setup, project states, greeting
- **Utilities:** Coming soon
- **About:** App info, credits, technical details

Click the nav items on the left to switch between sections.

---

### 🔍 Library Toolbar

**Professional organization tools!**

New toolbar in ORGANIZE tab:

```
[🔍 Search...] [L][U][E][+]  [+Set][≡][▦][ℹ]  [X]
```

| Button | Action | Shortcut |
|--------|--------|----------|
| 🔍 | Search containers | — |
| L | Load | Cmd+L |
| U | Update | Cmd+U |
| E | Export | Cmd+E |
| + | Add current | — |
| ℹ | Info drawer | Cmd+I |
| X | Delete | Delete |

---

### 📋 Info Drawer

**Container details at a glance!**

Click ℹ to open a slide-out panel showing:
- Container name
- BPM
- Audio file (with Swap/Remove)
- Duration
- Loop points
- Created date

Make changes and click OK to apply, or Cancel to discard.

---

### 🗑️ Safe Delete

**No more accidental deletions!**

Delete confirmation now works via status bar:
1. Press Delete → "Press Delete again to confirm"
2. Press Delete again within 3 seconds → Actually deletes
3. Wait 3 seconds → "Delete cancelled"

No browser dialog popups!

---

### 🟡 Loaded Container Indicator

**Know what's loaded!**

The currently loaded container shows with:
- Yellow border
- Yellow tint background

Clear visual distinction from selected (cyan) state.

---

### 🎵 Audio Controls Strip

**Refined audio controls!**

The audio strip now has:
- **G button:** Toggle beat grid visibility
- **S button:** Snap audio end to bar
- **X button:** Remove audio

Buttons stay visible but disable when no audio is loaded (no UI shift).

---

## KEYBOARD SHORTCUTS

### Tab Switching (NEW!)

| Key | Tab |
|-----|-----|
| Ctrl+1 | CREATE |
| Ctrl+2 | ORGANIZE |
| Ctrl+3 | PERFORM |
| Ctrl+4 | ADMIN |

### Library (ORGANIZE)

| Key | Action |
|-----|--------|
| Cmd+L | Load container |
| Cmd+U | Update container |
| Cmd+E | Export container |
| Cmd+I | Toggle info drawer |
| Cmd+S | Swap shader |
| Cmd+A | Swap audio |
| ↑/↓ | Navigate list |
| Enter | Load selected |
| Delete | Delete (with confirm) |

### Everything Else

Unchanged from V4_4_0 — all your existing shortcuts still work!

---

## MIGRATION FROM V4_4_0

### What Changed

| Before (V4_4_0) | After (V4_5_0) |
|-----------------|----------------|
| 8 tabs (C P W L LIVE S U A) | 4 tabs (CREATE ORGANIZE PERFORM ADMIN) |
| W tab for code | [Code] button in CREATE |
| S tab for settings | ADMIN → Settings |
| U tab for utilities | ADMIN → Utilities |
| A tab for about | ADMIN → About |

### Your Data

✅ All containers work immediately  
✅ All projects load normally  
✅ All keyboard shortcuts preserved  
✅ No migration needed

---

## TAB COLOR GUIDE

| Color | Meaning | Tabs |
|-------|---------|------|
| **Cyan** | Creation/editing | CREATE |
| **Yellow** | Organization | ORGANIZE |
| **Magenta** | Performance | PERFORM |
| **Grey** | Administration | ADMIN |

---

## COMPATIBILITY

✅ Fully compatible with V4_4_0  
✅ Fully compatible with V4_3_0_A  
✅ All .oms containers work  
✅ All .omsp projects work

---

## PRODUCTION READINESS

**Overall: ~94%**

**100% Complete:**
- 4-tab workflow
- Code view toggle
- ADMIN consolidation
- Library toolbar
- Info drawer
- Delete confirmation

**Remaining:**
- Grid view mode
- Custom sets
- MIDI integration

---

## QUICK START

1. Open `OMS_Beta_V4_5_0.html`
2. You're on CREATE tab — shader running!
3. Click [Code] to edit shader code
4. Go to ORGANIZE to manage containers
5. Go to PERFORM to DJ
6. Go to ADMIN for settings

---

## FEEDBACK

- Social: @itswessmithyo
- YouTube: @itswessmithyo
- Website: https://onemanshyo.com

---

**END OF RELEASE NOTES**

Version: Beta V4_5_0
Date: December 10, 2025
License: GPL-3.0

*"Create → Organize → Perform"*
