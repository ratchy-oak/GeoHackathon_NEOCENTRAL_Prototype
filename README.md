# NEO CENTRAL

A static browser prototype for displaying stored province scores and exploring a six-slider arithmetic score. The ranking and detail flows are currently blocked by a JSON filename mismatch. ([Ranking code](index.html#L75), [detail code](city.html#L69), [available data](data/province.json))

The existing page describes the project as a “Smart Capital Ranking” dashboard and a THEOS-2 hackathon prototype. This is the stated framing **per the existing page copy**, not evidence of satellite processing in this repository. ([Page copy](index.html#L52), [footer](index.html#L73))

## Implementation status

| Component | Verified behavior and status | Source |
| --- | --- | --- |
| Ranking | Implemented, but blocked as checked in: fetches a missing JSON path, sorts stored `score` values descending, and builds rows linking to province details. | [`loadProvinces`](index.html#L76) |
| Province details | Implemented, but blocked by the same missing path: selects an exact `province` query match, otherwise the first record, then displays its stored base score. | [`initCityPage`](city.html#L69) |
| Score controls | Logic verified with supplied data and stubbed browser dependencies: initializes six sliders and computes their rounded, equally weighted mean. This is a deterministic arithmetic heuristic, not trained-model inference. | [`setSlider`, `calculateAdjustedScore`](city.html#L97) |
| Radar chart | Chart.js configuration plots the six slider values divided by 100; every update destroys and recreates the chart. Real browser rendering was not verified in this audit. | [`updateRadarChart`](city.html#L107) |
| Map | Implemented Google Maps initialization using stored latitude/longitude and zoom 7; the API key is a placeholder, so a working map is unverified. | [`city.html`](city.html#L89), [API script](city.html#L121) |
| QR page | Constructs a QR code for a hardcoded Colab URL, although its caption displays a different website URL. Payload construction was verified with a stub; QR rendering/scanning was not tested. | [`QRcode.html`](QRcode.html#L37) |
| Analysis exports | Stored per-image, aggregate, and regression JSON files are unused by all three page scripts. They do not drive the displayed province scores. | [`data/per_image_summary.json`](data/per_image_summary.json), [`data/aggregated_summary.json`](data/aggregated_summary.json), [`data/regression_results.json`](data/regression_results.json); scripts in [`index.html`](index.html#L75), [`city.html`](city.html#L66), [`QRcode.html`](QRcode.html#L41) |

## Stack and architecture

The application consists of HTML, inline CSS, and plain browser JavaScript. There is no application server or build step in the supplied implementation; the page scripts fetch static JSON and call browser libraries. ([`index.html`](index.html), [`city.html`](city.html), [`QRcode.html`](QRcode.html))

| Dependency | Version in source | Use |
| --- | --- | --- |
| Bootstrap CSS | 5.3.2, pinned CDN URL | Table, layout, and control styling in [`index.html`](index.html#L7) and [`city.html`](city.html#L7) |
| Chart.js | Unpinned CDN URL; resolved version unverified | Radar chart in [`city.html`](city.html#L9), [`updateRadarChart`](city.html#L107) |
| Google Maps JavaScript API | No version specified | Map in [`city.html`](city.html#L89), [script URL](city.html#L121) |
| QRCode.js | 1.0.0, pinned CDN URL | QR generation in [`QRcode.html`](QRcode.html#L6) and [constructor call](QRcode.html#L42) |
| Google Fonts: Inter | No version specified | Typography in [`index.html`](index.html#L8) and [`city.html`](city.html#L8) |

The ranking page passes the province name in the detail-page URL. The detail page independently fetches the same JSON, rounds NDVI/NDWI/NDBI multiplied by 100, and uses air quality, roads, and flood risk directly. The adjusted score is `Math.round(sumOfSixSliderValues / 6)`; it does not use the base score or regression coefficients. Slider changes update the score and chart, with no persistence or ranking update. ([Navigation](index.html#L90), [initialization](city.html#L69), [update functions](city.html#L97))

## Run locally

Use a browser and a static HTTP server. For example, from the repository root with Python 3 installed:

```sh
python3 -m http.server 8000 --bind 127.0.0.1
```

Open [the ranking page](http://127.0.0.1:8000/index.html), [a province detail URL](http://127.0.0.1:8000/city.html?province=Chiang%20Mai), or [the QR page](http://127.0.0.1:8000/QRcode.html). These entry points were checked with Python's local HTTP server; this verifies file serving, not full browser or external-service behavior. Their implementations are [`index.html`](index.html), [`city.html`](city.html), and [`QRcode.html`](QRcode.html).

**The checked-in dashboard cannot load its province data.** Both pages request `data/provinces.json`, but only `data/province.json` exists. A local HTTP check returned 404 for the requested path. To try the data-driven UI, first change the fetch URL to `data/province.json` in both [`index.html`](index.html#L78) and [`city.html`](city.html#L72). This audit documents the defect without modifying application code.

For the map, replace `YOUR_GOOGLE_MAPS_API_KEY` in [`city.html`](city.html#L121) with a key configured for your deployment. A key alone does not address the duplicate initialization described below. External network access is needed for the CDN libraries, Google Fonts, and Maps URLs referenced in the [ranking head](index.html#L7), [detail head](city.html#L7), [Maps script](city.html#L121), and [QR script](QRcode.html#L6).

After correcting the data path, a representative interaction is to select Chiang Mai and adjust the sliders. Its initial slider average is 58, calculated from the stored fields; changing Flood Risk to 100 produces 64. These are reproducible arithmetic examples, **not model predictions or validated suitability scores**. ([Input record](data/province.json), [scaling](city.html#L79), [formula](city.html#L100))

## Data scope

- [`data/province.json`](data/province.json) contains three fixed records: Chiang Mai, Phuket, and Bangkok. Scores and environmental/infrastructure fields are stored literals with no provenance fields or generating code supplied. `water_resource` and `soil_quality` are not consumed by the detail page. ([Consumer](city.html#L69))
- [`data/per_image_summary.json`](data/per_image_summary.json) contains 25 records across seven category labels: Agri (4), Coastal (3), Disaster (2), LULC (3), Miscellaneous (3), RAW Data (PAN+MS) (7), and Urban (3). These counts are directly reproducible by counting records and `category` values; the labels are not classifier predictions produced by this application.
- Those records have 25 distinct `virtual_path` strings but only 13 distinct timestamp strings in their filenames, including repeated timestamps across categories and image products. This is **not** a verified count of independent scenes or physical locations. None of the referenced TIFF files is included, and the JSON provides no train/test assignment. ([All paths and metadata](data/per_image_summary.json))
- The summary records contain three one-band entries with null NDVI/EVI/NDWI fields, five entries with null CRS, and six entries with null vegetation/water areas. The files are summaries with incomplete metadata, not a supplied training dataset of raw imagery. ([`count`, `crs`, index and area fields](data/per_image_summary.json))
- Aggregate values can be arithmetically reconciled to the stored records, but the original image measurements cannot be reproduced here. No verifiable measurement source was found for the stored regression evaluation metrics; their values are therefore not reported as project results. ([Aggregate export](data/aggregated_summary.json), [input summaries](data/per_image_summary.json), [regression export](data/regression_results.json))

## Known limitations

- The missing JSON endpoint leaves the ranking's loading message in place; its catch block only logs the error. Detail initialization has no catch or HTTP-status/schema validation. ([`index.html`](index.html#L76), [`city.html`](city.html#L69))
- Detail initialization is registered both on `DOMContentLoaded` and as the asynchronous Maps callback. This can run twice or attempt map creation before the Maps API is available. ([`city.html`](city.html#L119), [map access](city.html#L89))
- All six sliders contribute positively and equally, so increasing “Flood Risk” raises the score. NDVI/NDWI/NDBI controls accept only 0–100 after scaling. There is no domain validation, weighting calibration, or evaluation in the scoring function. ([Controls](city.html#L31), [formula](city.html#L100))
- The “AI Score” and THEOS-2 analysis labels do not establish an AI pipeline: executable code displays stored scores and computes a slider mean. No trained model, training/evaluation script, image processing code, or connection from the analysis exports to province records is supplied. ([Labels](index.html#L53), [ranking](index.html#L76), [detail script](city.html#L66), [regression export](data/regression_results.json))
- The supplied application has no checked-in test suite, dependency manifest/lockfile, or CI configuration. The source entry points are the three HTML files above; audit checks used a local HTTP server and Node script execution with stubbed DOM, fetch, and library objects. Those checks do not establish a working Google Maps integration or browser rendering. ([Application sources](index.html), [`city.html`](city.html), [`QRcode.html`](QRcode.html))

## Attribution and license

The existing footer attributes the prototype to NEO CENTRAL and dates it 2025. No individual author or license file is supplied; the footer is not a license grant. ([`index.html`](index.html#L73))
