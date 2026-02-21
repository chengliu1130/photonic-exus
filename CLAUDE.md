# CLAUDE.md — Photonics Nexus

## Project Overview

Photonics Nexus is a **single-file 3D interactive optical system design & simulation platform** (~34,000 lines, single HTML file). It enables users to place optical components on a virtual optical bench, trace laser beams through the system in real-time, and analyze pulse/dispersion/power characteristics. This is a **pre-sales tool** for WaveQuanta — it recommends WaveQuanta high-end modules when appropriate (e.g., damage threshold warnings, pulse broadening alerts).

**Main file:** `photonics_nexus_v7_command (12).html`

## Tech Stack

- **Three.js** — 3D rendering, scene graph, raycasting (loaded via CDN)
- **Vanilla JavaScript (ES6+)** — all application logic, no frameworks
- **HTML5 / CSS3** — UI layout (CSS Grid), theming via CSS variables
- **No build system** — open the HTML file directly in a WebGL-capable browser

## Architecture

### Pattern: Data-Driven Entity-Component-System (ECS)

```
UI Layer (HTML/CSS)
  ↕
Manager/System Layer (SceneManager, PhysicsEngine, BeamSystem, ValidatorManager, HistoryManager, …)
  ↕
Data & Physics Layer (PRODUCT_CATALOG, PRODUCT_SCHEMA, ProductRepository, WavelengthColorSystem)
  ↕
Rendering Layer (Three.js Scene, Camera, Renderer, UnrealBloomPass for glow effects)
```

### Core Principles (MUST follow)

1. **Visual First / "Deep Dark Lab" aesthetic** — Use UnrealBloomPass (glow) for lasers and grids. Sci-fi/cyberpunk look.
2. **Strict ECS + Data-Driven** — Never hard-code product parameters. Always use `LibraryManager` / `PRODUCT_CATALOG` JSON configs.
3. **Physics Accuracy** — Optical calculations (FFT, dispersion, Jones Matrix) must be physically correct. Visuals follow physics, not the other way around.
4. **Single-File HTML delivery** — All CSS/JS inline or via CDN. Must preview immediately in browser/Artifacts.
5. **Business-Oriented** — Implement logic that recommends WaveQuanta modules when thresholds are exceeded.

## Code Organization (within the single HTML file)

| Line Range (approx.) | Section |
|---|---|
| 1 – 3,000 | **CSS** — Themes, UI components, animations |
| 3,000 – 6,500 | **HTML** — App container, sidebars, viewport, modals, toolbars |
| 6,500 – 12,000 | **Data** — `PRODUCT_CATALOG`, `PRODUCT_SCHEMA`, `WavelengthColorSystem`, `THEME_CONFIGS` |
| 12,000 – 18,000 | **Physics** — `PhysicsEngine`, `PrismPairPhysics`, `GratingPairPhysics`, `PulseAnalyzer`, `DispersionOptimizer` |
| 18,000 – 27,000 | **Core Managers** — `SceneManager`, `BeamSystem`, `ValidatorManager`, `MeasurementTool`, `HistoryManager` |
| 27,000 – 31,000 | **UI Managers** — `ThemeManager`, `LayoutManager`, `ToolbarManager`, `ProductSelectorModal` |
| 31,000 – end | **Initialization** — `DOMContentLoaded`, manager instantiation, event setup, animation loop |

## Key Classes

| Class | Responsibility |
|---|---|
| `SceneManager` | Three.js scene, entity CRUD, raycasting, viewport management |
| `PhysicsEngine` | Laser physics simulation (power, wavelength, GDD, pulse width) |
| `BeamSystem` | Ray tracing / beam propagation through optical chain |
| `ValidatorManager` | Wavelength/power/aperture compatibility checks |
| `HistoryManager` | Undo/redo via Command pattern (`CmdAdd`, `CmdDelete`, `CmdMove`, `CmdRotate`, `CmdAutoAlign`, `CmdGroup`, `CmdBatch`, …) |
| `ProductRepository` | Catalog lookup and filtering |
| `MeasurementTool` | Distance measurement with visual overlay |
| `OpticalPathTool` | Optical path length calculation |
| `AutoAlignSystem` | Automatic beam-to-component alignment |
| `ProjectSerializer` | Save/load project state as JSON |
| `ThemeManager` | 5 themes: light, dark, realistic, arctic, sunset |

## Key Data Structures

### PRODUCT_CATALOG entry
```js
{
  name: "Model Name",
  price: 1000,
  type: "SOURCE",          // SOURCE | OPTIC | OPA | COMPRESSOR | DETECTOR | ANALYZER | CHIRPED_MIRROR | ND_FILTER | PRISM_PAIR_COMPENSATOR | GRATING_PAIR_COMPENSATOR | PULSE_SHAPER | MEASUREMENT_TOOL
  physics: { wavelength, initialPower, beamDiameter, pulseWidth },
  visual: { width, depth, height },
  constraints: { ... }
}
```

### Entity userData
```js
entity.userData = {
  productId, label, params: { wavelength, initialPower, pulseWidth, … },
  position: { x, z }, rotation,
  ports: { input: { x, z }, output: { x, z } },
  locked, isPumped, outputPosition, parent
}
```

### Physics Payload (beam propagation)
```js
{ power, wavelength, pulseWidth, laserPulseWidth, accumulatedGDD, beamDiameter, sourceLabel }
```

### GlobalState
```js
const GlobalState = { beamHeight: 0, componentCounts: {}, componentLabels: {} };
```

## Component Categories

- **Sources** — Laser Sources, Y-Laser (Ytterbium fiber), OPA
- **Optics** — Lenses, Mirrors, Chirped Mirrors, ND Filters, Beam Splitters
- **Compressors** — Pulse Compressors, Prism Pair, Grating Pair
- **Analyzers** — Power Meters, Oscilloscopes, Autocorrelators, Photodetectors, Virtual Beam Analyzer

## Physics Features

- Real-time ray tracing with wavelength-dependent color (UV → IR)
- Power tracking through optical chain (accounting for losses)
- Group Delay Dispersion (GDD) accumulation and compensation
- Pulse width evolution (broadening / compression)
- FROG pulse comparison analysis
- Dispersion optimizer with component recommendations

## Validation Framework

The `ValidatorManager` checks in real-time:
- Wavelength compatibility (source → optic operating range)
- Power rating compliance (input ≤ max rated)
- Aperture sizing (beam diameter ≤ optic aperture)
- Auto-recommends alternative WaveQuanta components on failure

## Development

- **Run:** Open the HTML file in any modern browser with WebGL
- **Edit:** Directly modify the single HTML file
- **No npm / no build / no compilation** required
- **Version tracking:** 41+ development sprints documented in inline comments (search `SPRINT`)

## Coding Conventions

- ES6+ class syntax for all managers/systems
- Command pattern objects prefixed with `Cmd` (e.g., `CmdAdd`, `CmdMove`)
- CSS custom properties (variables) for theming — scoped under `:root` and `[data-theme]`
- Component type constants are UPPER_SNAKE_CASE strings
- Physics calculations use SI-adjacent units: nm (wavelength), W (power), fs (pulse width), mm (beam diameter), fs² (GDD)
- Inline comments reference sprint numbers for traceability

## Common Tasks

### Adding a new optical component
1. Add product entry to `PRODUCT_CATALOG` with `physics`, `visual`, `constraints`
2. Add schema entry to `PRODUCT_SCHEMA` if custom parameters are needed
3. Add a `type` handler in `BeamSystem` for beam interaction logic
4. Add validation rules in `ValidatorManager` if applicable
5. Add 3D mesh creation logic in `SceneManager`

### Adding a new physics engine
1. Create a new class (e.g., `NewPhysics`) with `calculate()` method
2. Wire it into `PhysicsEngine` or `BeamSystem` propagation chain
3. Update the measurement hub UI for displaying results

### Changing themes
- Modify `THEME_CONFIGS` object or add a new `[data-theme="name"]` CSS block
- Register in `ThemeManager`

## Gotchas

- **Grouped components** use hierarchical transforms — always convert to world coordinates via `entity.getWorldPosition()` / `entity.getWorldQuaternion()` before physics calculations (see Sprint 28 fix)
- **BeamHeight** is globally fixed via `GlobalState.beamHeight` — all beams propagate on the same Y-plane
- **Single-file constraint** — all changes must stay within one HTML file; do not split into separate JS/CSS modules
- **UnrealBloomPass** requires a specific Three.js post-processing pipeline setup — do not remove the composer chain
