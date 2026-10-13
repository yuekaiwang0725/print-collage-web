# Slides in the print-collage style

Contents
1. Deck shell (canvas, scaling, navigation, notes)
2. Slide types
3. Restyling an existing .pptx
4. Pitfalls

---

## 1. Deck shell

One HTML file, all slides inside a fixed 1600×900 canvas that is scaled to fit the window. Designing at a fixed size means every slide looks the same on a laptop, a projector or a phone.

```html
<div class="viewport"><div class="canvas" id="canvas">
  <section class="slide night cover active" data-notes="…">…</section>
  <section class="slide paper" data-notes="…">…</section>
</div></div>
<aside class="notes" id="notes" hidden><span class="lbl">Speaker notes</span><p id="notes-text"></p></aside>
<nav class="hud">← 01 / 09 → · Notes · Full screen</nav>
```

```css
html, body { height: 100%; } body { margin: 0; overflow: hidden; background: var(--night); }
.viewport { position: fixed; inset: 0; overflow: hidden; }
.canvas { position: absolute; left: 50%; top: 50%; width: 1600px; height: 900px;
  transform: translate(-50%, -50%) scale(var(--s, .5)); }
.slide { position: absolute; inset: 0; overflow: hidden; isolation: isolate;
  opacity: 0; visibility: hidden; transition: opacity .45s ease, visibility 0s linear .45s; }
.slide.active { opacity: 1; visibility: visible; transition: opacity .45s ease; }
```

```js
const fit = () => canvas.style.setProperty('--s', Math.min(innerWidth / 1600, innerHeight / 900));
```

Navigation to include: ←/→, Space, PageUp/PageDown, Home/End, click on empty canvas = next, swipe on touch, `#3` in the URL opens slide 3, **N** toggles speaker notes (from `data-notes`), **F** toggles fullscreen (wrap `requestFullscreen()` in try/catch; some viewers refuse it). Keep a small HUD in the bottom-right corner, half-transparent until hovered. Don't also print a page number in the slide footer — it collides with the HUD.

Under `prefers-reduced-motion`, stop the grain flicker and the slide fade.

## 2. Slide types

**Night slide** (cover, section opener, closing) — the hero from `assets/template.html` squeezed into the canvas:
- gradient sky. A horizon (arc or flat band, see techniques.md §2/§2b) is optional; if used, put it on the cover only or cover + closing, not on every night slide. For an arc at canvas scale use a fixed circle (`width/height: 4600px`) instead of `280vw`.
- For a lighter deck (school, internal reports), night slides can be swapped for paper "title" slides: cream paper, big dark ink title, scraps on the edges.
- scraps on both edges at `--scrap-w: 72px`; vary the card/photo split on different night slides so they don't look copy-pasted.
- brand in letter-spaced caps top-left (glow colour), meta eyebrow top-right.
- cover: date eyebrow, ink title ~150px, dotted-leader TOC of the deck's parts (clickable, jumps to the slide).
- closing: two ink headings side by side (on the horizon if there is one) (e.g. "Next" / "Your view", second one in warm `#ffc27a`), lists below in the dark.

**Paper slide** (content) — grid rows `auto auto 1fr auto`, padding `64px 96px 56px`:
- top row: `(n) SECTION NAME` eyebrow (number in ember) left, brand right.
- title: 64px serif with the lighter `#ink-sm` filter (displacement scale ~1.1, pinhole intercept ~7.1) — the full `#ink` is too rough at this size.
- body: the content.
- foot: hairline rule + italic serif caption (the slide's takeaway sentence).
- grain overlay at `opacity .22; mix-blend-mode: multiply`.

Paper-slide components:
- **Pasted photo** for screenshots: white 12px mat, soft shadow, `rotate(-0.9deg)` / `rotate(0.7deg)` alternating, a strip of masking tape on top (`::before`, beige translucent, lightly torn with a small displacement filter). Two side by side with a 64px gap; `max-height ~520px`.
- **Torn card** for lists: put the background and `filter: url(#torn)` on a `::before` layer (`z-index: -1`) — never on the card itself, or the text gets warped too. Yellow card stock for "live / done", off-white paper for "in progress".
- **Status tag**: small caps + a dot (`ink` = available, `ember` = in development).
- **Line art / diagrams** with white backgrounds: `mix-blend-mode: multiply` so they print onto the paper.
- **Steps**: numbered `(1) (2) (3)` only when it is a real sequence; join with dotted leaders ending in a small arrowhead.
- **Stamp** for a call to action ("Live demo next"): ember border, caps, `rotate(-5deg)`, full `#ink` filter.

Fonts: Fraunces + Hanken Grotesk (add Noto Serif SC / Noto Sans SC for Chinese).

## 3. Restyling an existing .pptx

1. **Read it.** Dump every slide's text, shapes and speaker notes (python-pptx, or markitdown), and render thumbnails to see the original layout.
2. **Extract media.** Save each picture's blob from the slide (`shape.image.blob`, `shape.image.ext`). Note any crop values. Icons can be reused and turned into ink with `filter: brightness(0) saturate(0) opacity(.85)`.
3. **Map slides to types.** Title → night cover; closing / next steps / Q&A → night closing; everything else → paper. If the notes describe the talk's parts, turn them into the cover TOC and the section eyebrows.
4. **Keep the content faithful.** Same titles, bullets, captions and numbers; carry every speaker note into `data-notes`. Don't move facts from the notes onto the slides unless asked — the presenter kept them off the slide on purpose.
5. **Build.** Write the slide HTML with placeholders like `{{s3_0}}` for images, then a small script swaps them for `data:` URIs so the deck is one self-contained file that works offline. A deck of a dozen 2000px screenshots ends up around 3 MB, which is fine.
6. **Leave the original .pptx untouched**; write the HTML next to it or in the working folder.

## 4. Pitfalls

- Don't centre the canvas with `display: grid; place-items: center` — the 1600px canvas makes the grid track 1600px wide, so in a narrow window it overflows to the right and the right edge (scraps, HUD) gets cut off. Centre with `left/top: 50%` + `translate(-50%, -50%)` as above.

- Headless Chrome screenshots: `--timeout=4000`, not `--virtual-time-budget` (infinite grain animation → hang). Hard-limit each run.
- `transform` on a card creates a stacking context; that's what keeps its `::before` (`z-index: -1`) behind the text but inside the card. A card without a transform needs `position: relative; z-index: 0`.
- Long titles: the canvas is fixed, so a 60-character title at 64px still fits on one line (≈1400px) but anything longer must wrap; `text-wrap: balance` keeps it tidy.
- Screenshots with small UI text: don't rotate more than ~1°, or the text gets blurry.
