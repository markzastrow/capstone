# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CAPSTONE is a single-file HTML application for magazine editors that manages images and captions with dynamic reference numbering. The entire application is contained in `index.html` with embedded CSS and JavaScript - no build process required.

## Development Commands

This is a static HTML application - no build, test, or lint commands are needed:

- **Run locally**: Open `index.html` in any web browser
- **Test functionality**: Load the file and test drag/drop, reference updates, export functionality
- **No package.json or dependencies** - everything is self-contained

## Architecture Overview

### Single-File Structure
- **`index.html`** - Complete application (HTML + CSS + JavaScript)
- **External dependency**: JSZip loaded via CDN for ZIP export functionality
- **No build process** - edit the HTML file directly

### Key Technical Patterns

**Dual-Layer Text Editor**
- Transparent textarea overlays styled display div
- Enables smart reference highlighting while maintaining text input functionality
- Located in lines ~334-374 (CSS) and ~954-1047 (JavaScript)

**ArrayBuffer File Storage**
- Files stored as ArrayBuffer in IndexedDB for reliable persistence
- Reconstructs File objects on load to avoid blob corruption
- See `createThumbnail()` (~767-814) and `loadAutoSavedProject()` (~1285-1311)

**Event Delegation System**
- Single parent listeners handle all drag/drop events for performance
- Avoids listener management during DOM updates
- Implemented in `setupImageListDragHandlers()` (~883-952)

**Dynamic Reference System**
- `_01`, `_02` patterns in text automatically update when images reordered
- Blue highlighting for valid references, red for broken/missing
- Core logic in `parseAndUpdateReferences()` (~968-992)

### Critical Functions

**Image Management**
- `addImages()` - Processes files, creates thumbnails, updates references
- `createThumbnail()` - Converts to ArrayBuffer + generates 60x60 canvas thumbnails
- `deleteImage()` - Removes image, marks references as `!!`
- `updateFilenames()` - Renames based on story code or sequential pattern

**Reference System**
- `updateTextReferencesAfterReorder()` - Updates `_01`, `_02` when images move
- `updateTextDisplay()` - Applies blue/red highlighting
- `parseAndUpdateReferences()` - Finds reference patterns in text

**Persistence**
- `autoSave()` - Triggers IndexedDB save with ArrayBuffer data
- `performSave()` - Actual write operation
- `loadAutoSavedProject()` - Reconstructs File objects from stored ArrayBuffer

## Development Guidelines

### Making Changes
1. **Edit `index.html` directly** - all code is embedded
2. **Test in browser immediately** - no compilation step needed
3. **Preserve ArrayBuffer storage** - don't break file persistence system
4. **Maintain event delegation** - avoid individual listeners on dynamic elements
5. **Keep single-file architecture** - resist urge to split into multiple files

### Common Editing Areas
- **Styling**: `<style>` section (~lines 8-523)
- **JavaScript**: `<script>` section (~lines 563-1377)
- **HTML structure**: Body content (~lines 525-562)

### Testing Checklist
- Drag/drop images works
- Text references (`_01`, `_02`) highlight correctly
- Reordering updates references properly
- Export generates correct ZIP
- Auto-save persists across browser refresh
- Lightbox navigation functions

### Current Status
The application is production-ready. The backup file `CLAUDE_bak.md` contains detailed development history. All major issues have been resolved, and the interface achieves optimal balance of modern polish with practical usability.