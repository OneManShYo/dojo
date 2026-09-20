# OMS DOJO V2_9 - PROJECT INSTRUCTIONS

**Version:** V2_9
**Date:** November 2025
**Purpose:** Complete project context for Claude AI development

This document provides comprehensive context for AI-assisted development of ONEMANSHYO Dojo and the broader OMS ecosystem.

For the complete, detailed project instructions including:
- About Wes Smith (Creator Context)
- OneManShYo Ableton Edition (Current Workflow)
- OMS Ecosystem Overview
- ONEMANSHYO Dojo architecture
- LIVE Tab Roadmap
- Development Guidelines
- Communication Protocols

Please refer to the comprehensive project instructions document uploaded separately, which contains:
- Full workflow documentation
- Development session rules  
- Iteration guidelines
- Communication protocols
- Future roadmap through V3.0.0

**Key Points for V2_9:**

## LIBRARY-First Architecture

V2_9 enforces LIBRARY folder for all saves:
- Two modes: LIBRARY set (persistent) vs not set (temp session)
- Auto-save .oms files to LIBRARY folder
- Container name = .oms filename (1:1 match)
- Control States save as {Name}_ControlState.oms
- No localStorage dependency for user data

## Development Process

**Iteration Rules:**
1. One feature per iteration (surgical changes)
2. Alphabetical naming (A-Z, then AA-AG, etc.)
3. Ask before building (numbered questions, lettered options)
4. Wait for "yes go" or specific answer
5. Update version display in HTML (title tag + UI header)

**Version Format:**
- Production: V2_9 (periods, no letter)
- Development: V2_9_0_A (underscores, letter suffix)

## File Structure

**Single-File Philosophy:**
- All functionality in one HTML file (~272KB)
- No external dependencies
- Portable, offline-capable

**Deliverables Package (9 files):**
1. OMS_Dojo_V2_9.html
2. _A_Deliverables.txt
3. _B_Complete_Changelog.md
4. _C_Developer_Documentation.md
5. _D_Release_Notes.md
6. _E_Claude_ShaderCreation_Guide.md
7. _F_UserGuide_All.md
8. _G_UserGuide_FFmpeg_Video_Guide.md
9. _Z_Project_Instructions.md (this file)

## Next Steps (V2.10.0+)

**LIVE Tab Integration:**
- Add 5th LIVE tab to Dojo
- Performance grid (4 tracks × 8 scenes)
- Mixer section (4 tracks + master)
- Real-time shader rendering (4 simultaneous)
- Real-time audio mixing
- Load containers from LIBRARY into grid

## Communication

**Wes's Style:**
- Direct, efficient, token-conscious
- Numbered questions with lettered options
- "yes go" means proceed immediately
- Expects exact technical language

**Do NOT Suggest:**
- Business models, pricing, revenue
- "This could be a business"
- Max for Live versions
- Server-side processing

## Critical Reminders

**NEVER:**
- Skip iteration letters
- Forget to update version display
- Embed changelog in HTML
- Use periods in development version names

**ALWAYS:**
- Ask before building
- Use underscores in V2_X_X format
- Single-file architecture
- Comprehensive iteration notes

---

For complete project context, workflows, and detailed guidelines, refer to the comprehensive project instructions document.

**END OF PROJECT INSTRUCTIONS**
