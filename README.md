# Geo Distance Finder

Geo Distance Finder is a small browser app that calculates the straight-line distance between two places. It looks up each location with the OpenStreetMap Nominatim API, reads their latitude and longitude, then uses the Haversine formula to estimate the distance in kilometres.

## Features

- Search for two place names or addresses.
- Fetch location coordinates from OpenStreetMap Nominatim.
- Calculate straight-line distance between the two results.
- Show latitude, longitude, and OpenStreetMap links for both places.
- Display the raw JSON API responses for inspection.
- Run entirely from a single `index.html` file.

## Getting Started

No installation is required. Open `index.html` in a modern web browser.

Because the app calls the Nominatim API from the browser, you need an internet connection when searching for places.

## Usage

1. Enter a starting location in **From location**.
2. Enter a destination in **To location**.
3. Select **Calculate distance**.
4. Review the distance, coordinates, map links, and optional raw JSON output.

For best lookup results, include extra context such as the country, state, or region. For example:

- `Adelaide, Australia`
- `Tokyo, Japan`
- `Paris, France`

## How It Works

The app sends each search term to:

```text
https://nominatim.openstreetmap.org/search
```

It requests JSON results with `format=jsonv2` and limits each search to the best single match with `limit=1`.

Once both locations are found, the app converts their coordinates from degrees to radians and applies the Haversine formula using an Earth radius of `6371 km`.

## Project Structure

```text
.
├── index.html
└── README.md
```

All HTML, CSS, and JavaScript live in `index.html`.

## Notes

- The result is a straight-line distance, not a driving, walking, or transit route.
- Location lookup quality depends on the Nominatim search result returned for each query.
- Public Nominatim usage is subject to OpenStreetMap Foundation policies, including fair-use limits.

