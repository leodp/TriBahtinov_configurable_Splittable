# TriBahtinov_configurable_Splittable

Configurable and splittable Tri-Bahtinov mask generator with integrated FFT focal-plane inspection, implemented as standalone HTML tools.

## Scope

This project provides browser-based generators and test benches for producing:

- SVG geometry of a tri-Bahtinov focusing mask
- STL geometry for 3D printing
- Approximate focal-plane FFT renderings of the mask response
- Split-mask variants for printers with smaller build volumes
- Experimental comparison pages for slit rendering and extrusion strategies

The project is intentionally framework-free: each tool is a single HTML file with embedded JavaScript and CSS.

## Main Tool

- `3Bahtinov_split_config.html`

Release status: `v1.6`

### What it does

- Exposes optical and mechanical parameters (diameters, offsets, slit spacing, rounding, split mode, 3x/4x mounting holes)
- Computes derived slit geometry from telescope and Bahtinov parameters
- Builds ring/sector domains and clipped slit polygons
- Generates SVG output with physical dimensions for export; split masks use an anti-sector white polygon so the split is visible in all CAD tools without requiring SVG clipPath support
- Uses the generated SVG, or an uploaded SVG, as the source for FFT evaluation
- Renders an approximate focal-plane image for quick parameter inspection, with Airy radius shown in micrometers
- Supports red, green, blue, and RGB FFT rendering modes
- Supports FFT defocus simulation: enter a signed wave count (positive = front focus, negative = back focus, 0 = in focus) referenced to 530 nm green
- Exports STL with side walls emitted directly from polygon boundaries for improved manifold quality
- Scale bar below mask preview shows the tube/mask diameter on screen
- **Parameter import/export**: Save all UI parameters to CSV file and load them back for configuration preservation
- **Defocus sweep export**: Batch-export FFT focal-plane PNG images at multiple defocus values for creating focus sequences

### Implementation overview

1. Parameter input and unit normalization
2. Derived geometry computation (slit pitch/width/depth and ring boundaries)
3. Polygon construction and clipping for sectors and slits
4. Optional slit-corner rounding with tangent-based fillets
5. SVG rendering and serialization for download
6. FFT aperture rasterization and focal-plane rendering
7. STL extrusion:
   - Top and bottom face triangulation
   - Side-wall generation from boundary loops
   - Download as ASCII STL

## Test Tools

Files under `test/` are focused experiments to validate geometric and export strategies (offset behavior, slit rendering methods, extrusion variants).

## License

This project is licensed under the GNU General Public License v3.0 or later.

- See `LICENSE` for the full license text.
- See `NOTICE` for project licensing note.
