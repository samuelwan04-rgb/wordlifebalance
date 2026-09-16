# Bible Cross-References — Project Handoff

This file is project context for Claude Code (or any fresh coding session) picking up this project. It was exported from a Cowork session that built the app as a Claude Artifact. The app is now a real, live website: the repo is created and pushed to GitHub, and GitHub Pages is turned on and serving it. Read this document for orientation, then read `index.html` directly for the actual implementation — this doc is the map, not the territory, and the code is the source of truth for exact behavior.

**Live site:** https://samuelwan04-rgb.github.io/wordlifebalance/
**Repo:** https://github.com/samuelwan04-rgb/wordlifebalance

## What this is

A single-page Bible study web app called **Bible Cross-References**. It has two modes:

1. **Home / Timeline** — a horizontally-scrollable chronological timeline of all 66 Bible books (Old and New Testament), grouped into historical eras, with filters by ruler, prophet, character, and group, plus a person-search feature. Clicking a book shows a detail panel (date range, genre, rulers/prophets/characters, a short intro). This is a "browse all of Scripture chronologically" view — conceptually the same idea as the user's other project, **Chronos** (easybiblestudy.com, a nonprofit chronological Bible study timeline), so there may be a reason to eventually merge or cross-link the two.
2. **Study** — a deep, verse-by-verse reading mode for a subset of books that have been fully authored (see "Content coverage" below). This is where most of the recent feature work has gone: reading settings, highlighting, cross-references, tradition notes, outside commentary links, sticky notes, and printing.

### Who it's for

The author (Sam) is a Methodist in Singapore, a beginner programmer working with Claude's help (both Cowork and Claude Code). The theological framing throughout leans broadly Wesleyan/Methodist but is intentionally cross-denominational — the "How Different Traditions Read It" feature deliberately surfaces Wesleyan, Reformed, Catholic, and Anabaptist readings side by side, on the premise that all four are "reasonably sound" even where they differ. Keep that spirit when adding content or features: cross-referencing across traditions is a feature, not a bug, but nothing theologically fringe should be added without it being a clearly labeled, mainstream-within-that-tradition view.

## Tech stack — read this before assuming a build step exists

**There is no build system, no package.json, no dependencies, no framework, and no backend.** The app is almost entirely one self-contained static HTML file (`index.html`) with inline `<style>` and `<script>` tags, plus a `maps/` folder of static JPEG images (see "Maps" below) — no other assets. `index.html` is roughly 12,000+ lines, mostly because the full text of nine Bible books in three translations is embedded directly in the file.

- Plain vanilla JavaScript, wrapped in a single IIFE at the bottom of the file.
- No `fetch`/`XMLHttpRequest`/API calls anywhere at runtime. Zero runtime API costs. The only external network request is a Google Fonts stylesheet `@import` (free, no auth); map images are local files committed to the repo, not hotlinked.
- State lives in one JS object (`state`) and a handful of module-level variables; the UI re-renders by rebuilding `innerHTML` strings (mostly `mainEl.innerHTML = '<div>...' + ... + '</div>'` string concatenation, not a templating library or virtual DOM).
- Persistence is entirely `localStorage`, per browser/device — there is no shared database, no user accounts, no sync across devices. See "Persistence" below for the exact keys.
- Deployed as a static site on **GitHub Pages** — already live (see the URL at the top of this file). No server, no environment variables, no secrets, no CI/CD pipeline beyond what Pages does automatically.

Do not introduce a bundler, framework, or backend unless the user explicitly asks for one — the whole design intent so far has been "zero-cost, zero-maintenance static site."

## Where the code lives

`index.html` — a complete standalone HTML document (`<!doctype html>` through `</html>`) — is already committed to the repo and is what GitHub Pages is serving. This is distinct from how the same content was published as a Claude Artifact, which strips the outer `<html>/<head>/<body>` tags because the Artifact host supplies its own — if you ever pull a fresh copy from the Artifact instead of this repo, remember to re-wrap it in a full document before it'll work as a standalone file again.

**The GitHub repo's `index.html` is now the single source of truth** — not the Claude Artifact. Any further edits should happen here (via Claude Code, a local editor, or GitHub's web editor) and get pushed to the repo; the Claude Artifact is the old copy and will drift out of date from this point on.

## Content coverage

Only six books currently have full study content authored: **Matthew (28 ch), Judges (21 ch), Malachi (4 ch), Revelation (22 ch), James (5 ch), Romans (16 ch)** — 363 individually-authored passages (pericopes) in total, each with cross-references and four tradition notes. All 66 books appear in the Home/Timeline view with metadata, but only these six have a "Study" mode.

```js
const BOOK_ORDER = ["matthew","judges","malachi","revelation","james","romans"];
const CONTENT_BOOKS = new Set(BOOK_ORDER); // used by the timeline to mark "has content"
```

Expanding to more books means writing more `PASSAGES` entries by hand (see structure below) — it's a content-authoring task, not a coding task, and is almost certainly the highest-value next piece of work if the user wants a fuller app.

### `PASSAGES` — the core data structure

One entry per pericope (a titled chunk of a chapter, usually a handful of verses). Shape:

```js
{
  id: "genealogy",              // unique slug, used as state.passageId
  chapter: 1,
  section: "birth-early-life",  // groups passages under SECTION_LABEL[book]
  ref: "Matthew 1:1–17",
  title: "The Genealogy of Jesus",
  speechMode: "marked",         // or other values — controls red-letter (words of Christ) rendering
  verses: {
    web: [[1, "verse text..."], [2, "verse text..."], ...],
    kjv: [[1, "..."], ...],
    esv: [[1, "..."], ...]
    // no niv/nkjv — those translations are link-out only, see below
  },
  xrefs: [
    { ref:"Luke 3:23–38", book:"luk", ch:3, v:23, note:"..." },
    ...
  ],
  trad: {
    wesleyan: "...",
    reformed: "...",
    catholic: "...",
    anabaptist: "..."
  }
}
```

Book/chapter-level lookups (`CHAPTER_LABEL`, `CHAPTER_ORDER`, `SECTION_LABEL`, `SECTION_ORDER`, `BLB_BOOK_CODE`) are separate constants keyed by book name — all defined near the top of the script, just above `PASSAGES`.

### `BOOK_TIMELINE` — the other 66-book data structure

Built via a `bk(key, name, testament, era, genre, dateRange, rulers, prophets, characters, opts)` helper, one call per Bible book, used only by the Home/Timeline view (not the Study mode). This is where era groupings, date ranges, and character/ruler/prophet metadata for the "browse all Scripture" experience live.

## Translations

```js
const TRANS_ORDER = ["web","kjv","esv","niv","nkjv"];
```

WEB (World English Bible) and KJV are public domain and fully embedded. ESV is embedded under Crossway's blanket quotation permission (with attribution, shown in `TRANS_CAPTION`). **NIV and NKJV are NOT embedded** — their licenses require separate written permission that hasn't been obtained, so selecting either one switches to "link-out" mode: buttons linking to Bible Gateway / Blue Letter Bible instead of inline text. This is handled by `isLinkoutTranslation()` / `LINKOUT_TRANSLATIONS`. **Do not embed NIV or NKJV verse text directly without confirming a license is in place** — this was a deliberate legal decision, not an oversight.

## Feature inventory

- **Reading**: a full chapter renders as one continuous scroll (all its pericopes back to back), not one passage at a time. Adjustable font (4 choices), line spacing (4 presets), and a draggable-width reading column (300–900px), all persisted.
- **Highlighting**: tap-or-drag word selection, 7 highlight colors, persisted per translation-independent verse key.
- **Cross-References panel**: sits sticky beside the reading column (not below it) so it's visible while scrolling — this was a deliberate redesign; see "Things already tried and rejected" below for why it isn't outside commentary sites instead.
- **Tradition tabs**: "How Different Traditions Read It" — four tabs (Wesleyan/Reformed/Catholic/Anabaptist) below the reading column, per current passage.
- **Commentary Elsewhere**: plain new-tab links to GotQuestions, Enduring Word, Desiring God, and Blue Letter Bible for the current passage. These are ordinary `<a target="_blank">` links — see below for why they aren't embedded inline.
- **Sticky notes**: click "Sticky note" near the chapter title to drop a draggable, colored (yellow/pink/blue/green) note anywhere on the reading page. Notes are freeform-positioned (not tied to a specific verse), scoped per chapter (`book-chapter` key), saved to `localStorage` on every keystroke/drag/color change. See `buildStickyNoteEl`, `renderStickyNotes`, `addStickyNote`, and the drag handlers (`beginDragNote`/`updateDragNote`/`endDragNote`).
- **Print chapter**: a "Print chapter" button calls `window.print()`; a `@media print` block hides all app chrome (nav, toolbars, side panels) and prints just the scripture text plus any sticky notes (notes swap from an editable `<textarea>` to a plain auto-height text block for printing, since a textarea only prints what fits its on-screen box), plus the book's map (see below) below the text.
- **Light/dark theme**: toggle in the header, persisted, plus `prefers-color-scheme` support when no explicit choice is made.
- **Person search**: search Bible characters across the Home/Timeline data.
- **Maps**: a "Map" button (beside "Sticky note" / "Print chapter") opens a modal showing a real reference map for the current book, from `BOOK_MAPS[book]` — see "Maps" below for the asset/license details. A freehand pen (SVG overlay, `state.mapDraw`, localStorage `smcv-mapdraw`) and sticky notes (reusing the chapter-notes component, keyed `map::<book>` in `state.notes`) both work on top of it, and both print.

## Maps

`maps/*.jpg` are the **"Biblica Open Bible Map"** series (Biblica, Inc. / Biblica Open Study Bible Resources), sourced from `https://open.bible/resources/` via Wikimedia Commons, licensed **CC BY-SA 4.0**. Each original PNG (several MB, up to 4000px) was downsized/re-compressed to a ~300KB JPEG (max 1400px) for web use — see `BOOK_MAPS` in `index.html` for the exact source file and Commons URL per book, which is also what's shown as the on-page credit. **Do not replace these with unlicensed images**, and if a book is added that needs a new map, look for another entry in the same Commons series first (search "Biblica Open Bible Map" on Wikimedia Commons) before sourcing elsewhere — keeping one consistent, pre-cleared source avoids a per-image licensing review each time.

## Things already tried and rejected (don't redo without a new reason)

- **Embedding outside commentary sites in an iframe next to the text.** This was built, tested, and removed. GotQuestions, Enduring Word, Desiring God, and Blue Letter Bible all refuse to be framed by another site (their own `X-Frame-Options`/CSP headers) — this is a restriction on their end, unrelated to hosting platform, so it will fail the same way on GitHub Pages as it did as a Claude Artifact. If revisiting this, the fallback UX (an "Open in new tab" button plus best-effort blocked-frame detection) was written and then deleted — worth searching the artifact's version history before rebuilding it from scratch, but don't expect the underlying blocking to have changed.
- A first attempt at the sticky-note click handling used real `<a href>` tags for the outside-commentary links and tried to intercept clicks with `preventDefault()` to redirect them into an in-page panel — this failed because the Claude Artifact hosting environment's own sandbox forces external-domain anchor clicks into a new tab before page JS gets a chance to intervene. That constraint is specific to the Artifact host, not to GitHub Pages, but the links were left as plain `target="_blank"` anchors anyway since the destination sites can't be embedded regardless (see above).

## Persistence — all `localStorage`, all per-browser/device

| Key | Holds |
|---|---|
| `smcv-book` | last-viewed book |
| `smcv-tradition` | last-selected tradition tab |
| `smcv-translation` | last-selected translation |
| `smcv-navmode` | "section" or "chapter" nav grouping in the sidebar |
| `smcv-highlights` | word/verse highlight state (JSON) |
| `smcv-notes` | all sticky notes, keyed by `book-chapter` (chapter notes) or `map::<book>` (map notes) (JSON) |
| `smcv-mapdraw` | freehand pen strokes drawn on each book's map, keyed by book (JSON) |
| `smcv-font` | reading font choice |
| `smcv-spacing` | line-spacing preset |
| `smcv-readingwidth` | dragged reading-column width in px |
| `smcv-theme` | "light" or "dark" |

There is no server-side storage. If the user ever wants notes/highlights to follow a person across devices, that requires adding a real backend (accounts + database) — a significant scope change from the current zero-backend design, flag it as such if requested.

## Design system

CSS custom properties defined once on `:root`, redefined under `@media (prefers-color-scheme: dark)` (guarded `:root:not([data-theme="light"])`) and again under `:root[data-theme="dark"]` for an explicit toggle. Key tokens: `--bg`, `--surface`, `--surface-2`, `--line`, `--ink`/`--ink-soft`/`--ink-faint`, `--accent`, `--gold` (primary accent, historically named for an earlier palette), `--jesus` (red-letter color), per-tradition colors (`--wesleyan`, `--reformed`, `--catholic`, `--anabaptist`, each with a `-tint` variant), and seven `--hl-*` highlight colors (also reused for sticky-note colors). Typography: Lora (serif, reading/headings) + Work Sans (UI) + Merriweather (one of the optional reading fonts), loaded via a single Google Fonts `@import`. Layout is CSS Grid/Flexbox throughout, no framework.

## Known limitations worth stating plainly if asked

- No accounts, no sync, no backend — everything is local to one browser.
- Only 6 of 66 books have deep study content; the rest are timeline-only.
- NIV/NKJV can't show verse text without a licensing conversation.
- Outside commentary sites can only ever be linked to, not embedded, for reasons outside this app's control.
- No automated tests exist; changes were manually validated (`node --check` for JS syntax, manual brace-balance checks for CSS, and a single visual pass before publishing) rather than through a test suite. Worth adding real tests if the codebase keeps growing.

## Deployment (GitHub Pages) — already live

No API costs, no server, no build step. Pages is enabled and `index.html` is being served as-is straight from the repo. (One thing to keep in mind either way: Pages is free on any plan for a public repo, but hosting Pages from a private repo needs a paid GitHub plan — worth knowing if the repo's visibility ever changes.) Updating the live site going forward is just: edit `index.html`, commit, push — GitHub Pages picks up the new commit automatically within a minute or so, no redeploy step.

If a custom domain is wanted later (e.g. tying into the `easybiblestudy.com` domain from the Chronos project), that's a "Custom domain" field in repo Settings → Pages, plus a DNS record (`CNAME`, or `A` records for an apex domain) at wherever the domain is registered.
