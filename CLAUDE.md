# CLAUDE.md

Guidance for working in this repo.

## What this is

A single-page **kitchen shopping checklist**. You tick items from a master
list, optionally add a quantity, then share the result to WhatsApp or the
native share sheet. Mobile-first, installable as a PWA-style web app.

No build step, no framework, no dependencies. Just static files.

## Files

- **`index.html`** — the entire app: markup, CSS (in `<style>`), and logic
  (one IIFE in `<script>`). Everything lives here.
- **`master-list.txt`** — the catalog, edited by hand. Format: a line that
  **starts with an emoji** is a category header; every following plain line
  is an item under it. Blank lines are ignored. Parsed at runtime by
  `parseCatalog()`.

## Running it

Open `index.html` in a browser. `master-list.txt` is fetched over HTTP, so
for local testing serve the folder (e.g. `python3 -m http.server`) rather
than opening via `file://`, or the fetch will be blocked.

## How the app works (in `index.html`)

- **State** lives in the `state` object (`catalog`, `theme`, `sound`, `extras`)
  plus module-level `currentSearch`, `currentFilter`, `selectedOnly`.
- **Persistence**: the in-progress list auto-saves to `localStorage`
  (`kitchenlist_state_v1`), keyed by category + item **name** (not position),
  so editing `master-list.txt` never corrupts a saved list. Theme is saved
  separately.
- **Render path**: `render()` → `renderPills()` + `renderGroups()` +
  `updateCount()`. `render()` is the full redraw; `updatePillCounts()` and
  `renderGroups()` are lighter partial updates called on ticks/search.
- **Tabs (pills)**: built from catalog categories, with `All Items` first and
  a dedicated **`✏️ Others`** tab last (`data-category="__others__"`) for wild
  items not in any category. Pills show **selected** progress (hidden at 0);
  per-category **totals** appear in each list group header instead. Each pill
  is built as `ring + label` via `setPill()` so the lightweight
  `updatePillCounts()` can refresh progress without rebuilding the strip. The
  **ring** is a masked conic-gradient donut (`--p` angle, registered with
  `@property`) that fills as the category completes and glows when done.
- **Micro-interactions** (the app leans into tasteful delight — keep it that
  way): per-item **emoji icons** via `itemIcon()` (`ITEM_ICONS` keyword→emoji
  table, substring match, graceful blank when no match); a **cart badge** on
  the logo showing the selected count with a bump animation; **fly-to-cart**
  (`flyToCart()`) arcing a ticked item's icon into the badge; a **clear sweep**
  (`clearSweep()`); and synthesised **sounds** (`playTone()` + `playTick` /
  `playUntick` / `playNav` / `playClear`, Web Audio, no files) gated by the
  🔊/🔇 toggle (`state.sound`, saved separately). Tapping the **cart icon**
  opens the selected-only review; the **logo text** goes home. All animations
  respect `prefers-reduced-motion`.
- **"Other items"** (extras): user-added items not on the master list. The
  adder shows only on **All Items** and the **Others** tab. Extras are part of
  the shared message and the selected count like any checked item.
- **Sharing**: `buildMessage()` produces the WhatsApp-formatted text. Set
  `OWNER_WHATSAPP` (top of the script) to a number to send straight to a chat;
  leave `""` to pick the chat after tapping share.

## Planned / next

- **Quick-add row of most-bought items** (deferred — build when the user is
  ready). Idea: track how often each item gets ticked in `localStorage` (a
  separate key, e.g. `kitchenlist_history_v1`, incremented on tick/share), then
  surface a horizontal **"Quick add"** chip row at the top of **All Items**
  with the user's most-frequent picks for one-tap selection. Keep it
  dependency-free and single-file; reuse `toggleItem`/`setPill` plumbing.

## Conventions

- Keep it **dependency-free and single-file** — match the existing vanilla-JS
  style (ES5-ish, IIFE, `el()`/`esc()` helpers, no bundler).
- Always `esc()` any user/catalog text put into `innerHTML`.
- Touch `master-list.txt` only to change the actual shopping catalog.
