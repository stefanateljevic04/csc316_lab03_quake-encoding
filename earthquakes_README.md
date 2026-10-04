# USGS Earthquakes — M6.0+, trailing year

Source: **U.S. Geological Survey** earthquake catalog (ANSS Comprehensive Catalog),
fetched from the FDSN event web service. **Public domain.**

- File: `earthquakes_M6.0_1year.csv`
- **155 rows, one per earthquake**, magnitude **6.0 and larger**, worldwide, over one
  trailing year (2025-05-28 → 2026-05-28). ~30 KB.
- Refresh: change `minmagnitude`, `starttime`, `endtime` in the query below.

```
https://earthquake.usgs.gov/fdsnws/event/1/query?format=csv&starttime=2025-05-28&endtime=2026-05-28&minmagnitude=6.0
```

## Columns that matter for this lab

| Column | Type | Range (this file) | Notes |
| --- | --- | --- | --- |
| `longitude` | float | −179.45 … 179.63 | Degrees East. **A quantitative variable** — maps straight to an x-scale. |
| `latitude` | float | −61.19 … 61.57 | Degrees North. Maps to a y-scale (remember: y grows *down* in SVG). |
| `depth` | float | 5.64 … 636.26 | Kilometres below the surface. Long right tail (deep-focus quakes). |
| `mag` | float | 6.0 … 8.8 | **Moment magnitude. This scale is *logarithmic*** (see below). Mean 6.35. |
| `magType` | string | mostly `mww` | 145 `mww`, 7 `mb`, 2 `mwb`, 1 `ml`. Categorical. |
| `time` | ISO 8601 | — | Event timestamp, UTC. Not a channel in this lab (stretch goal). |
| `place` | string | — | Human-readable location, e.g. "western Indian-Antarctic Ridge". Good for tooltips. |

All 155 rows are complete — **no nulls, no duplicates** in the columns above. The CSV
loads every value as a **string**; convert `longitude`, `latitude`, `depth`, `mag` to
numbers before plotting.

## Quirks that matter for *judging* an encoding

- **Magnitude is logarithmic.** Each whole step up (6→7→8) is ~32× more energy released.
  So the difference between a 6.0 and an 8.8 is *enormous*, even though the numbers look
  close. When you encode `mag` as **circle size, encode it as *area*, not radius** — human
  eyes read area. Use `d3.scaleSqrt` (radius ∝ √value ⇒ area ∝ value). A plain
  `scaleLinear` on the radius **exaggerates** the big quakes and lies.
- **This is only the big ones (M6.0+).** The catalog is *truncated* — thousands of smaller
  quakes are filtered out. Any claim like "earthquakes are rare" is false; you removed the
  common ones. A magnitude histogram here shows only the tip of the Gutenberg–Richter curve.
- **Depth has a long tail.** Most quakes are shallow (<70 km); a few are 300–636 km
  (deep-focus, in subduction slabs). A naive sequential color scale gets swamped by the tail —
  the deep outliers make everything else look the same color. Consider the domain carefully.
- **Longitude wraps at ±180°.** Plotting `longitude` on a linear x-scale over [−180, 180]
  is a real *plate carrée* map projection and works fine; just know the Pacific is split at
  the edges (the Aleutians and Fiji sit near both borders).
