# Pay Bill API Visibility — live dashboard + flow instrumentation

> **© Adeptio — public demonstration template.** All data is mock/generated in-browser or frozen day-logs; every figure is illustrative. Not affiliated with or describing any named institution.

Two linked pages that make one argument: a Pay Bill journey at a Myanmar
commercial bank can be watched end to end, and when it breaks the checks
themselves name the layer and the team.

| | Page | What it is |
|---|---|---|
| 1 | [`index.html`](index.html) | **Live status dashboard.** 16 nodes, 17 links, a frozen 7-day mock week (2016 steps × 5 min) with nine incident windows, three live synthesis tables, and a scrubable timeline. |
| 2 | [`flow-instrumentation.html`](flow-instrumentation.html) | **The use case, fully instrumented.** Pay Bill top to bottom — the software each step crosses, the hardware it rides, the 52 health checks that watch both, and five fault fingerprints that turn a check pattern into a routed ticket. |

Vanilla HTML + CSS + SVG + JS. No framework, no build step, no dependencies, no
network calls at runtime.

**All data is MOCK and every threshold is ILLUSTRATIVE.** The week is a
hand-authored scenario written to demonstrate the instrument, not a measurement
of any real bank. `{BANK}` is a placeholder. Treat cadences and thresholds as
calibration seeds for a real engagement, not as recommendations.

## Quickstart

**Just open `index.html`** — double-click it. Everything loads through plain
`<link>` / `<script src>` tags, so it runs straight off disk over `file://`.
There is deliberately no `fetch()` / `XHR` anywhere: browsers block those for
local files, and avoiding them is what makes the folder portable.

If you would rather serve it:

```sh
python3 -m http.server 8000     # then open http://localhost:8000/
```

Only needed if you want a real origin (e.g. to test with dev-tools throttling).
Nothing in the site requires it.

## GitHub Pages

Works as-is, no workflow and no Jekyll config:

**Settings → Pages → Build and deployment → Deploy from a branch → `main` / `/root` → Save.**

Every link, asset and data path in both pages is relative, and a `.nojekyll`
file is included so Pages serves the folder verbatim (without it, Jekyll would
ignore nothing here today, but it also would not have to — `.nojekyll` removes
the whole class of surprise).

## Page map and the deep-link trick

The dashboard reads `#t=<step>` from the URL on boot: it clamps the value to
`[0, 2015]`, seeks the timeline there, leaves playback **paused**, and refreshes
the tables. That is how page 2 links into page 1 — the fault-fingerprint matrix
in §03 carries a *view live →* link per scenario that opens the dashboard on the
exact moment that fingerprint was drawn from.

```
index.html#t=684      D3 09:00  A · carrier multi-path degradation
index.html#t=1614     D6 14:30  B · LB pool loss (2 of 4 members)
index.html#t=66       D1 05:30  C · OTP / SMS delivery dip
index.html#t=428      D2 11:40  D · replica lag -> silent false declines
index.html#t=1710     D6 22:30  E · EOD batch overrun
index.html#t=1394     D5 20:10  F · core outage after failed failover
```

Step index is minutes-since-start ÷ 5, i.e. `step = (day - 1) * 288 + (hh * 60 + mm) / 5`.
An out-of-range or malformed `#t=` is ignored and the dashboard opens live at
the end of the week, as it does without a hash.

## Repo layout

| Path | Role |
|---|---|
| `index.html` | Page 1 — markup and boot order only. No logic, no data. |
| `flow-instrumentation.html` | Page 2 — content page. Section-scoped `<style>` blocks stay inline on purpose (each is component-scoped to one chapter). |
| `assets/styles.css` | All page-1 styling, both themes. |
| `assets/engine.js` | Page-1 engine: hydration, rendering, interaction, tables, timeline. |
| `assets/flow.css` | Page-2 global stylesheet (extracted from that page's `<head>`). |
| `data/manifest.js` | Topology + objective definitions. No time series. |
| `data/log_day1..7.js` | The frozen week, one file per mock day. |
| `docs/SPEC-Dashboard-v1.2.md` | Engine spec — layout, interactions, verification checklist (§9). |
| `docs/SPEC-Dashboard-v1.3-Addendum.md` | v1.3 Pay Bill delta on top of that spec. |
| `build_singlefile.py` | Re-inlines page 1 into one portable HTML file. |
| `.nojekyll` | Serve verbatim on GitHub Pages. |

`index.html` loads its scripts in exactly this order, and the order matters:

1. `data/manifest.js` — defines `window.ADEPTIO_DATA`
2. `data/log_day1.js` … `log_day7.js` — each appends one day to `window.ADEPTIO_LOGS`
3. `assets/engine.js` — reads both, hydrates, renders

## Data contract

Each day file is one flat object of 288-step slices:

```js
window.ADEPTIO_LOGS = window.ADEPTIO_LOGS || {};
window.ADEPTIO_LOGS.d4 = {
  "dbr.1": { "vals": [1.21, 1.18, /* …288 numbers */], "stat": ["ok","ok", /* …288 strings */] },
  "KPI":   { "vals": [/* … */], "stat": [/* … */] }
};
```

* Key is `"<nodeId>.<objIndex>"` — the index into that node's `objs[]` in
  `data/manifest.js` — plus one special `"KPI"` key for the headline number.
* `vals` are rounded to 2 dp. `stat` is the pre-evaluated `ok` / `warn` / `crit`
  for that step, so the renderer never re-derives thresholds.
* The engine concatenates `d1…d7` in order into the 2016-step `vals[]` / `stat[]`
  arrays it consumes. Day length is read from the data, not assumed.

Because the week is frozen, every load tells the identical story — which is what
you want for a demo or a screenshot set.

If `window.ADEPTIO_LOGS` is missing or incomplete, the engine falls back to
`gen()` — the synthetic generator still in `assets/engine.js` — and synthesises a
fresh random week from the amplitudes in `manifest.js`. The scenario banner is
tagged **live-generated** so you can tell at a glance, and `window.ADEPTIO_MODE`
reports `frozen-logs` or `live-generated`.

### Wiring a live feed

Publish the same object shape and nothing else changes. Emit a script that sets
`window.ADEPTIO_LOGS.dN` per day (or per any slice length — the engine
concatenates whatever it is given and only checks the total equals `N`), keyed by
the same `"<nodeId>.<objIndex>"` strings, and swap the `<script src>` tags in
`index.html` for your endpoint. To stream instead, replace `collectLogs()` /
`stitch()` in `assets/engine.js` — they are the only two functions that know
where series come from. Keep `stat` pre-computed server-side, or derive it from
the thresholds already in `manifest.js`.

### Retargeting to another bank or another journey

**Edit `data/manifest.js` and regenerate the logs. Never edit `assets/engine.js`.**

The manifest owns everything bank-specific: node ids, display names, coordinates,
link weights, and the objective definitions (label, unit, baseline, warn/crit
thresholds, direction, amplitude, noise, incident bindings). The engine owns
rendering and interaction and knows nothing about Pay Bill. A retarget that
touches `engine.js` is a retarget that has gone wrong — the divergence will cost
you at the next upgrade.

After editing the manifest, regenerate `data/log_day1..7.js` so the frozen series
still line up with the new `objs[]` indices; a stale log keyed to an old index
silently paints the wrong objective.

## Rebuilding the single-file variant

This folder is the source of truth; the one-file dashboard is a build product.

```sh
python3 build_singlefile.py                 # -> ../adeptio_paybill_live_dashboard.html
python3 build_singlefile.py somewhere.html  # or an explicit path
```

It inlines every `<link rel=stylesheet>` and `<script src>` verbatim, in the same
order, with the frozen data embedded, and drops anything wrapped in
`<!--SF-STRIP-START--> … <!--SF-STRIP-END-->` — currently just the page-2 link
chip, which would dangle in a file shipped on its own. Nothing is minified or
reordered, so the single file behaves identically: same statuses, same values,
same timeline, same `#t=` deep links.

Edit the site sources, never the built file, then re-run the build.
