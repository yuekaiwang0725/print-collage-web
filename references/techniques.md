# Techniques and tuning

Contents
1. Ink filter (`#ink`)
2. Horizon arc (optional)
2b. Flat horizon band (optional)
3. Torn scraps and fake photo textures
4. Film grain
5. Dotted-leader TOC
6. Palette variants
7. Using real photos

---

## 1. Ink filter (`#ink`)

```html
<filter id="ink" x="-4%" y="-15%" width="108%" height="130%" color-interpolation-filters="sRGB">
  <feTurbulence type="fractalNoise" baseFrequency="0.9" numOctaves="2" seed="4" result="n"/>
  <feDisplacementMap in="SourceGraphic" in2="n" scale="2" xChannelSelector="R" yChannelSelector="G" result="rough"/>
  <feGaussianBlur in="rough" stdDeviation="0.5" result="soft"/>
  <feComponentTransfer in="soft" result="ink"><feFuncA type="linear" slope="1.2"/></feComponentTransfer>
  <feTurbulence type="fractalNoise" baseFrequency="0.75" numOctaves="2" seed="12" result="s"/>
  <feColorMatrix in="s" type="matrix" values="0 0 0 0 0  0 0 0 0 0  0 0 0 0 0  -8 0 0 0 6.6" result="holes"/>
  <feComposite in="ink" in2="holes" operator="in"/>
</filter>
```

Pipeline: noise → push stroke edges around (rough edge) → blur (ink bleed) → firm up alpha → second noise turned into a mask with a few transparent specks → keep ink only where the mask is opaque (missed-ink pinholes).

Knobs:
| Param | Effect | Range |
|---|---|---|
| `feDisplacementMap scale` | edge roughness | 0.6 (subtle, Anthropic-like) – 4 (woodcut) |
| `stdDeviation` | ink bleed | 0.3 – 1 |
| colour-matrix last row `-8 … 6.6` | pinhole density; lower the 6.6 → more holes | 6.0 (heavy wear) – 7.5 (almost none) |
| `baseFrequency` of 2nd noise | pinhole size; lower → bigger blotches | 0.4 – 1.0 |
| `seed` | different but equally random pattern | any |

Scale the roughness with font size: values that look right at 160px look mushy at 48px. For a second, smaller ink heading, make a copy `#ink-sm` with `scale="1"` and the matrix intercept at ~7.

## 2. Horizon arc (optional)

One of several hero backdrops, not a requirement — see "Hero backdrop" in SKILL.md.

```css
.stage { position: relative; }
.stage::before {
  content: ""; position: absolute; z-index: 0;
  left: 50%; top: calc(100% - .32em);      /* .32em of headline size: horizon crosses the letters' lower third */
  font-size: clamp(56px, 11vw, 168px);     /* same as h1 so em resolves to headline size */
  width: max(2600px, 280vw); aspect-ratio: 1; transform: translateX(-50%);
  border-radius: 50%; background: var(--night);
  box-shadow:
    0 -2px 3px   #ffe2b0,                  /* hot rim */
    0 -8px 18px  #ff9a4a,                  /* orange band */
    0 -26px 50px rgba(240,100,40,.85),
    0 -70px 110px rgba(210,90,60,.55),     /* red haze */
    0 -170px 180px rgba(140,150,200,.55),  /* lavender transition */
    0 -320px 260px rgba(90,135,200,.6);    /* blue sky lift */
}
.hero { background: linear-gradient(#060d22 0%, #12285a 45%, #2b4f8c 100%); overflow: hidden; }
```

- A wider circle → flatter horizon (more "from orbit"); narrower (e.g. 160vw) → visible curvature.
- Move the horizon by changing `.32em`: `0` puts it under the baseline, `.6em` cuts through the middle.
- The shadow stack *is* the sky. Reorder colours from hot (inner) to cool (outer).

## 2b. Flat horizon band (optional)

A straight glow line instead of the curved planet edge. Calmer, reads as landscape rather than orbit.

```css
.stage::before {
  content: ""; position: absolute; z-index: 0; left: -50vw; right: -50vw;
  top: calc(100% - .32em); bottom: -2000px;          /* the "ground" below the line */
  font-size: clamp(56px, 11vw, 168px);
  background: var(--night);
  box-shadow: 0 -2px 3px #ffe2b0, 0 -10px 24px #ff9a4a, 0 -50px 90px rgba(230,100,50,.6), 0 -200px 220px rgba(110,140,200,.55);
}
```
For no horizon at all, delete the `.stage::before` rule and let the hero gradient run to the bottom (end it in a dark colour so the TOC stays readable).

## 3. Torn scraps and fake photo textures

Edge rip — anisotropic noise (low horizontal, high vertical frequency) displaces the rectangle edges into fibres:

```html
<filter id="torn" x="-10%" y="-5%" width="120%" height="110%">
  <feTurbulence type="fractalNoise" baseFrequency="0.05 0.35" numOctaves="3" seed="7"/>
  <feDisplacementMap in="SourceGraphic" scale="12" xChannelSelector="R" yChannelSelector="G"/>
</filter>
```
For a horizontal tear (top/bottom edges) swap to `baseFrequency="0.35 0.05"`. Raise `scale` for a rougher rip.

Fake photos = striped/radial gradients warped by low-frequency displacement:
- **Dunes** (`#dune`, `baseFrequency 0.012 0.03`, scale 38): `repeating-linear-gradient(168deg, sand, dark shadow, mid sand…)`.
- **Rock** (`#rock`, `0.03 0.09`, scale 46): vertical red/orange stripes + a highlight and shadow radial.
- **Water**: `repeating-linear-gradient(178deg, #0f3a52 0 4px, #2f7392 4px 7px, #a9d3df 7px 8px)` + `#dune`-like filter at scale ~20.
- **Foliage**: radial-gradient dots in 3 greens on dark, filter `baseFrequency 0.05`, scale 30.
- **Card stock**: flat gradient of a single saturated colour (yellow `#f1c43a`, red `#d8452b`, blue `#2b55c8`, black `#121110`).

Each scrap: two pseudo-elements stacked (card on top ~58–62%, photo below), the whole strip filtered by `#torn`. Use different splits left vs right so it looks hand-pasted. A slight `rotate(-1deg … 2deg)` on a scrap adds to the collage feel.

## 4. Film grain

```css
--grain: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='240' height='240'><filter id='g'><feTurbulence type='fractalNoise' baseFrequency='.85' numOctaves='3' stitchTiles='stitch'/><feColorMatrix values='0 0 0 0 .5  0 0 0 0 .5  0 0 0 0 .5  0 0 0 1.6 -.3'/></filter><rect width='100%' height='100%' filter='url(%23g)'/></svg>");
```
- `#` must be URL-encoded as `%23` inside the data URI.
- `stitchTiles='stitch'` hides tile seams.
- Hero: `mix-blend-mode: overlay; opacity: .9` plus a vignette radial-gradient. Paper: `position: fixed` overlay, `multiply` (light) / `screen` (dark) at .15–.25.
- Flicker: animate `background-position` with `steps(4)` over ~1.2s. Disable under reduced motion.
- `baseFrequency` .6 = coarse 35mm grain, 1.2 = fine.

## 5. Dotted-leader TOC

```html
<a href="#intro"><span class="n">(1)</span><span class="dots"></span><span class="label">Introduction<span class="zh">介绍</span></span></a>
```
```css
.toc a { display: flex; align-items: baseline; gap: 12px; }
.toc .dots { flex: 1; border-bottom: 1px dotted rgba(243,236,224,.45); transform: translateY(-5px); min-width: 24px; }
```

## 6. Palette variants

Pick the scene from the subject's world. Change: hero gradient, arc `box-shadow`, scrap content, paper `--accent`.

| Scene | Hero gradient (top → bottom) | Glow stack (inner → outer) | Scraps | `--accent` |
|---|---|---|---|---|
| **Dusk from orbit** (default) | `#060d22 → #12285a → #2b4f8c` | `#ffe2b0, #ff9a4a, rgba(240,100,40), rgba(210,90,60), lavender, blue` | yellow card + dunes / black + red rock | `#c4552b` |
| **Dawn coast** | `#1b1f3a → #5a5f8f → #c7a6b8` | `#fff4d6, #ffd08a, #f6a07a, pink rgba(230,140,160), pale blue` | pale blue card + water / cream + sand | `#c8607a` |
| **Polar night / aurora** | `#02060c → #071a24 → #0b3a3c` | `#d9fff0, #7ff2c1, rgba(60,200,150), rgba(80,90,200), deep violet` | black + ice (white/blue stripes) / green card + foliage | `#2f8f7a` |
| **High noon, mineral** | `#9ec3e6 → #cfe1ef → #f3e9d6` (light hero; set `--night-ink` dark) | `#fff, #fff1c9, rgba(255,220,150), rgba(200,220,240)` | terracotta card + rock / ochre + dunes | `#b5532a` |
| **Ember / volcanic** | `#0a0605 → #2a0d08 → #5a160c` | `#fff1c0, #ffb000, #ff5a1a, rgba(160,20,10), rgba(60,10,10)` | black + rock / red card + ash (grey stripes) | `#d2451e` |

For a light hero (noon), flip hero text tokens to dark and use `multiply` grain instead of `overlay`.

## 7. Using real photos

Replace a scrap's pseudo-element background with `url(photo.jpg) center / cover` and keep the `#torn` filter on the strip — the rip still applies. Remove that scrap's `#dune`/`#rock` filter (it would warp the photo). A photo can also become the whole hero sky: set it as `.hero` background, keep the arc for the horizon or remove it, keep grain and vignette on top. Desaturate slightly (`filter: saturate(.85) contrast(1.05)`) so it sits in the printed look.
