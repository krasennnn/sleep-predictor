# Sleep Predictor

Predicting how long someone sleeps the following night from their daytime activity, using Fitbit data from the PMData dataset.

The whole project lives in one notebook, [`01_explore_and_prepare_data.ipynb`](01_explore_and_prepare_data.ipynb): data exploration, cleaning, building the model table, and training and evaluating the model.

## The question

Can a participant's activity during the day help predict how long they will sleep that night?

Predictions are made at **21:00**, using only what is known by then:

- steps and exercise sessions between waking up and 21:00
- the previous night's sleep (bedtime, duration, and the average over the last three nights)

Nothing about the night being predicted is used as an input, to prevent information leakage.

## Data

This project uses the **PMData** dataset: <https://osf.io/vx4bk/overview>

The data is not included in this repository (it is about 3 GB). After downloading it, place it so that the folder layout looks like this:

```
data/
└── raw/
    └── pmdata/
        ├── participant-overview.xlsx
        ├── p01/
        │   ├── fitbit/        (sleep.json, steps.json, exercise.json, ...)
        │   ├── googledocs/    (reporting.csv)
        │   └── pmsys/         (wellness.csv, ...)
        ├── p02/
        └── ... p16/
```

The notebook reads the files from `data/raw/pmdata` and never changes them.

## Running the notebook

Requires Python 3.12 or newer.

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter lab 01_explore_and_prepare_data.ipynb
```

Then run all cells from top to bottom.

## What the notebook does

1. **Setup** and a check of which participants have which files
2. **Sleep data**: main sleep only, duplicates removed, 3-night average sleep
3. **Activity data**: steps and exercise sessions counted only between wake-up and 21:00
4. **Combine activity and sleep**: one row per participant per day, matched to the following night's sleep; participant p12 and unreliable nights (daytime sleep, days without step data) are removed
5. **Model table**: time-of-day inputs, activity types turned into numbers
6. **Train and test**: random forest on a random 80/20 split, with error measures, a predicted-vs-real chart, and which inputs the model relies on
