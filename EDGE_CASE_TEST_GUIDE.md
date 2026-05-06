# Edge Case Validation Guide

## Overview
Three specific edge cases need validation to ensure no orphan line segments remain in SVG exports or degenerate geometry in STL exports:

1. **Scenario A**: 3x mounting holes, rotation 60°, split 1/3  
   - Tests holes that straddle the sector boundary
   
2. **Scenario B**: Mounting position 110mm, rotation 0°, split 1/3  
   - Tests holes positioned at radial boundaries (curved radii)
   
3. **Scenario C**: Bahtinov offset 90°, split 1/6  
   - Tests slits positioned at offset boundary

## Quick Start

### 1. Console Geometry Validation (No Exports)
Open the HTML file in a browser and run in Developer Console (F12):

```javascript
testAllScenarios()
```

This checks geometry BEFORE export clipping.

### 2. Full Export Analysis (Recommended)
For each scenario, run:

```javascript
// Scenario A
exportAndAnalyze('A')

// Or B or C
exportAndAnalyze('B')
exportAndAnalyze('C')
```

This will:
- Set up the scenario
- Generate both SVG and STL
- Analyze for orphan edges/faces
- List any suspicious structures

Expected output shows:
```
VERDICT:
✓ NO ISSUES DETECTED - exports appear clean
```

### 3. Visual Inspection with Interactive Tools

#### Step 1: Generate exports from scenario
```javascript
exportAndAnalyze('A')
// Downloads will start automatically (allow popups)
```

#### Step 2: Open the visual inspector
- Open file: `test/export_visualizer.html` in your browser
- This opens a 3-panel viewer

#### Step 3: Load and inspect files
**Left Panel (SVG Analysis):**
1. Click "Load SVG File"
2. Select the downloaded `3Bahtinov.svg`
3. Red-highlighted edges = suspicious boundary edges
4. Click on any path to highlight it
5. View all path details in the list below

**Middle Panel (STL Analysis):**
1. Click "Load STL File"
2. Select the downloaded `3Bahtinov_3D_scanline.stl`
3. Click on any facet in the 3D view to identify it
4. The info box shows: Facet ID, Type (top/bottom/side), Normal vector
5. Red facets = degenerate (normal=[0,0,0])
6. Use "Color by Type" button to see different face categories

**Right Panel (Report):**
1. Click "Run Full Analysis" to generate comprehensive report
2. Shows matching issues between SVG and STL
3. Lists specific problem edges/facets with IDs

#### Step 4: Report and Identify Issues
- **Facet ID**: Number assigned by STL exporter (0-indexed)
- **Edge**: Path number in SVG (also 0-indexed)
- **Type**: Whether it's a boundary cut, slit edge, or hole edge
- **Location**: Normally shown in the visualizer

## Test Commands Quick Reference

```javascript
// Geometry tests (console only)
testScenario('A')          // Test Scenario A geometry
testScenario('B')          // Test Scenario B geometry
testScenario('C')          // Test Scenario C geometry
testAllScenarios()         // Run all three

// Export analysis (console, with file generation)
exportAndAnalyze('A')      // Full export analysis for Scenario A
exportAndAnalyze('B')      // Full export analysis for Scenario B
exportAndAnalyze('C')      // Full export analysis for Scenario C
fullExportComparison()     // Analyze current SVG/STL in page

// Direct export analysis (current page state)
analyzeExportedSVG()       // List all SVG paths, find suspicious short ones
analyzeExportedSTL()       // List all STL facets, find degenerate ones
```

## What Each Analysis Checks

### Console Geometry Validation (`testScenario()`)
- Checks hole geometry BEFORE clipping
- Identifies boundary edges that SHOULD be suppressed
- Verifies they ARE marked in `computeSuppressedSideEdges()`
- **Limitation**: Doesn't catch issues created during export/clipping

### Full Export Analysis (`exportAndAnalyze()`)
- Generates fresh SVG and STL
- **SVG**: Lists short paths (<150 chars) as orphan candidates
- **STL**: Counts facet types, checks for degenerate facets
- **Comparison**: Verifies top/bottom face counts match
- **Best for**: Catching clipping artifacts

### Visual Inspector (`test/export_visualizer.html`)
- Interactive 3D view of STL (top view projection)
- Clickable edges in SVG, facets in STL
- Shows exact coordinates and normals
- **Best for**: Pinpointing exact location of orphans

## Files Structure

```
3Bahtinov_split_config.html          (Main: generator + console tools)
test/
  export_visualizer.html             (Visual inspector: SVG + STL viewer)
  test_*.html                        (Other reference tests)
EDGE_CASE_TEST_GUIDE.md              (This file)
```

## Expected Results

### Clean (All Pass)
```
Scenario A: ✓ PASS
Scenario B: ✓ PASS
Scenario C: ✓ PASS

VERDICT:
✓ NO ISSUES DETECTED - exports appear clean
```

### Issues Detected
```
Scenario A: ✗ FAIL
VERDICT:
⚠ SVG has N suspicious short paths
⚠ STL has M degenerate facets
⚠ STL top/bottom mismatch

→ Use export_visualizer.html to inspect in detail
```

## Troubleshooting

### "No SVG source found"
- Make sure you've run `drawMask()` first
- Or download an SVG from the HTML page

### STL visualization shows mostly blank
- The canvas is showing top-view projection
- Try "Color by Type" to see facets better
- Zoom with browser zoom (Ctrl/Cmd +)

### Can't find the orphan edge
- Use the console analysis first to get edge/facet IDs
- Then search for those specific IDs in the visualizer
- The ID is shown on click

### Files not downloading
- Check if browser blocked popups
- Allow popups for this site
- Or manually use "Download SVG" and "Download STL" buttons

## Key Code Points

If issues are found and need fixing:

1. **Clipping**: `clipFeatureToMaskSolid()` (line ~1214)
   - Applies Sutherland-Hodgman + circle clip
   - May need tolerance adjustment
   
2. **Suppression**: `computeSuppressedSideEdges()` (line ~1268)
   - Marks boundary/chord edges for skip
   - Checks: circle chords, radial boundaries
   
3. **STL Gen**: `scanlineSideWallsSelective()` (line ~2181)
   - Skips marked edges when creating side walls
   - Check if skip flags are being passed correctly

4. **SVG Export**: `downloadSVG()` (line ~2395)
   - Clones SVG, removes export-ignore elements
   - Check if all orphan paths are being included

## Notes

- All measurements in mm by default (can switch to inches)
- Coordinate system: math coords (x right, y up), then flipped for SVG (y down)
- Orphan detection tolerance: ~0.15mm for boundaries
- Short path threshold: <150 characters in SVG path data
