# svg-character-redraw

An Agent Skill that teaches Claude to redraw a character illustration (for example an anime character sheet) as
hand-written SVG that closely matches a reference image.

The workflow in short:

1. Measure the reference precisely (labelled coordinate grid, palette sampling, hem/silhouette profiles).
2. Build an anatomical construction layer first (skull, hairline, parting, joints, hidden legs).
3. Write a shared brief: one coordinate system, layer order, one owner per element, rules against the "uncanny" look.
4. Split the figure into parts (face, hair, accessories, upper outfit, arms/hands, skirt/boots) and iterate each one at
   3–8x zoom against the reference — in parallel with subagents when available.
5. Assemble, then check every seam zone at zoom (ear/face/hair, shoulder locks, waist, sleeves, background holes).
6. Write fixes that preserve what is already right and respect anatomy (hand orientation, ear ↔ jaw, shading that
   shares its edge with the contour); detect when the reference itself is anatomically inconsistent.

The skill embeds the helper tools (`lib.py`, `assemble.py`, `compare.py`, `grid.py`, palette and hole-scan snippets),
so Claude can recreate the whole working environment in a new session.

## 中文說明

讓 Claude 依照參考圖，用手寫 SVG 重繪角色立繪的技能。流程：精確量測 → 先建人體構造層 → 共用規範 →
拆部位放大迭代 → 組合後逐區檢查接縫 → 依解剖結構下修正指示。比對、格線、組合等工具程式碼都附在技能裡。

## Install

The skill is the folder [`svg-character-redraw/`](svg-character-redraw/) (a single `SKILL.md`).

### Claude app (web / desktop)
1. Make a ZIP whose top level is the skill folder:
   ```bash
   git clone https://github.com/OtohaTouka/svg-character-redraw
   cd svg-character-redraw
   zip -r svg-character-redraw.zip svg-character-redraw
   ```
   (GitHub's "Download ZIP" adds an extra top-level folder, so re-zip the inner `svg-character-redraw/` folder.)
2. In Claude, open **Customize → Skills**, click **+ → Create skill → Upload a skill**, and upload the ZIP.
3. Code execution must be enabled (Settings → Capabilities).

Skills enabled in your Claude account are also loaded by Claude Code (v2.1.273+) when you sign in with the same account.

### Claude Code (manual)
```bash
git clone https://github.com/OtohaTouka/svg-character-redraw
mkdir -p ~/.claude/skills
cp -r svg-character-redraw/svg-character-redraw ~/.claude/skills/
```

### Other agents
`SKILL.md` follows the Agent Skills format, so tools that support it can load the `svg-character-redraw/` folder.

## Requirements at run time
Python 3 with `cairosvg`, `pillow`, `numpy`, `scipy` (Claude installs them if missing). A high-resolution reference
image of the character; detail crops (face, clothing, accessories) help a lot.

## Notes
- Only redraw characters you own, have a licence for, or that are licensed for derivative use; credit the original
  creator and licence in the SVG `<desc>`.
- Hand-written SVG gives clean cel shading and line art; painterly brush gradients and fine hair texture are the ceiling.

## License

[MIT](LICENSE) © 2026 OtohaTouka
