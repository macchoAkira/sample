# CLAUDE.md

Guidance for AI assistants (Claude Code and others) working in this repository.

## Project overview

**ATELIER — 服づくり工程シミュレーター** (clothing-making process simulator) is a
single-file browser game. The player role-plays an apparel brand and builds one
garment from scratch, advancing through seven production steps:

1. **企画 (Planning)** — brand name, item name, garment type, concept, target season
2. **デザイン (Design)** — silhouette, neckline, sleeve length, detail
3. **生地選び (Fabric)** — material and color (with a budget)
4. **裁断 (Cutting)** — timing mini-game for precision
5. **縫製 (Sewing)** — timing mini-game (more reps)
6. **仕上げ (Finishing)** — timing mini-game for final quality
7. **完成 (Result)** — scored S/A/B/C with a breakdown and a brand tag

Choices in steps 1–3 update a live **SVG preview** of the garment. The final
score combines design coherence, cutting/sewing/finishing precision, material
quality, and cost management.

The language of the UI is Japanese.

## Codebase structure

```
index.html    The entire game: HTML + CSS + JavaScript in one file
CLAUDE.md     This file
```

There is **no build step, no dependencies, and no framework.** Everything is
vanilla HTML/CSS/JS in `index.html`. Open the file in a browser to play.

### How `index.html` is organized

The `<script>` block is divided by comment banners (search for `====`):

- **データ** — game content as plain JS arrays/objects: `GARMENTS`, `CONCEPTS`,
  `SEASONS`, `SILHOUETTES`, `NECKLINES`, `SLEEVES`, `DETAILS`, `FABRICS`,
  `COLORS`, `CONCEPT_PREF` (concept→preference map for scoring), `BUDGET`.
- **状態** — the single mutable `state` object (current step, all player choices,
  `mini` scores keyed by step index).
- **ユーティリティ** — helpers: `yen`, `shade` (lighten/darken a hex color),
  `byId` / `byName`, `spent` (running cost).
- **プレビューSVG** — `buildSVG()` generates the garment outline as an SVG path
  from the current `state`, plus garment-specific extras (hood, lapels, pocket,
  texture). `renderPreview()` writes it into the preview panel.
- **ステップ描画** — `renderPanel()` is the main view switch on `state.step`;
  per-step option lists are rendered with `optButton()` and wired with
  `bindOpts()`.
- **ミニゲーム** — `renderMini()` drives the timing-bar mini-game (a `requestAnimationFrame`
  loop; "stop in the green zone" scoring), shared by steps 4–6 via config.
- **採点** — `scoreDesign()`, `scoreCost()`, `finalScore()`, `rankOf()`.
- **結果画面** — `renderResult()` builds the final scorecard.
- **ナビゲーション** — `canAdvance()` gates the "next" button per step;
  `goNext` / `goBack` / `resetGame` manage flow.

## Development workflows

- **Run / play:** open `index.html` in any modern browser
  (e.g. `file:///.../index.html`). No server required.
- **There are no automated tests or linters configured.** Validation is manual
  (play through the steps) or via a throwaway Playwright script.

### Smoke-testing with the pre-installed browser

Chromium + Playwright are available in the environment. Playwright is installed
**globally**, so ESM `import` by bare name fails — import the package's default
export by absolute path and launch with the bundled Chromium:

```js
import pkg from '/opt/node22/lib/node_modules/playwright/index.js';
const browser = await pkg.chromium.launch({ executablePath: '/opt/pw-browsers/chromium' });
```

Then drive the UI by clicking `.opt[data-v="..."]` options and `#next`, and read
back `.rank` / `.scorebig` on the result screen. Do **not** run
`playwright install`.

## Conventions

- **Single file:** keep the game self-contained in `index.html` unless there's a
  strong reason to split. No bundler, no `node_modules`, no external CDN assets.
- **Vanilla JS only.** No frameworks or libraries in the shipped page.
- **Add game content via the data arrays**, not by hardcoding into render logic.
  A new fabric/color/garment should be a new entry in `FABRICS` / `COLORS` /
  `GARMENTS`; the render and scoring code reads from those arrays. New garment
  types may also need a branch in `buildSVG()` for their distinctive shape.
- **Scoring lives in the 採点 section** — adjust weights in `finalScore()` and
  concept affinities in `CONCEPT_PREF`.
- UI copy is **Japanese**; keep new user-facing strings in Japanese to match.
- The visual theme uses CSS custom properties defined in `:root` (`--accent`,
  `--ink`, etc.); reuse them rather than introducing new ad-hoc colors.

## Git workflow

- The active development branch is `claude/claude-md-docs-7j2wpm`.
- Push with `git push -u origin <branch-name>`.
- Do **not** open a pull request unless explicitly asked.
- Write clear, descriptive commit messages.

## Notes for AI assistants

- Keep this file current: when you change the step flow, scoring, or data model,
  update the relevant section here in the same change.
- Prefer verifying behavior by actually loading `index.html` in the browser over
  reasoning about it from memory.
