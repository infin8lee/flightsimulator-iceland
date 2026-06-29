# hellissandur · eclipse flight

A single-file WebGL flight simulator: fly the **real Snæfellsnes peninsula** (Iceland)
and watch a total solar eclipse sweep the sky to totality. Three.js (r128, via CDN), no build step.

Live: https://precious-arithmetic-5d64fc.netlify.app/

## What's real here

The terrain is genuine topography, not procedural noise. Elevation comes from **ArcticDEM v4.1**
(10 m), baked once into a compact heightmap that drives **both** the terrain mesh and the flight
collision, so they can never disagree:

- accurate coastline (the 0 m contour of the real DEM),
- the true ridges, valleys, and fjord-cut north coast of the peninsula,
- **Snæfellsjökull** (~1,446 m) as the real ice-capped landmark you climb toward.

You spawn over **Hellissandur** (64.917° N, 23.88° W) heading ~158°, nose pointed straight at the glacier.

## Controls

↑ ↓ pitch · ← → bank · W / S throttle · space boost · E hasten eclipse · R reset

## Run locally

The sim fetches `assets/*` and reads the heightmap pixels back from a canvas, so it **must be served
over HTTP** — opening the file with `file://` fails on fetch/CORS. From this folder:

```
python3 -m http.server 8000
# then open http://localhost:8000/
```

## Deploy (Netlify)

Purely static — no build, no env vars, no API keys. Publish this folder: `index.html` is the default
document and `assets/*` are CDN-cached. Drag-and-drop the folder into Netlify, or connect the repo with
**publish directory = repo root** and **no build command**.

## Repository layout

```
index.html                       the whole sim (HTML + CSS + Three.js)
assets/snaefellsnes-height.png   baked elevation, 2048², 16-bit packed into R (high) / G (low)
assets/snaefellsnes-height.json  sidecar: bbox, scale, min/max, landmark coords, attribution
tools/bake-dem.mjs               offline DEM baker (run once; its output is committed)
tools/package.json               bake-only dev deps (geotiff, pngjs, proj4)
```

## Re-baking the terrain (optional)

Only needed to change the area, resolution, or DEM source. The outputs are committed, so the deployed
site never bakes anything.

```
cd tools
npm install
npm run bake      # fetches ArcticDEM 10 m, reprojects EPSG:3413 → bbox, writes ../assets/*
```

Key knobs in `tools/bake-dem.mjs`: `BBOX` (area), `OUT` (heightmap size), `GEOID` / `SEA_CUT`
(coastline cleanup). Source tiles are pulled anonymously from the public `pgc-opendata-dems` S3
bucket — **no API key**. ArcticDEM heights are ellipsoidal, so the baker subtracts the ~64 m Iceland
geoid to put sea level at ~0.

## Tuning the flight (in `index.html`)

- `EXAG` (1.8) — vertical exaggeration; 1.4 km of relief over 33 km reads flat from the air.
- `SEG` (1024 desktop / 512 touch) — terrain mesh density. Collision stays full-resolution via the
  shared `H()` sampler regardless, so lowering this only softens the silhouette, never the physics.
- `MINS` / `MAXS` (55 / 130 m/s) — throttle speed range; at 1 unit = 1 m the HUD shows true km/h.
- `START_ALT` (2400) and the spawn heading are derived from the baked landmark coordinates.

For a faceted, low-poly look instead of the smooth shading, add `flatShading:true` to the terrain
`MeshStandardMaterial`.

## Data, attribution & licensing

- **Elevation — ArcticDEM** (Polar Geospatial Center): open data, no key, no usage fee. On-screen
  credit is shown in the HUD, pulled from the sidecar JSON. Recommended citation:
  > Porter, Claire, et al. (2022). *ArcticDEM — Mosaics, Version 4.1.* Harvard Dataverse.
  > Geospatial support provided by the Polar Geospatial Center under NSF-OPP awards
  > 1043681, 1559691, and 2129685.
- **Three.js** r128 — MIT (loaded from cdnjs).
- **Fonts** — Google Fonts (Cormorant Garamond, Space Grotesk), open licenses.
- No runtime or bake-time API keys. The deployed site makes only same-origin `assets/*` requests
  plus the Three.js CDN and Google Fonts.
