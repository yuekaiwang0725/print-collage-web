---
name: print-collage-web
description: Build web pages in an editorial print-collage style (印刷拼贴风) — a serif headline that looks pressed in ink, a dusk horizon glowing behind it, torn photo scraps pasted on the edges, film grain over everything, a dotted-leader table of contents and letter-spaced all-caps eyebrows. Pure HTML/CSS/SVG filters, no images needed. Use this skill whenever the user asks for 印刷拼贴风 / 复古印刷风 / 杂志感 / 胶片颗粒 / 撕纸拼贴 / 油墨字 / "像 Anthropic 发布页那种风格" / editorial, letterpress, risograph, grainy, collage, zine or magazine-cover look for a landing page, launch or announcement page, product page, event page, portfolio, personal homepage, report cover, HTML slides or poster — even if they only say "make it look printed / vintage / textured / like a magazine" without naming this style.
---

# Print-Collage Web Style

A page that looks printed and pasted together rather than rendered: ink on paper, photo scraps, grain. The reference implementation is `assets/template.html` (named "Horizon Press"). Read it before building — it is the source of truth for structure, tokens and filter parameters. `references/techniques.md` explains each effect, its tuning knobs, and palette variants; read it when you need to change an effect rather than copy it.

## The six signatures

A page reads as this style when most of these are present. They work together; dropping two or three makes it look like a generic dark landing page.

1. **Ink-pressed serif headline** — large serif display type with SVG filter `#ink`: edges displaced by noise, slight blur, random pinholes punched out. This is the single most recognisable element, so give the headline room and keep it short (1–4 words).
2. **Horizon arc** — a giant dark circle (the planet's edge) whose layered `box-shadow` paints a sunset glow and blue sky above it. The headline sits right on the horizon line so the glow passes behind the lower half of the letters.
3. **Torn collage scraps** — vertical strips on the left/right edges of the hero: coloured card stock on top, a "photo" (dunes, rock, water, foliage…) below, edges ripped by filter `#torn`. The photo textures are CSS gradients warped by displacement filters, or real images if the user provides them.
4. **Film grain** — an inline SVG `feTurbulence` tile blended over the hero (`overlay`) and lightly over the paper sections (`multiply` in light, `screen` in dark).
5. **Book-style TOC** — `(1) ........ Introduction 介绍` with a dotted leader, linking to the sections below.
6. **Magazine typography** — serif for voice (headline, body), grotesk sans for labels; eyebrows/dates in 12px caps with ~0.12em tracking; headings tightly tracked (−2.5% to −3.5%).

## Workflow

1. **Pin down the subject.** What is the page about, who reads it, what is its one job? Collect the real content (title, date, sections). Don't use lorem ipsum — write plausible real copy in the user's language. If the user writes in Chinese, use bilingual labels the way the template does (`Introduction 介绍`) unless they want one language.
2. **Start from the template.** Copy `assets/template.html` and edit it rather than writing from scratch. The filters, the arc geometry and the scrap layering are fiddly; the template has tuned values that work together.
3. **Adapt the world to the subject.** The default is "dusk from orbit + desert". Pick a scene that belongs to the subject (see palette variants in `references/techniques.md`): change the arc glow colours, the sky gradient, and what the two scraps depict. Keep one dominant warm/cool contrast — the glow line is the accent.
4. **Replace content.** Wordmark, nav, date (both languages if bilingual), headline, TOC entries, sections. Keep TOC `href`s and section `id`s in sync. Delete the template's own demo sections ("Typography", "Texture", "How to use") — they document the template and aren't meant for the user's page — unless the user wants a style guide.
5. **Body sections.** The cream "paper" sections use a label column + body column grid with hairline rules between chapters. Reuse that rhythm; add components (quotes, figures, stat rows, cards) sparingly and style them with the same tokens. Structural numbering `(1) (2)` should mirror the TOC.
6. **Check it.** Open the page if you can. Look for: headline still legible after the ink filter (raise font size or lower filter `scale` if not), scraps not covering content at phone width, grain not too heavy, dark mode readable.

## Rules that keep it from breaking

- **Ink filter only on big type** (≥ ~40px). On small text the pinholes destroy legibility. Body text stays clean.
- **Grain is seasoning.** Hero grain can be strong because the photo hides it; on paper sections keep opacity around 0.15–0.25 with `multiply` (≈ 0.03–0.12 if you use `normal` blend). Above that it looks like a rendering bug.
- **Hero height comes from content**, not `100vh`, so the horizon position stays tied to the headline. The arc is positioned relative to `.stage` (the date + headline block): `top: calc(100% - .32em)` with the headline's font-size.
- **Hero is always night**; only the paper sections follow light/dark theme. Keep hero colours as fixed tokens (`--night`, `--night-ink`) and paper colours as themed tokens with both dark blocks.
- **Phone width**: scraps shrink to ~14px (`--scrap-w` clamp), nav collapses to the CTA, the two-column chapters stack. Never let the body scroll horizontally — the hero needs `overflow: hidden` because the arc is ~280vw wide.
- **Fonts**: Google Fonts only if publishing somewhere with a strict CSP. Defaults: Fraunces (display/body, `SOFT 50`), Noto Serif SC (Chinese serif), Hanken Grotesk + Noto Sans SC (UI), JetBrains Mono (code). Swap for other serif/grotesk pairs if the subject calls for it, but keep the serif-voice / sans-label split.
- **Reduced motion**: the grain flicker and headline "develop" animation must stop under `prefers-reduced-motion`. The headline animation starts from a visible state (opacity .4), never from 0.
- **Don't impersonate brands.** The style is inspired by Anthropic's Claude launch pages; reproduce the *techniques*, not their logo, wordmark (e.g. a backslash in the name), proprietary fonts or copy, unless the user is that brand.
- **Safari**: CSS `filter: url(#svgFilter)` on HTML text is less consistent there. If the ink effect fails the headline simply renders clean, which is acceptable — don't add hacks.

## Output

A single self-contained `.html` file (inline CSS, inline SVG filters, Google Fonts link as the only external resource). If the host has its own page rules (e.g. an artifact publisher that wraps the page in its own `<html>/<head>`), follow those and drop the template's doctype wrapper accordingly.

For HTML slides in this style: one hero-style slide per section title (arc + ink headline), paper-style slides for content, same tokens and grain.
