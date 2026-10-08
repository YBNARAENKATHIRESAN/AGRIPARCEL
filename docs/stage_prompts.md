# Stage prompts for Claude Code

Paste one prompt at a time. Wait for the Stage report. Do the "You check" item. Then type `Commit this stage.`

---

## Stage 0: Setup
```
Read CLAUDE.md, START_HERE.md and everything in docs/. This is Stage 0 only.
Set up the project on my laptop: detect my operating system, check Python is installed (help me install it if not), create a project Python environment, install the packages we need (earthengine-api, geopandas, rasterio, rasterstats, shapely, pandas, numpy, scikit-learn, lightgbm, scikit-image, matplotlib, streamlit, folium, streamlit-folium, duckdb, pyyaml), create the folders (src, data/raw, data/interim, data/processed, reports, app), and a README with how to run each stage.
Then ask me for my Earth Engine project ID and put it in config.yaml.
Then ask me for my GitHub repo link and connect this folder to it (ask before pushing).
Finish with a Stage report.
```
**You check:** the Stage report says Python and the packages work, and config.yaml has your project ID.

---

## Stage 1: Earth Engine login and study area
```
Stage 1. Give me the exact command to log in to Earth Engine in my own terminal, and tell me what will pop up and what to click. After I confirm, test the connection.
Then find the Ambasamudram and Cheranmahadevi taluk boundaries as CLAUDE.md says, and propose a study area of about 30-40 sq km that includes land from both taluks. Show me a picture with the boundaries and town names and the size in sq km. Wait for my OK before saving it.
Finish with a Stage report.
```
**You check:** the picture shows both taluks and the area is at least 20 sq km. Reply OK, or ask for a change.

---

## Stage 2: Satellite data
```
Stage 2. Download 12 months of Sentinel-2 NDVI for the study area exactly as CLAUDE.md says. Save the 12-layer raster, a preview picture for each month, and a gap report (clear photos per month, which months were filled). Explain in simple words what I should look at in the pictures.
Finish with a Stage report.
```
**You check:** 12 pictures exist and the area looks like farmland.
**Tip:** while this runs, you can start labeling (see Stage 4 "You do").

---

## Stage 3: Plot outlines
```
Stage 3. Get the plot outlines as CLAUDE.md says: try A, then B, and use C only if I agree. Save data/processed/plots.gpkg. Make a preview map. Tell me how many plots, the median plot size, which route worked, and any merged or missing plots you can see.
Finish with a Stage report.
```
**You check:** zoom into the preview map and say if the outlines look badly merged.

---

## Stage 4: Labels (your main job)
```
Stage 4. First update tools/crop_label_tool.html to match our study area. Then build src/check_labels.py, src/label_curves.py and src/split_labels.py as CLAUDE.md says, and explain in simple steps how I run each one. Then walk me through my first labeling session using docs/labeling_guide.md. Do not invent any labels.
Finish with a Stage report.
```
**You do**
1. Open `tools/crop_label_tool.html` in Chrome or Safari (from the folder, with internet).
2. Label fields. Download the CSV, rename it (for example `labels_rohith.csv`), and put it in `labels/`. Teammates can do the same with their own names.
3. Type: `Run check_labels and tell me what to label next.` Repeat until targets are met.
4. Type: `Run label_curves and show me the suspicious spots.` Fix or delete wrong labels.
5. When targets are met, type: `Run split_labels.`

**If labeling is slow**
```
Build the optional labeling helpers from CLAUDE.md: app/label_plots_app.py and src/links_to_labels.py. Explain how to run them.
```

---

## Stage 5: Model
```
Stage 5. Train the Random Forest as CLAUDE.md says, using labels/train.csv only. Evaluate on labels/test.csv only. Save the model, the per-pixel probabilities and the accuracy report. Explain each number in one plain sentence. If accuracy is low, tell me the most likely reasons (label problems first) before changing anything.
Finish with a Stage report.
```
**You check:** the accuracy report exists and you understand what the score means.

---

## Stage 6: One crop per plot
```
Stage 6. Give every plot a crop as CLAUDE.md says. Save data/processed/plot_results.gpkg and reports/hectares.csv. Make a preview map coloured by crop.
Finish with a Stage report.
```
**You check:** the coloured map looks sensible in places you labeled.

---

## Stage 7: Web page
```
Stage 7. Build the Streamlit page as CLAUDE.md says. It must only read saved files. Tell me the one command to start it, and start it for me.
Finish with a Stage report.
```
**You check:** the page opens in your browser and shows the map, hectares, accuracy and limits.

---

## Stage 8: Finish the base
```
Stage 8. Run everything from scratch following the README and fix anything that breaks. Show me the "Definition of done" checklist from CLAUDE.md with yes or no for each item. Then ask me before tagging the commit base-working and pushing to GitHub.
```
**You check:** every item says yes.

---

## Stage 9: LightGBM comparison (only after the tag)
```
Stage 9. Train LightGBM on labels/train.csv and score it on labels/test.csv, the same files the Random Forest used. Show both scores side by side. Keep the better one as the model the page uses, and tell me which one that is.
```
