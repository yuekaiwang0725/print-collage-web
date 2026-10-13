# print-collage-web · 印刷拼贴风网页

A Claude skill for building web pages that look printed and pasted together rather than rendered: an ink-pressed serif headline, torn photo scraps on the edges, film grain, a dotted-leader table of contents and letter-spaced caps eyebrows. The glowing horizon arc is one optional hero backdrop, not a requirement.

一个 Claude skill：让 Claude 做出「像印出来、再剪贴拼起来」的网页 —— 油墨压印的衬线大标题、两侧撕纸拼贴、胶片颗粒、点线目录、字距拉开的全大写眉题。地平线（日全食式的弧形光晕）只是可选的首屏背景之一。

Everything is HTML, CSS and inline SVG filters. No images, no JavaScript framework. 全部由 HTML、CSS 和内联 SVG 滤镜实现，不用图片，不用框架。

## Install 安装

**Claude Code**

```bash
git clone https://github.com/yuekaiwang0725/print-collage-web ~/.claude/skills/print-collage-web
```

**Skills CLI**

```bash
npx skills add https://github.com/yuekaiwang0725/print-collage-web
```

**Claude.ai / Claude desktop**: download this repo as a zip and upload it in Claude's skill settings. 下载 zip 后在 Claude 的 Skills 设置里上传。

## Use 使用

Just ask for the style. 直接描述你要的页面：

> 用印刷拼贴风做一个我们读书会的活动页面，10月25日，主题是《沙丘》

> Make a launch page for our app in that grainy, printed magazine-collage style

> 把 ~/Downloads/季度汇报.pptx 改成油墨风的 HTML 幻灯片

The skill also triggers on 油墨风 / 油墨画风 / 复古印刷 / 杂志感 / 胶片颗粒 / 撕纸拼贴 / editorial / letterpress / risograph / collage / zine.

## What's inside 内容

| File | Purpose |
|---|---|
| `SKILL.md` | When to use it, workflow, the six signatures of the style, rules that keep it from breaking |
| `references/techniques.md` | Every effect with code and tuning knobs (ink, horizon arc, torn edges, grain, TOC), 5 scene palettes, using real photos |
| `references/slides.md` | HTML slide decks in this style, and how to restyle an existing .pptx 把现有 PPT 改成这个风格 |
| `assets/template.html` | "Horizon Press", the reference page. Open it in a browser to see the style 用浏览器打开即可预览 |

## Credits

Inspired by the visual language of Anthropic's Claude model launch pages. This skill reproduces the techniques only; it contains no Anthropic logos, fonts or copy. Fonts are from Google Fonts (Fraunces, Noto Serif SC, Hanken Grotesk, Noto Sans SC, JetBrains Mono).

## License

[MIT](LICENSE)
