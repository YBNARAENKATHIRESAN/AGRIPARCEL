# AgriParcel AI: paddy vs banana plot map

## The task (VIT Mapathon, Problem Statement 1)
Differentiate land parcels, and identify/differentiate paddy and banana cultivation in Tirunelveli district, covering Ambasamudram and Cheranmahadevi taluks, using openly available satellite imagery. Minimum study area: 20 sq km.
- Event: VIT Mapathon, 8-9 Oct 2026. Time is short. Get the base version working before anything else.
- Do not present goals that are not in the problem statement (insurance, water sharing, loans) as part of the task.

## About the user
- Second-year student, Python fundamentals only. You write and run all the code. The user does not code by hand.
- Explain in plain words. Give a one-line explanation of any technical term the first time you use it.
- Tell the user exactly what to run, click or provide. One thing at a time.
- The user's laptop OS is unknown: detect it and give commands that fit it.
- Never invent data, labels, accuracy numbers or results. If something could not be run or checked, say so.
- If you are unsure about a coordinate, boundary, dataset name or command option, say so and verify it instead of guessing.

## How to work
- Do only the stage the user asks for. The stages and their prompts are in `docs/stage_prompts.md`.
- Run and test everything you write before saying a stage is done.
- End every stage with a short "Stage report" (10 lines max): what was done, files created, what the user should look at, what comes next.
- If the same step fails twice, stop and explain the problem simply with 2 options.
- Ask before installing large packages, deleting files, or running git push.

## Build order (base version)
0. Setup: Python environment, folders, `config.yaml`, `.gitignore`, connect GitHub
1. Earth Engine login and study area
2. Monthly greenness (NDVI) for 12 months
3. Plot outlines
4. Labels (see the Labels section)
5. Random Forest model and accuracy
6. One crop per plot, plus hectares
7. Web page
8. Final check, tag `base-working`
9. Only after the tag: LightGBM comparison
No extra features until the user says the base works and asks for them.

## Rules that keep the base easy to extend (always follow)
- One script per stage in `src/` (for example `src/s2_fetch_ndvi.py`). Each stage reads files written by earlier stages and writes its own files.
- Save every result as a file under `data/` (raw, interim, processed) or `reports/`.
- Save class probabilities for every prediction, not only the winning label.
- Keep every plot's 12 monthly NDVI values (its curve) in a saved table.
- All settings live in `config.yaml`. No hard-coded paths, years or coordinates in scripts.
- The web page only reads saved files. It never calls Earth Engine and never trains a model.
- Never use `labels/test.csv` for training or tuning. Create it once with `src/split_labels.py`. Do not regenerate it unless the user asks.
- Large data files stay out of git (see `.gitignore`). Label CSVs and code go into git.
- Keep code simple, with plain-language comments.
- Commit after each finished stage when the user says so. Tag the finished base `base-working`.

## Stage 1: Earth Engine login and study area
- Earth Engine login is interactive. Give the user the exact command to run in their own terminal, inside the project's Python environment (for example `earthengine authenticate`). Wait for them to confirm. Then use `ee.Initialize(project=<project_id from config.yaml>)`.
- Study area: at least 20 sq km and must include land from both Ambasamudram and Cheranmahadevi taluks.
- Find taluk boundaries from open sources (try OpenStreetMap, then geoBoundaries). If you cannot find reliable boundaries, say so and use the towns' locations instead, with the user's approval.
- Start with a strip of about 30-40 sq km that crosses both taluks (fast to download). It can be enlarged later.
- Make a picture of the chosen area with the boundaries and town names, show its size in sq km, and get the user's OK before saving it to `config.yaml` and `data/processed/study_area.geojson`.

## Stage 2: Satellite data (Google Earth Engine)
- Sentinel-2: `COPERNICUS/S2_SR_HARMONIZED`. Remove clouds with the SCL band (mask classes 3, 8, 9, 10, 11). NDVI = (B8 - B4) / (B8 + B4).
- One median NDVI picture per month of `year` (default 2025), named m01..m12, at 10 m.
- If a month has no clear photos, leave it as a gap first, then fill by interpolating neighbouring months. Record which months were filled.
- Earth Engine limits the size of each direct download. If a download fails for size, split it into tiles or export through Google Drive.
- Save one 12-layer raster plus a preview picture per month and a gap report.
- Sentinel-1 radar (`COPERNICUS/S1_GRD`, VV and VH) is NOT in the base. Add it only if the user asks.

## Stage 3: Plot outlines (Fields of The World, FTW)
Repo: https://github.com/fieldsoftheworld/ftw-baselines (package `ftw-tools`; its README recommends `uv`). Try A, then B, then C.
- A) Published FTW field polygons for 2024 and 2025 (CC-BY, each with a confidence score): https://source.coop/ftw/global-data . Read https://data.source.coop/ftw/global-data/predictions/vectors/llms.txt to find the file(s) covering Tamil Nadu (India is split into several files). Query with DuckDB and keep only polygons inside the study area.
- B) `ftw inference all --bbox minx,miny,maxx,maxy --year 2025 --model <name> --out <dir>`. `ftw model list` shows model names. It uses the CPU if there is no GPU. By default polygonizing drops shapes under 500 sq m and simplifies by 15 m. If small plots vanish, run `ftw inference polygonize` separately with smaller `--min_size` and `--simplify`.
- C) Last-resort fallback, only with the user's OK: segment the 12-month NDVI image into small uniform patches (for example SLIC from scikit-image) and use those as plots. Call them "image segments" everywhere, never "parcels".
- Save `data/processed/plots.gpkg` with `plot_id`, `area_ha`, `source` (A, B or C).
- Always say plainly: these are satellite-derived field units, not legal parcels, and small Tamil Nadu plots may merge.
- Compute areas in a metric CRS: UTM zone 43N (EPSG:32643).

## Stage 4: Labels (the user wants every possible help here)
Labeling is the biggest job and only a human can judge crops. Make it as easy and as correct as possible. Never fabricate labels.
- First update `tools/crop_label_tool.html` so its dashed box and jump buttons match the confirmed study area. Tell the user to open it from the project folder in Chrome or Safari, with internet. It does not work inside a chat preview.
- Label files: `labels/labels*.csv` (one per person is fine), columns `latitude,longitude,crop`, where crop is `paddy`, `banana` or `other`.
- Targets: paddy 55, banana 55, other 45 (40/40/30 for training plus 15 each for testing), spread across both taluks.
- Read `docs/labeling_guide.md` and walk the user through their first session.
- Build these helpers:
  - `src/check_labels.py`: merge all `labels/labels*.csv` into `labels/all_labels.csv`, then report in plain language: count per crop against target and how many more are needed; missing or invalid values; duplicates and spots closer than 30 m; spots outside the study area; spots not inside any plot (for information only); whether spots are spread across both taluks or clustered. End with exactly what to label next.
  - `src/label_curves.py`: draw the 12-month greenness curve of every labeled spot, grouped by crop. Flag suspicious ones (for example labeled banana but swings like paddy, or labeled paddy but stays flat and high) in `reports/label_review.csv` with pictures. Flags come from guessed thresholds, so they are hints, not proof. The user decides.
  - `src/split_labels.py`: spatial split into `labels/train.csv` and `labels/test.csv`. Group spots into blocks of about 500 m, keep whole blocks together, stratify by crop, fixed seed. Run it only when targets are met and the user agrees.
- Optional helpers. Build them when asked, or offer them if labeling is slow:
  - `app/label_plots_app.py`: Streamlit page showing plot outlines with each plot's greenness curve. The user clicks paddy, banana, other or skip. Saves `labels/labels_plots_<name>.csv` as points at plot centres.
  - `src/suggest_labels.py`: after a first model, list the spots or plots the model is least sure about, so the user labels those next.
  - `src/links_to_labels.py`: turn pasted Google Maps links plus a crop name (from a local contact) into label rows.
- When a stage needs labels and they are missing, too few or unbalanced: STOP. Say exactly which file to make, which columns, and how many rows. Do not continue with fake, random or simulated labels. If you must test code without real labels, mark the output DUMMY and never save it in `labels/`.
- Be honest: one-date imagery makes labeling imperfect. Ask the user to label only clear cases, and suggest asking a local farmer or the organizers to confirm some spots.

## Stage 5: Model
- Random Forest (scikit-learn) on pixel-level features: the 12 monthly NDVI values.
- Training rows: each labeled point's pixel plus its 8 neighbours. All rows from one point stay in the same split.
- Predict every pixel in the study area. Save per-pixel class probabilities as a raster.
- Evaluate only on `labels/test.csv`. Report overall accuracy, per-class precision and recall, and a confusion table, each with one plain-language sentence. Save to `reports/accuracy.json` and `reports/accuracy.md`.
- If accuracy is low, look at label problems first before changing the model.

## Stage 6: One crop per plot
- For each plot, class = highest mean probability over its pixels. Confidence = that mean probability.
- Save `data/processed/plot_results.gpkg`: plot_id, crop, confidence, the three mean probabilities, m01..m12 mean NDVI, area_ha, source.
- Save `reports/hectares.csv`: hectares per crop (and per taluk if boundaries exist).

## Stage 7: Web page (Streamlit + folium)
- Start with one command (`streamlit run app/app.py`). Reads only saved files.
- OpenStreetMap basemap. Plots coloured: paddy green #2E8B57, banana gold #FFD700, other grey #CCCCCC. Popup per plot: crop, confidence, hectares.
- Panels: hectares per crop, accuracy and confusion table, a "limits" box, a download button for plots as GeoJSON.
- If the map is slow, simplify polygon shapes for display only.

## Definition of done (base)
- Runs from scratch with the commands in README.
- Web page shows the coloured plot map, hectares per crop, accuracy on the test spots, and the limits box.
- Saved files exist: study area, 12-month NDVI raster, plots, train/test labels, model, per-pixel probabilities, plot results with confidence and curves, accuracy report.
- Tagged `base-working`.

## Honest limits to show in the app and report
- Plots are satellite field units, not legal parcels, and small plots may merge.
- Coconut and other tree crops can look like banana. They are labeled `other` on purpose.
- Accuracy depends on label quality and count, and is measured on a limited test set.
