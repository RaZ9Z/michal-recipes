# CLAUDE.md — המתכונים של מיכל (Michal's Recipes)

Handoff from a Claude chat session (Nov 2026). Read this before changing anything.

## What this is
A Hebrew, RTL, installable web app (PWA) for **Michal** (~60, Android, the mother of Raz's girlfriend May) to save recipe videos from Instagram / TikTok / Facebook.
- Live: https://raz9z.github.io/michal-recipes/ (GitHub Pages, `main` branch, root)
- Repo: github.com/RaZ9Z/michal-recipes
- Pushing to `main` redeploys in ~1 min. The service worker is network-first for pages, so her installed app picks up changes on next open with internet.

## Users and design rules
- Primary user is 60 and not technical. Big text (19px base), 44px+ targets (most are 50-64px), every icon has a text label, no swipe-only or long-press-only actions, confirm before deleting, max ~3 main screens.
- All UI text is Hebrew, neutral-plural imperative style ("לוחצים", "בוחרים"). Michal is addressed by name only on the empty home screen.
- Raz (the developer) wants direct, no-filler communication, replies in English.

## Architecture (no build step, no framework)
- `index.html`: the whole app. Inline CSS + one inline `<script>` wrapped in `{ }`. Views rendered as HTML strings into `#app`; event delegation on `document`.
- `manifest.json`: install info + `share_target` (GET to `./` with `title`/`text`/`url`). This is what puts the app in Android's Share menu after Chrome "Install app" (Chrome makes a WebAPK; decided NOT to build our own APK, see below).
- `sw.js`: caches app shell; navigation = network-first with cached fallback. Bump `C` (cache name) if you change the asset list.
- Icons: `icon-192.png`, `icon-512.png`, `icon-maskable-512.png` (generated with PIL: red pot on tomato background).
- `.nojekyll` for GitHub Pages.

### Data
- Recipes + categories: `localStorage['michal-recipes-v1']` → `{cats:[{id,name}], recipes:[{id,created,url,plat,title,text,kos,cats[],fav,notes,hasImg?,imgUrl?}]}`.
- Images: IndexedDB `michal-recipes` / store `img`, key = recipe id, value = JPEG Blob (resized to 520px).
- Theme: `localStorage['michal-recipes-skin']`, applied by a tiny inline script in `<head>` before paint.
- Backup: Settings → "שמירת גיבוי" builds JSON (images as data URLs) as a `.txt` file and opens the Android share sheet (WhatsApp to herself). Restore replaces everything after a confirm.
- Data lives only on her phone. Planned: Supabase sync.

### Views (stack-based nav, Android Back button works via pushState/popstate)
`home` (search, kashrut tiles, favorites, "not yet sorted", category tiles) → `list` (filter chips) → `recipe` (hero image, watch button, fav, cooking mode, written recipe, autosaved notes, edit/delete) → `edit` (link + paste, auto-fill, own photo, title, kashrut, multi-category + add new, written recipe). `settings`: themes, how-to, manage categories, bulk link import, backup/restore.
Overlays (confirm dialog, cooking mode with Wake Lock + text size) close on Back.

### Domain model
- Two independent axes: **kashrut** (`kos`: dairy/meat/parve, single, optional) and **categories** (multi). Default categories: ראשונות, עיקריות, דגים, סלטים, תוספות, מרקים, מאפים ולחמים, עוגות וקינוחים, חגים.
- Kashrut colours are fixed across themes: blue חלבי, red בשרי, green פרווה.

### Sharing / links
- Share launch: reads `url`, then `text`, then `title` for the first URL; caption text (TikTok sends one) minus URL and hashtags prefills title/written recipe.
- `cleanUrl()` strips tracking params (igsh, utm_*, mibextid, fbclid, ...) but keeps meaningful ones (e.g. Facebook `v=`). `sameUrl()` dedupes: sharing a saved video opens it with toast "המתכון הזה כבר שמור".
- Auto-fill `fetchMeta()`: TikTok oEmbed first, then microlink.io free API (~50 req/day, no key). Instagram often blocks previews; fallback is the platform-coloured placeholder + "תמונה משלי" photo upload. **Not yet verified against live networks** (dev sandbox had no internet).

### Themes (Settings → צבעים)
`data-skin` on `<html>`: `tomato` (default, no attribute), `lemon`, `news`, `retro`, `olive`. Each skin sets `--brand --onbrand --bg --card --ink --mute --line --red --redbg --sun --onsun --hl --focus` with `light-dark()` for dark mode. All text pairs checked ≥4.5:1. Never hardcode `#fff` on brand colours; use `--onbrand`/`--onsun`.

### RTL rules (audited)
- `<html lang="he" dir="rtl">`, logical properties only, no physical left/right.
- Every free-text input/textarea has `dir="auto"`, with `:placeholder-shown{direction:rtl}` so Hebrew placeholders stay right. URL field is `dir="ltr"`.
- `.rtext` (written recipe, notes) uses `unicode-bidi:plaintext` so each line picks its own direction. `rtx()` swaps digit-hyphen-digit for a non-breaking hyphen so phone numbers don't wrap.
- User strings inside Hebrew sentences are wrapped in `<bdi>` (or U+2068/U+2069 inside `ask()` messages).
- Back chevron flips with `.dirico:dir(rtl)`. Play ▶ and heart ♥ are not mirrored.

### Security
All user/remote strings go through `esc()` before entering HTML. Keep it that way.

## Decisions already made
- PWA over native app; Chrome install = real WebAPK with share target. A custom APK (TWA/Bubblewrap) was rejected: sideload warnings for a 60-year-old, needs assetlinks on raz9z.github.io, keystore management. Revisit only if Chrome install fails on her phone or Play Store is wanted.
- No login, no backend yet.
- In the claude.ai artifact preview (iframe) the app seeds 5 demo recipes; on the real site it starts empty.

## Status
- Deployed and live. Pending: real-phone test of install + Share menu (Raz's phone first, then Michal's). Tell her to install with Chrome, and in Instagram/Facebook use Share → "Share to..."/More to reach the Android share sheet; pin "המתכונים" there.

## Next ideas (not started)
1. Supabase sync so May can add recipes and nothing is lost on phone change.
2. AI extraction of clean ingredients/steps from caption or video audio.
3. Verify/replace microlink if Instagram previews are mostly empty.

## Testing
No test suite. Previously tested with Playwright + headless Chrome serving the folder over `http://localhost` (service worker needs localhost/https): share-launch URLs (`/?text=...`), save/edit/delete, Back button, cooking mode, themes, `Page.getAppManifest` and `Page.getInstallabilityErrors` via CDP. Syntax check: extract the inline script and run `node --check`.
