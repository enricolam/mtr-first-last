# Programmatic Display for MTR First &amp; Last Train Info
A tiny, dependency-free web app that shows the first and last train times for any MTR station in Hong Kong. Everything lives in one index.html file — no build step, no framework, no install. Drop it on any static host (or open it locally) and it works.

---

## Features

### 🔍 Smart station search
- Type-ahead dropdown that matches on **English name**, **Chinese name**, or **station code** (e.g. `Lo Wu`, `羅湖`, `LOW`).
- Covers the whole network: Airport Express, Disneyland Resort, East Rail (incl. Lo Wu / Lok Ma Chau branches), Island, Kwun Tong, South Island, Tung Chung, Tseung Kwan O (incl. LOHAS Park branch), Tuen Ma, and Tsuen Wan lines.
- Bilingual display (`English · 中文 · CODE`) so you can confirm you picked the right stop.

### 🎯 Two ways to query
| Mode | What you do | What you get |
|---|---|---|
| **Station only** | Pick an origin | Automatically discovers *every* line and direction serving that station and fetches first/last trains toward each terminus |
| **Station + target** | Pick an origin **and** a target | Fetches the specific journey both ways: services **departing** your station toward the target, and services **arriving from** the target |

The target field is optional — leave it blank for a full station overview.

### 🧭 Direction-aware results
- Results are grouped and sorted by **line name**, then **Depart / Arrive**, then **direction**, so the list reads predictably.
- Each card shows a **First** and **Last** time with a coloured `Depart` / `Arrive` tag.
- Terminus-based enumeration means you see exactly which direction each time applies to (e.g. *to Lo Wu*, *from Admiralty*).

### ⚡ Fast & resilient loading
- Queries run with a **concurrency limit of 4**, so large stations (like Admiralty, with many lines) load quickly without hammering the API.
- Each card fails independently — one bad request shows *"Failed to load"* on that card instead of breaking the page.
- Live spinner and status text while fetching.

### 🔗 Shareable deep links
The current selection is written to the URL, so any view is shareable and bookmarkable:

```
index.html?sta=LOW
index.html?sta=ADM&tar=TST
```

- `sta` (alias: `station`) — origin station, by **code** or numeric ID.
- `tar` (alias: `target`) — optional destination.
- When either parameter is present, the page automatically enters **embed mode**.

### 🖼️ Embed mode
Loading with `?sta=…` adds an `embed` class to `<body>` that hides the header, search box, and footer — ideal for dropping into an `<iframe>` on another site. Only the status line and result cards remain.

### 📱 Responsive, themed UI
- Mobile-first layout capped at `480px`, comfortable on phones and in narrow embeds.
- Clean card design with a teal accent, driven by **CSS custom properties** (`--accent`, `--card`, `--border`, …) so the whole palette is easy to reskin in one place.
- Focus rings, hover states, and a spinning loader for polish.

### 🪶 Zero dependencies
- Pure HTML + CSS + vanilla JS in a single file.
- **All station, line, and direction data is embedded** in the `DATA` object — no separate JSON fetch, no CDN, no npm.
- Deep links are updated with `history.replaceState`, so selecting stations doesn't spam browser history.

---

## How it works

1. **On load**, the page reads `sta` / `tar` from the query string. If present, it resolves them (by code or ID) and immediately fetches.
2. **Without a target**, it looks up `stationToLines[stationId]`, derives every terminus on every line serving the station from `lineDirs`, and builds a depart + arrive query per terminus.
3. **Each query** calls the MTR Journey Planner endpoint:
   ```
   https://www.mtr.com.hk/share/customer/jp/api/HRRoutes/?o=<originId>&d=<destId>&lang=E
   ```
   and reads `firstTrain.time` / `lastTrain.time` from the response.
4. **Results** are sorted and rendered as cards. The URL is kept in sync with the current selection.

### Embedded data shapes
- `DATA.stations` — `{ id: { c: code, e: English, z: 中文 } }`
- `DATA.lineDirs` — `{ "LINE|DIR": [ { i: stationId, e: name }, … ] }` (ordered stop lists per direction)
- `DATA.stationToLines` — `{ stationId: ["LINE|DIR", …] }`

Adding a new station means adding one entry to `stations` and the relevant `lineDirs` / `stationToLines` arrays.

---

## Usage

```bash
# Just open it
open index.html

# Or serve it (any static server works)
python3 -m http.server 8000
# → http://localhost:8000/index.html
```

No install, no config, no build.

---

## Limitations & notes

- **Unofficial**: this talks to MTR's public Journey Planner endpoint, which is not a documented, stable API. It can change or be rate-limited without notice, and browser CORS rules may block it depending on where the page is hosted.
- Times are **platform departure times** as returned by the planner; the app does not model the current date, service-day rollover, or special holiday timetables.
- Times are shown as-is (`—` when the API returns nothing), and there is **no caching** — every page load hits the network.
- Station/line data is **hard-coded** in the file and must be updated manually when the network changes.

---

## Disclaimer

Not affiliated with or endorsed by MTR Corporation. Data belongs to MTR; use responsibly and avoid aggressive polling. Provided as-is, without warranty of accuracy — always confirm against official MTR sources before relying on a first/last train time.
