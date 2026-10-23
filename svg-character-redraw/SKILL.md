---
name: "svg-character-redraw"
description: "Redraw a character illustration (e.g. an anime character sheet) as hand-written SVG closely matching a reference, via parts iterated at zoom on a shared anatomy layer, then assembly and seam fixes."
license: MIT
---

# SVG character redraw (reference-matched, part-by-part)

Use when someone wants a character, figure or similarly detailed illustration drawn as hand-written SVG that
should look like a given reference image (full-body standing pose, character sheet, mascot design), not a loose sketch.
It produced a full-body anime character whose assembled SVG is hard to tell from the reference at normal viewing size.

Core idea: **measure first, build on an anatomical construction, split into parts that are iterated at high zoom against
the reference in ONE shared coordinate system, then assemble and fix the seams.** Most visible flaws come from the seams
between parts and from fixes that ignore anatomy — budget for that phase.

Only redraw characters the person owns, has a license for, or that are clearly licensed for derivative non-commercial
use; put the original creator and licence in the SVG `<desc>`. Decline protected commercial characters.

---

## 0. Expectations to state up front
- Hand-written SVG = flat cel shading + line art. Painterly soft gradients, textured brushwork and fine hair texture are the
  ceiling; say so before starting.
- A first naive attempt (one file, guessed coordinates, whole-figure comparison at small size) lands in the uncanny valley:
  realistic proportions drawn with stiff placeholder shapes. A stylised chibi is the safe alternative if the person prefers.
- Ask the person which they want if unclear: close match to the reference, or a stylised/cute redraw.

## 1. Gather references
- Get the highest-resolution reference available (a repo/setting sheet with close-ups of face, expressions, clothing,
  accessories is ideal). If a URL won't fetch as an image, ask the person to attach it.
- Crop the main target view to a tight box and use **its pixel size as the SVG viewBox** (e.g. 937 x 1678). Every part uses
  these coordinates; never rescale parts at assembly.
- Crop one detail reference per part (face/expressions, clothing pieces, accessories) into `ref/parts/`.
- Sample the palette with k-means (snippet below) per area: hair, skin, eyes, cloth, gold, black. Real values beat guesses.
- Measure silhouettes that the eye is sensitive to, e.g. a skirt hem profile: per x column, the lowest non-white y.
  (Hems, sleeve bottoms and every band that runs across fabric must follow these curves — a horizontal hem was the
  first thing a viewer noticed.)

## 2. Measure anchors precisely (tools/grid.py)
Zoom the reference with a labelled coordinate grid (6–10x) and write anchors to ±4 px: eye corners and iris centres,
lash line, nose, mouth, chin, face contour points, ear, neck, collar, brooch, shoulder line, cape corners, sleeve, elbow,
wrist, hand box, corset/waist, accessories, bow centres, hem arc, boot boxes, hair width at eye level and its fan extent.
Eyeballed anchors were off by 10–20 px (eye spacing, a ring position); measure, don't estimate.
Note the true symmetry axis (face and arms may mirror about slightly different x than the body centre).

## 3. Anatomical construction layer (construction.py) — before any part is drawn
Estimate and overlay on the reference (render it and check visually):
- Head: skull ellipse (from ear position, eye line, chin), hairline, parting, crown whorl, ears, jaw, eye line,
  hair thickness outside the skull (~20 px on top, 35–50 px at the sides for voluminous hair).
- Torso: neck, jugular notch, clavicles, trapezius, shoulder joints, rib cage, waist, pelvis crest and hips.
- Limbs: shoulder→elbow→wrist, palm centre and fingertip line; hip→knee→ankle, sole line (infer hidden legs from hips and
  boots). Check head count (~8 heads for adult anime figures) and limb ratios.
Why: without it, hair is traced as an outline (helmet look), the face has nothing for hair to grow from, clothes have no
body inside, and parts read as an inorganic assembly. Hair grows from parting/hairline and wraps the skull; skin extends
under the hair to the skull outline; sleeves wrap limb volumes between joints; cloth hangs from shoulders/waist/hips.

## 4. Shared brief (STYLE.md) — every part worker reads it
Include: project layout, the part-module contract, the global layer order with one owner per layer, all anchors, the
construction summary, the target look and the iteration process below. Template for the look section:
- Clean anime character-sheet rendering: dark warm outlines (outer ~1.8, inner ~0.9 or finer, tapered where useful),
  flat base + one or two shadow shapes following the form + small highlights; gradients only where the reference has
  them (eyes, metal, satin), subtle.
- Cloth and hair edges are curved and slightly irregular; folds radiate from where fabric is gathered/pinned;
  repeated elements vary a little. Never placeholder shapes (mitten hands, ball bows, box frills, straight hems).
- Each part must look good alone at zoom, not just sit in the right place. Match the reference's level of detail.
- Keep files compact: smooth Catmull-Rom/Bezier paths with few points, loops for repeats.

## 5. Split into parts
Split by (a) what overlaps whom and (b) which detail reference exists. A good split for a full figure:
face (+neck), hair (back / shoulder / front layers), accessories (shared motif function used everywhere, ornaments,
neckwear, bows), upper outfit (collar, torso, cape), arms+hands, lower outfit (skirt layers, petticoat, boots).
Rules:
- Each layer and each element has exactly ONE owner (a tassel drawn by two parts showed up twice).
- If a part copies another part's geometry as constants (edges it must meet), record that in the brief; whoever changes
  those edges must report the new points.
- Build one reusable function for a repeated motif (e.g. a flower) and perfect it at high zoom first.

## 6. Iterate each part (in parallel if subagents are available)
Per part: study references at zoom → write structure notes → v1 → render with `tools/compare.py` at 3–8x next to the
reference crop and the detail reference → list the 3 biggest differences in priority order
(silhouette/proportion/placement → structure/folds → rendering → small details) → fix → repeat.
Also ask every round: "does this look appealing, or off?". At least 4 rounds, usually 5–8; save every render and a one-line
log per round; finish with a `--context` render (all parts) and `--guide` (construction overlay).
Work in fixed zones every round (e.g. skirt: upper, mid incl. accessories, hem band, boots) so nothing stays at
"far view" precision. The regions never zoomed are where errors survive.

Subagent prompt skeleton: "Read STYLE.md and lib.py. YOUR PART: <name> → write parts/<name>.py, you own layers <…>.
Character spec for this part: <…>. References: front crop box <…>, detail refs <…>. Compare commands: <…>. Iterate per
STYLE.md, save renders/<name>_rN.png, log to parts/<name>_log.md. Don't edit shared files or other parts. Report: summary,
rounds, what still differs, anchor changes."

## 7. Lead review of each part (do not skip)
Open the part's final render yourself at zoom before accepting it. Typical rejections:
- Hands: tube-like back of hand, noodle fingers of equal width, thumb as a spike. Require a broad back-of-hand plane to a
  knuckle line, 3-segment fingers of different lengths, thumb from the thenar pad on the body side.
- Hand orientation: a hand hanging at rest shows its palm to the body and the back outward; from the front, fingers show
  their thumb-side edge — **no fingernails on the fingers, only the thumb nail is visible**; no creases across the finger
  fronts. Nails, creases and knuckle shading must agree with the orientation of the palm and fingers.
- Bangs: symmetric even clumps where the reference has one dominant asymmetric lock.
- Waist/peplum or any "block" shape where the reference has diagonal strips.
- Rendering: too saturated, every strand outlined in near-black (stripes) — use sampled colours and coloured, thinner
  inner lines; keep the dark line for silhouettes and big clump edges.

## 8. Assemble and fix seams
- `tools/assemble.py` stacks layers in the global order; render full + fixed zones (head, chest, waist, each arm, mid
  skirt, each hem side, boots, each hair fan side) side by side with the reference.
- Seam checklist: ear↔face↔hair, side locks↔shoulder locks (one lock continues, no cut line, no stacked pointed ends),
  hair behind neck, braid and highlights spilling across lock edges, waist pieces, sleeves↔torso↔hair, chains/ribbons
  hidden under other layers, skirt layer gaps.
- Scan for background holes: enclosed white regions in the render (snippet below), then inspect them against the
  reference; small slits between parts can be closed with an underpaint patch part drawn below the covering layers.
- Background showing through between parts is an assembly bug, not "style".

## 9. How to write fix instructions (the big lessons)
- **State what must stay correct, not just what to change.** "Attach the ear" made the jaw run straight to the ear and the
  face looked crooked; the fix had to say "keep the measured convex cheek line".
- **Check anatomy before sending a fix.** "Align the nails with the fingers" produced nails on fingers whose backs face
  away. Think through the 3D orientation first.
- **A separate colour block must share its edge with a contour line**; a shade block whose edge runs elsewhere reads as a
  misaligned second outline. Prefer form shadows centred on the contour (clipped by the skin) with stacked widths for a
  soft falloff.
- **The reference itself can be anatomically inconsistent and hide it** (here: the ear sat ~13 px outside the jaw line
  at lobe height, covered by a lock of hair). When a fix keeps feeling wrong, measure whether the reference is coherent.
  Then either resolve it consistently (e.g. jaw starts at the ear lobe as one convex curve; move ear/earrings slightly
  inward) or hide it the way the reference does — explain the trade-off and let the person choose.
- Moving one part can expose or hide another (moving the ear inward hid it behind a hair lock): re-check neighbours and
  re-measure them against the reference.
- Keep a backup of each part before every person-requested change and show original | before | after crops.

## 10. Deliver
Final SVG (validate XML; credit + licence in `<desc>`), PNG render, full side-by-side comparison, zoom comparisons of the
most scrutinised areas, and a source archive (parts, tools, construction, brief, logs, backups; not the reference images
unless their licence allows). State honestly what still differs.

---

## Project layout
```
project/
  lib.py            palette P, line widths, curve helpers (shared, read-only for part workers)
  construction.py   skull / joints / guide_svg() overlay
  STYLE.md          shared brief
  parts/<part>.py   defs() + layers() per part; <part>_log.md
  tools/assemble.py tools/compare.py tools/grid.py
  ref/front.png     target crop (its size = viewBox);  ref/parts/*.png detail references
  renders/  out/  backup/
```
Part contract:
```python
from lib import *
def defs():   return ''          # gradient/clip ids MUST start with '<part>-'
def layers(): return {'<layer>': '<svg elements>'}   # only layers this part owns
```
Rendering needs `pip install cairosvg pillow numpy scipy --break-system-packages` (if missing).

## tools: lib.py (core helpers)
```python
import math
W, H = 937, 1678          # = size of ref/front.png
CX = 470                  # body centre line
P = dict(  # fill from k-means samples of the reference
    hair='#e8743a', hair_s1='#c85a2c', hair_s2='#9e4228', hair_hl='#f6a06a', hair_line='#5c2416',
    skin='#fcf1ec', skin_s='#f0d2c6', skin_line='#7a4436', cream='#f8f0e9', cream_s='#e8d9cd', cream_line='#5a3a2a',
    black='#2c2626', black_s='#171213', black_hl='#4c4444', gold='#d6a46a', gold_s='#a87348', gold_l='#f3d9a8')
LW_OUT, LW_IN, LW_FINE = 1.8, 0.9, 0.6
def f(v): return f'{v:.2f}'.rstrip('0').rstrip('.')
def pt(p): return f'{f(p[0])} {f(p[1])}'
def smooth_d(pts, close=False, tension=1.0):
    n = len(pts)
    ext = ([pts[-1]] + list(pts) + [pts[0], pts[1]]) if close else ([pts[0]] + list(pts) + [pts[-1]])
    segs = n if close else n - 1
    d = 'M' + pt(ext[1])
    for i in range(1, segs + 1):
        p0, p1, p2, p3 = ext[i-1], ext[i], ext[i+1], ext[i+2]
        c1 = (p1[0] + (p2[0]-p0[0])/6*tension, p1[1] + (p2[1]-p0[1])/6*tension)
        c2 = (p2[0] - (p3[0]-p1[0])/6*tension, p2[1] - (p3[1]-p1[1])/6*tension)
        d += f' C{pt(c1)} {pt(c2)} {pt(p2)}'
    return d + (' Z' if close else '')
def sample_catmull(pts, n=10):
    res = []
    for i in range(len(pts)-1):
        p0 = pts[i-1] if i > 0 else pts[i]; p1, p2 = pts[i], pts[i+1]; p3 = pts[i+2] if i+2 < len(pts) else pts[i+1]
        for k in range(n):
            t = k/n; t2 = t*t; t3 = t2*t
            res.append(tuple(0.5*((2*p1[j]) + (-p0[j]+p2[j])*t + (2*p0[j]-5*p1[j]+4*p2[j]-p3[j])*t2
                                  + (-p0[j]+3*p1[j]-3*p2[j]+p3[j])*t3) for j in (0, 1)))
    res.append(tuple(pts[-1])); return res
def ribbon_pts(center, w0, w1, taper_start=0.0, taper_end=0.15, wfun=None, n=8):
    Q = sample_catmull(center, n); m = len(Q); L = []; R = []
    for i, (x, y) in enumerate(Q):
        a = Q[max(i-1, 0)]; b = Q[min(i+1, m-1)]; dx, dy = b[0]-a[0], b[1]-a[1]; d = math.hypot(dx, dy) or 1
        nx, ny = -dy/d, dx/d; t = i/(m-1); w = wfun(t) if wfun else w0 + (w1-w0)*t
        if taper_end and t > 1-taper_end: w *= max(0.02, (1-t)/taper_end)
        if taper_start and t < taper_start: w *= max(0.02, t/taper_start)
        L.append((x+nx*w/2, y+ny*w/2)); R.append((x-nx*w/2, y-ny*w/2))
    return L, R
def path(d, fill='none', stroke=None, sw=LW_IN, extra=''):
    st = f' stroke="{stroke}" stroke-width="{f(sw)}" stroke-linejoin="round" stroke-linecap="round"' if stroke else ''
    return f'<path d="{d}" fill="{fill}"{st} {extra}/>'
def ribbon(center, w0, w1, fill, stroke=None, sw=LW_IN, extra='', **kw):   # tapered filled stroke: hair locks, lines
    L, R = ribbon_pts(center, w0, w1, **kw)
    return path(smooth_d(L + R[::-1], close=True, tension=0.6), fill, stroke, sw, extra)
def shape(pts, fill, stroke=None, sw=LW_OUT, close=True, smooth=True, extra=''):
    return path(smooth_d(pts, close=close) if smooth else 'M' + ' L'.join(pt(p) for p in pts) + (' Z' if close else ''),
                fill, stroke, sw, extra)
def line(pts, stroke, sw=LW_IN, extra=''): return path(smooth_d(pts), 'none', stroke, sw, extra)
def mirror(svg, cx=CX): return f'<g transform="translate({f(2*cx)} 0) scale(-1 1)">{svg}</g>'
```

## tools/assemble.py
```python
import glob, importlib.util, os, sys, traceback
ROOT = os.path.dirname(os.path.dirname(os.path.abspath(__file__))); sys.path.insert(0, ROOT)
from lib import W, H
ORDER = ['hair_back', 'boots', 'petticoat', 'skirt', 'skirt_acc', 'skirt_bows', 'arms', 'torso', 'cape',
         'hair_shoulder', 'neck', 'collar', 'neckwear', 'face', 'hair_front', 'ornament']   # adapt per figure
def load_parts(only=None):
    mods = {}
    sys.path.insert(0, os.path.join(ROOT, 'parts'))
    for fn in sorted(glob.glob(os.path.join(ROOT, 'parts', '*.py'))):
        name = os.path.splitext(os.path.basename(fn))[0]
        if name.startswith('_') or (only and name not in only): continue
        try:
            spec = importlib.util.spec_from_file_location(f'part_{name}', fn); m = importlib.util.module_from_spec(spec)
            spec.loader.exec_module(m)
            if hasattr(m, 'layers'): mods[name] = m
        except Exception: print(f'[assemble] skip {name}', file=sys.stderr); traceback.print_exc(limit=2)
    return mods
def build(only_parts=None, only_layers=None, viewbox=(0, 0, W, H), bg='#ffffff', title='', desc=''):
    mods = load_parts(only_parts); defs, stacks = [], {k: [] for k in ORDER}
    for name, m in mods.items():
        try:
            if hasattr(m, 'defs'): defs.append(m.defs())
            for ln, svg in m.layers().items():
                if ln not in stacks: print(f'[assemble] {name}: unknown layer {ln}', file=sys.stderr); continue
                if only_layers and ln not in only_layers: continue
                stacks[ln].append(f'<g id="{name}-{ln}">{svg}</g>')
        except Exception: print(f'[assemble] skip {name}', file=sys.stderr); traceback.print_exc(limit=3)
    x0, y0, x1, y1 = viewbox
    return (f'<svg xmlns="http://www.w3.org/2000/svg" viewBox="{x0} {y0} {x1-x0} {y1-y0}" width="{x1-x0}" height="{y1-y0}">'
            f'<title>{title}</title><desc>{desc}</desc><defs>{"".join(defs)}</defs>'
            f'<rect x="{x0}" y="{y0}" width="{x1-x0}" height="{y1-y0}" fill="{bg}"/>'
            + ''.join(''.join(stacks[k]) for k in ORDER) + '</svg>\n')
if __name__ == '__main__':
    open(sys.argv[1], 'w').write(build())
```

## tools/compare.py
```python
# python3 tools/compare.py --part face --box x0,y0,x1,y1 --scale 5 [--layers a,b] [--context] [--guide]
#        [--detail B02.png --detail-box ...] --out renders/face_r1.png        (--part ALL = assembled figure)
# panels: reference crop | part alone | (--context) all parts | (--detail) detail reference
import argparse, io, os, sys, cairosvg
from PIL import Image
ROOT = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
sys.path.insert(0, os.path.join(ROOT, 'tools')); sys.path.insert(0, ROOT)
from assemble import build
ap = argparse.ArgumentParser()
for a_, kw in (('--part', dict(required=True)), ('--layers', {}), ('--box', dict(default='0,0,937,1678')),
               ('--scale', dict(type=float, default=3.0)), ('--context', dict(action='store_true')),
               ('--guide', dict(action='store_true')), ('--detail', {}), ('--detail-box', {}),
               ('--maxw', dict(type=int, default=3000)), ('--out', dict(required=True))):
    ap.add_argument(a_, **kw)
a = ap.parse_args()
x0, y0, x1, y1 = [float(v) for v in a.box.split(',')]; w, h = int((x1-x0)*a.scale), int((y1-y0)*a.scale)
layers = a.layers.split(',') if a.layers else None
def render(parts):
    svg = build(parts, layers if parts else None, viewbox=(x0, y0, x1, y1))
    return Image.open(io.BytesIO(cairosvg.svg2png(bytestring=svg.encode(), output_width=w, output_height=h))).convert('RGB')
panels = [Image.open(os.path.join(ROOT, 'ref', 'front.png')).convert('RGB').crop((int(x0), int(y0), int(x1), int(y1))).resize((w, h), Image.LANCZOS)]
if a.part != 'ALL': panels.append(render(a.part.split(',')))
if a.context or a.part == 'ALL': panels.append(render(None))
if a.guide:
    import construction
    g = f'<svg xmlns="http://www.w3.org/2000/svg" viewBox="{x0} {y0} {x1-x0} {y1-y0}" width="{w}" height="{h}">{construction.guide_svg()}</svg>'
    gi = Image.open(io.BytesIO(cairosvg.svg2png(bytestring=g.encode(), output_width=w, output_height=h))).convert('RGBA')
    for i, p in enumerate(panels): b = p.convert('RGBA'); b.alpha_composite(gi); panels[i] = b.convert('RGB')
if a.detail:
    d = Image.open(os.path.join(ROOT, 'ref', 'parts', a.detail)).convert('RGB')
    if a.detail_box: d = d.crop(tuple(int(v) for v in a.detail_box.split(',')))
    panels.append(d.resize((max(1, int(d.width*h/d.height)), h), Image.LANCZOS))
out = Image.new('RGB', (sum(p.width for p in panels) + 8*(len(panels)-1), h), (200, 200, 205)); x = 0
for p in panels: out.paste(p, (x, 0)); x += p.width + 8
if out.width > a.maxw: out = out.resize((a.maxw, int(out.height*a.maxw/out.width)), Image.LANCZOS)
os.makedirs(os.path.dirname(os.path.abspath(a.out)), exist_ok=True); out.save(a.out)
```

## tools/grid.py
```python
# python3 tools/grid.py [--image renders/full_1x.png] --box x0,y0,x1,y1 --scale 7 --step 10 --out renders/g.png
import argparse, os
from PIL import Image, ImageDraw
ROOT = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
ap = argparse.ArgumentParser(); ap.add_argument('--image', default=os.path.join(ROOT, 'ref', 'front.png'))
ap.add_argument('--box', required=True); ap.add_argument('--scale', type=float, default=5)
ap.add_argument('--step', type=int, default=10); ap.add_argument('--out', required=True); a = ap.parse_args()
x0, y0, x1, y1 = [int(v) for v in a.box.split(',')]; s = a.scale
im = Image.open(a.image).convert('RGB').crop((x0, y0, x1, y1)); im = im.resize((int(im.width*s), int(im.height*s)), Image.LANCZOS)
d = ImageDraw.Draw(im)
for gx in range(-(-x0 // a.step) * a.step, x1+1, a.step):
    X = (gx-x0)*s; c = (255, 0, 0) if gx % (a.step*5) == 0 else (0, 160, 255)
    d.line([(X, 0), (X, im.height)], fill=c); d.text((X+2, 2), str(gx), fill=(0, 0, 200))
for gy in range(-(-y0 // a.step) * a.step, y1+1, a.step):
    Y = (gy-y0)*s; c = (255, 0, 0) if gy % (a.step*5) == 0 else (0, 160, 255)
    d.line([(0, Y), (im.width, Y)], fill=c); d.text((2, Y+2), str(gy), fill=(0, 0, 200))
im.save(a.out)
```
To grid your own render in the same coordinates, rasterise the assembled SVG at 2x with cairosvg and downsample to W x H.

## construction.py (shape)
Module constants (SKULL dict cx/cy/rx/ry, HAIRLINE, PARTING, EAR_L/R, JAW, NECK, CLAVICLES, SHOULDER_L/R, RIBCAGE,
WAIST, PELVIS, ELBOW/WRIST/PALM, HIP/KNEE/ANKLE, SOLE_Y, BONES list) plus `guide_svg()` returning a
`<g fill="none" stroke="#1a8cff">` with the ellipses, polylines, bone lines and joint circles, so `compare.py --guide`
can overlay it. Save `renders/construction_overlay.png` (guide over the reference) and look at it before briefing parts.

## Snippets
Palette sampling (k-means on non-white pixels of a crop):
```python
import numpy as np; from PIL import Image
def dom(path, box, k=5, seed=0):
    a = np.asarray(Image.open(path).convert('RGB').crop(box)).reshape(-1, 3).astype(float); a = a[a.sum(1) < 735]
    rng = np.random.default_rng(seed); c = a[rng.choice(len(a), k, replace=False)]
    for _ in range(15):
        l = ((a[:, None] - c[None])**2).sum(2).argmin(1)
        c = np.array([a[l == i].mean(0) if (l == i).any() else c[i] for i in range(k)])
    return sorted((('#%02x%02x%02x' % tuple(int(v) for v in c[i])), int((l == i).sum())) for i in range(k)), key=lambda t: -t[1])
```
Hole scan (white regions not connected to the border; inspect each against the reference — gaps between hair strands are fine):
```python
import numpy as np; from scipy import ndimage; from PIL import Image
ours = np.asarray(Image.open('renders/full_1x.png').convert('RGB')).astype(int)
lab, n = ndimage.label(ours.min(2) >= 252)
border = set(np.unique(np.concatenate([lab[0], lab[-1], lab[:, 0], lab[:, -1]])))
holes = sorted(((int((lab == i).sum()), *np.argwhere(lab == i).mean(0)[::-1].astype(int)) for i in range(1, n+1)
                if i not in border and (lab == i).sum() >= 4), reverse=True)
```
Soft form shadow along a contour (no misaligned block edge): draw 3 ribbons centred on the contour points, widths
13/9/5 at opacity .14/.16/.2 in the shadow colour, inside a clip-path of the skin shape.