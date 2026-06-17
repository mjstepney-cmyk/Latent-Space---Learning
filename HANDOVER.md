# Latent Space — Handover / Kickoff

_Last updated: 17 Jun 2026. Keep this with the project; update the "Current state" and "Backlog" sections as you go._

---

## 0. Kickoff prompt (paste this to start a new session)

> I'm iterating **Latent Space Productions** — a self-contained single-file HTML educational tool (`index.html`) explaining AI to non-technical UK public-sector execs. The canonical source is **GitHub `main`** at `mjstepney-cmyk/Latent-Space---Learning` (pull it fresh as the base — the project-knowledge copy has been behind before). Read `HANDOVER.md` first. House rules: surgical `str_replace` edits not regenerations; self-contained file, Google Fonts only; validate after every change (node --check + tag balance + jsdom smoke); preview new visual features as standalone files before integration. Today I want to: **[describe change]**.

---

## 1. What the product is

- **Brand:** Latent Space Productions. Logo = dot-cloud with projection lines converging to a glowing focal node; wordmark "latent / space / productions".
- **Primary deliverable:** `index.html` — one self-contained HTML file, 10-chapter explainer on how AI works.
- **Audience:** non-technical UK public-sector executives. Conceptual fluency, not technical depth.
- **Hosting:** GitHub Pages → https://mjstepney-cmyk.github.io/Latent-Space---Learning/
- **Repo:** `mjstepney-cmyk/Latent-Space---Learning`, branch `main`, file `index.html` at repo root.
- **Sibling products (separate files, not in this repo workflow):** `understanding-ai-complete.html` (full reference edition), "The Data Mind" (data-literacy app).

## 2. Canonical source — READ THIS FIRST

- **The good copy lives on GitHub `main`.** Always start an edit by pulling it fresh:
  `curl -s https://raw.githubusercontent.com/mjstepney-cmyk/Latent-Space---Learning/main/index.html -o prod.html`
- The Claude **project-knowledge copy has repeatedly been several commits behind GitHub.** Editing the stale copy silently reverts newer work (this exact trap dropped the fullscreen toggle + blip fix once — recovered from git history). Do **not** trust `/mnt/project/` as the base.
- Commit messages in the repo are all generic ("Update index.html"), which makes recovery hard. Recovering a lost version means walking commit SHAs via `raw.githubusercontent.com/.../<sha>/index.html`.
- Current file: **~2,322 lines, ~207 KB.**

## 3. Design system (match exactly)

```
--bg0 #081418  --bg1 #0d1f26  --panel #10262e  --panel2 #143039
--line rgba(94,234,212,.13)   --line2 rgba(94,234,212,.28)
--ink #e9f4f1  --dim #9db8b4  --faint #7a9a96
--aqua #5eead4  --coral #ff8c6b  --gold #f4c95d
--disp 'Sora'   --body 'Inter'   --mono 'JetBrains Mono'   --r 14px   --max 880px
```
Only external dependency is Google Fonts. No frameworks, no build step.

## 4. Architecture & conventions

- **One file.** Inline `<style>` (ends ~L565) and one `<script>` (the big block at the end). All JS is plain ES6, no modules/imports.
- **Scope new JS in an IIFE** with prefixed ids/classes to avoid clobbering existing demos (they share the global scope). The scale animation uses the `lsz-` prefix and exposes only `window.lszOpen` / `window.lszClose`.
- **"Go deeper" pattern:** collapsible `<details class="godeeper">`, collapsed by default (exec view), open by default in the reference edition. Deferred demo init via `ontoggle` with a `typeof` guard, e.g.
  `ontoggle="if(this.open){setTimeout(function(){if(typeof fn==='function')fn();},120);}else{…}"`
- **Demos** use shared classes: `.demo`, `.demo-tag`, `.demo-h`, `.dsub`, `.why`/`.wl`, `.controls`, `.btnrow`, `.mini-btn`(`.acc`), `.readout`.
- **Two-track content:** exec-facing collapsed view vs. full reference edition (`understanding-ai-complete.html`).

## 5. Validation workflow (run after every change)

```bash
# 1. JS syntax
awk '/<script>/{f=1;next}/<\/script>/{f=0}f' index.html > /tmp/s.js && node --check /tmp/s.js
# 2. tag balance
python3 -c "import re;s=open('index.html').read()
[print(t,len(re.findall(r'<%s[ >]'%t,s)),len(re.findall(r'</%s>'%t,s))) for t in ['div','details','section','button','canvas']]"
# 3. runtime smoke (jsdom): stub getContext/rAF/matchMedia/IntersectionObserver,
#    load page, assert 0 load-time errors, then exercise new functions.
```
The jsdom smoke test is the high-value one — it catches ReferenceErrors and confirms the new IIFE doesn't break existing demos. Stub canvas `getContext` to a no-op Proxy and `requestAnimationFrame` to noop so canvas-less jsdom doesn't false-fail.

## 6. Key interactive features currently live

| Feature | Where | Notes |
|---|---|---|
| Latent-space logo modal | click the logo | Canvas animation; **blip fix** = captures actual dot positions at the phase boundary (`if(!captured){captured=true…}`) and lerps from those. Don't revert. |
| Fullscreen toggle | header | ~7 refs; restored after a regression. |
| Fit-the-line demo | Ch 3 | Two-slider regression; "① your turn / ② then the machine" staged cue. |
| Spam-filter demo | Ch 4 | Hand-set dials → "Reveal trained values"; same staged cue. |
| **Parameter-scale animation** | Ch 04 "From four knobs to a trillion" go-deeper | The big build from this session — see §7. |

## 7. The parameter-scale animation (most recent build)

**Location:** Chapter 04 ("Parameters") go-deeper `<details>`. Panel order: intro `<p>` → **bars chart** (with log→linear flip + Moon anchor) → **zoom animation**.

**Concept:** generative branching tree on a tilted/offset plane. Starts with exactly 4 gold "knobs" and nothing else; each sprouts 4, those sprout 4 — counter reads the literal node count (4 → 16 → 64 → 256 → 1,024) while countable, then hands off to a procedural halo racing to ~2T. Connecting lines persist the whole way; active branches carry an outward-flowing pulse (additive dashed overlay). Camera pulls back continuously (`vg` "virtual generation").

**Wiring:**
- IIFE, all DOM ids/classes prefixed `lsz-`. Exposes `window.lszOpen` / `window.lszClose`.
- Panel `ontoggle`: open → `animateBars()` + `lszOpen()`; close → `lszClose()` (stops rAF).
- **Autoplay** = `IntersectionObserver` (threshold 0.35) plays once when the stage scrolls into view (chosen because the animation sits below the chart). Fallback = play-on-open if no IO.
- Reduced-motion: no autoplay, no pulse/rotation; scrub + chips still work.

**Tuning constants** (top of the IIFE — all one-liners to change):
- `EXP=1.95` branch/ring spread · `TILT=0.52` plane angle · `DOTK=0.013` dot size · `GROW=0.085` spawn speed
- `B=4, MAXGEN=4` explicit generations before procedural halo
- `GEN_P[]` per-generation reveal points · `VG[]` camera keyframes · `CNT[]` counter keyframes · `DUR=15.5` total seconds (still at v3 default — never tuned in context)
- Pulse: the `globalCompositeOperation='lighter'` block — `setLineDash([2,26])` glint size/gap, `lineDashOffset=-(time*0.06…)` flow speed.
- `MILE[]` = milestone labels (Perceptron…Frontier). Chips use **K/M/B/T** suffixes (thousand→K, not t).

**Linear flip (merged into existing bars):** bars carry `data-w` (log %) and `data-lin` (linear %). `lszToggleScale()` swaps widths + shows the Moon anchor card; `lszResetScale()` runs inside `animateBars()` so reopening always resets to log.

**Standalone preview files** (in case you want to iterate the animation in isolation again before re-integrating): `scale-preview.html` (v1 flat zoom — superseded), `scale-preview-v2.html` (generative), `scale-preview-v3.html` (lines persist + pulse — this is what was integrated).

## 8. Backlog / open items

- [ ] **`understanding-ai-complete.html`** has none of the recent work (UX fixes, blip fix, fullscreen, scale animation). Port when ready.
- [ ] **Responsive laptop vs large-screen** layout optimisation — two options were proposed (tighter breakpoints vs. collapsed sidebar for laptop mode); not yet actioned.
- [ ] **Scale-animation pacing** (`DUR` / `GEN_P` holds) left at default — revisit now it's in context if it feels fast/slow.
- [ ] Consider descriptive commit messages going forward to make future recovery easier.

## 9. How the collaborator likes to work

- Terse and decisive: short evaluation, then "build it." Prefers Claude makes reasonable assumptions over asking lots of clarifying questions.
- **Surgical `str_replace` edits to the existing file**, not full regenerations. Avoid unnecessary rewrites.
- Iterative, fix-by-fix, with verification after each change.
- New visual features previewed as standalone files first, then integrated once the motion/look is approved.
- Concise, signal-over-noise responses; highlight only material decisions/trade-offs.
