# Pull Request Changes Summary

## Overview
This pull request adds four main enhancements to the GeoGebra MCP server:
1. **Direct file-saving capabilities** for PNG, SVG, PDF, and GGB exports
2. **New GGB file export** to save constructions as native GeoGebra files
3. **New GGB file import** to load constructions from .ggb files
4. **Auto-zoom feature** to automatically fit all visible objects in the view

## Files Modified

### 1. `src/tools/geogebra-tools.ts`

#### Added Imports (Lines 1-6)
```typescript
import fs from 'fs';
import path from 'path';
```

#### PNG Export Tool (Lines 658-795)
- **Added parameter**: `filename` (optional string)
- **New behavior**: When `filename` is provided, saves PNG to file instead of returning base64
- **File path resolution**: Uses `process.env['VSCODE_WORKSPACE_FOLDER']` or `process.cwd()`
- **Backward compatible**: Without `filename`, returns base64 data as before

#### SVG Export Tool (Lines 797-888)
- **Added parameter**: `filename` (optional string)
- **New behavior**: When `filename` is provided, saves SVG to file instead of returning SVG string
- **File path resolution**: Uses `process.env['VSCODE_WORKSPACE_FOLDER']` or `process.cwd()`
- **Backward compatible**: Without `filename`, returns SVG data as before

#### PDF Export Tool (Lines 956-1008)
- **Added parameter**: `filename` (optional string)
- **Updated description**: Now mentions "optional direct file saving"
- **New behavior**: When `filename` is provided, saves PDF to file instead of returning base64
- **File path resolution**: Uses `process.env['VSCODE_WORKSPACE_FOLDER']` or `process.cwd()`
- **Backward compatible**: Without `filename`, returns base64 data as before

#### GGB Export Tool (Lines 1367-1504) - **NEW**
- **New tool**: `geogebra_export_ggb`
- **Description**: Export construction as native GeoGebra file (.ggb)
- **Parameter**: `filename` (optional string)
- **Behavior**:
  - With `filename`: Saves .ggb file to disk
  - Without `filename`: Returns base64-encoded .ggb data
- **File path resolution**: Uses `process.env['VSCODE_WORKSPACE_FOLDER']` or `process.cwd()`
- **Use case**: Save constructions that can be opened in GeoGebra desktop or web app

#### GGB Load Tool (Lines 1507-1563) - **NEW**
- **New tool**: `geogebra_load_ggb`
- **Description**: Load a GeoGebra construction from a .ggb file or base64 data
- **Parameter**: `source` (required string)
  - Can be a file path (e.g., "construction.ggb")
  - Can be base64-encoded .ggb data
- **Behavior**:
  - Detects if source is a file path or base64 data
  - Reads file if path provided
  - Loads construction into GeoGebra using `setBase64()`
  - Returns list of loaded objects
- **File path resolution**: Uses `process.env['VSCODE_WORKSPACE_FOLDER']` or `process.cwd()`
- **Use case**: Quickly reload saved constructions without recreating objects

#### Auto-Zoom Tool (Lines 660-789) - **NEW**
- **New tool**: `geogebra_auto_zoom`
- **Description**: Automatically calculate and set optimal view to fit all visible objects
- **Parameters**:
  - `padding` (optional number): Padding percentage around objects (default: 0.2 for 20% margin)
  - `minRange` (optional number): Minimum range for each axis (default: 2)
- **Behavior**:
  - Gets all visible point objects
  - Calculates bounding box from point coordinates
  - Adds configurable padding
  - Sets coordinate system to fit all objects
- **Returns**: Success message with calculated view range and bounding box
- **Use case**: Automatically focus view on relevant construction elements

#### All Export Tools Enhanced with AutoZoom
All export tools now support optional auto-zoom functionality:

**PNG Export Tool** (Lines 842-854, 900-951)
**SVG Export Tool** (Lines 1070-1082, 1088-1157)
**PDF Export Tool** (Lines 1235-1247, 1248-1317)
**GGB Export Tool** (Lines 1374-1386, 1387-1456)

- **Added parameters** (to all export tools):
  - `autoZoom` (optional boolean): Automatically zoom to fit all visible objects before exporting (default: false)
  - `zoomPadding` (optional number): Padding percentage for auto-zoom (default: 0.2 for 20% margin)
- **New behavior**: When `autoZoom` is true, automatically calculates optimal view before export
- **Smart logic**:
  - PNG/SVG: Auto-zoom only activates if manual xmin/xmax/ymin/ymax are not specified
  - PDF/GGB: Auto-zoom activates when enabled (no manual coordinate options)
- **Backward compatible**: Default behavior unchanged (autoZoom defaults to false)
- **Consistent API**: Same parameters across all export formats

### 2. `src/utils/geogebra-instance.ts`

#### GGB Export Method (Lines 1058-1113) - **NEW**
- **New method**: `exportGGB()`
- **Implementation**: Uses GeoGebra's native `getBase64()` API method
- **Returns**: Base64-encoded .ggb file
- **Validation**: Ensures valid base64 format and reasonable file size
- **Error handling**: Comprehensive validation with retry mechanism

#### GGB Load Method (Lines 1116-1173) - **NEW**
- **New method**: `loadGGB(ggbBase64OrPath: string)`
- **Implementation**: Uses GeoGebra's native `setBase64()` API method
- **Parameters**: Accepts either file path or base64-encoded .ggb data
- **Behavior**:
  - Auto-detects if input is file path (contains `/`, `\`, or ends with `.ggb`)
  - Reads file from disk if path provided
  - Converts to base64 if needed
  - Loads construction using `applet.setBase64()`
- **File path resolution**: Resolves relative to workspace or current directory
- **Error handling**: Validates base64 format, checks file existence
- **Use case**: Quickly restore saved constructions

## Key Features

### Direct File Saving
- **Optional feature**: Only activates when `filename` parameter is provided
- **Workspace-aware**: Resolves paths relative to VS Code workspace or current directory
- **Format support**: PNG (binary), SVG (UTF-8 text), PDF (binary), GGB (binary)
- **Response format**: When saving to file, returns success message with full file path

### GGB File Export
- **Native format**: Uses GeoGebra's `getBase64()` API to export constructions
- **Portable**: .ggb files can be opened in GeoGebra desktop, web app, or mobile apps
- **Complete**: Includes all objects, settings, and construction history
- **Convenient**: Direct file saving eliminates need for manual base64 decoding

### GGB File Import
- **Flexible input**: Accepts file paths or base64-encoded .ggb data
- **Auto-detection**: Automatically detects if input is a file path or base64 data
- **Native loading**: Uses GeoGebra's `setBase64()` API to load constructions
- **Fast workflow**: Quickly restore saved constructions without recreating objects
- **Workspace-aware**: Resolves file paths relative to VS Code workspace or current directory

## Backward Compatibility

All changes are **100% backward compatible**:
- Existing code without `filename` parameter works exactly as before
- Return values unchanged when `filename` is not provided
- No breaking changes to API signatures

## Testing Recommendations

1. **PNG Export**:
   - Test with `filename`: Should save file and return path
   - Test without `filename`: Should return base64 data
   - Test with `autoZoom`: Should automatically fit all objects in view
   
2. **SVG Export**:
   - Test with `filename`: Should save file and return path
   - Test without `filename`: Should return SVG string
   - Note: May fail with some GeoGebra versions due to API limitations
   
3. **PDF Export**:
   - Test with `filename`: Should save file and return path
   - Test without `filename`: Should return base64 data
   - Test with `autoZoom`: Should automatically fit all objects in view

4. **GGB Export**:
   - Test with `filename`: Should save .ggb file and return path
   - Test without `filename`: Should return base64 data
   - Test with `autoZoom`: Should save with optimal view settings
   - Verify saved .ggb file can be opened in GeoGebra

5. **GGB Import**:
   - Test with file path: Should load construction from .ggb file
   - Test with base64 data: Should load construction from base64
   - Verify all objects are loaded correctly
   - Test with relative and absolute paths

## Example Usage

### Before (returns data):
```typescript
// PNG
const result = await geogebra_export_png({ scale: 2 });
// Returns: { success: true, data: "base64...", encoding: "base64" }

// SVG
const result = await geogebra_export_svg({});
// Returns: { success: true, data: "<svg>...</svg>", encoding: "utf8" }
```

### After (with file saving):
```typescript
// PNG - save to file
const result = await geogebra_export_png({ filename: "output.png", scale: 2 });
// Returns: { success: true, saved: true, filename: "/full/path/output.png" }

// SVG - save to file
const result = await geogebra_export_svg({ filename: "diagram.svg" });
// Returns: { success: true, saved: true, filename: "/full/path/diagram.svg" }

// GGB - save to file (NEW!)
const result = await geogebra_export_ggb({ filename: "construction.ggb" });
// Returns: { success: true, saved: true, filename: "/full/path/construction.ggb" }

// GGB - load from file (NEW!)
const result = await geogebra_load_ggb({ source: "construction.ggb" });
// Returns: {
//   success: true,
//   message: "GGB file loaded successfully",
//   objectCount: 8,
//   objects: ["A", "B", "C", "lineAB", "perp", "D", "a", "b"]
// }

// GGB - load from base64 (NEW!)
const result = await geogebra_load_ggb({ source: "UEsDBBQACAgIAA..." });
// Returns: { success: true, message: "GGB file loaded successfully", objectCount: 8, objects: [...] }

// Still works without filename (backward compatible)
const result = await geogebra_export_png({ scale: 2 });
// Returns: { success: true, data: "base64...", encoding: "base64" }

// Auto-zoom before export (NEW!)
const result = await geogebra_export_png({
  filename: "focused.png",
  autoZoom: true,
  zoomPadding: 0.15  // 15% padding
});
// Returns: { success: true, saved: true, filename: "/full/path/focused.png" }
// View automatically adjusted to fit all objects before export

// Manual auto-zoom (NEW!)
const result = await geogebra_auto_zoom({ padding: 0.2 });
// Returns: {
//   success: true,
//   pointsAnalyzed: 4,
//   boundingBox: { minX: -3, maxX: 4, minY: 0, maxY: 9 },
//   viewRange: { xmin: -4.4, xmax: 5.4, ymin: -1.8, ymax: 10.8 },
//   padding: "20%"
// }
```

## Benefits

1. **Convenience**: Users can directly save exports without manual base64 decoding
2. **Efficiency**: Reduces data transfer for large exports
3. **Flexibility**: Optional parameters maintain backward compatibility
4. **User-friendly**: Automatic workspace path resolution
5. **Native format support**: GGB export/import allows saving and loading constructions in GeoGebra's native format
6. **Portability**: GGB files can be shared and opened across all GeoGebra platforms
7. **Fast workflow**: Load saved constructions instantly without recreating objects
8. **Smart visualization**: Auto-zoom automatically focuses on relevant construction elements
9. **Configurable padding**: Adjustable margins ensure objects aren't cut off at edges
10. **Export optimization**: Auto-zoom ensures all objects are visible in saved images
11. **Flexible input**: GGB load accepts both file paths and base64 data

## Code Quality Improvements (GitHub Copilot Feedback)

After the initial PR submission, GitHub Copilot identified several code quality and security issues. All critical and medium priority issues have been addressed:

### Security Enhancements
1. **Path Traversal Protection**: Added `validateFilePath()` method to prevent directory traversal attacks in all file operations
2. **Async File Operations**: Replaced synchronous file operations (`fs.readFileSync`, `fs.writeFileSync`) with async versions (`fs.promises.readFile`, `fs.promises.writeFile`) to prevent event loop blocking

### Code Quality Improvements
3. **Eliminated Code Duplication**: Refactored auto-zoom logic into reusable helper methods (`calculateAutoZoomCoordinates()`, `applyAutoZoom()`), reducing ~275 lines of duplicated code to ~90 lines (67% reduction)
4. **Consistent Error Handling**: Standardized error handling pattern across all tools to use `error instanceof Error ? error.message : String(error)`
5. **Configurable minRange**: Added `minRange` parameter to all export tools for consistency with standalone auto-zoom tool

See [`GITHUB_COPILOT_FIXES.md`](./GITHUB_COPILOT_FIXES.md) for detailed information about these improvements.

## Notes

- All file operations now use async Node.js `fs.promises` API
- File paths are validated to prevent directory traversal attacks
- File paths are resolved relative to workspace or current directory
- TypeScript compilation required before changes take effect
- No changes to package dependencies required
