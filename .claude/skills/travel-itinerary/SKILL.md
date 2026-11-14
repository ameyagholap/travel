---
name: travel-itinerary
description: Build a shareable, day-by-day travel itinerary as one self-contained HTML file (collapsible days, left navigation, Google Maps route embeds, click-a-sight photo pane) in the same format and depth as the Scandinavia and Iceland trip. Use when the user gives a destination, country list or rough day plan and wants an itinerary to share with friends or family.
---

# Travel itinerary builder

Produces one HTML file the user can send to a friends group. Reference implementation: `scandinavia-itinerary.html` in the project root (if it exists, open it and copy its CSS, markup and script structure so the new file looks and behaves the same). This skill is the spec; the reference file is the proof of what "same format" means.

## 1. Before writing anything: ask

The user's standing rule is to ask whenever a prompt is ambiguous. Ask in **one** AskUserQuestion call (up to 4 questions), then proceed. Cover what is not already stated:

- **Season or month** (changes sights, opening hours, daylight, ferries, snow/ski options, wildlife and aurora).
- **Days per country** and the **route order**, including where they land and where they leave from.
- **Transport style** between and inside places (train, flights, rental car, tours, cruise).
- **Output**: local file only, or also a published share link (default: local file only).

Don't ask what you can decide with a sensible default. Do state the defaults you picked in the final message.

When the user supplies their own day plan, keep every sight they listed. Add what is missing as alternatives, never as replacements.

## 2. Research and honesty

- Use WebSearch (`mode: "standard"`) to check anything that can be wrong or has changed: ferry and cruise links, closures and rebuilds, operator changes, seasonal opening, safety advisories. Examples that mattered before: a museum closed for a rebuild, a "cruise" that does not exist, a volcanic area that closes attractions.
- If the user asks for a link that does not exist (for example a direct ferry), say so, explain the nearest real option, and build the plan around it.
- Anything written from memory (opening hours, prices, transport durations, restaurant names) gets a hedge ("about", "check", "from memory") and goes in the final message as something to verify. Prefer well-known, long-established venues. Never invent a place.
- Never include the user's personal details in searches.

## 3. Content spec (per day)

One card per day, numbered continuously across the whole trip. Travel days are real days. An optional extra day is badged "Optional +1 day" and does not renumber the rest.

Each day has:

1. **Title, place line and Day badge.**
2. **How (getting there):** exact mode, route, rough duration, where to book, the main pitfall (for example seasonal timetable).
3. **See & do:** crisp bullets with named places. Mark world-famous sights with a leading `★`. Add the "peculiar and popular" detail (record, quirk, film location, Nobel or royal link).
4. **Eat:** named dishes and named, established places. Include the local must-try dish. Note vegetarian or Indian options where they exist.
5. **Tip:** one practical line (timing, tickets, weather, crowd-avoidance).
6. **Alternatives & popular add-ons:** a collapsible list. This is where season variants, weather fallbacks, famous sights that did not fit, and "too far" options go. Include a short list of tap-to-open sights for the alternatives.
7. **Tap-a-sight chips** for the main sights (see section 5).

Rules:

- Keep days realistic: arrival days are light, no more than two or three anchor sights a day, and flag the long drive or hike day.
- Seasonal logic: pick the variant for the user's month and put the other in Alternatives.
- Audience: friends travelling from India. Surface world-renowned names first so people can relate and bring back memories (for example Nobel laureates with Indian links, famous brands, film locations, the blue tin of Danish cookies). Mention safety items explicitly (for example sneaker waves, ice, wind).
- Brands: add a "world-famous brands" item per country to the "Famous, but left out" box, tagged as too far, no public tour, or easy shop stop. Put easy shop stops and one brand store on a real day.

## 4. Page-level content

Top of page, in order:

1. Hero: title naming **every** country (and the `<title>` tag too), "N days" summary, route chip strip, Expand all / Collapse all buttons.
2. "★ = world-famous" legend box.
3. **At a glance** table: Day, Where, Highlights, for every day.
4. **Famous, but left out (too far for this route)** box, including brands.
5. **Check these before booking** box: closures, seasonal openings, daylight, safety advisories. Do not repeat notes the user asked to remove.
6. One section per country (and a short section for any cruise or long transfer), each with an overview line, a map block, and its day cards. Add a short "essentials" box to a country when it needs one (for example self-drive rules for Iceland).
7. **Essentials** at the end: flights to and from home (open-jaw where it saves doubling back), visa (Schengen applies across many countries; say where to apply), money and city transport apps, advance bookings, a must-try food list per country.
8. Footer: prices and timetables change; confirm before booking.

## 5. Page spec (what "same format" means)

- **One self-contained `.html` file**, no build step, no local images. Inline CSS and JS. Day content lives in JS data objects and is rendered on load.
- **Design:** CSS tokens on `:root`, a dark-mode override under `prefers-color-scheme`, a per-country accent colour, responsive down to phone width, print styles that expand every day.
- **Days collapsed by default.** Click or press Enter/Space on the header to toggle. Expand all / Collapse all in the hero and in the left pane. Deep links `#day-N` open that day.
- **Left navigation (screens 1100px and wider):** fixed, 360px, dark background (navy in light mode, near-black in dark mode), light accent colours for country headings, one line per day with an ellipsis and a `title` tooltip, active day highlighted while scrolling, page content offset by the pane width. Below 1100px show a sticky chip bar of country anchors instead.
- **Maps:** one or more Google Maps embeds per country block, with a numbered "stops in order" line and an "Open full route in Google Maps" button. Embed URL pattern: `https://maps.google.com/maps?f=d&saddr=A&daddr=B+to:C+to:D&output=embed` (add `&dirflg=w` for walking). Full link: `https://www.google.com/maps/dir/A/B/C`. Keep each map to 10 stops or fewer. Split big countries into walkable city sights, day trips by train, and drives. This embed is a keyless legacy format, so always keep the button.
- **Right-hand sight pane:** clicking a sight chip slides in a fixed pane (max 440px, full width on phones, Esc or × closes). It shows up to 3 photos, **Why visit**, **How to enjoy it**, an **About** paragraph, and an **Open in Google Maps** button.
  - Photos and the About text are fetched live in the browser from public sources and embedded, never downloaded: Wikipedia REST `page/summary/{title}` and `page/media-list/{title}` (CORS is open; take the largest `srcset` entry, skip svg, flags, logos, maps), with Wikimedia Commons search (`action=query&generator=search&gsrnamespace=6&prop=imageinfo&iiurlwidth=800&origin=*`) to top up to 3 photos.
  - If the Wikipedia title 404s, fall back to the Wikipedia title search by name. For sights with no English Wikipedia page set `w:""` so the pane uses Commons only.
  - Credit line: photos and summary come from Wikipedia / Wikimedia Commons under their open licences.
- Data shapes used by the reference file:
  - `S[id] = {name, q, w, why, how}` for every sight (`q` is the Google Maps query, `w` the Wikipedia title).
  - `DS[dayKey] = [sightIds]` for the main chips and `DA[dayKey]` for chips inside Alternatives.
  - `ALTS[dayKey] = ["html", ...]` for the Alternatives list.
  - Day objects `{n, t, loc, how, see:[], eat:[], tip, alt?, badge?}`.
  - `MAPS[countryKey] = [{label, mode, stops:[]}]`.
  - Glance rows `[day, where, highlights]`.

## 6. Build and verify

1. Write the file to the project folder as `<destination>-itinerary.html`. Edit the existing file in place when the user asks for changes, and keep everything the user already approved.
2. Syntax-check every `<script>` block with Node (`new Function(src)`), and check that every sight id used in `DS`, `DA` and the day data exists in `S`.
3. Check that Wikipedia titles resolve, one request at a time with about a 1.5 second pause, because the API rate-limits bursts (HTTP 429 is not "missing"). Fix 404s with a better title or `w:""`.
4. **Never run Playwright or any headless-browser test unless the user asks in that turn.** Say plainly that the page has not been viewed in a browser.
5. Update the day count, route chips, at-a-glance table, sidebar and checklist whenever days or countries are added. Re-check that the title and heading name all countries.
6. Do not add a `spec/` or `prompts/` folder for this unless the project already uses them.

## 7. Final message

Start with a one-line audience note ("Written for: ..."). Then report briefly:

- What was built and where the file is.
- The defaults and assumptions chosen.
- Anything that does not exist or changed from what the user assumed (for example a missing ferry).
- What was written from memory and needs checking before booking.
- Whether the page was viewed in a browser (it was not, unless the user asked).

Offer a published share link only if the user chose it earlier; otherwise mention it in one line.
