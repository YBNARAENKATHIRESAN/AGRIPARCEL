# AgriParcel AI: paddy vs banana plot map

VIT Mapathon 2026, Problem Statement 1: differentiate land parcels and identify paddy and banana
cultivation in Ambasamudram and Cheranmahadevi taluks (Tirunelveli district) using open satellite
imagery, over at least 20 sq km.

**Status:** Stage 0 (setup) done. Scripts for later stages do not exist yet; they are listed below
with the names they will get. This README is updated as each stage is finished.

## One-time setup (Windows, PowerShell, inside the `AgriParcel` folder)

Needs Python 3.12 and [uv](https://docs.astral.sh/uv/).

```powershell
uv venv .venv --python 3.12
uv pip install --python .venv\Scripts\python.exe -r requirements.txt
```

Every command below uses `.venv\Scripts\python.exe`, so you do not need to "activate" anything.
(On Mac/Linux, use `.venv/bin/python` instead.)

## Settings

All settings are in `config.yaml` (Earth Engine project ID, year, study area, file paths, label targets).
Scripts read from it. Nothing is hard-coded in scripts.

## Stages and how to run them

| Stage | What it does | Command | Main output |
|---|---|---|---|
| 1 | Earth Engine login (once) | `.venv\Scripts\earthengine.exe authenticate` | login saved on your laptop |
| 1 | Choose study area | `.venv\Scripts\python.exe src\s1_study_area.py` | `data/processed/study_area.geojson` |
| 2 | 12 months of NDVI greenness | `.venv\Scripts\python.exe src\s2_fetch_ndvi.py` | `data/processed/ndvi_monthly.tif`, previews, gap report |
| 3 | Plot outlines | `.venv\Scripts\python.exe src\s3_plots.py` | `data/processed/plots.gpkg` |
| 4 | Labels (human work) | open `tools\crop_label_tool.html` in Chrome, save CSVs in `labels\` | `labels/labels_<name>.csv` |
| 4 | Check labels | `.venv\Scripts\python.exe src\check_labels.py` | `labels/all_labels.csv` + report |
| 4 | Review label curves | `.venv\Scripts\python.exe src\label_curves.py` | `reports/label_review.csv` |
| 4 | Train/test split (once) | `.venv\Scripts\python.exe src\split_labels.py` | `labels/train.csv`, `labels/test.csv` |
| 5 | Random Forest + accuracy | `.venv\Scripts\python.exe src\s5_train_model.py` | model, `class_probabilities.tif`, `reports/accuracy.md` |
| 6 | One crop per plot + hectares | `.venv\Scripts\python.exe src\s6_plot_crops.py` | `data/processed/plot_results.gpkg`, `reports/hectares.csv` |
| 7 | Web page | `.venv\Scripts\streamlit.exe run app\app.py` | opens in your browser |

Script names for Stages 1-7 are planned names; they may be adjusted when each stage is built.

## Folders

- `src/` one script per stage
- `data/raw`, `data/interim` downloads and in-between files (not in git, can be re-made)
- `data/processed/` final data files
- `labels/` label CSVs (in git). `labels/test.csv` is never used for training.
- `reports/` accuracy, hectares, label review
- `app/` the Streamlit web page (it only reads saved files)
- `tools/` the browser labeling tool
- `docs/` problem statement, stage prompts, labeling guide

## Honest limits

- Plots are satellite-derived field units, not legal parcels. Small plots may merge.
- Coconut and other tree crops can look like banana; they are labeled `other` on purpose.
- Accuracy depends on label quality and count, and is measured on a limited test set.
