# Changelog

All notable changes to this project are documented in this file.

## [1.3] - 2026-05-05

### Changed

- **UI reorganization**: Split and mounting holes controls merged into single "Mask modifications" section
- **Split mask**: Replaced checkbox with 3-option selector (No / 1/3 / 1/6)
  - 1/3 mode: clips mask from -60° to +60° (same as previous behavior)
  - 1/6 mode: clips mask from 0° to 60° (new)
- **3x mounting holes**: Renamed checkbox, added round/square shape option, radial position field,
  diameter/side field (default 5mm, was radius 2.5mm)
- **4x mounting holes**: New option creating holes at 0°/90°/180°/270° with 0°/60° rotation,
  round/square shape, configurable position and diameter
- **FFT controls**: Moved Compute FFT and Download FI PNG buttons above Mask source selector
  in the FFT panel

### Removed

- 3D thickness field and 3D export buttons (bridge merge + scanline) from the UI
- "Mounting holes" section title (controls now under "Mask modifications")

## [1.2.2] - 2026-05-04

### Changed

- Restored M2 guarded slit-rounding behavior in production path:
  - `buildWithSkipping` now rejects fillet centers outside the slit polygon
  - `buildWithSkipping` now enforces tangent-distance validation on both sides
    (`|d - r| <= 0.3 * r`)
- This release intentionally prioritizes stable, conservative rounding (M2) over
  aggressive corner completion.

## [1.2.1] - 2026-05-04

### Changed

- Reverted unwanted rounding exclusion logic — all convex corners are now always rounded:
  - Removed local per-corner radius cap (`localMaxR`) that was preventing rounding on
    arc-boundary corners with short polygon segments
  - Removed `pointInPolygon(arcMid)` skip in `buildWithSkipping` — was silently dropping
    fillets on concave arc-boundary slits. Now uses corner-vertex-side test to determine
    correct arc sweep direction
  - Removed area-growth ceiling, radial bounds check, and retry-with-smaller-radius loop
    from `safeApplyRounding`
  - Removed `!in1 && !in2` skip and distance-from-corner guard in `applyRounding`
  - When arc-line intersection has no solution (`h² < 0`), falls through to offset-lines
    method instead of skipping the corner
  - Invalid geometry is caught only by `sanitizeSlitPolygon` (self-intersection, area collapse)
  - Rounding radius is now auto-capped per slit to half the effective slit width (`A/P`)
    so that fillet circles from opposite sides cannot overlap; fixes R4 inner slits not
    getting rounded at rounding ≥ 0.7
  - When rounding still produces invalid geometry (slit too thin for any fillet), the slit
    is removed entirely instead of falling back to unrounded
  - Reverted the latest stem-meets-arc experiment that made rounding worse:
    - Restored arc-corner MA to use the prior slit-normal construction (`px, py`)
    - Restored stem-corner bisector direction chooser (centroid-distance tie-break)
    - This intentionally returns to the previous behavior where a few critical vertices may
      stay partially rounded instead of producing outward/pointy artifacts
  - Added a targeted MA acceptance check at arc corners (notably near stem↔R3):
    arc-line center is used only if it is inside the slit polygon and tangent distance
    matches rounding radius on both adjacent sides; otherwise that corner falls back to
    bisector rounding. This removes external circular blob artifacts at high rounding.
  - Replaced centroid-based offset-lines stem-corner method with angle bisector approach;
    the centroid could be on the wrong side of edges on concave slit polygons, causing
    fillet centers to land outside the polygon and produce outward-bulging fillets
  - Added `pointInPolygon` safety check on fillet centers in `buildWithSkipping`; fillets
    whose center is outside the slit polygon are now skipped
  - Removed all remaining rounding reduction/disabling guards:
    - Removed `pointInPolygon` rejection of fillet centers in `buildWithSkipping` (unreliable
      for centers near polygon boundaries, especially at stem-meets-arc junctions)
    - Removed tangent-distance validation (30% tolerance) that was skipping valid fillets
    - Removed per-slit radius cap (`effHalfWidth * 0.95`) in `safeApplyRounding`
    - Replaced `pointInPolygon` bisector-direction test with centroid-distance test
      (closer-to-centroid = interior side), which is reliable even at boundary edges
  - Fixed root cause of incorrect rounding at stem-meets-arc junctions: the arc-line
    method now computes the actual edge normal of the adjacent straight edge (slit edge
    or stem boundary) instead of always assuming the slit-direction normal `(px, py)`.
    Previously, stem-boundary corners on arc boundaries got fillet centers tangent to a
    phantom slit edge instead of the actual stem edge, producing outward-bulging fillets.
  - Bisector method now always uses `+bisector` direction (`u1 + u2`), which is
    mathematically proven to point inward for convex corners — no direction guessing.
  - Added tangent-distance validation in `buildWithSkipping`: both back and forward tangent
    distances must be within 30% of the fillet radius; rejects misplaced centers that cause
    spikes or outward expansion
  - When rounding produces invalid geometry, keeps the unrounded slit instead of removing it

## [1.2.0] - 2026-05-04

### Changed

- Replaced slit-rounding algorithm with Analytical Arc-Line Tangency (MA method):
  - For corners where a straight slit edge meets a circular sector boundary, the fillet center
    is now computed as the intersection of the offset slit line and the offset arc circle —
    exact geometry, no iteration
  - For stem corners (two straight edges), the fillet center is at the intersection of the
    two inward-offset edge lines
  - New `buildWithSkipping` polygon builder removes ALL intermediate vertices inside each
    fillet zone, eliminating spike artifacts
  - New helper functions: `closestOnSideEx`, `buildWithSkipping`, `inwardNormal`
  - `applyRounding` and `safeApplyRounding` now accept `tiltDeg`, `rInner`, `rOuter` parameters
    so the analytical method can use the true arc geometry
- Fixed severe rounding failures at high rounding values (notably BahF=100, A_off=90°):
  - Fixed runtime bug in `applyRounding`: `in1`/`in2` were referenced before definition,
    which could break rendering when rounding was enabled
  - Removed major-arc fallback in `buildWithSkipping`: when arc midpoint validation fails,
    the fillet is now rejected instead of switching to near-full-circle subtraction (the
    "circle cut into slit" artifact)
  - Added local per-corner radius cap in `applyRounding` based on adjacent edge lengths and
    corner half-angle, preventing fillet-circle superposition on narrow inner slits
  - Added strict arc-corner candidate check: when both analytical solutions are outside the
    slit polygon, the fillet is skipped rather than forcing an invalid center
  - Added extra geometric guards in `safeApplyRounding` to reject explosive rounded polygons
    before SVG clipping, preventing full-mask disappearance
- Mounting holes angle: added pulldown menu with 0° and 60° options
- FFT display gain: decoupled from per-mask peak normalization; fixed multiplier with frozen normRef
- Removed split+holes chord from SVG without disjoining continuity
- UI: regrouped controls by functional section (Mask modifications → Mounting holes → Export)

### Added

- `test/testRounding_arcfix.html`: 4-method comparison (M0 bisector baseline, MA analytical,
  MB iterative, MC hybrid) × 2 rounding values (0.5, 0.99), with zoomed fillet views showing
  center placement, tangent points, and distance equality annotations

## [1.1.0] - 2026-05-03

### Changed

- Merged the FFT focal-plane renderer into the main tool:
  - `3Bahtinov_split_config.html`
- Reworked the main page into two clearly separated sections:
  - mask generation/export
  - FFT rendering/inspection
- Updated default objective diameter to 203 mm
- Hardened slit-rounding geometry for edge cases:
  - convex-only fillets
  - adaptive rounding fallback for unstable slit polygons
  - interior-center / interior-arc validation to prevent outward fillet application on external slits
- Slit/Stem ratio: changed from pull-down menu to free-text field; default changed to 1
- Added physical unit labels `[mm]`/`[in]` to all length fields in mask UI; labels update when units are switched
- FFT Display gain: decoupled from per-mask peak normalization; now a fixed multiplier (default 1) applied to a mask-independent reference (openPixels²); added read-only Central peak intensity field for cross-mask comparison
- Added `test_rounding_approaches.html`: 4-panel comparison of slit-fillet placement strategies for the problematic case (BahF=100, 3rd order, rounding=0.99)

### Added

- In-page FFT rendering controls next to the simulated focal image
- SVG-driven FFT source mode using the mask SVG generated in the page
- Optional uploaded-SVG FFT source mode
- Resolution boost control for the FFT mask approximation
- Wavelength preset selector with monochrome and RGB modes
- RGB focal-plane rendering using separate red, green, and blue FFT passes

### UI

- Simplified FFT control labels and defaults for practical use
- Added a dedicated file chooser with filename shown on its own line
- Removed redundant FFT subsection title text
- Improved page balance between the FFT viewer and controls column

### Notes

- `1.1.0` is the first merged release where mask generation and FFT inspection live in the same main page.

## [1.0.0] - 2026-05-03

### Added

- Main configurable Tri-Bahtinov generator page:
  - `3Bahtinov_split_config.html`
- Test pages under `test/` for geometry and rendering validation:
  - `test.html`
  - `test_OffsetAngle.html`
  - `test_parallelogram.html`
  - `test_slit_methods.html`
  - `test_3d_extrusion.html`
  - `test_3d_extrusion_v2.html`

### Features

- Parametric slit and ring geometry generation
- Split-mask generation mode
- SVG export with physical sizing metadata
- STL export with selectable triangulation approaches
- Slit rounding logic and geometric clipping workflows

### Notes

- This release is the baseline production configuration validated in-project.
