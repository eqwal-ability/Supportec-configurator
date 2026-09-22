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
- The footrest is drawn on its own stacked `<canvas>` (`#footrest-canvas`,
  positioned directly over `#seat-canvas`) rather than the main one. Both
  canvases share the same drop-shadow filter on desktop, so it's visually
  seamless there — but the footrest sits well clear of the main seat, so
  its shadow reads as its own floating blob rather than part of one
  grounded object; on the small mobile hero it lands on plain white
  (rather than the grey circle, like on desktop) and looks like an
  unwanted extra shape, so the mobile media query switches its filter off
  while leaving the main seat's shadow (and the desktop footrest's)
  untouched. The PDF export composites both canvases together.
- On mobile, the hero column is taller than the seat preview needs
  (`min-height: calc(min(100vw,520px) + 130px)`, `+130px` more than the
  image itself requires) so vertical centering pushes the preview down
  far enough that the headrest add-on doesn't sit under the logo. The
  grey circle backdrop's vertical offset is expressed as a `calc()` that
  reproduces hero's *old*, unpadded height formula directly, rather than
  a plain `%` of hero's own (now taller) box — otherwise growing hero to
  make room for the shift would also drag the circle down with it.
- Vinyl (15), Platilon (4), Airmesh (7) and Leather (18) swatches come
  from the Eqwal Ability reference sheets included in the source
  hand-off — Vinyl/Platilon/Airmesh are the same palette used by the
  Janton configurator; Leather's hex values were sampled directly from
  the swatch scans in its own reference sheet.
- Tabs: **Shell** (leather on the Side and back panel only — Frontal
  moved to the Upholstery tab, and Vinyl was dropped here in favour of
  Leather) → **Headrest** (optional, toggle switch; when on, choose
  Vinyl or Leather for the shell and Platilon/Airmesh/Leather for the
  inside upholstery, each pair/trio behind its own subtabs) →
  **Footrest** (same structure as Headrest, plus a Bare foam upholstery
  option — now shown as an actual black tint rather than an untinted
  "natural" swatch) → **Upholstery** (five collapsible sections — Frontal,
  Back, Back lateral, Seating, Abduction wedge — each stacking its own
  finish options: Frontal offers Vinyl/Leather like a shell panel, the
  other four offer Platilon/Airmesh/Leather). There is no separate
  "Sides" section: that area is part of the Side and back shell panel,
  not a soft-upholstery zone.
- **View order** unlocks once the shell panel and all five upholstery
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
mismatch between layers will visibly misalign them. Vinyl/Platilon/
Airmesh/Leather swatches live in the `VINYL` / `PLATILON` / `AIRMESH` /
`LEATHER` arrays near the top of the inline script; which materials each
zone offers, and in what order, is set per zone in `UPHOLSTERY_ZONES`'
`materials` field (`buildAccordion()` renders whatever is listed there),
and separately in `buildAddonFinish()` for the Headrest/Footrest tabs.

## Fonts

- Body text uses **Noto Sans**, loaded from Google Fonts (falls back to
  the system sans-serif if offline).
- The title ("Customize your Supportec seating.") uses the licensed
  **Bogue Semibold Italic**, embedded in the `@font-face` rule in
  `index.html` (same asset used by the AFO, Cheneau and Janton
  configurators).
