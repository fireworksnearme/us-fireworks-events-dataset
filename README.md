# 🎆 US Fireworks Events Dataset

An open dataset of verified public fireworks displays across the United States — New Year's Eve, Independence Day (July 4th), seasonal winter series, and recurring landmark shows — published in [RFC 7946 GeoJSON](https://datatracker.ietf.org/doc/html/rfc7946), CSV, and JSON formats, with ready-to-read Markdown pages per state.

![Format: GeoJSON](https://img.shields.io/badge/format-GeoJSON%20%7C%20CSV%20%7C%20JSON-blue)
![License: MIT](https://img.shields.io/badge/license-MIT-green)
![Standard: RFC 7946](https://img.shields.io/badge/standard-RFC%207946-orange)

> **Canonical source & live feed**
>
> This repository is a static snapshot. The **canonical, authoritative source** of this dataset is
> [**Fireworks Near Me**](https://fireworksnearme.top), and the **live, always-up-to-date feed** is served at:
>
> ```
> https://fireworksnearme.top/events.geojson
> ```
>
> The live endpoint is refreshed as events are verified, rescheduled, or cancelled. If you build on this data, prefer polling the live feed — the files here are periodic exports of it.

---

## Table of Contents

- [Coverage at a Glance](#coverage-at-a-glance)
- [Repository Layout](#repository-layout)
- [File Manifest](#file-manifest)
- [Quickstart](#quickstart)
  - [Python — GeoPandas](#python--geopandas)
  - [Python — pandas (CSV)](#python--pandas-csv)
  - [JavaScript — Leaflet Map](#javascript--leaflet-map)
  - [curl](#curl)
- [Data Dictionary](#data-dictionary)
  - [Event Records](#event-records)
  - [Enumerated Values](#enumerated-values)
  - [City Viewing Guides](#city-viewing-guides)
- [Format Notes](#format-notes)
- [Live Feed & Refresh Policy](#live-feed--refresh-policy)
- [Attribution & Citation](#attribution--citation)
- [Contributing & Corrections](#contributing--corrections)
- [License](#license)
- [Disclaimer](#disclaimer)

---

## Coverage at a Glance

| What | Count |
|---|---|
| Geo-referenced events (`events.geojson`) | **115** Point features |
| 2026 US event records (JSON/CSV) | 93 |
| New Year's Eve 2026 → 2027 events | 12 |
| Chicago Navy Pier winter series shows | 6 |
| Recurring landmark series (Disney, SeaWorld, Waikiki) | 4 |
| City viewing guides | 49 |
| State Markdown pages | 25 (24 states + Washington, DC) |

**States covered:** AZ, CA, CO, DC, FL, GA, HI, IL, IN, LA, MA, MD, MI, MN, MO, NC, NV, NY, OR, PA, SC, TN, TX, UT, WA — plus two signature international New Year's Eve events (Sydney, AU and London, UK) in the top-level GeoJSON.

Every record carries geographic coordinates, local-time scheduling with UTC offsets, an official source URL, and editorially written viewing, parking, transit, and safety tips.

---

## Repository Layout

```text
us-fireworks-events-dataset/
├── README.md                        ← you are here
├── LICENSE                          ← MIT
├── data/
│   ├── cities/
│   │   ├── cities.csv               ← 49 city viewing guides (CSV)
│   │   └── cities.json              ← 49 city viewing guides (JSON)
│   └── events/
│       ├── events.geojson           ← master RFC 7946 feed (115 features)
│       ├── upcoming-events.geojson  ← upcoming-only feed (2 features)
│       ├── 2026-us.json / .csv      ← full 2026 US season (93 records)
│       ├── 2026-nye.json / .csv     ← New Year's Eve 2026→2027 (12 records)
│       ├── 2026-chicago-winter.json / .csv   ← Navy Pier winter series (6 records)
│       └── recurring-us.json / .csv ← recurring landmark series (4 records)
└── events/                          ← 25 human-readable state pages (Markdown)
    ├── arizona.md
    ├── california.md
    ├── …
    └── washington-dc.md
```

---

## File Manifest

| Path | Format | Records | Description |
|---|---|---|---|
| `data/events/events.geojson` | GeoJSON (RFC 7946) | 115 | Master dataset. All events as Point features with full properties. Mirror of the [live feed](https://fireworksnearme.top/events.geojson). |
| `data/events/upcoming-events.geojson` | GeoJSON (RFC 7946) | 2 | Only events with `status` of `confirmed` or `tentative` that have not yet occurred. |
| `data/events/2026-us.json` | JSON array | 93 | Full 2026 US season — July 4th displays (marked `past`), NYE, and winter events — with the richest per-event schema (descriptions, tips as arrays). |
| `data/events/2026-us.csv` | CSV | 93 | Same records flattened to CSV; tips are pipe-delimited (` \| `). |
| `data/events/2026-nye.json` | JSON array | 12 | New Year's Eve 2026 → 2027 displays (Boston, NYC, LA, Chicago, and more). |
| `data/events/2026-nye.csv` | CSV | 12 | CSV export of the NYE set. |
| `data/events/2026-chicago-winter.json` | JSON array | 6 | Chicago Navy Pier Winter Fireworks series (Saturday nights, Nov–Dec 2026). |
| `data/events/2026-chicago-winter.csv` | CSV | 6 | CSV export of the Navy Pier series. |
| `data/events/recurring-us.json` | JSON array | 4 | Recurring landmark fireworks: Disney's *Happily Ever After* (Orlando), SeaWorld, and Waikiki shows. Adds `seriesId` and `verificationStatus`. |
| `data/events/recurring-us.csv` | CSV | 4 | CSV export of the recurring set. |
| `data/cities/cities.json` | JSON array | 49 | Per-city viewing guides: intros, best viewing areas, transit tips, and local fire-safety links. |
| `data/cities/cities.csv` | CSV | 49 | CSV export of the city guides. |
| `events/*.md` | Markdown | 25 pages | Human-readable state pages (24 states + DC), each linking back to live pages on [fireworksnearme.top](https://fireworksnearme.top). |

---

## Quickstart

### Python — GeoPandas

Read the live feed straight from the canonical source and start analyzing:

```python
import geopandas as gpd

# Live, always-current feed (canonical source)
url = "https://fireworksnearme.top/events.geojson"
gdf = gpd.read_file(url)

# Or use the snapshot in this repo
# gdf = gpd.read_file("data/events/events.geojson")

print(gdf.crs)                        # EPSG:4326 (WGS 84) per RFC 7946
print(len(gdf), "events")             # 115

# Free, confirmed shows in California
ca = gdf[
    (gdf["state"] == "CA")
    & (gdf["isFree"] == True)
    & (gdf["status"] == "confirmed")
]
print(ca[["title", "city", "startDate", "venue"]])
```

### Python — pandas (CSV)

The CSV and JSON files are snapshot exports in this repo; the live endpoint is GeoJSON. To work with tabular data, read the snapshot:

```python
import pandas as pd

df = pd.read_csv(
    "data/events/2026-us.csv",           # snapshot export from this repo
    parse_dates=["startDate", "endDate"],
)

# Shows sorted by date, tip lists split on the pipe separator
df = df.sort_values("startDate")
df["viewingTips"] = df["viewingTips"].str.split(r"\s*\|\s*")

print(df[["title", "state", "city", "startDate", "isFree"]].head(10))
```

> The CSV columns match the [event data dictionary](#event-records) below, with `latitude` / `longitude` columns replacing the GeoJSON geometry. To go from the **live** feed to a DataFrame instead, use `pd.json_normalize(requests.get("https://fireworksnearme.top/events.geojson").json()["features"])`.

### JavaScript — Leaflet Map

Drop-in HTML page — fetches the live feed and plots every event on an interactive map:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Fireworks Near Me — Open Data</title>
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <style>#map { height: 100vh; }</style>
</head>
<body>
  <div id="map"></div>
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <script>
    const FEED_URL = "https://fireworksnearme.top/events.geojson"; // canonical live feed

    const map = L.map("map").setView([39.5, -98.35], 4); // continental US
    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      maxZoom: 18,
      attribution: '&copy; OpenStreetMap contributors',
    }).addTo(map);

    fetch(FEED_URL)
      .then((res) => res.json())
      .then((geojson) => {
        L.geoJSON(geojson, {
          onEachFeature: (feature, layer) => {
            const p = feature.properties;
            layer.bindPopup(`
              <strong>${p.title}</strong><br/>
              ${new Date(p.fireworksAt ?? p.startDate).toLocaleString()}<br/>
              ${p.venue} — ${p.city}, ${p.state}<br/>
              ${p.isFree ? "🎉 Free admission" : "🎟️ " + p.admissionType}<br/>
              <a href="${p.url}" target="_blank" rel="noopener">Event details →</a>
            `);
          },
        }).addTo(map);
      });
  </script>
</body>
</html>
```

### curl

```bash
# Live GeoJSON feed (canonical)
curl -s https://fireworksnearme.top/events.geojson | jq '.features | length'

# Upcoming events only
curl -s https://fireworksnearme.top/events.geojson \
  | jq '[.features[] | select(.properties.status != "past")]'
```

---

## Data Dictionary

### Event Records

The table lists the superset of fields. GeoJSON files place these under each feature's `properties` (with coordinates in `geometry.coordinates`); JSON files are flat arrays; CSV files flatten tips to pipe-delimited strings and use `latitude` / `longitude` columns.

| Field | Type | Present in | Description |
|---|---|---|---|
| `id` | string | GeoJSON, JSON, CSV | Stable event identifier (also mirrored as the GeoJSON feature `id`). Derived from the slug. |
| `title` | string | all | Display name of the event, e.g. *"Macy's 4th of July Fireworks 2026"*. |
| `slug` | string | all | URL-friendly identifier used by fireworksnearme.top. |
| `url` | string (URI) | GeoJSON | Canonical event page on [fireworksnearme.top](https://fireworksnearme.top). |
| `country` | string | all | ISO 3166-1 alpha-2 country code (`US`, `AU`, `GB`). |
| `state` | string | all | Two-letter US state (or DC) code. `NSW` / `England` are used for the two international NYE events. |
| `city` | string | all | City or metro name. |
| `venue` | string | all | Venue or viewing area, e.g. *"Navy Pier"*. |
| `address` | string | all | Street address of the venue. |
| `latitude`, `longitude` | number | JSON, CSV | WGS 84 coordinates. In GeoJSON, see `geometry.coordinates` (`[longitude, latitude]` per RFC 7946). |
| `startDate` | string (ISO 8601) | all | Event start, local time with UTC offset, e.g. `2026-07-04T21:00:00-04:00`. |
| `fireworksAt` | string (ISO 8601) | GeoJSON, JSON | Scheduled launch moment of the fireworks themselves (may differ from event start). |
| `endDate` | string (ISO 8601) | all | Event end, local time with UTC offset. |
| `timezone` | string | all | IANA time zone name, e.g. `America/Chicago`. |
| `status` | enum | all | `confirmed` · `tentative` · `past`. See [enumerated values](#enumerated-values). |
| `eventType` | enum | JSON, CSV | Event classification, e.g. `public-display`, `festival`. |
| `displayType` | enum | GeoJSON, JSON | What the audience sees: `aerial-fireworks`, `ball-drop`, `hybrid`. |
| `admissionType` | enum | GeoJSON, JSON | `free` · `paid` · `rsvp` · `unknown`. |
| `isFree` | boolean | all | Convenience flag derived from `admissionType`. |
| `officialUrl` | string (URI) | all | Official organizer/municipality page for the event. |
| `sourceName` | string | all | Human-readable name of the source used for verification. |
| `verifiedAt` | string (date) | all | ISO date the record was last verified against its source. |
| `lastUpdated` | string (date) | all | ISO date the record was last modified in the dataset. |
| `verificationStatus` | enum | recurring JSON | Verification pipeline state, e.g. `pending`. |
| `seriesId` | string | recurring JSON | Identifies the recurring series a show belongs to (e.g. Navy Pier Saturdays, Disney nightly). |
| `image` | string (path) | JSON | Site-relative image path on fireworksnearme.top. |
| `summary` | string | all | One-to-two-sentence plain-language summary. |
| `description` | string | JSON | Longer editorial description of the show and viewing experience. |
| `viewingTips` | array (JSON) · string (CSV) | JSON, CSV | Where to stand and how early to arrive. Pipe-delimited (` \| `) in CSV. |
| `parkingTips` | array (JSON) · string (CSV) | JSON, CSV | Parking guidance. |
| `transitTips` | array (JSON) · string (CSV) | JSON, CSV | Public-transport guidance. |
| `safetyNotes` | array (JSON) · string (CSV) | JSON, CSV | Crowd and safety guidance. |

### Enumerated Values

| Field | Values |
|---|---|
| `status` | `confirmed` — verified against an official source · `tentative` — expected but not yet confirmed · `past` — the event has already taken place |
| `eventType` | `public-display` · `community-event` · `festival` · `parade` · `stadium-fireworks` |
| `displayType` | `aerial-fireworks` · `ball-drop` · `hybrid` |
| `admissionType` | `free` · `paid` · `rsvp` · `unknown` |
| `verificationStatus` | `verified` · `pending` |

### City Viewing Guides

`data/cities/cities.json` and `cities.csv` — 49 metro-area guides for planning around major displays:

| Field | Type | Description |
|---|---|---|
| `country` | string | ISO 3166-1 alpha-2 code (`US`). |
| `state` | string | Two-letter state code. |
| `stateSlug` | string | URL-friendly state identifier. |
| `city` | string | City name. |
| `slug` | string | URL-friendly city identifier. |
| `timezone` | string | IANA time zone name. |
| `intro` | string | Short editorial overview of the city's fireworks scene. |
| `bestViewingAreas` | array (JSON) · string (CSV) | Curated viewing spots. Pipe-delimited in CSV. |
| `transitTips` | array | Transit advice for event nights (JSON only). |
| `localSafetyUrl` | string (URI) | Local fire-department or official safety page. |

---

## Format Notes

- **GeoJSON** files conform to **[RFC 7946](https://datatracker.ietf.org/doc/html/rfc7946)**: WGS 84 (`EPSG:4326`), `Point` geometries, and `[longitude, latitude]` coordinate order. They load directly in GeoPandas, `L.geoJSON()`, QGIS, ArcGIS, D3, Mapbox GL, and every RFC 7946-compatible tool.
- **Timestamps** are ISO 8601 with the local UTC offset (`2026-12-31T23:30:00-08:00`), so no separate time-zone math is needed to display local showtimes. `timezone` gives the IANA name for DST edge cases.
- **CSV** files are UTF-8 with a header row, RFC 4180 quoting, and use ` | ` (space-pipe-space) as the list separator inside multi-value tip fields.
- **JSON** array files (`2026-us.json`, etc.) are the richest export: they include `description`, `fireworksAt`, `admissionType`, `displayType`, and tips as true arrays.
- **State Markdown pages** (`events/*.md`) are intended for humans, GitHub browsing, and static-site consumption; each event links back to its canonical page on fireworksnearme.top.

## Live Feed & Refresh Policy

| Endpoint | Description |
|---|---|
| `https://fireworksnearme.top` | Canonical website — browse events, cities, and states with full context. |
| `https://fireworksnearme.top/events.geojson` | **Live master feed** (RFC 7946). What `data/events/events.geojson` in this repo is snapshotted from. |

- The live feed is the authoritative version: it is updated as events are added, verified, rescheduled, or cancelled.
- This repository receives periodic snapshot exports of that feed.
- Records carry `verifiedAt` / `lastUpdated` so consumers can judge freshness.
- Poll respectfully — a reasonable interval is once per hour or slower; showtimes rarely change minute-to-minute.

## Attribution & Citation

The dataset is released under the MIT License — free for commercial and non-commercial use. If it helps you, an attribution like the following is appreciated but not required:

```bibtex
@dataset{fireworks_near_me_open_data_2026,
  title  = {US Fireworks Events Dataset},
  author = {{Fireworks Near Me Open Data Contributors}},
  year   = {2026},
  url    = {https://fireworksnearme.top/events.geojson}
}
```

## Contributing & Corrections

Found a missing show, a wrong time, or a cancelled event?

1. **Fastest:** use the event feedback flows on [fireworksnearme.top](https://fireworksnearme.top) — corrections there flow into the live feed and the next snapshot.
2. **Repo issues:** open a GitHub issue on this repository with the event `id`, the field in question, and an authoritative source URL.
3. **Pull requests** against snapshot files are welcome for editorial fixes (tips, formatting); schedule changes should be sourced from an official URL.

## License

Released under the [MIT License](LICENSE) — © 2026 Fireworks Near Me Open Data Contributors.

## Disclaimer

Event details — dates, times, venues, and admission — change frequently and are subject to weather, permits, and organizer decisions. This dataset is provided as-is: **always confirm details via each record's `officialUrl` before attending.** Neither the contributors nor fireworksnearme.top are liable for outcomes arising from use of this data.
