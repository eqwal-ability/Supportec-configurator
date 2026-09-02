# Supportec-Configurator

Custom seating Supportec configurator — a single, responsive page
(`index.html`) that works on both desktop and mobile, no separate mobile
link required (`mobile.html` just redirects to `index.html` for old links).

`index.html` is fully self-contained: every layer image, all vinyl /
platilon / airmesh swatches, the vendored jsPDF library, the Eqwal
Ability logo/circle backdrop and the title font are embedded directly in
the file as data URIs. You can open it straight from disk (double-click
it, or drag it into a browser tab) with nothing else alongside it, or
serve it from any static file host / GitHub Pages — both work identically.

## How it's built

- Plain HTML/CSS/JS, no build step, no dependencies at runtime other than
  Google Fonts (Noto Sans body text) loaded over the network — everything
  else works offline.
- Same layout system as the Janton and AFO configurators: white
  background, Noto Sans body copy, the licensed Bogue Semibold Italic
  title font, the Eqwal Ability logo top-left, and the soft grey circle
  backdrop behind the seat preview, at a 54/46 hero/panel split (a modest
  step up from Janton/AFO's 50/50). `.page` is also capped at 1900px and
  centered (`margin:0 auto`) as a safety net on ultra-wide monitors.
- The original hand-off's layer PNGs were a 2752×1536 canvas, but the
  chair itself only occupied its middle ~48% (huge transparent margins,
  worst on the right — up to 30% of the canvas). Sizing the viewer to
  fill that canvas meant most of what filled the hero column was dead
  space, not image — and since the panel starts right where the hero
  column ends, that dead space is exactly what was pushing the panel
  column off toward the edge of the screen instead of sitting close to
  the chair the way it does on Janton. Every layer was re-cropped (see
  "Updating an asset" below) to a shared box that tightly bounds the
  chair across every combination of zones and add-ons — 1392×1536,
  still not perfectly square like Janton's art, so the canvas keeps its
  own real aspect ratio via CSS `aspect-ratio` rather than a hardcoded
  1:1 (the same approach AFO uses for its own non-square 1536×2752
  boot artwork) — but now almost entirely image, which is what actually
  lets a modest hero column reproduce a big, close-to-the-panel chair.
- Layer images (bare seat, frontal, side-and-back, back, back lateral,
  seating, abduction wedge, headrest shell/upholstery, footrest
  shell/upholstery) are drawn back to front, each tinted with a canvas
  `multiply` blend against its own artwork so shading/highlights always
  show through, and each zone's mask fully abuts its neighbours —
  there's no separate outline/trim layer drawn on top, since that
  previously left a visible white seam along every zone boundary once
  all zones were coloured. "Complete" — the bare seat with no headrest
  or footrest — is always drawn first, in the back; the headrest/footrest
  layers are only ever drawn additively on top when their toggle is on,
  so there's no cutout/erase step and no dependency on a separate
  "all layers" reference render staying in sync with the individual
  zone masks.
- Vinyl (15), Platilon (4) and Airmesh (7) swatches come from the Eqwal
  Ability reference sheets included in the source hand-off — the same
  material palette used by the Janton configurator.
- Tabs: **Shell** (vinyl on the Frontal and Side and back panels) →
  **Headrest** (optional, toggle switch; when on, choose Vinyl for the
  shell and Platilon/Airmesh for the inside upholstery) → **Footrest**
  (same structure as Headrest, plus a Bare foam upholstery option) →
  **Upholstery** (four collapsible sections — Back, Back lateral,
  Seating, Abduction wedge — each offering Platilon and Airmesh). There
  is no separate "Sides" section: that area is part of the vinyl-colored
  Side and back shell panel, not a soft-upholstery zone.
- **View order** unlocks once both shell panels and all four upholstery
  sections are chosen (and the headrest/footrest finish too, if that
  add-on is switched on). It opens an order-review screen with the full
  breakdown and a **Save as PDF** button that renders the current preview
  image (at 262.5pt wide on the A4 page) plus the full selection list to
  a PDF, entirely client-side.

## Updating an asset

Every layer must stay pixel-aligned to the same 1392×1536 canvas. If the
studio sends a fresh export, it'll be on their original 2752×1536 canvas —
crop it to the box `(585, 0, 1977, 1536)` (left, top, right, bottom) first,
the same box every current layer was cropped to, then base64-encode the
result and paste it into the matching entry in the `LAYER_SRC` object in
`index.html`. If a new zone's content falls outside that box (unlikely,
but the footrest already sits close to its bottom-right edge), the box
needs to grow to fit it — on every layer, `#seat-canvas`'s `aspect-ratio`,
and the JS `LAYER_SRC` loader all assume one shared canvas size, and a
mismatch between layers will visibly misalign them. Vinyl/Platilon/Airmesh
swatches live in the `VINYL` / `PLATILON` / `AIRMESH` arrays near the top
of the inline script.

## Fonts

- Body text uses **Noto Sans**, loaded from Google Fonts (falls back to
  the system sans-serif if offline).
- The title ("Customize your Supportec seating.") uses the licensed
  **Bogue Semibold Italic**, embedded in the `@font-face` rule in
  `index.html` (same asset used by the AFO, Cheneau and Janton
  configurators).
