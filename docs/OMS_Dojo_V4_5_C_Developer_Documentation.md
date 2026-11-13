# ONEMANSHYO BETA V4_5_0 - DEVELOPER DOCUMENTATION

**Version:** Beta V4_5_0  
**Date:** December 10, 2025

---

## TABLE OF CONTENTS

1. [Tab Architecture](#tab-architecture)
2. [CREATE Tab](#create-tab)
3. [ORGANIZE Tab](#organize-tab)
4. [PERFORM Tab](#perform-tab)
5. [ADMIN Tab](#admin-tab)
6. [Audio Controls Strip](#audio-controls-strip)
7. [CSS Classes Reference](#css-classes-reference)
8. [Element IDs Reference](#element-ids-reference)
9. [Functions Reference](#functions-reference)
10. [Keyboard Shortcuts](#keyboard-shortcuts)

---

## TAB ARCHITECTURE

### 4-Tab System

```
CREATE | ORGANIZE | PERFORM | ADMIN
```

| Tab | Internal ID | Color | CSS Class | Purpose |
|-----|-------------|-------|-----------|---------|
| CREATE | controlsTab | Cyan (#00d4ff) | tab-cyan | Shader creation, code editing |
| ORGANIZE | libraryTab | Yellow (#ffcc00) | tab-yellow | Container/library management |
| PERFORM | liveTab | Magenta (#ff00ff) | tab-magenta | 4-track DJ mixer |
| ADMIN | adminTab | Grey (#888) | tab-grey | Settings, Utilities, About |

### Tab Switching

```javascript
function switchTab(tabName) {
    // Remove active from all tabs/content
    document.getElementById('controlsTabContent').classList.remove('active');
    document.getElementById('libraryTabContent').classList.remove('active');
    document.getElementById('liveTabContent').classList.remove('active');
    document.getElementById('adminTabContent').classList.remove('active');
    
    document.getElementById('controlsTab').classList.remove('active');
    document.getElementById('libraryTab').classList.remove('active');
    document.getElementById('liveTab').classList.remove('active');
    document.getElementById('adminTab').classList.remove('active');
    
    // Activate selected tab
    switch(tabName) {
        case 'controls':
            document.getElementById('controlsTabContent').classList.add('active');
            document.getElementById('controlsTab').classList.add('active');
            break;
        case 'library':
            document.getElementById('libraryTabContent').classList.add('active');
            document.getElementById('libraryTab').classList.add('active');
            break;
        case 'live':
            document.getElementById('liveTabContent').classList.add('active');
            document.getElementById('liveTab').classList.add('active');
            break;
        case 'admin':
            document.getElementById('adminTabContent').classList.add('active');
            document.getElementById('adminTab').classList.add('active');
            break;
    }
}
```

### Keyboard Shortcuts

```javascript
// Tab switching (Ctrl+1-4)
if (e.ctrlKey && !e.metaKey && !e.shiftKey) {
    switch(e.key) {
        case '1': switchTab('controls'); break;
        case '2': switchTab('library'); break;
        case '3': switchTab('live'); break;
        case '4': switchTab('admin'); break;
    }
}
```

---

## CREATE TAB

### Structure

```
┌─────────────────────────────────────┐
│ [SHADER CONTROLS          ] [Code] │  ← Header bar with toggle
├─────────────────────────────────────┤
│                                     │
│ [Controls View OR Code View]        │  ← Toggled content
│                                     │
└─────────────────────────────────────┘
```

### Controls View

```html
<div id="shaderControlsContent">
    <!-- Timing section -->
    <div class="controls-section">
        <label class="section-label">Timing</label>
        <!-- tempo, duration controls -->
    </div>
    
    <!-- Reactivity section -->
    <div class="controls-section">
        <label class="section-label">Reactivity</label>
        <!-- modulator controls -->
    </div>
    
    <!-- OUTPUT section -->
    <div class="controls-section">
        <div class="output-header">OUTPUT</div>
        <!-- render settings -->
    </div>
</div>
```

### Code View

```html
<div id="shaderCodeContent" style="display: none;">
    <!-- Toolbar -->
    <div class="code-toolbar">
        <button onclick="compileShader(false)">Compile</button>
        <button onclick="copyShaderCode()">Copy</button>
        <button onclick="saveShaderToPresets()">Save</button>
        <button onclick="exportShaderTxt()">Export</button>
    </div>
    
    <!-- Shader code textarea (SINGLE SOURCE OF TRUTH) -->
    <textarea id="shaderCode">...</textarea>
</div>
```

### Toggle Function

```javascript
function toggleCodeView() {
    const controls = document.getElementById('shaderControlsContent');
    const code = document.getElementById('shaderCodeContent');
    const btn = document.getElementById('codeToggleBtn');
    
    if (code.style.display === 'none') {
        controls.style.display = 'none';
        code.style.display = 'flex';
        btn.textContent = 'Controls';
        btn.classList.add('active');
    } else {
        code.style.display = 'none';
        controls.style.display = 'block';
        btn.textContent = 'Code';
        btn.classList.remove('active');
    }
}
```

### Important Notes

- Single `shaderCode` textarea (no sync needed)
- Code section is sibling to controls-section (not nested)
- All existing shaderCode references work unchanged

---

## ORGANIZE TAB

### Structure

```
┌─────────────────────────────────────┬───────────────────────────────────────┐
│            LEFT PANEL               │            RIGHT PANEL                │
├─────────────────────────────────────┼───────────────────────────────────────┤
│ [LIBRARY                          ] │ [🔍 Search...] [L][U][E][+] ... [X]   │
├─────────────────────────────────────┼───────────────────────────────────────┤
│ ┌─────────────────────────────────┐ │ Name ▲              BPM    Audio      │
│ │ All                             │ │─────────────────────────────────────│
│ └─────────────────────────────────┘ │ [thumb] Container1   128    Y         │
│ ┌─────────────────────────────────┐ │ [thumb] Container2   124    Y         │
│ │ Presets                         │ │                                       │
│ └─────────────────────────────────┘ │ [Info Drawer - slide out]             │
└─────────────────────────────────────┴───────────────────────────────────────┘
```

### Library Toolbar

| Button | ID | Action | Shortcut |
|--------|-----|--------|----------|
| 🔍 | librarySearchInput | Filter containers | — |
| L | libraryLoadBtn | Load selected | Cmd+L |
| U | libraryUpdateBtn | Update loaded | Cmd+U |
| E | libraryExportBtn | Export as ZIP | Cmd+E |
| + | libraryAddBtn | Add current | — |
| +Set | libraryAddSetBtn | Create set | — |
| ≡ | libraryListViewBtn | List view | — |
| ▦ | libraryGridViewBtn | Grid view | — |
| ℹ | libraryInfoBtn | Toggle info drawer | Cmd+I |
| X | libraryDeleteBtn | Delete selected | Delete |

### Info Drawer

```html
<div id="libraryInfoDrawer" class="library-info-drawer">
    <div class="info-drawer-header">
        <span>Container Info</span>
        <button onclick="closeInfoDrawer()">×</button>
    </div>
    
    <div class="info-drawer-content">
        <div class="info-field">
            <label>NAME</label>
            <input id="infoDrawerName" readonly>
        </div>
        <div class="info-field">
            <label>BPM</label>
            <span id="infoDrawerBpm">—</span>
        </div>
        <div class="info-field">
            <label>AUDIO</label>
            <span id="infoDrawerAudio">—</span>
            <button class="info-drawer-swap-btn" onclick="swapAudio()">Swap</button>
            <button class="info-drawer-swap-btn" onclick="removeAudio()">Remove</button>
        </div>
        <!-- Duration, Loop, Created fields -->
    </div>
    
    <div class="info-drawer-footer">
        <button onclick="cancelInfoChanges()">Cancel</button>
        <button onclick="applyInfoChanges()">OK</button>
    </div>
</div>
```

### Search Implementation

```javascript
function filterLibraryList(searchTerm) {
    const containers = libraryContainersCache;
    const term = searchTerm.toLowerCase();
    
    return containers.filter(c => {
        // Search by name
        if (c.name.toLowerCase().includes(term)) return true;
        // Search by BPM
        if (c.audioBPM && c.audioBPM.toString().includes(term)) return true;
        // Search by audio (y/n)
        if (term === 'y' && c.audioFilename) return true;
        if (term === 'n' && !c.audioFilename) return true;
        // Search by shader code
        if (c.shaderCode && c.shaderCode.toLowerCase().includes(term)) return true;
        return false;
    });
}
```

### Container Card Indicators

```css
/* Selected state - cyan */
.library-container-card.selected {
    border: 1px solid #00d4ff;
    background: rgba(0, 212, 255, 0.1);
}

/* Loaded state - yellow */
.library-container-card.loaded {
    border: 1px solid #ffff00;
    background: rgba(255, 255, 0, 0.1);
}
```

### Delete Confirmation

```javascript
let pendingDeleteId = null;
let pendingDeleteTimeout = null;

function handleDelete() {
    const selected = getSelectedContainerId();
    if (!selected) return;
    
    if (pendingDeleteId === selected) {
        // Second press - actually delete
        clearTimeout(pendingDeleteTimeout);
        deleteContainer(selected);
        pendingDeleteId = null;
        showStatus('Container deleted', 'success');
    } else {
        // First press - start confirmation
        pendingDeleteId = selected;
        showStatus('Press Delete again to confirm removal', 'warning');
        
        pendingDeleteTimeout = setTimeout(() => {
            pendingDeleteId = null;
            showStatus('Delete cancelled', '');
        }, 3000);
    }
}
```

---

## PERFORM TAB

### Structure

(Unchanged from V4_4_0 - see System Reference for full details)

```
┌───────────────────────────────────────────────────────────────┐
│ [PERFORM                                                    ] │
├───────────────────────────────────────────────────────────────┤
│ ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐            │
│ │  1  │  2  │  3  │  4  │  5  │  6  │  7  │  8  │ ← Scenes   │
│ ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤            │
│ │     │     │     │     │     │     │     │     │ Track 1    │
│ │     │     │     │     │     │     │     │     │ Track 2    │
│ │     │     │     │     │     │     │     │     │ Track 3    │
│ │     │     │     │     │     │     │     │     │ Track 4    │
│ └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘            │
│                                                               │
│ [Waveform displays]                                           │
│                                                               │
│ [Mixer controls]                                              │
└───────────────────────────────────────────────────────────────┘
```

---

## ADMIN TAB

### Structure

```
┌─────────────────────────────────────┬───────────────────────────────────────┐
│            LEFT PANEL               │            RIGHT PANEL                │
├─────────────────────────────────────┼───────────────────────────────────────┤
│ [ADMIN                            ] │                                       │
├─────────────────────────────────────┤                                       │
│                                     │                                       │
│ ○ Settings                          │  [Content for selected section]       │
│ ○ Utilities                         │                                       │
│ ○ About                             │                                       │
│                                     │                                       │
└─────────────────────────────────────┴───────────────────────────────────────┘
```

### Navigation

```javascript
function adminNavSelect(section) {
    // Remove active from all nav items
    document.querySelectorAll('.admin-nav-item').forEach(item => {
        item.classList.remove('active');
    });
    
    // Hide all content sections
    document.getElementById('adminSettingsContent').style.display = 'none';
    document.getElementById('adminUtilitiesContent').style.display = 'none';
    document.getElementById('adminAboutContent').style.display = 'none';
    
    // Show selected section
    switch(section) {
        case 'settings':
            document.getElementById('adminSettingsContent').style.display = 'block';
            document.querySelector('[data-section="settings"]').classList.add('active');
            break;
        case 'utilities':
            document.getElementById('adminUtilitiesContent').style.display = 'block';
            document.querySelector('[data-section="utilities"]').classList.add('active');
            break;
        case 'about':
            document.getElementById('adminAboutContent').style.display = 'block';
            document.querySelector('[data-section="about"]').classList.add('active');
            break;
    }
}
```

### Settings Content

```html
<div id="adminSettingsContent">
    <!-- Application Folder -->
    <div class="admin-field">
        <label>Application Folder</label>
        <span id="adminApplicationFolderName">Not set</span>
        <button onclick="setupApplicationFolder()">Set Folder</button>
    </div>
    
    <!-- Project State -->
    <div class="admin-field">
        <label>Project State</label>
        <select id="adminProjectStateSelect">...</select>
    </div>
    
    <!-- Greeting Text -->
    <div class="admin-field">
        <label>Greeting Text</label>
        <input id="adminGreetingText" type="text">
    </div>
</div>
```

---

## AUDIO CONTROLS STRIP

### Layout

```
┌──────────┬─────────────────┬─────────────┬──────────────────┬─────────────────┐
│Transport │ Audio Start|End │ Tempo|Pitch │ Loop In|Out|Loop │ Beat Grid    |X │
├──────────┼─────────────────┼─────────────┼──────────────────┼─────────────────┤
│ [Play]   │ [00:00] [00:00] │ [70px][50px]│ [50px][50px][50px│ [G][S][field]|[X]
└──────────┴─────────────────┴─────────────┴──────────────────┴─────────────────┘
```

### Cell Definitions

| Cell | Label | Contents | Width |
|------|-------|----------|-------|
| 1 | Transport | Play button | 120px fixed |
| 2 | Audio Start / Audio End | Time input fields | flex |
| 3 | Tempo / Pitch | 70px + 50px inputs | auto |
| 4 | Loop In / Loop Out / (toggle) | 50px + 50px + 50px | auto |
| 5 | Beat Grid | G, S, field, divider, X | 200px fixed |

### Beat Grid Cell

```html
<div class="audio-controls-cell" style="width: 200px;">
    <div class="cell-top">
        <label class="cell-label">Beat Grid</label>
    </div>
    <div class="cell-bottom">
        <button id="audioGridBtn" class="btn-square" onclick="toggleBeatGrid()">G</button>
        <button id="audioSnapBtn" class="btn-square inactive" onclick="snapToBar()" disabled>S</button>
        <input id="audioSnapLastBarField" type="text" readonly style="width: 40px;">
        <div class="cell-divider"></div>
        <button id="audioRemoveBtn" class="btn-square inactive" onclick="removeAudio()" disabled>X</button>
    </div>
</div>
```

### Button State Management

```javascript
function updateAudioButtonStates(hasAudio) {
    const snapBtn = document.getElementById('audioSnapBtn');
    const removeBtn = document.getElementById('audioRemoveBtn');
    
    if (hasAudio) {
        snapBtn.disabled = false;
        snapBtn.classList.remove('inactive');
        removeBtn.disabled = false;
        removeBtn.classList.remove('inactive');
    } else {
        snapBtn.disabled = true;
        snapBtn.classList.add('inactive');
        removeBtn.disabled = true;
        removeBtn.classList.add('inactive');
    }
}
```

---

## CSS CLASSES REFERENCE

### Tab Classes

```css
.tab { /* Base tab button */ }
.tab.active { /* Active tab */ }
.tab-cyan { /* CREATE tab color */ }
.tab-yellow { /* ORGANIZE tab color */ }
.tab-magenta { /* PERFORM tab color */ }
.tab-grey { /* ADMIN tab color */ }
```

### Button Classes

```css
.toolbar-btn { /* Library toolbar buttons */ }
.toolbar-btn.active { /* Active view mode */ }
.toolbar-btn.danger { /* Delete button - red hover */ }
.btn-square { /* 32px square buttons (G, S, X) */ }
.inactive { /* Disabled appearance */ }
.info-drawer-swap-btn { /* Swap/Remove buttons in info drawer */ }
```

### Container Classes

```css
.library-container-card { /* Base card */ }
.library-container-card.selected { /* Selected state */ }
.library-container-card.loaded { /* Loaded state - yellow */ }
```

### Info Drawer Classes

```css
.library-info-drawer { /* Drawer container */ }
.info-drawer-header { /* Header with close button */ }
.info-drawer-content { /* Field container */ }
.info-drawer-footer { /* Cancel/OK buttons */ }
.info-field { /* Individual field row */ }
```

### ADMIN Classes

```css
.admin-nav { /* Navigation container */ }
.admin-nav-item { /* Nav item */ }
.admin-nav-item.active { /* Active nav item */ }
.admin-section { /* Content section */ }
.admin-section-header { /* Section header */ }
.admin-field { /* Field row */ }
```

---

## ELEMENT IDS REFERENCE

### Tabs

| ID | Element | Purpose |
|----|---------|---------|
| controlsTab | Button | CREATE tab |
| libraryTab | Button | ORGANIZE tab |
| liveTab | Button | PERFORM tab |
| adminTab | Button | ADMIN tab |
| controlsTabContent | Div | CREATE content |
| libraryTabContent | Div | ORGANIZE content |
| liveTabContent | Div | PERFORM content |
| adminTabContent | Div | ADMIN content |

### CREATE Tab

| ID | Element | Purpose |
|----|---------|---------|
| shaderControlsContent | Div | Controls view container |
| shaderCodeContent | Div | Code view container |
| codeToggleBtn | Button | Code/Controls toggle |
| shaderCode | Textarea | Shader code (single source) |

### ORGANIZE Tab

| ID | Element | Purpose |
|----|---------|---------|
| librarySearchInput | Input | Search field |
| libraryContainersList | Div | Container cards container |
| libraryInfoDrawer | Div | Info drawer panel |
| libraryLoadBtn | Button | Load button |
| libraryUpdateBtn | Button | Update button |
| libraryExportBtn | Button | Export button |
| libraryAddBtn | Button | Add button |
| libraryDeleteBtn | Button | Delete button |
| libraryInfoBtn | Button | Info drawer toggle |

### ADMIN Tab

| ID | Element | Purpose |
|----|---------|---------|
| adminSettingsContent | Div | Settings section content |
| adminUtilitiesContent | Div | Utilities section content |
| adminAboutContent | Div | About section content |
| adminApplicationFolderName | Span | Folder name (synced) |
| adminProjectStateSelect | Select | Project dropdown (synced) |

### Audio Controls

| ID | Element | Purpose |
|----|---------|---------|
| audioPlayBtn | Button | Play/pause |
| audioLoopBtn | Button | Loop toggle |
| audioTempoField | Input | BPM (70px) |
| audioPitchField | Input | Pitch (50px) |
| audioStartField | Input | Audio start time |
| audioEndField | Input | Audio end time |
| audioLoopInField | Input | Loop in point |
| audioLoopOutField | Input | Loop out point |
| audioGridBtn | Button | Grid toggle (G) |
| audioSnapBtn | Button | Snap to bar (S) |
| audioRemoveBtn | Button | Remove audio (X) |
| audioSnapLastBarField | Input | Last bar display |

---

## FUNCTIONS REFERENCE

### Tab Functions

```javascript
switchTab(tabName)         // Switch between tabs
toggleCodeView()           // Toggle Controls ↔ Code view
adminNavSelect(section)    // Switch ADMIN content section
```

### Library Functions

```javascript
filterLibraryList(term)    // Filter containers by search term
toggleInfoDrawer()         // Open/close info drawer
applyInfoChanges()         // Apply pending info changes
cancelInfoChanges()        // Discard pending changes
handleDelete()             // Handle delete with confirmation
```

### Audio Functions

```javascript
updateAudioButtonStates(hasAudio)  // Update G/S/X button states
toggleBeatGrid()                   // Toggle grid visibility
snapToBar()                        // Snap audio end to bar
removeAudio()                      // Remove audio from container
```

### Sync Functions

```javascript
updateFolderDisplay(name)          // Update folder display (both panels)
updateProjectStateSelect(options)  // Update project dropdown (both panels)
```

---

## KEYBOARD SHORTCUTS

### Tab Switching

| Key | Action |
|-----|--------|
| Ctrl+1 | CREATE |
| Ctrl+2 | ORGANIZE |
| Ctrl+3 | PERFORM |
| Ctrl+4 | ADMIN |

### Library (ORGANIZE)

| Key | Action |
|-----|--------|
| Cmd+L | Load full container |
| Cmd+S | Swap shader |
| Cmd+Shift+S | Remove shader |
| Cmd+A | Swap audio |
| Cmd+Shift+A | Remove audio |
| Cmd+U | Update container |
| Cmd+E | Export container |
| Cmd+I | Toggle info drawer |
| ↑/↓ | Navigate list |
| Enter | Load container |
| Delete | Delete (with confirmation) |

### Audio

| Key | Action |
|-----|--------|
| Space | Play/pause |
| ←/→ | Navigate by beat |
| G | Toggle grid |

---

**END OF DEVELOPER DOCUMENTATION**

Version: Beta V4_5_0
Date: December 10, 2025
License: GPL-3.0
