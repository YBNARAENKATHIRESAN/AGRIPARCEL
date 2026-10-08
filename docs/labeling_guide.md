# Labeling guide

## What labeling is
You make an answer key. For each spot, you write down what is growing there: paddy, banana or other. The computer studies some of your answers, and you keep the rest hidden to grade it.

## The three answers and their clues
- **Paddy:** a flat rectangle with thin ridges around it. It may be flooded, bright green, or tan after harvest.
- **Banana:** dark green, rough-looking, planted in rows, green all year.
- **Other:** coconut groves, houses, river, bare land, other trees. Label a few coconut groves on purpose, so the model learns they are not banana.

## How to label with the tool
1. Open `tools/crop_label_tool.html` from the project folder in Chrome or Safari, with internet on. (Claude Code updates its study-area box in Stage 4.)
2. If a red warning says the satellite picture is not loading, open the file directly in Chrome or Safari, not inside a chat app.
3. Tap **Ambasamudram** or **Cheranmahadevi** to jump there. Turn on "Place names" to check where you are.
4. Pick Paddy, Banana or Other at the top. Tap the middle of a field. A coloured dot appears.
5. Tap a dot to delete it. Undo removes the last one.
6. Press **Download labels.csv** when you finish. Rename it with your name, like `labels_asha.csv`, and put it in the `labels/` folder.

## How many and where
- Targets: paddy 55, banana 55, other 45.
- Spread spots across both taluks. Do not cluster them in one corner.
- Include different looks: young and old banana, flooded and green paddy.

## Do and don't
- Do tap the middle of a field, away from its edges.
- Do skip a spot when you are not sure. A wrong label hurts more than a missing one.
- Don't label from one look at a harvested field if you cannot tell what it was.
- Remember the picture is from one date, and fields change through the year.
- The satellite picture is only for looking. The final map is built from Sentinel data.

## Getting confirmed labels from a local person
A farmer or someone who lives there beats any guess from a picture. Message template:

> Hi, I'm a college student mapping paddy and banana farms near Ambasamudram and Cheranmahadevi for a project. Could you send me the location (a map pin) of 5 to 10 fields where you are sure it is paddy, and 5 to 10 where it is banana? Please tell me which season or year you mean. Thank you!

Paste the map links and crops into a text file. Claude Code can turn them into label rows (`src/links_to_labels.py`).

## Checking your labels
- Run `src/check_labels.py`. It tells you how many more you need and what looks wrong.
- Run `src/label_curves.py`. It draws each spot's greenness curve. A spot labeled banana whose curve swings like paddy deserves a second look.
- Flags are hints. You decide.

## Common mistakes
- All spots in one area, so the test score looks better than it is.
- Labeling coconut as banana.
- Labeling a fallow or harvested paddy field as "other" because it looks bare.
- Labeling a spot on a field boundary.

## Working in a team
Everyone labels a different part of the area, downloads their own CSV, and puts it in `labels/`. The check script merges all `labels/labels*.csv` files.
