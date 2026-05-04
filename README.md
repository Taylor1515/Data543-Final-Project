# Hurricane Impact Prediction and Visualization

A neural network that predicts county-level hurricane damage from storm hazard, social vulnerability, and exposure data, paired with an interactive map for exploring predictions on real historical storms.

## What This Project Does

Given a hurricane's wind field and a set of impacted counties, the model predicts five damage metrics for each county:

- Average inspected damage in 2024 dollars
- FEMA registrations as a fraction of housing units
- Total damage as a fraction of building value
- Total damage as a fraction of total property value
- Major damage claims as a fraction of housing units

Predictions are visualized on an interactive choropleth map. Users can select any historical storm from the training data, optionally scale its intensity (a "what if this storm had been stronger" slider), and choose which damage metric to color the map by.

## Repository Contents

- `Hurricane_Impact_Model.ipynb` — the full Google Colab notebook (data prep, model training, evaluation, prediction, visualization)
- `FemaHousingAssistanceLosses.xlsx` — historical FEMA loss data per county per storm
- `HurricaneHazardData.xlsx` — per-county wind speeds and storm-level intensity for each historical storm
- `SocialVulnerabilityData.xlsx` — county-level social vulnerability, resilience, and exposure features
- `README.md` — this file

(Exact spreadsheet filenames may vary; the notebook auto-detects them by matching "Losses", "Hazard", and "Vulnerability" in the filename.)

## How to Replicate This Work

### 1. Download the repository

Clone or download this repo so you have the three spreadsheets and the notebook locally:

```
git clone <repo-url>
```

### 2. Open the notebook in Google Colab

Go to [colab.research.google.com](https://colab.research.google.com), click **File → Upload notebook**, and select `Hurricane_Impact_Model.ipynb`.

You do not need to install anything locally. Colab provides Python, PyTorch, and all required libraries.

### 3. Run the notebook top to bottom

The notebook is divided into nine sections. Run cells in order; each section's purpose is explained in a markdown header.

When you reach **Section 1, Cell 2** (the file upload prompt), upload all three spreadsheets at once. The notebook will automatically identify which file is which based on the filename.

The sections are:

1. **Setup** — installs packages and imports libraries
2. **Data Preparation** — merges the three spreadsheets on storm name, year, and county FIPS
3. **Data Exploration** — plots target distributions and feature correlations
4. **Model Configuration** — sets hyperparameters and prepares the train/test split
5. **Model Training** — trains the neural network (takes ~2-3 minutes)
6. **Model Evaluation** — reports R squared, RMSE, MAE per target and shows diagnostic plots
7. **Architecture Comparison (optional)** — sweeps a few configurations; skip if you only want to use the default
8. **Save Model** — writes `hurricane_model.pt` so the prediction section can reload it
9. **Interactive Prediction and Visualization** — provides a UI for selecting a storm and viewing predicted impacts on a map

### 4. Use the interactive predictor

After running Cell 25 in Section 9, an interactive panel appears with three controls:

- **Historical storm** — pick any storm from the training data
- **Intensity multiplier** — scale all wind speeds up or down (0.5x to 1.5x) to ask hypothetical "what if" questions
- **Map impact metric** — choose which of the five damage metrics drives the map's color scale

Click **Predict and Map** to run the model and render the choropleth. Hover over any county to see the predicted value, the storm category, and the severity classification.

## How the Model Works

The neural network is a fully-connected feedforward regressor implemented in PyTorch. It takes 17 input features per county-storm pair:

- Storm hazard: county wind speed, storm peak wind, storm minimum pressure
- Social vulnerability: SOVI score, resilience score, coastal risk factor
- Exposure: building value, agricultural value, total property value, population, coastal shoreline flag
- Categorical (one-hot encoded): storm passage direction, NOAA coastal tier

Targets are log-transformed and standardized before training to handle the heavy right-skew typical of damage data. The training loop uses Huber loss with a small delta (0.2) and per-sample weighting that emphasizes the rare high-damage examples, which prevents the model from collapsing to mean predictions.

Training and evaluation use a random 80/20 split. An option to hold out specific storms for testing (a more realistic but harder evaluation) is available in the hyperparameters cell.

## Visualization Details

The map uses GeoPandas to load Census TIGER county polygons and Folium to render the interactive choropleth. Each of the four primary impact metrics has its own color gradient and value range:

- Average Inspected Damage: white to red, $0 to $6,500
- Total Damage / Building Value: white to dark orange, 0% to 0.145%
- Registrations / Housing Units: white to blue, 0% to 20%
- Major Damage Claims / Housing Units: white to dark purple, 0% to 0.275%

Color ranges are fixed (not rescaled per storm), so a deep-red county in one storm represents the same predicted damage level as a deep-red county in a different storm.

The list of impacted counties and the per-county wind speeds are pulled directly from the training data for each historical storm. The model itself does not decide which counties are exposed; it predicts damage given the exposure provided.

## Requirements

All requirements are installed automatically by the notebook's first cell. For reference:

- Python 3.10+
- PyTorch
- pandas, numpy, scikit-learn, openpyxl
- matplotlib, seaborn
- geopandas, folium, branca, mapclassify
- ipywidgets

## Limitations

- The training set contains 701 county-storm rows, which is small for predicting rare catastrophic events. The model will under-predict the upper tail of damage outcomes for the most extreme storms.
- The "approximate landfall point" shown on each map is the centroid of the most-wind-affected county for that storm, not an actual track coordinate. The training data does not include storm-track latitude and longitude.
- Predictions for hypothetical storms (using the intensity multiplier) are extrapolations beyond the training distribution and should be treated as illustrative rather than predictive.

## Project Structure

```
.
├── Hurricane_Impact_Model.ipynb    # Main notebook
├── FemaHousingAssistanceLosses.xlsx
├── HurricaneHazardData.xlsx
├── SocialVulnerabilityData.xlsx
└── README.md
```
