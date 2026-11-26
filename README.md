# המתכונים של מיכל (Michal's Recipes)

An installable web app (PWA) for saving recipe videos from Instagram, TikTok and Facebook, with categories, kashrut, notes, search and a kitchen mode.

## Files
- `index.html`: the whole app (UI + logic)
- `manifest.json`: install info + **share_target** (makes the app show up in Android's Share menu)
- `sw.js`: service worker (offline + required for install)
- `icon-*.png`: app icons

## Deploy (free, ~5 min)
It needs to be on HTTPS. Pick one:
- **Netlify:** sign in at app.netlify.com → Add new site → Deploy manually → drag this folder in.
- **GitHub Pages:** push the folder to a repo → Settings → Pages → Deploy from branch `main` / root.

## Install on Michal's Android
1. Open the site in **Chrome**.
2. Menu ⋮ → **Install app** (or "Add to Home screen").
3. Done. The app now appears in the Share sheet as "המתכונים".

To save from Instagram: Share → "Share to..." / More → המתכונים.
Tip: long-press the app in the Android share sheet and **Pin** it so it's always first.

## How data is stored
- Recipes: `localStorage`. Photos: IndexedDB. All on her phone only.
- The app asks Chrome for persistent storage, but **backups matter**: Settings → שמירת גיבוי opens the share sheet so she can send the backup file to herself on WhatsApp.

## Auto-fill (title, photo, caption)
- TikTok: official oEmbed (free, no key).
- Instagram / Facebook: microlink.io free tier (~50 lookups/day, no key). Instagram often blocks previews, so sometimes only the link is saved. She can then add a photo with "תמונה משלי" (a screenshot works well).
- The caption text goes into the written recipe, so the recipe survives if the video is deleted.

## Next steps
1. Cloud sync (Supabase) so May can add recipes too and nothing is lost if the phone changes.
2. AI extraction: turn a caption or the video's spoken audio into clean ingredients and steps.
