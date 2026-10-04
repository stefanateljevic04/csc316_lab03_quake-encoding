# Repository Guidelines

## Project Structure & Module Organization

This repository is a minimal SvelteKit application with Svelte 5, Vite, and D3 for earthquake visualization work. The current home page is still the starter scaffold.

- `src/routes/+page.svelte`: home page and visualization entry point.
- `src/routes/+layout.svelte`: shared layout and favicon setup.
- `src/lib/`: reusable components, data helpers, and bundled assets. Keep one component per `.svelte` file and import with the `$lib` alias.
- `static/`: directly served assets, including `earthquakes_M6.0_1year.csv`, available at `/earthquakes_M6.0_1year.csv`.
- `earthquakes_README.md`: dataset provenance, column definitions, and encoding considerations.
- `vite.config.js`: SvelteKit plugin, adapter, and runes configuration.

## Build, Test, and Development Commands

- `npm ci`: install dependencies from `package-lock.json`.
- `npm run dev`: start the local Vite development server at http://localhost:5173; append `-- --open` to open a browser.
- `npm run build`: create the production build.
- `npm run preview`: serve the production build locally after building.
- `npm run prepare`: synchronize generated SvelteKit files; also runs during dependency installation.

## Coding Style & Naming Conventions

Use JavaScript ES modules and Svelte components. Match existing tab indentation, single-quoted JavaScript strings, and semicolons. Use Svelte 5 runes (`$state`, `$derived`, `$props`), not legacy `$:` statements. Preserve SvelteKit route filenames such as `+page.svelte` and `+layout.svelte`. Name reusable components in PascalCase and functions and variables in camelCase. No formatter or linter is currently configured.

Use D3 for scales, axes, and CSV loading. Svelte must create earthquake marks with `{#each}`; do not create them with `d3.select`/`append`. D3 axis generators may use selections inside dedicated axis `<g>` elements. Follow the D3 margin convention and label axes with units.

## Testing Guidelines

There is currently no test framework, test script, or coverage threshold. Run `npm run build` for application changes, then inspect the affected page locally. For visualization changes, verify CSV loading, numeric conversion, scales, labels, and browser console errors. If adding automated tests, document the framework and command in `package.json` and the README.

## Commit & Pull Request Guidelines

The history contains only `scaffold + data`, so no formal commit convention is established. Use short, descriptive imperative subjects, such as `Add magnitude encoding controls`. Keep changes focused. Pull requests should explain the resulting behavior, list validation performed, link relevant issues, and include screenshots for visual changes.

## Dataset Guidelines

Load `static/earthquakes_M6.0_1year.csv` client-side with `d3.csv` inside `onMount`, using `/earthquakes_M6.0_1year.csv`. Convert `longitude`, `latitude`, `depth`, and `mag` to numbers on load. These columns have no nulls, but CSV values arrive as strings.

The dataset contains only M6.0+ earthquakes; smaller quakes are filtered out. Do not imply earthquakes are rare. Consult `earthquakes_README.md` for provenance and encoding context; document changes to the source, date range, or magnitude filter there.

## Encoding Rules

- Use `d3.scaleSqrt` for circle radius. Area proportional to a nonnegative value requires radius proportional to its square root, with both domain and range starting at 0. If using `[6, 8.8] → [3, 30]`, explicitly state that size preserves order, not value ratios.
- Magnitude is logarithmic: each +1 corresponds to approximately 32× energy. Size represents magnitude, not energy. Include an M6/M7/M8 size legend and explain this distinction.
- Use `scaleLinear` for x/y position. SVG y grows downward, so use `[height, 0]` for the y range: high latitude or high magnitude belongs at the top.
- Encode the chosen quantitative color column with a sequential scale, such as `d3.scaleSequential` with `interpolateViridis`. Depth has a long right tail; if limiting its domain, enable clamping and label the cutoff in the legend.
- `magType` is categorical; use an ordinal scale if encoding it.

## Definition of Done

- `npm run dev` runs with no browser console errors.
- Four channels are present: longitude → x, latitude → y, magnitude → size (`scaleSqrt`), and the chosen quantitative column → color.
- Size and color legends include units; the color legend names the encoded column. Latitude is north-up.
