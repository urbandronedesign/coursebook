# Diagrams — hand-written inline SVG

A course manual needs 5–10 real diagrams. Not decoration: the pipeline, the folder structure, the shot sizes, the thing that cannot be said in a paragraph.

**Always inline SVG. Never an image of a diagram.** A raster diagram cannot be searched, does not follow the theme (a light diagram on a dark page looks broken), blurs when printed, and cannot be corrected without the original file — which will be lost.

---

## The wrapper

```html
<figure class="diagram">
  <svg viewBox="0 0 700 220" role="img" aria-labelledby="fig-pipe-t">
    <title id="fig-pipe-t">Le pipeline en sept étapes</title>
    <defs>
      <marker id="ah" viewBox="0 0 8 8" refX="7" refY="4"
              markerWidth="6" markerHeight="6" orient="auto">
        <path d="M0 0 L8 4 L0 8 z" fill="var(--ink)"/>
      </marker>
    </defs>
    …
  </svg>
  <figcaption class="fr-only">Ce qu'il faut regarder.</figcaption>
  <figcaption class="en-only">What to look at.</figcaption>
</figure>
```

- `viewBox` and no `width`/`height` — the CSS makes it fluid.
- A `<title>` with an `id` referenced by `aria-labelledby`. It is the accessible name and costs one line.
- Arrow markers go in `<defs>`. **Marker ids are global to the page**: if two diagrams both define `#ah`, the second wins everywhere. Either define it once in the first diagram, or suffix per figure (`#ah-pipe`).
- Wrap a diagram wider than ~660 px in `<div class="diagram-wide">` so it scrolls on a phone instead of squashing.

---

## Colours: never literal

Every stroke and fill uses a variable, so the diagram follows light/dark/print automatically.

| Class | Use |
|---|---|
| `dg-band` | a box on the paper background |
| `dg-fill` | a filled panel, no stroke — for emphasis behind a group |
| `dg-node` | a node with a grey outline |
| `dg-deliver` | a box that matters more (a deliverable, an output) |
| `dg-ink` | a plain line |
| `dg-good` / `dg-bad` | accent / warning strokes — right way, wrong way |
| `dg-dash` | a secondary or optional relation |
| `dg-grid` | rules, baselines, axes |
| `dg-arrow` | a line with the arrowhead marker |
| `dg-return` | dashed warning arrow — the return path |

Text:

| Class | Use |
|---|---|
| `dg-tb` | box title, 12 px semibold |
| `dg-ts` | annotation, 10.5 px grey |
| `dg-tn` | monospace, ink — filenames, values |
| `dg-tm` | monospace, accent — parameters |
| `dg-t` | 12 px body, for longer labels |
| `dg-numt` | white number inside a filled circle (step markers) |

If you need a shape the classes do not cover, add a `dg-*` class to the stylesheet using existing variables. Do not inline `fill="#333"` — it will be invisible in dark mode or black-on-black in print.

---

## Bilingual labels

Duplicate the `<text>` element with the same coordinates:

```svg
<text class="dg-tb fr-only" x="18" y="50">Modèle</text>
<text class="dg-tb en-only" x="18" y="50">Model</text>
```

`display:none` works on SVG elements, so the language rules apply unchanged. For a group of labels, wrap in `<g class="fr-only">`.

Numbers, filenames and parameter names need no duplication. Do not duplicate a whole diagram just for two labels — but **do** duplicate the whole `<figure>` if the diagram lives inside a language block and both editions need it (suffix the second id with `-en`).

The single most common diagram bug in this format: French labels hardcoded with no `.en-only` twin. The English reader sees a French diagram, and no automated check catches it. Grep your `<text>` elements for accented characters without a sibling.

---

## Drawing without a drawing tool

Positioning by hand is fine if you work on a grid. Pick a column width and stick to it.

```
Boxes:     x = 8, 160, 318, 486, 618      (rhythm, not symmetry)
Box title: box.x + 10,  box.y + 20
Sub-label: box.x + 10,  box.y + 36
Arrows:    from box.x+w+4 to next.x-4, mid-height
```

Practical notes:

- **Baselines, not centres.** SVG `y` on `<text>` is the baseline. Title at `y+20` inside a box starting at `y` looks right for a 46 px box.
- `text-anchor="middle"` for centred labels — do not compute string widths.
- `vector-effect="non-scaling-stroke"` on strokes that must stay hairline when the SVG scales up.
- `clipPath` when content must be cropped to a frame (a shot-size figure, a viewport).
- Reuse a repeated motif with `<defs><g id="person">…</g></defs>` and `<use href="#person" transform="translate(…) scale(…)"/>`.
- Keep a diagram under ~60 elements. Past that, it is two diagrams.

---

## What to draw

The five that earn their place in almost any course manual:

1. **The whole pipeline on one page**, with the stages, what each decides, and **the return path** — a dashed warning arrow showing that a fault found at stage 6 was made at stage 3. This single arrow changes how students read the entire manual.
2. **The project folder** as a drawn container, so the structure is a shape and not a list.
3. **The decision that has two routes** — where a course forks and rejoins.
4. **The scale or gradient** the vocabulary depends on (shot sizes, quality levels, zoom levels).
5. **The one thing everyone gets wrong**, drawn right way / wrong way side by side with `dg-good` and `dg-bad`.

Diagrams that do not earn their place: an "ecosystem" of logos, a four-quadrant matrix with nothing in the quadrants, a numbered list drawn as boxes.

---

## Checking

- Toggle FR/EN — every label switches, nothing overlaps.
- Toggle dark — nothing disappears, nothing goes black on black.
- Print preview — `.diagram svg{max-height:9cm}` applies; the diagram is not two pages.
- Phone width — either it scales legibly or it scrolls in `.diagram-wide`.
- Two diagrams on one page — arrowheads still render (marker id collision).
