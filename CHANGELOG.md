# Changelog

All notable changes to this project are documented in this file.

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
