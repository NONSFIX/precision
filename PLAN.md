# Precision — PLAN

Fill text frames in Illustrator artwork from JSON (text, size, color, font), batch-export print PDFs.
Phase 1: web demo (browser preview). Phase 2: Illustrator UXP plugin (real CMYK PDF).

The **job JSON** is the contract between both phases.

## Locked decisions (grilling 2026-11-23)

| # | Decision |
|---|---|
| 1 | Illustrator renders the final PDF. No true headless; web is preview only. |
| 2 | Frames matched by **Layers-panel name** (`textFrames.getByName`). Templates use **area type**. |
| 3 | Shrink-to-fit: reduce font size, then horizontal scale, never below floors. Still overflows → flag. |
| 4 | Floors: global `defaults` (`minSize: 6.1`, `minScale: 90`), per-field override. JSON `size` = max size. |
| 5 | Font = exact **PostScript name**. Not installed → flag. No `font` → keep template font. |
| 6 | **CMYK only.** RGB template or RGB/hex color in JSON → flag. |
| 7 | Export with named PDF preset; default = our existing preset (exact name TBD). |
| 8 | Order: web demo first, then local Illustrator plugin. |
| 9 | Demo template = **SVG export** (Object IDs: Layer Names). Demo PDF is RGB preview only. |
| 10 | Demo = static HTML/JS, GitHub Pages, no backend. Files stay in browser. |
| 11 | Demo input = form generated from template frames + Import/Export JSON. |
| 12 | "Repeatedly" = **batch**: one template, many rows → many outputs. |
| 13 | Batch format = JSON job file (below). No CSV. |
| 14 | Flagged row → skip, continue, report at end (web: red rows; plugin: `report.json`). |
| 15 | **Every text frame needs data**, else flag. Fixed text in templates must be outlined. |
| 16 | Plugin = **UXP panel** wrapping the fill logic. |
| 17 | Plugin runs on **another PC**; user tests. Need its Illustrator version first (UXP needs 2025+). |
| 18 | Script forces **World-Ready composer + Thai language** on every filled frame. |
| 19 | Demo fonts = **Google Fonts only**, with a family/weight → PostScript name map. Plugin PC must have same fonts installed. |
| 20 | Each row has its own `filename`. Missing → flag. |
| 21 | **Mixed styles = runs.** A field is `text` or `runs: [{text, size, color, font}]`. Shrink-to-fit scales all runs together; no run may go below the floor. |
| 22 | **Template per row.** Each label is its own `.ai` with the same frame names; row sets `template`. Image/color swaps deferred until a real need. |
| 23 | **Barcode stays artwork** in each template (digits outlined). Tool ignores it. |
| 24 | **Shared values via `vars`.** Row `vars: {abv: 5.5}`; text uses `{abv}` (as-is), `{abv,}` (comma decimal), `{abv.}` (dot decimal). Unknown var → flag. Works inside runs. |

Reference job: a beer-label series (≈10 frames per label: description, type line, ingredients with bold words, brewer info, URL, IBU/SRM/PLATO, volume+ABV, 2-line title, style+ABV line). Build an **original** sample in this shape — no real brand.

## Job JSON

```json
{
  "template": "card.svg",
  "pdfPreset": "<our preset name>",
  "defaults": { "minSize": 6.1, "minScale": 90 },
  "rows": [
    {
      "template": "label-01.svg",
      "filename": "label-01_th",
      "vars": { "abv": 5.5 },
      "fields": {
        "title": { "text": "ดำ\nมืด", "size": 24, "font": "Sarabun-Bold", "color": { "cmyk": [0, 0, 0, 100] }, "minSize": 10 },
        "volume": { "text": "330ml {abv,}% vol." },
        "style_abv": { "runs": [
          { "text": "BROWN PORTER ", "color": { "cmyk": [0, 90, 80, 0] } },
          { "text": "{abv.}", "size": 14 },
          { "text": "% ABV", "size": 9 }
        ] }
      }
    }
  ]
}
```

`template` may sit at the top (default) or per row (override).

Flag reasons: `overflow`, `font-missing`, `rgb-color`, `rgb-template`, `unknown-frame`, `frame-no-data`, `filename-missing`, `template-missing`, `unknown-var`.

## Slices

### Phase 1 — web demo
- [x] **S1** Load SVG → list text frames by id, render preview. *(verified on mock sample; box = text bbox, not area-type box — re-check with a real Illustrator export)*
- [x] **S2** Original beer-label sample SVG (≈10 frames). Generated form per frame (text, size, font, CMYK color, floors) → live preview.
- [x] **S2b** JSON editor panel beside the form, two-way sync with form + preview; parse errors shown inline.
- [x] **S2c** Runs (mixed styles in one frame) + `vars` substitution (`{abv}`, `{abv,}`, `{abv.}`).
- [x] **S3** Shrink-to-fit in browser (size → scale → flag) + Thai wrap in box. *(box = `<frame>_box` rect on a hidden layer, since SVG export drops area-type bounds; fallback = template text footprint. Thai breaks via `Intl.Segmenter`.)*
- [x] **S4** Google Fonts picker + PostScript name map. *(10 Thai families, 75 PostScript names like `Sarabun-Bold`; names follow Google's static-file pattern — confirm on the plugin PC.)*
- [x] **S5** Import/Export job JSON; validation flags (all reasons above). *(`rgb-template` is plugin-only: SVG has no CMYK mode. `font-missing` in the demo = not in the Google Thai font map.)*
- [x] **S6** Batch: rows list, per-row template, per-row preview, red flagged rows, report. *(Open SVG takes several files; every row is fitted + validated, not just the shown one; Export report = same shape as the plugin's `report.json`.)*
- [ ] **S7** Export preview PDF (RGB) per row. Deploy to GitHub Pages.
  - [x] Preview PDF: browser print → "Save as PDF", one page per passing row, page size = artboard. Template CSS scoped per SVG (Illustrator reuses `.st0…` in every file).
  - [ ] Deploy to GitHub Pages (waiting for go-ahead: new public repo).

### Phase 2 — Illustrator plugin (needs Illustrator version from test PC)
- [ ] **P1** Fill logic: open `.ai`, fill frames by name, World-Ready + Thai, CMYK, font check.
- [ ] **P2** Shrink-to-fit with floors, overflow detection.
- [ ] **P3** Batch export PDFs with preset, `report.json`.
- [ ] **P4** UXP panel UI: load job, row list with flags, Run. Package as `.ccx`.

## Open
- Exact PDF preset name (Edit › Adobe PDF Presets).
- Illustrator version on the test PC.
- Demo: SVG export of area type may come as positioned lines, not a box — S1 must verify how box bounds survive export.
