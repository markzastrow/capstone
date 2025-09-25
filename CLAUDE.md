# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CAPSTONE is a single-file HTML application for magazine editors that manages images and captions with dynamic reference numbering. The entire application is contained in `index.html` with embedded CSS and JavaScript - no build process required.

**Print Workflow Support**: The application now fully supports TIFF files (.tif, .tiff), which are essential for professional print workflows, in addition to standard web image formats (JPEG, PNG, GIF, WebP, SVG).

## Development Commands

This is a static HTML application - no build, test, or lint commands are needed:

- **Run locally**: Open `index.html` in any web browser
- **Test functionality**: Load the file and test drag/drop, reference updates, export functionality
- **No package.json or dependencies** - everything is self-contained

## Architecture Overview

### Single-File Structure
- **`index.html`** - Complete application (HTML + CSS + JavaScript)
- **External dependencies**:
  - JSZip loaded via CDN for ZIP export functionality
  - UTIF.js v3.1.0 loaded via CDN for TIFF decoding support
- **No build process** - edit the HTML file directly

### Key Technical Patterns

**Dual-Layer Text Editor**
- Transparent textarea overlays styled display div
- Enables smart reference highlighting while maintaining text input functionality
- Located in lines ~334-374 (CSS) and ~954-1047 (JavaScript)

**ArrayBuffer File Storage**
- Files stored as ArrayBuffer in IndexedDB for reliable persistence
- Reconstructs File objects on load to avoid blob corruption
- Enhanced for TIFF files with on-demand decoding workflow
- See `createThumbnail()` (~776-907) and `loadAutoSavedProject()` (~1415-1441)

**Event Delegation System**
- Single parent listeners handle all drag/drop events for performance
- Avoids listener management during DOM updates
- Implemented in `setupImageListDragHandlers()` (~883-952)

**Dynamic Reference System**
- `_01`, `_02` patterns in text automatically update when images reordered
- Blue highlighting for valid references, red for broken/missing
- Core logic in `parseAndUpdateReferences()` (~968-992)

**TIFF Processing System**
- Automatic detection of TIFF files by MIME type (`image/tiff`) and file extension (`.tif`, `.tiff`)
- Uses UTIF.js library to decode TIFF files to raw RGBA pixel data
- Dual processing: thumbnail generation (60x60) + full-resolution lightbox display
- Graceful error handling with fallbacks for corrupted or unsupported TIFF variants
- Memory efficient: decodes TIFF data only when needed, discards after processing

### Critical Functions

**Image Management**
- `addImages()` - Processes files, creates thumbnails, updates references (supports TIFF)
- `createThumbnail()` - Converts to ArrayBuffer + generates 60x60 canvas thumbnails (with TIFF decoding via UTIF.js)
- `deleteImage()` - Removes image, marks references as `!!`
- `updateFilenames()` - Renames based on story code or sequential pattern

**TIFF-Specific Functions**
- `showLightboxImage()` - Enhanced lightbox display with TIFF decoding for full-resolution viewing (~1155-1201)
- TIFF detection logic integrated into file processing pipeline
- Error handling and fallback systems for TIFF processing failures

**Reference System**
- `updateTextReferencesAfterReorder()` - Updates `_01`, `_02` when images move
- `updateTextDisplay()` - Applies blue/red highlighting
- `parseAndUpdateReferences()` - Finds reference patterns in text

**Persistence**
- `autoSave()` - Triggers IndexedDB save with ArrayBuffer data
- `performSave()` - Actual write operation, saves projects with or without images
- `loadAutoSavedProject()` - Reconstructs File objects from stored ArrayBuffer
- Auto-saves text-only projects (no images required for persistence)

## Development Guidelines

### Making Changes
1. **Edit `index.html` directly** - all code is embedded
2. **Test in browser immediately** - no compilation step needed
3. **Preserve ArrayBuffer storage** - don't break file persistence system
4. **Maintain event delegation** - avoid individual listeners on dynamic elements
5. **Keep single-file architecture** - resist urge to split into multiple files
6. **TIFF considerations** - always include error handling and fallbacks for TIFF operations

### Common Editing Areas
- **Styling**: `<style>` section (~lines 8-523)
- **JavaScript**: `<script>` section (~lines 567-1507)
- **HTML structure**: Body content (~lines 525-566)

### TIFF Development Guidelines

**Error Handling Patterns**
- Always wrap TIFF operations in try-catch blocks
- Provide graceful fallbacks for decode failures
- Log errors to console for debugging
- Never crash the app due to TIFF processing issues

**Performance Considerations**
- TIFF decoding is CPU-intensive (~100-500ms per file)
- Memory usage spikes during decode (temp canvas creation)
- Decode only when needed (thumbnails + lightbox display)
- Clean up temporary canvases and ImageData objects

**TIFF Detection Logic**
```javascript
const isTiff = file.type === 'image/tiff' ||
              file.name.toLowerCase().endsWith('.tif') ||
              file.name.toLowerCase().endsWith('.tiff');
```

**Memory Management**
- Use temporary canvases for TIFF processing
- Convert to data URLs quickly and discard raw pixel data
- Monitor for memory leaks during repeated TIFF operations
- UTIF.js handles internal cleanup automatically

### Testing Checklist
- Drag/drop images works (including TIFF files)
- Text references (`_01`, `_02`) highlight correctly
- Reordering updates references properly
- Export generates correct ZIP (including TIFF files)
- Auto-save persists across browser refresh (including text-only projects)
- Lightbox navigation functions (including TIFF display)

**TIFF-Specific Testing**
- TIFF file upload via file picker and drag-and-drop
- TIFF thumbnail generation (60x60 with proper cropping)
- TIFF lightbox display (full-resolution, decoded rendering)
- TIFF error handling (corrupted files, unsupported variants)
- TIFF memory management (no memory leaks during processing)
- Mixed workflows (TIFF + standard formats in same project)

### Current Status
The application is production-ready with full TIFF support for print workflows. The backup file `CLAUDE_bak.md` contains detailed development history. All major issues have been resolved, and the interface achieves optimal balance of modern polish with practical usability.

**Latest Enhancement**: Added comprehensive TIFF file support via UTIF.js integration, enabling seamless handling of professional print formats while maintaining the single-file architecture and performance characteristics.