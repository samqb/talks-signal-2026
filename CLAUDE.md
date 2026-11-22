# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file HTML slide deck for Sam Bourton's talk "Under the Hood of Agentic AI" at Signal Geneva (18 September 2026). No build, no dependencies, no tests. The folder lives in Google Drive and is a git repo pushed to github.com/samqb/talks-signal-2026. The deck is `2026-09-17 - Signal Geneva - deck v9.html`: all CSS, markup, speaker notes and JS in one file.

Versioning is by filename and date. A new version is a new file (`... deck v10.html`), not an edit in place, and the title-slide notes carry a short "Deck vN, date" changelog paragraph that should be updated with it.

## Running it

- Open the HTML file in a browser. Nothing to build or serve.
- No print key. The `P` shortcut was removed in v9 because the browser PDF came out corrupted; the `@media print` CSS is still there but is not a supported path. The PDF in `share/` is built from per-slide screenshots.
- Deploy: the folder is linked to the Vercel project `talks-signal-2026` (team `sams-projects-e4e0a1e4`, production URL https://signal.sambourton.ai, A record at GoDaddy to 76.76.21.21; https://talks-signal-2026.vercel.app also works). `vercel.json` rewrites `/` to the deck file. Deploy with `vercel deploy --prod --yes --scope sams-projects-e4e0a1e4`. The GitHub repo is connected to Vercel, so a push to `main` deploys production automatically.
- Vault sync: a git `post-commit` hook (`.git/hooks/post-commit`, not versioned) copies the newest `* - deck v*.html` into `~/Obsidian/Vault/notes/talks/Geneva-Signal/latest/` after every commit and moves any other deck there to `_archive/`. It does not write the vault's NEW.md; log a session entry there by hand when the vault copy matters. If the repo is re-cloned, reinstall the hook.
- Smoke-test after edits: open the file, press `N` and step through every slide checking that notes render, press `F` for present mode, press `T` on slide 6 and `1`-`5` on slide 9.

## Deck anatomy

Every slide is a `<section>` with class `title-slide`, `slide` or `closing-slide`, plus `data-slide` and `id` set to the same value. The JS collects sections in DOM order, so slide order is DOM order.

Inside a content slide, in this order:

1. `.slide-number`: `NN · Section X · Section name`. Hardcoded text.
2. `.slide-label`: `Slide N · Short title`. Hardcoded text.
3. `h1` (with an `<em>` for the accent word), then `.slide-sub`.
4. Body content, using layout classes defined in the stylesheet (`two-col`, `three-col`, `spectrum-steps`, `callout`, `card`, `stat`, `onion-*`, `ledger`, `cap-grid`, and so on).
5. `.notes`: speaker notes. Hidden on screen and in print, rendered into the notes drawer. First `<p>` starts with `<span class="cue">2 min</span>` (or `90 sec`, `60 sec`, `45 sec`) giving the time budget for the slide.

Slide ids are historical (`s2`, `s5`, `s5b`, `s8b`, `s9b`, `s12b`, `s14c`, `s14d`, `s16b`) and do not match displayed slide numbers. Do not renumber ids; they are referenced from the section-jump sidebar and from JS.

### Three places to update when adding, removing or reordering a slide

- The `<section>` itself (DOM position sets order).
- The `.slide-number` and `.slide-label` text on every slide after it.
- The `#sectionJump` sidebar: one `<a class="section-jump-item" data-target="ID" onclick="go('ID')">` per slide, grouped under `.section-jump-group-label`. `data-target` must equal the section's `data-slide`. The nav dots (`#slideNav`) are generated automatically and need no edit.
- Also check the title-slide notes, which reference slide numbers for the cut plan ("drop slides 10 and 11").

## Interaction model (the JS at the bottom of the file)

- Scroll-snap deck: one section per viewport, `IntersectionObserver` at 0.4 threshold tracks `current` and updates dots, sidebar highlight and notes drawer.
- `.reveal` elements fade in when they scroll into view; stagger with `reveal-delay-1` to `reveal-delay-5`. Print CSS forces them visible.
- Keyboard: arrows / space / PageUp-Down / Home / End navigate. `N` toggles the notes drawer, `F` toggles present mode (fullscreen, `body.presenting` hides chrome), `Esc` closes notes.
- Slide-specific keys are hardwired to section ids in the keydown handler: `T` only fires when `sections[current].id === 's8b'` (heat map colour mode toggle), `1`-`5` only on a slide containing `[data-dual]` (cluster rings). If those slides move or are renamed, update the handler; the sidebar footer only lists N and F, so the slide-specific keys live in the speaker notes.
- Heat map (slide 6, `#heat`): data lives in `HEAT_ROWS` / `HEAT_COLS` in the JS, not in the markup. `buildHeat()` generates the grid; two colour modes, `mck` (ramp to the 31% max cell) and `pct` (ramp to 100%). The row flagged `'se'` (software engineering) gets the `heat-row-se` class for greying out in pct mode.
- Cluster rings (slide 9, `[data-dual]`): `CLUSTERS` defines which `.spectrum-step` and `.scope-stop` ranges each ring encloses. Rings are drawn as SVG paths from live `getBoundingClientRect()` positions, redrawn on resize and on present-mode toggle. Toggled by the `.cb[data-cluster]` buttons or keys 1-5; cluster 5 (frontier labs) also reveals the two frontier markers via `.show-frontier`.

## Styling

- Design tokens are CSS custom properties on `:root` (warm off-white background, coral / purple / orange / blue / green / red accents, `--accent-gradient`). Use the tokens, not raw hex, except inside the heat-map and cluster JS where colours are hardcoded to match.
- Fonts are system stacks (`Inter` and `JetBrains Mono` if installed, with fallbacks). No web font loading, no external requests at all: the file must work offline on stage.
- Breakpoints at 1400px (tighter slide padding) and 1100px (sidebar hidden, grids collapse to one column). Presentation target is a 16:9 laptop or projector at full width.
- Title and closing slides share the dark gradient background and are forced to print in colour.

## Content conventions

- Tone and vocabulary follow Sam's global rules: no em dashes in prose, no AI filler. Use `&middot;` as the separator in labels and eyebrows.
- Speaker notes are glanceable talking points, not messages to Sam: a cue `<p>` with the time budget and a one-line summary of the slide, then a `<ul>` of short bullets summarising the voiceover. Bold lead-ins label the bullet (`Bridge:`, `If asked:`, `Time check:`), verbatim lines go in `<em>` quotes. No build history or rationale in slide notes; the changelog lives in the title-slide notes as `<p class="log">`.
- Numbers on slides are sourced from published surveys (McKinsey State of AI and similar). When changing a figure, change it in the notes too if it is quoted there.
