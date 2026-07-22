# CLAUDE.md — VaCom E&C Panel Layout Editor

Save this file as CLAUDE.md at the project root so Claude Code auto-loads it each session.

INTERNAL USE ONLY // CONFIDENTIAL — VaCom Technologies (BITZER Group). Rev 1.

## What this project is
A standalone, single-file HTML tool for arranging control-panel components on a back panel
and door (2D drag editing + a 3D view), reading and writing one JSON layout model. It is the
"you arrange it" step of a larger E&C drawing pipeline. As of Rev 4 the app can also **ingest a
BOM (CSV/Excel), auto-arrange the parts, and export self-contained 2D + 3D drawings as HTML** —
the human still fine-tunes by dragging; the app enforces snap, spacing, and keep-outs.

## Where it fits (context, not scope for now)
Configurator (arrange) → this editor → arranged JSON → elevation/fab drawing generator →
ACADE schematic assembly → ACADE final BOM. Only the editor is in scope here. The arranged JSON
this app exports is the input the downstream elevation generator will consume.

## Current state — Rev 5
Working: **BOM import (CSV/TSV/Excel-paste or file) with auto column-mapping** (Tag/PN/Catalog/
Description/Qty/Type/Mount/Face), **catalog footprint lookup by PN** (dimsSource flips to
"catalog" on a hit; nominal keyword resolver is the fallback), **auto-arrange** (DIN devices packed
along rails with wrap; large panel-mount gear packed into the open bays between rails; duct
keep-outs avoided; hand-placed `geometry.pinned` devices are left alone), **three faces —
Back / Door / Side** (each with its own envelope in meta.subpanel / meta.door / meta.side), 2D drag
with grid + DIN-rail snap, keyboard nudge, all-face overlap/clearance/duct/envelope keep-out guard
(caution-yellow flag + clickable status list), 3D orbit/zoom view, inspector edit, Import/Export
JSON, add/rotate/delete device, and **Export Drawings (HTML)** = one self-contained file with
back / door / side 2D elevations (red mm dims to the lower-left datum), an interactive 3D view, and
a device/nameplate schedule table (with a catalog-vs-nominal dims column).
The embedded sample is the real **rCMP-1-1-1 compressor panel (Job S2240 / Panel V2029)** parsed
from the fab PDF: Saginaw SCE-903620FS Type 3R enclosure, 813×1981 back subpanel + 356×1981 side
subpanel, PowerFlex 755TS 250HP VFD, 600A AB fused disconnect (+ door handle), 2× 500 CFM side
filter fans, control gear on DIN rails.
Stubbed / known: catalog dims cover the parts in the built-in CATALOG map only (enclosure/subpanel/
fan dims are off the drawing; VFD/disconnect/xfmr are mfg frame data); everything else is still
nominal. The CATALOG map is the hook the Access .mdb export will later feed. Header uses a green
wordmark, not the real shield asset (logo slot reserved — never ship a placeholder logo).

## Data contract (the JSON model — single source of truth)
Top level: schemaVersion, meta, rails[], ducts[], devices[].
- meta: so, panelId, rev, units:"mm", datum:"lower-left", enclosure{mfg,pn,typeRating,width,height,depth}, subpanel{width,height}, door{present,width,height}, side{present,width,height}.
- rails[]: id, face, type, orientation, x, y, length — these are the snap targets.
- ducts[]: id, face, x, y, w, h, depth — these are keep-out zones.
- devices[]: id, tag, pn, description, circuitType, face("back"|"door"|"side"), mount("din"|"panel"), railId, geometry{x,y,w,h,depth,rotation,pinned?}, dimsSource("nominal"|"catalog"), catNo, clearanceMm, nameplate.

Schema is living/versioned — expected to evolve. Before any merge with the VaCom Panel
Configurator, field names must be aligned to that app's real export.

## Coordinate conventions (do not change without discussion)
- Units mm. Datum 0,0 = lower-left of the mounting face (VaCom fab standard).
- geometry.x/y = lower-left corner of the footprint at rotation 0.
- rotation ∈ {0,90,180,270}; at 90/270 the effective w/h swap (parts stay axis-aligned).
- depth = Z off the mounting face; used only for the 3D extrude + clearance.
- face: "back" = subpanel; "door" = door inner face; "side" = side-mount subpanel. Each face has its own envelope (meta.subpanel / meta.door / meta.side).

## Architecture + hard constraints
- Delivery artifact = ONE self-contained .html file, zero external dependencies. Internal
  dev structure is your choice, but the shipped file must stay single-file and dependency-free
  (it is viewed on iPhone and in Cowork). No CDN, no build step required to open it.
- The exported drawing package is likewise ONE self-contained .html (inline SVG + inline 3D canvas
  script). No external references.
- No localStorage/sessionStorage — they fail in sandboxed viewers. Persistence is via
  Import/Export JSON only.
- Rendering is HTML5 canvas (2D editor + a hand-rolled 3D orthographic painter's-algorithm view).

## Pitfalls already hit (keep these — they cost real revs)
- Use Pointer Events (pointerdown/move/up + setPointerCapture), NOT mouse events —
  otherwise dragging is dead on touch/iPhone. This was the Rev 1→3 blocker.
- Set touch-action:none on both canvases or a drag scrolls the page instead.
- Any overlay sitting on top of a canvas (hint bar, coord readout, labels) must be
  pointer-events:none or it swallows taps on components beneath it.
- Canvas must have guaranteed height: flex-column body + min-height on the work area,
  a ResizeObserver redraw, a deferred requestAnimationFrame first draw, and a zero-size
  guard in the draw functions. Without this the canvas collapses to 0 and nothing renders.
- Scale canvas by devicePixelRatio for crisp lines.
- 3D auto-fits scale unless the user has manually zoomed (cam.userScaled).

## VaCom delivery standards (apply to all output)
- Branding: VaCom green #3AAA35, near-black #1A1A1A, greys, pale-green #E6F1DD,
  caution-yellow #F4C20D. Arial body, Courier New for mono/readouts. // as the bullet/sep.
  Footer INTERNAL USE ONLY // CONFIDENTIAL. Real shield logo goes in the header slot when
  supplied (transparent on dark) — never ship a placeholder logo.
- Rev discipline: increment Rev on every deliverable; never overwrite a prior Rev.
  Filename ProjectName_DocumentType_Rev#.ext (e.g. EC-PanelLayout_Editor_Rev5.html).
- Honest status: say "in development"; never call anything "validated/final" unless it is.
- Audience: the owner is a non-developer — ship finished, working files, not code to run.

## Guardrails (non-negotiable)
- AI/code never authors SCCR, OCPD/breaker/fuse sizing, conductor ampacity, or NH₃ relief
  sizing. Anything in that space is labeled "SUGGESTED — PE VERIFY."
- No client names anywhere in output. Job/SO numbers are fine.
- Keep confidential content out of any web queries.

## Roadmap / candidate next tasks
1. Extend the catalog: Rev 5 added a PN→footprint CATALOG map + lookup (dimsSource "catalog");
   next is feeding it from the real Access .mdb export so every BOM PN resolves to true dims and
   the keep-out guard becomes fully trustworthy (today only mapped PNs are catalog-grade).
2. Elevation / fab drawing export refinement — per-device mm dimension chains (not just overall),
   duct/rail dimensions, formal title block. (Rev 4 ships the first cut of this.)
3. Configurator schema alignment — map field names to the live Panel Configurator export
   ahead of merge.
4. Interaction polish: multi-select, align/distribute, duplicate device, live dimension readout
   between parts. (Keyboard nudge landed in Rev 4.)
5. Header shield logo drop-in once the asset is provided.

## Files expected in this project
- EC-PanelLayout_Editor_Rev5.html — current app (start here).
- EC-PanelLayout_Editor_Rev4.html — prior rev (kept; rev discipline — never overwrite).
- (Rev 3 / earlier revs and the JSON-schema / workflow-plan reference docs are not yet in this repo.)

## Working style for Claude Code on this repo
Plan before large builds; ask focused numbered questions when scope is unclear. Increment the
Rev on the shipped HTML each iteration. Keep this file under ~200 lines; record only durable
conventions and pitfalls here, not narration.

> NOTE: This repo's root also contains an unrelated app (`index.html` = "Quality Temp HUB" HVAC
> workspace). The panel editor lives in its own `EC-PanelLayout_Editor_Rev#.html` files and does
> NOT touch `index.html`.
