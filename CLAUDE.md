# CLAUDE.md — Greece Trip Planning Site

## Project Overview

A single-page static web application for planning a Greece road trip (7–17 March 2026). The site covers an 11-day itinerary starting and ending in Athens, with stops at Meteora, Delphi, Nafplio, and Cape Sounion.

**Deployed as:** A single static HTML file — no build step, no server required.
**Open in browser:** `open index.html` or serve with any static file server.

---

## Architecture

This is a **zero-dependency, single-file application**:

```
greece-trip/
└── index.html    # The entire app — HTML, CSS, and JavaScript in one file (~1,250 lines)
```

- No package manager (`npm`, `pip`, etc.)
- No build tools (webpack, Vite, etc.)
- No JavaScript frameworks (React, Vue, etc.)
- No TypeScript — plain ES6+ JavaScript
- No backend or API keys required

All CSS lives in a `<style>` block in `<head>`. All JavaScript lives in a `<script>` block before `</body>`.

---

## Technologies

| Layer      | Technology                                      |
|------------|-------------------------------------------------|
| Markup     | HTML5                                           |
| Styling    | CSS3 (custom properties, Grid, Flexbox)         |
| Scripts    | Vanilla JavaScript (ES6+, async/await, fetch)   |
| Fonts      | Google Fonts (Cormorant Garamond, Cinzel, Jost) |
| Weather    | Open-Meteo API (free, no authentication)        |
| Video      | YouTube Embed API (lazy-loaded iframes)         |
| Map        | Leaflet.js (referenced in CSS, map section uses `<iframe>` embed) |

---

## Design System

### Color Palette (CSS Custom Properties)

```css
--bone:         #f5f0e8    /* page background */
--sand:         #e8dcc8    /* warm secondary background */
--terracotta:   #c4622d    /* accent, section tags, restaurant tabs */
--olive:        #6b7c4b    /* activity tags, checked packing items */
--aegean:       #1a4a6b    /* primary headings, nav background */
--aegean-light: #2a6a9b    /* lighter variant */
--marble:       #f8f5f0    /* card backgrounds, light text on dark */
--ink:          #1a1510    /* near-black body text, footer */
--gold:         #c9a84c    /* borders, accents, chevrons, progress */
--mist:         #8a9aa8    /* secondary/muted text */
```

### Typography

- **Cinzel** — serif, for section headings and card titles (classical feel)
- **Cormorant Garamond** — serif italic, for preview text and atmospheric copy
- **Jost** — sans-serif, for body text, labels, tags (weight 300/400/500)

### Sizing Convention

Font sizes use `clamp()` for responsive scaling on major headings. Small UI labels use `rem` in the `0.55–0.85rem` range, with `letter-spacing` and `text-transform: uppercase` for structure.

---

## Page Sections

The page is structured as a vertical scroll with the following sections (in order):

| Section ID      | Purpose                                           |
|-----------------|---------------------------------------------------|
| *(hero)*        | Title banner, travel dates, animated entrance     |
| *(budget-strip)*| Budget overview bar (terracotta background)       |
| `#map-section`  | Embedded map with route stops                     |
| `#weather`      | Live weather cards via Open-Meteo API             |
| `#itinerary`    | Interactive 11-day alternating timeline           |
| `#gallery`      | Photo gallery (placeholder cards, upload-ready)   |
| `#audio-guide`  | Video guide cards with YouTube embeds             |
| `#hotels`       | Hotel cards with reviews                          |
| `#restaurants`  | Filterable restaurant recommendations             |
| `#packing`      | Interactive checklist with progress bar           |
| `#practical`    | 8 info cards (flights, car rental, SIM, etc.)     |
| *(footer)*      | Route summary                                     |

---

## JavaScript Data Structures

All data is hardcoded as `const` arrays in the `<script>` block. There is no external data source.

### `DAYS` — 11-day itinerary
```js
{
  day: 'Day 1',
  date: 'Sat\n7 Mar',       // \n splits day-of-week from date
  title: 'Fly into Athens',
  transport: '✈',           // emoji shown in desktop timeline sidebar
  preview: '...',           // one-line teaser shown when card is collapsed
  desc: '...',              // full paragraph shown when card is expanded
  highlights: ['...'],      // bullet list shown when expanded
  tags: ['Label|type']      // pipe-separated: display label | CSS class suffix
                            // types: transport, hotel, activity
}
```

### `WEATHER_STOPS` — 5 locations for weather API
```js
{ city: 'Athens', dates: '7–9 Mar · Days 1–3', lat: 37.9715, lng: 23.7257 }
```

### `SITES` — 8 sites for video guide section
```js
{
  id: 'acropolis',
  name: 'Acropolis of Athens',
  location: 'Athens · Day 2',
  preview: '...',      // shown only if videoId is null
  videoId: '9XvUjdTkJ3o'   // YouTube video ID; null = "coming soon" card
}
```

### `PACKING` — 6 packing categories
```js
{
  icon: '👕',
  title: 'Clothing',
  items: ['Merino T-shirts × 4', ...]
}
```

---

## JavaScript Functions

| Function | Signature | Description |
|---|---|---|
| `buildTimeline()` | `() → void` | Renders DAYS into alternating timeline DOM nodes |
| `toggleDay(el)` | `(Element) → void` | Expands/collapses a day card; closes others; mobile scroll |
| `fetchWeather()` | `async () → void` | Fetches Open-Meteo API for all 5 stops; renders weather cards |
| `wmoCond(code)` | `(number) → {icon, text}` | Maps WMO weather code to emoji and description string |
| `fToC(f)` | `(number) → number` | Fahrenheit to Celsius conversion |
| `buildPacking()` | `() → void` | Renders PACKING categories as interactive checklist |
| `togglePack(key, el)` | `(string, Element) → void` | Checks/unchecks a packing item; updates progress |
| `updateProgress(total)` | `(number) → void` | Updates progress bar width and count label |
| `renderAudioCards()` | `() → void` | Renders SITES as video/coming-soon cards |
| `loadVideo(siteId, videoId)` | `(string, string) → void` | Swaps thumbnail for live YouTube iframe on click |
| `filterResto(city, btn)` | `(string, Element) → void` | Filters restaurant cards by city tab |

---

## External API

### Open-Meteo Weather API
- **URL:** `https://api.open-meteo.com/v1/forecast`
- **Auth:** None required
- **Called:** On page load via `fetchWeather()`
- **Parameters:** `latitude`, `longitude`, `current`, `daily`, `timezone=Europe/Athens`
- **Failure handling:** Replaces grid with a plain error message; does not throw

---

## CSS Animations

```css
@keyframes fadeUp   /* Hero text entrance — opacity + translateY */
@keyframes expandIn /* Day card body reveal — opacity + translateY */
@keyframes spin     /* Loading spinner — 360° rotation */
```

---

## Responsive Design

Breakpoint: `@media (max-width: 768px)`

Mobile changes:
- Nav links hidden; logo font reduced
- Timeline converts from 3-column CSS Grid to vertical flex list with left-border dot
- Transport emoji column (`.day-meta`) hidden with `display: none !important`
- Gallery, audio, hotel, restaurant grids collapse to 1–2 columns
- Packing and practical grids collapse to 1 column

---

## Conventions

### Making Changes

1. **Edit `index.html` directly** — there is no source/dist split.
2. **CSS changes** go in the `<style>` block in `<head>`. CSS is organized by section with comment blocks like `/* ── WEATHER ── */`.
3. **JavaScript changes** go in the `<script>` block before `</body>`. Sections are separated by `// ═══ SECTION NAME ═══` comments.
4. **Data changes** (itinerary, hotels, restaurants, etc.) are hardcoded in the JS data structures — update the relevant `const` array.

### Adding a New Itinerary Day

Add an object to the `DAYS` array following the existing schema. Tags must use the `'Label|type'` pipe format where type is one of `transport`, `hotel`, `activity`.

### Adding a Video Guide Entry

Add an object to `SITES`. Set `videoId` to a YouTube video ID string if a video exists, or `null` to show a "coming soon" card.

### Adding a Restaurant

Restaurants are rendered via hardcoded HTML in the `#restaurants` section (not from a JS array). Add a `.resto-card` div with the appropriate `data-city` attribute matching the filter tabs (`All`, `Athens`, `Meteora`, `Nafplio`).

### Styling Conventions

- Use existing CSS custom properties (`var(--aegean)`, etc.) — never hardcode color hex values directly in new rules.
- Add responsive overrides inside the existing `@media (max-width: 768px)` block.
- Section background colors alternate between `var(--bone)`, `var(--marble)`, `white`, and themed backgrounds (`var(--aegean)`, `var(--ink)`, `#f0ece4`).

---

## Git

- **Remote:** `http://local_proxy@127.0.0.1:34523/git/risha-pea/greece-trip`
- **Main branch:** `master`
- **Active development branch:** `claude/claude-md-mm3845fr8q1ntnzj-2cS3O`
- **Commit style:** Short imperative messages (e.g. `Update index.html`)
- No CI/CD, no hooks, no `.gitignore`

---

## Trip Reference

- **Dates:** 7–17 March 2026 (11 days)
- **Route:** Amsterdam → Athens → Meteora → Delphi → Nafplio → Cape Sounion → Athens → Amsterdam
- **Flights:** HV6865 (outbound, AMS 07:00 → ATH 11:20), HV6868 (return, ATH 20:35 → AMS 23:15)
- **Car hire:** Hertz, Mini Fiat 500, manual, unlimited mileage. Pick-up at Athens Airport (Day 4). Drop-off at 25 Syngrou Ave, Athens (Day 10).
- **Hotels:** Luwian Boutique Hotel (Athens, Days 1–3), Meteora Vibes (Meteora, Days 4–5), Kastalia Boutique Hotel (Delphi, Day 6), Hotel Ippoliti (Nafplio, Days 7–9)
