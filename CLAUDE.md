# CLAUDE.md

Guidance for AI assistants working in this repository.

## What this repo is

A single-page **reveal.js** presentation titled *AI 上下游产业链全景* (AI industry value chain). All slides, styles, and init code live in one file: `index.html`. Content is in Simplified Chinese (`lang="zh-CN"`).

There is no build step, no package manager, no test suite, and no backend.

## Layout

```
index.html                       # entire deck: HTML + inline <style> + inline init <script>
README.md                        # user-facing instructions (Chinese)
vendor/reveal.js-5.1.0/          # vendored reveal.js — do NOT edit
  dist/reset.css
  dist/reveal.css
  dist/reveal.js
  dist/theme/black.css           # base theme; custom overrides live in index.html
  plugin/notes/notes.js          # speaker notes plugin
```

Everything the deck needs is vendored under `vendor/` so it runs fully offline. Never add CDN `<script>`/`<link>` tags or fetch reveal.js assets from the network — that's the whole point of vendoring.

## Running locally

```bash
python3 -m http.server 8000
# open http://localhost:8000/
```

`file://` works for basic viewing but breaks the speaker-view popup (`S` key). Use the http server when verifying anything interactive.

## Editing conventions

- **One source of truth.** All slide markup, CSS, and the `Reveal.initialize({...})` call sit inside `index.html`. Don't split CSS/JS into separate files unless the user asks — keeping it inline is intentional for a zero-build, copy-and-share deck.
- **Slides are `<section>` elements** inside `<div class="slides">`. Add new slides by appending another `<section>`. Each existing slide has a `<!-- Slide N: ... -->` comment above it; keep that pattern when inserting.
- **Reusable layout primitives** already defined in the inline `<style>`:
  - `.cover` — centered title slide
  - `.stack` / `.tier.up|.mid|.down` — three-tier stacked rows (upstream/midstream/downstream)
  - `.cols`, `.cols.tri`, `.card` — 2- or 3-column card grids
  - `.flow` with `.node` + `.arrow` — left-to-right pipeline diagram
  - `.kpi-row` / `.kpi` — 4-up metric tiles
  - `.pill`, `.pill.pink`, `.pill.green` — inline section tags
  - `.footer-note` — bottom-right meta text on a slide
  - `table` — themed dark table, no extra class needed
  Prefer composing these over inventing new classes.
- **Color tokens** are CSS custom properties on `:root`: `--accent` (cyan `#00d4ff`), `--accent-2` (pink `#ff6ec7`), `--accent-3` (green `#7cffb2`), `--bg-grad` (radial dark teal). Reuse them instead of hardcoding hex values.
- **Typography.** Headings use `var(--accent)`; `h3` inside cards uses `--accent-3`. Body text sizing is unusually small (e.g. `0.7em`, `0.62em`, `0.55em`) so slides fit on one screen without scrolling — match the surrounding scale when adding content.
- **Bullets** use a custom `▹` marker via `.reveal ul li::before`. Don't add a `list-style` override that fights it.
- **Content language is Chinese.** Use the same mixed CN/EN style already present (e.g. "上游 · 算力 + 芯片"). Keep numbers/units consistent with the existing slides ($, %, MW, etc.).

## Reveal.js config

Initialized at the bottom of `index.html`:

```js
Reveal.initialize({
  hash: true,
  slideNumber: 'c/t',
  transition: 'slide',
  backgroundTransition: 'fade',
  controls: true,
  progress: true,
  center: false,
  plugins: [ RevealNotes ]
});
```

`center: false` is intentional — slides are top-aligned because the content is dense. Don't flip it without checking every slide.

Speaker notes use the `RevealNotes` plugin (`plugin/notes/notes.js`) — to add notes to a slide, put an `<aside class="notes">…</aside>` inside the `<section>`.

## Verifying changes

There's no test runner. Before reporting a change done:

1. Start `python3 -m http.server 8000` and open the deck.
2. Arrow through every slide you touched — reveal.js will silently render broken markup, so visually confirm layout, overflow, and that no card got pushed off-screen.
3. Press `S` to make sure speaker view still opens.
4. If a slide grew, check it at 1280×720 and 1920×1080 — slide sizing is tight.

## Git workflow

- Default branch: `main`.
- Feature work goes on a `claude/<topic>-<slug>` branch (see the active branch for the current task).
- Push with `git push -u origin <branch>` and open a **draft** PR via the GitHub MCP tools — there is no `gh` CLI in this environment.
- Don't commit anything under `vendor/` as a "fix" — that directory is a pinned upstream snapshot. If reveal.js itself needs an upgrade, replace the whole `vendor/reveal.js-5.1.0/` directory with the new version and update the `<link>`/`<script>` paths in `index.html` accordingly.

## Things to avoid

- Adding a bundler, package.json, or framework. This repo is deliberately dependency-free at the project level.
- Re-introducing the highlight plugin or any other reveal.js plugin that isn't already vendored — only `notes` ships with the deck (see commit `3758903`).
- Fetching fonts/CSS/JS from a CDN at runtime.
- Translating the deck to English wholesale; individual term tweaks are fine, but the audience is Chinese-speaking.
