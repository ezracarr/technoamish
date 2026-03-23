# Technoamish — The Nostr Barn / Nostr Village

An interactive **3D explainer and playground for the Nostr protocol**, delivered as a **single HTML file** with no build step. It presents a Tron-inspired “techno-Amish” world: a barn whose timbers map to real open-source repositories, a village of enterable buildings tied to well-known pubkeys, live data from relays, and a growing set of optional visual features.

The experience is meant to be **educational and atmospheric**—hover tooltips, GitHub links, note walls, a GM garden, river fish carrying trending notes, and HUD copy that shifts as you move from barn to village to “frontier.”

---

## What this project is

- **Protocol storytelling in 3D.** Roughly **100 curated repos, clients, relays, and tools** are represented as interactive pieces of a gambrel barn. Categories (Protocol, Relay, Client, Library, etc.) map to colors; the legend filters highlights.
- **Live Nostr.** The page opens **WebSocket** connections to public relays, subscribes for **kind 0** (profiles) and **kind 1** (notes), caches authors, and drives ground “radar” cells, village interiors, fish, gardens, and analytics-style panels.
- **Three.js world.** [Three.js r128](https://threejs.org/) (loaded from CDN) powers terrain, the barn, village, river, sky, particles, and camera work—including **orbit mode** for the world and **first-person look** inside buildings.
- **No compilation.** Everything ships in one document: inline CSS, HTML shell, and a large block of vanilla JavaScript. That keeps demos easy to copy, host, or archive.

---

## Repository structure

This repository is intentionally minimal:

| Path | Role |
|------|------|
| `nostr-barn-tron.html` | **Entire application** — styles, markup, scripts, and embedded data arrays. |
| `README.md` | This documentation. |

There is **no** `package.json`, bundler, or framework. Optional assets (textures, models) are generated at runtime (canvas 2D → `CanvasTexture`, procedural geometry) or pulled from the network (relay profile pictures, Three.js from CDN).

---

## How the HTML file is organized

All implementation detail lives inside `nostr-barn-tron.html`. The file begins with a **table of contents** in comments (section numbers and topics). The following is a high-level map aligned with that structure.

### Document shell

1. **`<head>`** — Meta, title (“The Nostr Barn”), and a large **`<style>`** block: full-screen canvas, cyan/orange Tron UI (HUD, tooltips, modals, feature panel, nav pad, explorer, zap log, etc.).
2. **`<body>`** — Fixed-position UI layers (no framework components): note modal, relay status, house exit button, explainer overlay, control buttons, feature toggles, search, proverb ticker, protocol explorer panel, compass/progress widgets, tooltip, legend, on-screen navigation.
3. **Scripts** — External **Three.js** script tag, then one **inline** `"use strict"` block containing the entire program.

### Major logical sections (inline script)

Rough order of concerns:

1. **Data** — `REPOS` (100 entries: name, description, URL, category). Color maps `CI` / `CIhex` for legend and materials.
2. **Legend** — Builds category chips; `applyCatHighlight()` syncs barn part edge colors with the active filter.
3. **Sound** — Web Audio API (e.g. construction hammer, bell) tied to barn state.
4. **Three.js core** — Scene, perspective camera, WebGL renderer, fog, lights, ground grid, sky elements (stars, clouds, moon), tone mapping.
5. **Terrain** — Procedural extruded “ranges” around the play area.
6. **Relay client** — Connects to a fallback list of `wss://` relays, fills `nostrNotes`, profile cache, filtering helpers (`isJunkNote`, etc.).
7. **Ground radar** — Grid of textured cells; sweep animation assigns notes; click opens a read modal.
8. **Barn** — ~100 meshes mapped 1:1 to `REPOS`; deconstruct/reconstruct animation; X-Ray interior; hex sign (quilt code feature); hover → tooltip, click → repo URL.
9. **Horse & buggy** — Decorative orbit around the barn.
10. **Camera & input** — Spherical orbit (`tgt`, `th`, `ph`, `rad`), drag, wheel zoom, keyboard and on-screen nav, first-person yaw/pitch when inside a building.
11. **Feature toggles** — `FT` object and `applyFeat()`; keyboard shortcuts (Z, H, /, S, P, F, Q, W, G, R, E, I, B, etc.). Includes zaps (visual), heartbeat pulse, search, sunrise, proverbs, fireflies, quilt code, web-of-trust lines, GM garden, relay constellation, NIP explorer, event inspector, banjo (Web Audio).
12. **GM garden** — Plants grown from “GM” notes; relay-specific fetches; hover/click for notes.
13. **Village** — `VILLAGE_NPUBS` (npub + display name per lot), `BLDG_LAYOUTS` (position, size, rotation, type: house / shop / church). `createBuilding()` builds geometry, fireplace, stick figure, label sprite, and defers **note walls** until `fetchNotes()` runs (per-building WebSocket to Primal for that author).
14. **Infrastructure** — Roads, square, pine tree (click to grow), villagers, small trees.
15. **River & bridge** — Catmull-Rom path, water mesh, flow lines, bridge placement.
16. **Mill, fish, riverside plots** — Fish carry trending notes; empty lots / signage.
17. **Enter / exit buildings** — `enterBuilding` / `exitBuilding`, camera lerp, wall visibility, ESC and UI button.
18. **Deep linking** — On load, reads `?npub=`, `?house=`, or hash forms; matches **npub** (case-insensitive) or **64-char hex** to `village[]` entries; calls `enterBuilding` when matched (first match if the same key appears twice).
19. **Render loop** — `anim()`: animations, transitions, feature updates, `renderer.render`.

### Important global data you may edit

- **`REPOS`** — What the barn represents; each index aligns with a barn “part.”
- **`VILLAGE_NPUBS`** — Which npub “owns” each building slot; must stay aligned with **`BLDG_LAYOUTS`** index order (`createBuilding(i)` uses both).
- **`npubToHex()`** — Bech32 decoding for npubs used in filters and deep links.

### Important runtime objects (globals)

Documented in-file near the TOC: `scene`, `camera`, `renderer`, `nostrNotes`, `profileCache`, `village`, `gmPlants`, `fishes`, `FT`, `insideHouse`, `activeBldg`, `modalOpen`, etc.

---

## How to run it

### Why not `file://`?

Browsers restrict **WebSocket** and some other APIs for `file://` pages. Relay connections expect a normal **http(s) origin**. Always serve the file over HTTP for full behavior.

### Local static server

From the repository root:

```bash
cd /path/to/technoamish
python3 -m http.server 8080
```

Then open:

```text
http://localhost:8080/nostr-barn-tron.html
```

Any static server works (e.g. `npx serve`, `php -S`, Caddy, nginx). Use the same origin when embedding or sharing links.

---

## Deep links (enter a specific house)

If the query or hash contains an npub (or 64-character hex pubkey) that matches a row in **`VILLAGE_NPUBS`**, the page **starts the same camera transition as clicking that building**—you begin inside that interior.

Supported forms:

- `?npub=npub1…`
- `?house=npub1…`
- `#npub=npub1…` (and similar hash patterns parsed in code)

Example:

```text
http://localhost:8080/nostr-barn-tron.html?npub=npub1…
```

**Note:** If the same npub is assigned to more than one building index, the **first** matching building in `village[]` wins. To deep-link a specific lot, ensure unique keys or extend the code with an index parameter.

---

## Dependencies

| Dependency | Source | Purpose |
|------------|--------|---------|
| **Three.js r128** | CDN (cdnjs) | 3D rendering |
| **Public Nostr relays** | Various `wss://` URLs in script | Events and profiles |

No npm install is required.

---

## Controls (summary)

- **Mouse:** Drag to orbit (outside); drag to look (inside building). Wheel zoom outside.
- **UI:** Deconstruct / X-Ray / Features; feature panel toggles; legend categories; nav pad and keyboard arrows; zoom +/- buttons.
- **Inside a building:** ESC or **ESC to exit** button to leave; click wall cells to read notes when loaded.

Exact shortcuts are listed in the in-file feature panel and TOC.

---

## Customization tips

1. **Add or change barn projects** — Edit `REPOS` and keep the array length consistent with barn geometry if you change part counts (advanced).
2. **Assign buildings to different people** — Edit `VILLAGE_NPUBS` in lockstep with `BLDG_LAYOUTS` indices.
3. **Change relay endpoints** — Search the HTML for `wss://` and adjust carefully (subscriptions use Nostr’s JSON-over-WebSocket protocol).
4. **Split the file** — For maintainability, you can move CSS and JS to separate files without changing behavior; the current layout favors single-file portability.

---

## License and credits

If the original author specified a license elsewhere, follow that. The inline comments describe the work as a community-facing Nostr visualization; repository metadata may be added over time.

---

## Summary

**Technoamish** is a **static, single-page 3D experience** that teaches and demonstrates the Nostr ecosystem through a stylized barn, a pubkey-mapped village, and live relay data—all inside **`nostr-barn-tron.html`**, served like any other static site.
