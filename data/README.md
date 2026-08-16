# `data/` — two ways to get the same week

`manifest.js` is always required: topology, objectives, incident windows. The
**series** behind them arrive one of two ways, both byte-identical — verified
step for step across the whole week.

## 1 · Seeded replay (what this repo ships)

No day-log files, and no `<script>` tags for them in `index.html`. On boot the
engine rebuilds all 2016 steps from `ADEPTIO_SEED = 20260815` (mulberry32,
re-seeded per series key, so output never depends on generation order). Every
load — and every visitor of the GitHub Pages copy — sees the identical canonical
week. Banner tag: `· seeded replay`; `window.ADEPTIO_MODE === 'seeded replay'`.

## 2 · Materialised day logs (`log_day1..7.js`)

Seven files, one per mock day, ~183 KB each, each setting

```js
window.ADEPTIO_LOGS.dK = { "<nodeId>.<objIndex>": {vals:[288], stat:[288]}, …, "KPI": {…} }
```

which the engine stitches into the 2016-step arrays (`ADEPTIO_MODE ===
'frozen-logs'`). **This is also the live-feed contract**: publish that shape from
a real collector and nothing else in the engine changes.

Not committed (the hosted copy runs seeded) but shipped in the release zip. To
enable: drop them in here and delete the two `<!--` / `-->` delimiters between the
`ADEPTIO-LOGS-START` / `-END` markers in `index.html`. They were dumped from the
seeded generator, so the switch changes nothing on screen.

`build_singlefile.py` never embeds them; the standalone build regenerates the week
from the seed instead (~140 KB, most of which is `rcameta.js`).

## 3 · `rcameta.js` — RCA copy, not series

Loaded right after `manifest.js`. `window.ADEPTIO_RCA` has `nodes[<id>]`
(`desc` / `page` / `systems` / `owner`) for all 16 objects and
`indicators["<nodeId>.<objective label>"]` (`checks` / `questions`) for all 61
objectives — the content of the incident panel, and the seed text for each pane's
editable "About this object". Keys mirror `manifest.js` exactly; if you retarget
the topology, re-key this file too. It is optional: without it the engine still
runs and the panel simply renders blank copy.
