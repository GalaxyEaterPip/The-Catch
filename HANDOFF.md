# The Catch: project handoff (current)

Read this first. It is the full state of the project as of cache version `muskie-v25`. The code is in the `muskie-log` folder next to this file. Read index.html before changing anything.

## What this is
"The Catch" is an offline-first catch log for muskie fishing trips in the Northwoods of Wisconsin that I take with my dad. It is a PWA (installable web app) with no backend. I have no coding experience (I used Notepad++ on Windows), and I want things explained in plain language. I now have a Claude Pro plan and am continuing with Opus 5.5 in the Claude desktop app, so you can edit the files in my folder directly: make changes in place, then tell me which files changed and what in plain terms. If you do not have access to the folder, ask me to attach it or upload index.html and sw.js.

## Hard rules and preferences
- Do NOT generate original icons or artwork. I supply my own PNG icons. Temporary placeholders are fine until mine arrive. Small plain UI glyphs (e.g. the "no picture" icon) are okay but I may replace them.
- No claude.ai hosting. The app is plain files hosted once on GitHub Pages; after install everything works offline. Data lives only on each device.
- Never rename the GitHub repository or change the web address (data is stored per address). Never rename the internal database "muskielog" (IndexedDB) or the localStorage key "muskie-view", or existing data disappears.
- Ask before big design changes. Say honestly what you could not test. Everything so far was verified only by running logic in Node with a fake page and previewing rendered images, never in a real browser. I have run it on my Android and desktop. If you can test in a real browser or ask me for screenshots, please do.
- I iterate on small visual details a lot. Do exactly what I list, flag anything extra you add, and keep changes easy to revert. When a feature does not work out, remove it completely rather than leaving code behind.
- Only update this handoff when I ask.
- Devices: my Android (Chrome, installed), my dad's iPhone (Safari, Add to Home Screen), and my Windows desktop (I sometimes open index.html straight from the folder, so nothing may depend on loading separate files via CSS or fetch from file://).

## Deploying (after every change)
1. In sw.js bump the cache name (`muskie-v25` now, next is `muskie-v26`) whenever ANY cached file changes, or phones keep showing the old version. A change appears on the second open after upload.
2. Upload changed files to my GitHub repository (Pages deploys from the main branch root). If you can set up git for the folder, commit and push so Pages redeploys itself. Pages needs a public repository on a free account; the code is public, my data never leaves my devices.
3. Installed icons and names update only on reinstall. Warn me to make a backup first, since removing a home-screen icon can erase data on some phones.

## Files in the folder
- index.html (about 120 KB): the whole app (HTML, CSS, JavaScript, icon sprite, embedded icon masks).
- sw.js: offline cache. FILES: ./, index.html, manifest.webmanifest, icon-192.png, icon-512.png, loading.gif, apple-touch-icon.png. Add new assets there.
- manifest.webmanifest: name "The Catch", standalone, dark theme, icons 192 and 512 (any) plus 512 maskable.
- icon-192.png, icon-512.png, apple-touch-icon.png (180): my hook icon in brass (#d9a441) on deep teal (#0d2526).
- loading.gif: launch screen, my own muskie_vector.gif with the "now loading..." text erased (plays once, 5.2 s, holds on the last frame).

## How index.html is organized
- CSS variables: --bg #0d2526, --panel #143537, --ink #e6efe9, --mute #8fb0a8, --brass #d9a441, --line #24494b. Georgia headings, system sans body.
- Icon sprite (`<symbol id="i-...">`): Lucide fishing rod (i-person, anglers), Lucide map pin (i-lake, lakes), Lucide crosshair (i-pin, location details). Lucide is ISC licensed; the credit comment is in the file. Other outline icons: calendar, clock, ruler, hook, sun, cloudsun, cloud, rain, snow, moon, note, camera, plus, trash, download, upload, search, sort, group, chev, chart, fishgen (generic fish), nopic (picture with a slash).
- Image icons are CSS masks with base64 PNG data URIs (they take the current text color). Data URIs are required because external mask images fail from file://.
  - `.fi.musky` and `.fi.pike`: share one canvas so the pike is about 79% of the musky size on purpose. `speciesIcon(species)`: Muskellunge gets musky, Northern Pike gets pike, any other species gets the generic outline fish. `ic('fish')` returns the musky (heading, Catches tab, labels, empty state).
  - `.li` lure icons: `.li.suick` is the DEFAULT lure icon everywhere (card, form labels, group headers, Insights heading, Insights tab). `.li.medussa` is used for Medusa/Medussa, Realistic Crankbait and Orange Tail. Bucktails (Mepps 5, Weedkiller, Giantkiller), topwater (Bobblehead) and the Blue Dart/Drifter Hellhound currently get class `li suick bucktail`, `li suick topwater` and `li suick bluedart`: Suick placeholders with their own classes, so when I supply an icon, add a mask rule for that class (with its aspect-ratio) after the `.li.suick` rule. `lureIcon(name)` chooses. On cards the lure icon is 1.2em tall and Medussa 1.7em, on purpose.
  - `.hk` is my hook icon as a mask (Add tab, and rotated 45 degrees for special notes).
  - To change an icon: make a cropped alpha mask from my PNG (composite on white, ink = 255 minus gray, crop to the drawn area, scale, smoothstep the alpha), save as LA PNG, embed as a data URI. My source art: musky.png, pike_1.png, thriller.png, medussa_f54d4e.png, the hook PNG (noun_Hook_7541036__700_alt.png).
- Tabs: Catches, Insights, Add, Import (Insights uses the Suick icon, Add uses the hook). Sections: splash, #log (search, sort, group, list, backup, restore), #ins (species filter, output), #add (form), #imp (importer).
- Script: splash (SPLASH_MS = 3000, tap to skip); IndexedDB; show(); add form with photo shrink (max 1600 px JPEG) and a "No picture taken" checkbox; card(); renderList() with SORTS and GROUPS; Insights; importer (parseNote plus preview); backup/restore (mergePlan); cleanRec() migration run on load.

## Data record (IndexedDB "muskielog", store "catches")
id, created, date (YYYY-MM-DD, or "Unknown"), time (24 h, shown as 12 h), length (inches as a number, or the text "??"), species, lake (may be "Unknown"), angler, lure, pattern (lure color), patternAssumed, weather (free text with suggestions), spot (location details), notes, noPic, figure8, firstCast (booleans), photo (Blob or null).
- Weather suggestions: Sunny, Sunny (in shade), Partly cloudy, Mostly cloudy, Overcast, Drizzle, Light rain, Rain, Sleet, Snow. Any other wording is allowed.
- On load, cleanRec() fixes older records: removes "nopic", "F8", "first cast"/"1st cast" and the camera emoji from notes, lure and pattern, and sets the matching flags; Medusa is renamed Medussa.
- Unknown dates sort last, group under "Not recorded" and are left out of the Insights day count. "??" lengths sort last, show without the inch mark, and are excluded from averages and bests.

## Catches tab and cards
- Search (any text), sort: newest, oldest, longest, shortest, with photo first, without photo first. Group by: day, month, year, lake, species, angler, lure, lure and color, lure color, weather (collapsible headers with count, average, longest). Sort and group choice is remembered per device.
- Card: header row with the big length, the species icon with the species name under it, and (if present) the lure icon with the lure name and color under it, right aligned in muted text. Then the detail lines in this order: date and time, lake (shown as "<name> Lake" unless it already contains lake/flowage/river/reservoir/pond/creek/chain/unknown), location details, weather, angler, notes (own icon, lighter text), "Figure 8!" and "1st Cast!" special notes (white bold text with the hook icon rotated 45 degrees), then "No picture taken" (shown only when flagged and no photo). Delete and "Add photo" buttons (Add photo is hidden when a catch is flagged no picture).
- Insights tab: species filter; personal bests by lake; lure rankings (most catches / average / longest; lure only or lure and color); time of day; weather; anglers. Counts come from catches only, with no hours fished.

## Importer (Import tab) and my Obsidian notes
- Format: a date line (M/D/YY), then the lake on its own line (unquoted), then optional location lines IN QUOTES, then one line per fish: length, species, weather, time, angler code, lure, notes. A new unquoted line later the same day is a new lake. A location note applies to the next fish, plus following fish while each is within 15 minutes of the previous one. Times may lack minutes (3PM).
- ?? handling: a length of ?? stays "??". A ?? lake becomes "Unknown". A ?? date becomes "Unknown": a line of ?? is a date when another unquoted lake line follows it and a lake when a fish or quoted line follows; patterns like ??/??/26 are dates.
- Ignored: the camera emoji (📸 📷).
- Special words anywhere in a fish line (removed from the notes): nopic sets noPic; F8 sets figure8; first cast or 1st cast sets firstCast. They are not lures, colors or locations.
- Angler codes: C = Connor, D = Dad. Species shorthand: muskie to Muskellunge, pike to Northern Pike, LMB to Largemouth Bass, SMB to Smallmouth Bass. Weather shorthand: sunny/shade to Sunny (in shade), light drizzle to Drizzle; unknown wording kept as written.
- Lure list defaults (editable in the Import tab; format `word in notes = full name | assumed color`): suick = Suick Thriller | Firetiger; medusa and medussa = Medussa; headbanger = Headbanger; blue dart = Drifter Hellhound; realistic crankbait = Realistic Crankbait; orange tail = Orange Tail; bobblehead = Bobblehead; mepps5 and mepps 5 = Mepps 5; weedkiller = Weedkiller; giantkiller = Giantkiller. Colors: perch, black/white, silver/black, firetiger, and ft = firetiger. A color counts only if written right next to the lure name; anything else after the lure is the note.
- The preview shows catches to add and existing ones that will get new special notes (no picture, Figure 8, 1st Cast). Duplicates (same date, time, length, lake) are skipped. The importer does not recognize an F8 or first-cast written on its own line with no fish; tell me if I use that form.

## Backup and merge
"Save a backup file" exports JSON including photos (share sheet on phones, download on desktop). "Restore or merge a backup" adds missing catches, attaches photos to existing catches that lack one, skips the rest, and keeps "??" lengths. This is also how my phone, my dad's phone and a new web address get merged.

## Waiting on me
Icons I will supply: Blue Dart (Drifter Hellhound), bucktail (Mepps 5, Weedkiller, Giantkiller) and topwater (Bobblehead). For each, add a mask class and wire it into lureIcon().

## Ideas and not built yet
1. Edit an existing catch (today only delete and add-photo exist).
2. Full-size photo view (tap a photo).
3. Species icons for other fish (I will supply art).
4. CSV export and merging duplicate catches.
5. Card background bubbles: tried and reverted. If revisited, make one fixed approved design (no randomness), bigger and thicker bubbles at lower opacity, no overlap, shown to me as a static preview before it goes in.
6. More Insights ideas (trip summaries, seasons, lure by lake).
