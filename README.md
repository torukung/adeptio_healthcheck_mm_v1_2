# ADEPTIO Pulse — Flow Inspection : Mobile Payment Module

Live status dashboard + flow instrumentation + incident trace portal.

> **© Adeptio Pulse — public demonstration template.** All data is mock — regenerated in-browser from a fixed seed; every figure is illustrative. Not affiliated with or describing any named institution.

Three linked pages that make one argument: a mobile bill-payment journey at a Myanmar
commercial bank can be watched end to end, when it breaks the checks themselves
name the layer and the team, and the case that follows is already on a board.

| | Page | What it is |
|---|---|---|
| 1 | [`index.html`](index.html) | **Live status dashboard.** 16 nodes, 17 links, a deterministic 7-day mock week (2016 steps × 5 min) with nine incident windows, three live synthesis tables, and a scrubable timeline. |
| 2 | [`flow-instrumentation.html`](flow-instrumentation.html) | **The use case, fully instrumented.** The payment flow top to bottom — the software each step crosses, the hardware it rides, the 52 health checks that watch both, and five fault fingerprints that turn a check pattern into a routed ticket. |
| 3 | [`incident-trace.html`](incident-trace.html) | **Incident Trace portal.** Where the routed ticket lands: a small board / list / detail ticketing surface over the same nine incident windows — 20 seeded cases, drag-to-transition, comments, linked issues, create. Editable; edits persist to `localStorage` in your browser only. |

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

Every link and asset path in both pages is relative, and a `.nojekyll` file is
included so Pages serves the folder verbatim (without it, Jekyll would ignore
nothing here today, but it also would not have to — `.nojekyll` removes the
whole class of surprise).

The hosted copy carries **no day-log data files** — 1.3 MB of series that the
browser can rebuild for free. The engine regenerates the identical week from a
fixed seed instead (`ADEPTIO_SEED = 20260815`), so Pages, a local `file://` open
and the single-file build all tell the same story, step for step. The banner
tags it `· seeded replay`. If you want the materialised day logs — because you
are wiring the same shape to a real feed — take them from the release zip and
follow [`data/README.md`](data/README.md).

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

The portal has the mirror-image trick: `incident-trace.html#INC-1030` opens that
ticket's detail overlay directly. The three pages therefore link in a loop —
dashboard incident panel → *Incident Trace ▸* → the case that covers the window
you are looking at → *view on dashboard →* → the dashboard back at that moment.

## Repo layout

| Path | Role |
|---|---|
| `index.html` | Page 1 — markup and boot order only. No logic, no data. |
| `flow-instrumentation.html` | Page 2 — content page. Section-scoped `<style>` blocks stay inline on purpose (each is component-scoped to one chapter). |
| `assets/styles.css` | All page-1 styling, both themes. |
| `assets/engine.js` | Page-1 engine: hydration, rendering, interaction, tables, timeline. |
| `assets/flow.css` | Page-2 global stylesheet (extracted from that page's `<head>`). |
| `incident-trace.html` | Page 3 — markup and boot order only, same pattern as page 1. |
| `assets/portal.css` | Page-3 styling. Reuses the page-1 design tokens; everything is prefixed `.p-`. |
| `assets/portal.js` | Page-3 engine: board, list, detail overlay, create, `localStorage` persistence. |
| `data/tickets.js` | Page-3 seed: 20 tickets keyed `INC-10xx`, plus `byWindow` mapping each incident window to the case that carries it. Read by page 1 too, for the incident panel's trace card. |
| `data/manifest.js` | Topology + objective definitions. No time series. |
| `data/rcameta.js` | RCA metadata behind the incident panel: per-object description / customer page / neighbour systems / owner, plus first checks + questions for all 61 objectives. |
| `data/README.md` | The two data modes, and the live-feed contract. |
| `data/log_day1..7.js` | *Optional.* Materialised week, one file per mock day — not committed; in the release zip. |
| `docs/SPEC-Dashboard-v1.2.md` | Engine spec — layout, interactions, verification checklist (§9). |
| `docs/SPEC-Dashboard-v1.2.1-Addendum.md` | v1.2.1 Pay Bill delta on top of that spec. |
| `build_singlefile.py` | Re-inlines page 1 into one portable HTML file. |
| `.nojekyll` | Serve verbatim on GitHub Pages. |

`index.html` loads its scripts in exactly this order, and the order matters:

1. `data/manifest.js` — defines `window.ADEPTIO_DATA`
1b. `data/rcameta.js` — defines `window.ADEPTIO_RCA` (optional: the engine runs
   without it, the incident panel simply shows blank copy)
1c. `data/tickets.js` — defines `window.ADEPTIO_TICKETS` (optional: without it the
   incident panel's Incident Trace row reports no linked case)
2. `data/log_day1.js` … `log_day7.js` — *optional*, each appends one day to
   `window.ADEPTIO_LOGS`. Absent in this repo; the tags sit commented between the
   `ADEPTIO-LOGS-START` / `-END` markers, ready to switch on.
3. `assets/engine.js` — reads what is there, hydrates, renders

## Data contract

Series reach the renderer one of two ways, and both yield identical numbers —
see [`data/README.md`](data/README.md) for the short version.

**Seeded replay (default here).** No log files present: the engine rebuilds the
week from `ADEPTIO_SEED` with a mulberry32 PRNG re-seeded per series key, so the
output does not depend on generation order and never drifts between loads.
`window.ADEPTIO_MODE === 'seeded replay'`.

**Materialised logs.** Each day file is one flat object of 288-step slices:

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

Every load tells the identical story either way — which is what you want for a
demo or a screenshot set. The shipped day files were dumped from the seeded
generator, so adding or removing them changes nothing on screen; that equality is
verified step-for-step. When logs are present `window.ADEPTIO_MODE` reports
`frozen-logs` and the banner stays unmarked; when they are absent it reports
`seeded replay` and the banner says so.

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
rendering and interaction and knows nothing about the payment flow. A retarget that
touches `engine.js` is a retarget that has gone wrong — the divergence will cost
you at the next upgrade.

Seeded replay follows the manifest automatically. If you keep materialised day
logs, regenerate them after editing the manifest so the series still line up with
the new `objs[]` indices — a stale log keyed to an old index silently paints the
wrong objective.

## Rebuilding the single-file variant

This folder is the source of truth; the one-file dashboard is a build product.

```sh
python3 build_singlefile.py                 # -> ../adeptio_paybill_live_dashboard.html
python3 build_singlefile.py somewhere.html  # or an explicit path
```

It inlines every `<link rel=stylesheet>` and `<script src>` verbatim, in the same
order, and leaves out two things: anything wrapped in `<!--SF-STRIP-START--> …
<!--SF-STRIP-END-->` (currently just the page-2 link chip, which would dangle in
a file shipped on its own), and the day logs — seeded replay rebuilds them, so
the build stays ~72 KB instead of ~1.4 MB. Nothing is minified or reordered, so
the single file behaves identically: same statuses, same values, same timeline,
same `#t=` deep links.

Edit the site sources, never the built file, then re-run the build.
