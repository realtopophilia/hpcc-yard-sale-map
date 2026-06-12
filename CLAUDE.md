# Highland Park Yard Sale Map

Interactive map for the 2026 Highland Park Neighborhood Yard Sale (**Sunday**, June 14, 2026, 9am–3pm, Pittsburgh PA), built for the Highland Park Community Council (HPCC).

The `STOPS` data is the real 2026 sign-up list (162 households), cleaned from the registration spreadsheet and geocoded with the US Census batch geocoder on June 10, 2026. Categories were auto-assigned by keyword matching on the item descriptions. One registrant ("Ben Orsburn (TBD)") had no address yet and is not on the map.

## Architecture

This is a single static file: `index.html`. There is no build step, no bundler, no framework, no backend. All HTML, CSS, JS, and the sale data live in that one file. Keep it that way — the whole point is zero-maintenance hosting.

Static assets sit alongside it: `hpcc-logo.jpg` (welcome modal + favicon), `tree-white.png` (white tree in the banner, links to hpccpgh.org/yardsale), `2026 Highland Park Yard Sale Printable Map.pdf` (linked from the welcome modal), and `qr.png` (QR code to the live site, for flyers). The PDF (4 pages) is generated from `..\yardsale-tmp\print.html` via headless Edge: `msedge --headless --no-pdf-header-footer --virtual-time-budget=25000 --print-to-pdf=<out> <file-url-of-print.html>`. Owner requirements baked into it: 0.25in margins (fewest pages possible); optimized for black-and-white printing (grayscale CARTO light basemap + grayscale filter, solid black numbered dots, festival = white circle with black ★); page 1 = map, page 2 starts with a Festival & Flea Market section (1–2 sentence intro + its own table), then street-sorted listings with full item descriptions (2026 data only, straight from `STOPS`); no Kid's Biz, no business specials, no map-pickup locations, no fundraising campaign. Regenerate if STOPS or the festival lineup change (refresh `yardsale-tmp\stops.js` from index.html first).

External dependencies (CDN, loaded with `defer`):

- Leaflet 1.9.4 (unpkg) — map engine
- Sortable.js 1.15.0 (cdnjs) — drag-to-reorder on My List
- CartoDB Voyager raster tiles — basemap
- Google Fonts (Playfair Display) — header typography

The entire app script is wrapped in a `DOMContentLoaded` listener (because the CDN scripts are deferred). Handler functions used by inline `onclick` attributes are exported via `Object.assign(window, {...})` at the bottom of the script. If you add a new function referenced from HTML, add it to that export list.

## Data model

- `STOPS` — array literal near the top of the script. Each entry: `{ address, items, categories[], lat, lon }`. The address string is the unique key everywhere (localStorage, lookups).
- `HOUSE_COUNT` — derived from `STOPS.length` before the festival entry is pushed. Don't hardcode house counts in copy; counts are computed.
- The Bryant St Festival entry is `STOPS.push(...)`-ed separately and styled with a 🎪 icon, pinned at Bryant St & N Euclid Ave with `zIndexOffset:1000` so it stacks above house pins. Its lineup lives near the top of the script — `FESTIVAL_SCHEDULE` (`{ time, act }`), `FOOD_TRUCKS` (`{ name, url? }`, linkified when url present), and `VENDOR_OVERVIEW` (HTML intro blurb) and `VENDORS` (the full 42-vendor list with names + wares, alphabetical — added at owner request June 11; never include sign-up emails). The print version mirrors this in `yardsale-tmp\print.html`. All rendered by the festival modal (`openFestival()`), reachable from the 🎪 popup and the welcome modal.
- Categories are **internal-only** filter/search helpers — by owner decision they are never displayed on a listing (no per-house tags, no category pin colors, no chip counts) so no house feels pigeonholed. All house pins use the uniform `PIN_COLOR`. The filter chips render from `CATEGORIES` (computed from stops, alphabetical, internal "Misc" bucket hidden).
- `SYNONYMS` — search-term expansion map.

Festival hours: flea market from 9am; music/festival from 10:30 (sale runs 9am–3pm).

"Find Me" (header button, `findMe()`): geolocation via `watchPosition` — pulsing blue dot + accuracy circle, flies to the user on first fix then follows quietly; tapping again stops the watch and removes the dot. Requires HTTPS (or localhost). Saved (starred) stops render as gold ★ pins in every view, not just My List.

## State (localStorage, per device)

| Key | Contents |
|---|---|
| `yardSaveSaved` | saved addresses (My List) |
| `yardSaveChecked` | checked-off saved stops |
| `yardSaveVisited` | visited (grayed-out) stops |
| `yardSaveOrder` | custom drag order, or absent = distance sort |
| `hpccWelcomeSeen_v2` | welcome modal dismissed flag — bump the suffix to re-show the modal to everyone |

The welcome modal doubles as the how-to guide (icon-led bullets) and is reopenable anytime via the "?" button in the header (`openHelp()`).

## Performance conventions (preserve these)

- **Icon cache:** `cachedIcon()` memoizes `L.divIcon` instances. Never create divIcons inline in update loops.
- **Diffed updates:** `setMarkerIcon(marker, key, fn)` skips DOM work when the icon key is unchanged; `setMarkerVisible()` adds/removes from the layer only on actual change. Never call `markerLayer.clearLayers()`.
- **Debounced search:** input goes through `onSearchInput()` (140 ms). Programmatic refresh calls `doSearch()` directly.
- Search text is precomputed per stop (`stop._text`); address lookup uses the `ADDR_IDX` map, not `findIndex`.

## Mobile behavior

≤640 px: the sidebar becomes a bottom sheet (peek height 84 px — the `PEEK` constant in `initSheetSwipe()` must match the CSS `translateY(calc(100% - 84px))`; kept tall to clear phone edge-gesture zones). The handle supports tap and swipe; the drag handler is non-passive and calls `preventDefault()` plus `overscroll-behavior-y:none` on body to stop pull-to-refresh. Tapping the map collapses the sheet. Uses `100dvh` with `100vh` fallback. Modals are `max-height`-capped and scroll so large system font sizes can always reach the buttons. Filter chips are multi-select (`activeCats` Set, OR semantics); "Food & Drink" is a real category on stops selling lemonade/baked goods/etc. Search blurs on Enter to dismiss the phone keyboard.

## Operations

- **Day-of announcements:** set the `ANNOUNCEMENT` constant (top of the script) to a short message and push — a dismissible gold banner appears under the header for everyone. Empty string = hidden.
- **Visit counts:** GoatCounter is registered and live — dashboard + CSV export at https://griessjason.goatcounter.com (owner's account).
- **Social previews:** Open Graph tags + `og-preview.png` (1200×630, generated with System.Drawing from the logo). If the canonical URL ever changes (e.g. custom domain), update `og:url` and `og:image`.

## Pending content (placeholders the owner will supply)

- "Ben Orsburn (TBD)" sign-up — add to `STOPS` when the address is known (then regenerate the PDF).

The "💛 Support HPCC" button at the sidebar bottom links to HPCC's donation form: https://hpcc.app.neoncrm.com/forms/donate (owner-confirmed).

## Testing checklist after any change

- Desktop: search "bike" → results render, pins resize/dim; star a result; My List tab shows numbered gold pins; drag to reorder; pins renumber.
- Category chips filter the map and the list; clicking the active chip again clears it.
- Mobile width (≤640 px): sheet swipes up/down and taps open/closed; searching auto-expands the sheet; tapping a result flies the map and collapses the sheet.
- Popup: ☆ Save pill toggles; "✓ Done" grays the pin; no category tags should appear on listings.
- Reload: list, order, checked, and visited states persist.
- No console errors on load.
