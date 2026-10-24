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
7. Optionally draw a base body under the outfit (fitted base layer, head-unit proportions checked for widths as well as
   heights, continuous limbs across joints, fitted cuffs and collar) and verify it never peeks out of the outfit view.
8. Optionally dress the same base body, face and pose in another outfit / hair style taken from a different character
   sheet: outfit sets in the assembler, the other sheet aligned onto the canvas with distortion-free transforms, the
   sheet's own pose stripped from the anchors, every garment fitted to the base body, every other set verified unchanged.

The skill embeds the helper tools (`lib.py`, `assemble.py` with outfit sets, `compare.py`, `grid.py`, `views.py`,
`peek.py`, an alignment pattern, palette and hole-scan snippets),
so Claude can recreate the whole working environment in a new session.

## 中文說明

讓 Claude 依照參考圖，用手寫 SVG 重繪角色立繪的技能。流程：精確量測 → 先建人體構造層 → 共用規範 →
拆部位放大迭代 → 組合後逐區檢查接縫 → 依解剖結構下修正指示 →（選用）在衣服底下畫素體，
並確認素體不會從衣服下露出 →（選用）把另一張設定圖的服裝和髮型移植到同一個素體上（多套服裝切換、
參考圖只用旋轉加等比例縮放對齊、去除原圖姿勢造成的形狀、每次修改都確認其他服裝不變）。
比對、格線、組合、素體檢查、參考圖對齊等工具程式碼都附在技能裡。

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
