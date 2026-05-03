# TroBahtinov_configurable_Splittable

Configurable and splittable Tri-Bahtinov mask generator implemented as standalone HTML tools.

## Scope

This project provides browser-based generators and test benches for producing:

- SVG geometry of a tri-Bahtinov focusing mask
- STL geometry for 3D printing
- Split-mask variants for printers with smaller build volumes
- Experimental comparison pages for slit rendering and extrusion strategies

The project is intentionally framework-free: each tool is a single HTML file with embedded JavaScript and CSS.

## Main Tool

- `3Bahtinov_split_config.html`

### What it does

- Exposes optical and mechanical parameters (diameters, offsets, slit spacing, rounding, split mode, mounting holes)
- Computes derived slit geometry from telescope and Bahtinov parameters
- Builds ring/sector domains and clipped slit polygons
- Generates SVG output with physical dimensions for export
- Exports STL using two approaches:
  - Bridge-merge triangulation
  - Scanline exact triangulation (recommended for smooth boundaries)

### Implementation overview

1. Parameter input and unit normalization
2. Derived geometry computation (slit pitch/width/depth and ring boundaries)
3. Polygon construction and clipping for sectors and slits
4. Optional slit-corner rounding with tangent-based fillets
5. SVG rendering and serialization for download
6. STL extrusion:
   - Top and bottom face triangulation
   - Side-wall generation from boundary loops
   - Download as ASCII STL

## Test Tools

Files under `test/` are focused experiments to validate geometric and export strategies (offset behavior, slit rendering methods, extrusion variants).

## License

This project is licensed under the GNU General Public License v3.0 or later.

- See `LICENSE` for the full license text.
- See `NOTICE` for project licensing note.
