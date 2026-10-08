# START HERE

You have: a laptop, a Google account, and Claude Code. That is enough. You do not need any data. Claude Code downloads the satellite data, and you make the labels later with a tool.

## What we are building (base version)
A web page with a map of farm plots in Ambasamudram and Cheranmahadevi taluks. Each plot is coloured paddy, banana or other, with hectare totals and an accuracy score.

## The order of everything

| Stage | Who | What happens |
|---|---|---|
| Before | You | Make an Earth Engine project, unzip this folder |
| 0 | Claude Code | Sets up Python, folders, GitHub |
| 1 | Claude Code + you | Earth Engine login, choose the study area |
| 2 | Claude Code | Downloads 12 months of satellite greenness |
| 3 | Claude Code | Gets the farm plot outlines |
| 4 | You (with help) | Label fields: paddy, banana, other |
| 5 | Claude Code | Trains the model and checks accuracy |
| 6 | Claude Code | Gives every plot a crop and counts hectares |
| 7 | Claude Code | Builds the web page |
| 8 | Claude Code + you | Final check, tag `base-working` |
| 9 | Claude Code | LightGBM comparison (only after the base works) |

Labeling (Stage 4) comes after the study area is fixed in Stage 1. You may start labeling while Stages 2 and 3 are running.

## Before Claude Code (about 15 minutes)
1. **Earth Engine project.** Open https://developers.google.com/earth-engine/guides/access and follow its link to the registration page. Choose **noncommercial** use (student project). Create a new project. Copy the **Project ID** (looks like `my-project-12345`). Keep it in a note.
2. **Folder.** Make a folder on your laptop named `AgriParcel`. Unzip this zip into it.
3. **Problem statement.** Copy the problem statement PDF into the `docs` folder.
4. **GitHub link.** Keep your GitHub repo link ready. Stage 0 asks for it.

## Starting Claude Code
1. Open a terminal **inside** the `AgriParcel` folder, then type `claude` and press Enter. If you use the Claude Code desktop app, open the `AgriParcel` folder instead.
2. Type `/context` and press Enter. Check that **CLAUDE.md** appears under **Memory files**. That means Claude Code has read the project rules.

## Running the stages
1. Open `docs/stage_prompts.md`.
2. Copy the Stage 0 prompt, paste it into Claude Code, press Enter.
3. If Claude Code asks permission to run something, read it. Approve it if it fits the stage.
4. When it finishes, do the "You check" item for that stage.
5. Type: `Commit this stage.`
6. Go to the next stage. One stage at a time.

## When something goes wrong
- Paste: `That failed. Explain simply what went wrong and fix it.`
- If it fails twice, paste: `Stop. Explain the problem in simple words and give me 2 options.`
- If Claude Code forgets the rules, paste: `Re-read CLAUDE.md and follow it.`

## Rough time guide (guesses, not promises)
Before: 15 min · Stage 0: 30-60 min · 1: 30 min · 2: 30-90 min · 3: 30-90 min · 4 (labeling): 2-3 hours of human time, faster with teammates · 5: 30 min · 6: 30 min · 7: 1 hour · 8: 30-60 min.

## The files in this folder
- `CLAUDE.md`: the rules Claude Code reads every session. Do not delete it.
- `config.yaml`: settings. Claude Code fills it in with you.
- `docs/stage_prompts.md`: what to paste, stage by stage.
- `docs/labeling_guide.md`: how to label paddy, banana, other.
- `tools/crop_label_tool.html`: the labeling tool. Open it from your Downloads or this folder in Chrome or Safari, with internet. It does not work inside a chat app preview.
- `labels/`: put your label CSV files here.
