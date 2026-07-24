# 🔥 Algerian Forest Fire — FWI Prediction

A Flask web application that predicts the **Fire Weather Index (FWI)** using a Ridge regression model trained on the Algerian Forest Fires dataset.

**🔗 Live app:** https://algerian-forest-fire-urz8.onrender.com

---

## Overview

This project builds a regression model that estimates the **Fire Weather Index (FWI)** — a numeric indicator of forest fire risk — from weather and fire-behavior-index readings, and serves it through a small Flask web app.

The underlying data is the **Algerian Forest Fires dataset**, which contains observations from two regions of Algeria (Bejaia and Sidi Bel-Abbes) collected over the summer of 2012.

## How it works

1. **Data cleaning** (`notebooks/Untitled copy.ipynb`)
   - Raw CSV (`Algerian_forest_fires_dataset_UPDATE.csv`) is loaded, with the two regions originally stacked in one file separated by a header row.
   - A `Region` column is added (0 = Bejaia, 1 = Sidi Bel-Abbes) based on row position.
   - Null rows and a stray header row embedded mid-file are dropped.
   - Column names are stripped of whitespace, and numeric columns (`day`, `month`, `year`, `Temperature`, `RH`, `Ws`, and the fire indices) are cast to proper numeric types.
   - The `Classes` label (fire / not fire) is cleaned (trimmed, lowercased) for consistency.
   - The cleaned data is exported to `Algerian_cleaned.csv`.

2. **Exploratory data analysis**
   - Correlation heatmaps to inspect relationships between weather variables and FWI.
   - Boxplots to check outliers across all numeric features.
   - Fire-count breakdowns by month for each region.

3. **Feature selection**
   - A multicollinearity check drops features with pairwise correlation above 0.85 (this removes `DC` and `BUI`, which are highly correlated with other fire indices).
   - Remaining features used to predict `FWI`: **Temperature, RH (relative humidity), Ws (wind speed), Rain, FFMC, DMC, ISI, Classes, Region**.

4. **Modeling**
   - Features are standardized with `StandardScaler`.
   - Several regressors are trained and compared on a 75/25 train-test split:

     | Model              | MAE   | R²     |
     |---------------------|-------|--------|
     | Linear Regression    | 0.547 | 0.9848 |
     | Lasso                 | 1.133 | 0.9492 |
     | **Ridge**             | **0.564** | **0.9843** |
     | LassoCV               | 0.547 | 0.9848 |
     | RidgeCV               | 0.564 | 0.9843 |
     | ElasticNet            | 1.882 | 0.8753 |
     | ElasticNetCV          | 0.658 | 0.9814 |

   - **Ridge Regression** was selected as the production model (a strong balance of accuracy and regularization against multicollinearity), and is saved along with the fitted `StandardScaler`.

5. **Serving** (`application.py`)
   - A Flask app loads `models/ridge.pkl` and `models/scaler.pkl` at startup.
   - The `/` route renders a home page (`templates/home.html`) with a form for the 9 input features.
   - The `/predictdata` route (GET/POST) reads the submitted form values, scales them with the saved `StandardScaler`, runs them through the Ridge model, and renders the predicted FWI back on the page.

## Project structure

```
Algerian-forest-fire/
├── application.py                 # Flask app (routes + inference)
├── models/
│   ├── ridge.pkl                  # Trained Ridge regression model
│   └── scaler.pkl                 # Fitted StandardScaler
├── notebooks/
│   └── Untitled copy.ipynb        # Data cleaning, EDA, model training & comparison
├── templates/
│   ├── home.html                  # Prediction form + result page
│   └── index.html                 # Simple test/landing page
├── requirements.txt                # Python dependencies
├── Procfile                        # Gunicorn start command (for deployment)
└── .gitignore
```

## Tech stack

- **Python / Flask** — web app and routing
- **scikit-learn** — `Ridge`, `StandardScaler`, and comparison against Linear/Lasso/ElasticNet variants
- **pandas / numpy / scipy** — data processing
- **Gunicorn** — production WSGI server (used for deployment, e.g. on Render)

## Getting started

### Prerequisites
- Python 3.8+

### Installation

```bash
git clone https://github.com/ishityadav/Algerian-forest-fire.git
cd Algerian-forest-fire
pip install -r requirements.txt
```

### Run locally

```bash
python application.py
```

The app runs at `http://localhost:5001`. Open it in your browser, fill in the 9 fields, and submit to get a predicted FWI value.

### Input fields

| Field       | Description                              |
|-------------|-------------------------------------------|
| Temperature | Temperature (°C)                          |
| RH          | Relative Humidity (%)                     |
| Ws          | Wind speed (km/h)                         |
| Rain        | Rainfall (mm)                             |
| FFMC        | Fine Fuel Moisture Code                   |
| DMC         | Duff Moisture Code                        |
| ISI         | Initial Spread Index                      |
| Classes     | Fire class indicator (0 = not fire, 1 = fire) |
| Region      | 0 = Bejaia region, 1 = Sidi Bel-Abbes region |

### Deployment

The app is deployment-ready for platforms like Render or Heroku via the included `Procfile`:

```
web: gunicorn application:app
```

## License

No license specified. Feel free to open an issue if you'd like this clarified.
