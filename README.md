# Sleep Predictor

Predicting how long someone sleeps the following night, using Fitbit data and daily self-reports from the PMData dataset.

## The question

How long will a participant sleep tonight?

Predictions are made at **21:00**, using only what is known by then. Nothing about the night being predicted is used as an input, to prevent information leakage.

## Project structure

```
sleep_prediction/
├── notebooks/
│   ├── 01_explore_and_prepare_data.ipynb          iteration 1: cleaning, first models
│   ├── 02_exploratory_data_analysis.ipynb         iteration 2: the EDA
│   └── 03_feature_engineering_iteration_2.ipynb   iteration 2: feature engineering, feature selection, models, SHAP
├── data/raw/pmdata/        the dataset (not in git, see below)
├── models/                 the exported best model (random forest) and its feature list
├── reports/                reports for the course
├── app/                    the web app (in progress)
├── requirements.txt
└── README.md
```

Run the notebooks in order: each one builds on the decisions of the one before.

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

The notebooks read the files from `data/raw/pmdata` and never change them.

## Running the notebooks

Requires Python 3.12 or newer.

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter lab notebooks/
```

Open a notebook and run all cells from top to bottom.

## Iteration 1: `01_explore_and_prepare_data.ipynb`

1. **Setup** and a check of which participants have which files
2. **Sleep data**: main sleep only, duplicates removed, 3-night average sleep
3. **Activity data**: steps and exercise sessions counted only between wake-up and 21:00
4. **Combine activity and sleep**: one row per participant per day, matched to the following night's sleep; participant p12 and unreliable nights (daytime sleep, days without step data) are removed
5. **Model table**: time-of-day inputs, activity types turned into numbers
6. **Train and test**: compare the models with last-night and seven-night-average baselines on the same chronological test set: the last 20% of each participant’s prepared nights. Earlier nights supply training and validation; the neural network uses validation for early stopping.

## Iteration 2

**`02_exploratory_data_analysis.ipynb`** explores the data before choosing any features: missing values, distributions, outliers and how each feature relates to the following night's sleep. Only the first 80% of each participant's nights are explored; the rest is the test set.

**`03_feature_engineering_iteration_2.ipynb`** applies the EDA decisions and builds the final model:

- **Missing values**: fatigue and readiness filled per participant with KNN (k = 10); nights under 3 hours replaced by the participant's own report or dropped
- **Feature selection**: feature groups compared on the validation nights, checked with SHAP and permutation importance
- **Models**: linear regression, random forest, gradient boosting, a neural network and KNN regression, all compared with the seven-night-average baseline on the same test nights
- **Result**: the random forest is the best model; it is exported to `models/`

## Using the exported model

```python
import joblib

model = joblib.load("models/sleep_random_forest.joblib")
minutes = model.predict(features)  # columns in the order listed in models/sleep_random_forest.json
```

`models/sleep_random_forest.json` lists the features, what each one means and the scikit-learn version the model was saved with.
