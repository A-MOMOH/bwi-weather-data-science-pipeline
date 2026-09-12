# BWI Weather Data Science Pipeline

An end-to-end data science pipeline for analyzing weather and precipitation patterns around Baltimore/Washington International Thurgood Marshall Airport (BWI) using NOAA weather observations.

The project covers data validation, exploratory data analysis, feature engineering, statistical inference, regression and classification modeling, time-series analysis, and unsupervised learning.

## Project Overview

Weather data contains a mix of seasonal patterns, extreme events, temporal dependencies, and relationships between atmospheric variables. This project explores those relationships using daily weather observations and hourly precipitation measurements from the BWI area.

The analysis focuses on three main goals:

- Explore relationships among temperature, precipitation, and wind conditions.
- Build predictive models for maximum temperature and the occurrence of rain.
- Examine temporal precipitation patterns and identify naturally occurring weather regimes through clustering.

## Data

The project uses two NOAA datasets:

### Daily Weather Data

`clean_daily_weather.csv`

Each observation represents one day and includes variables such as:

- Daily precipitation (`PRCP`)
- Snowfall and snow depth (`SNOW`, `SNWD`)
- Average, maximum, and minimum temperature (`TAVG`, `TMAX`, `TMIN`)
- Average wind speed (`AWND`)
- Wind direction and gust measurements

### Hourly Precipitation Data

`rain_hourly_clean.csv`

Contains timestamped precipitation measurements used for time-series analysis.

## Pipeline

The project follows the workflow:

**Data Validation → Exploratory Data Analysis → Feature Engineering → Statistical Inference → Predictive Modeling → Time-Series Analysis → Clustering**

## Exploratory Data Analysis

The daily weather data was explored using distributions, correlation analysis, and relationships between weather variables.

Maximum daily temperature showed strong seasonal variation, spanning nearly 80°F across the observed period. Temperatures near the lower and upper ends of the distribution were consistent with expected winter and summer conditions rather than erroneous outliers.

Correlation analysis was also used to investigate relationships among temperature, precipitation, and wind variables.

## Feature Engineering

Several features were created to capture additional weather characteristics:

### Temperature Range

```text
temp_range = TMAX - TMIN
```

Measures the difference between the daily maximum and minimum temperatures.

### Rain Day

```text
rain_day = 1 if PRCP > 0 else 0
```

Converts precipitation into a binary indicator representing whether measurable rainfall occurred.

### Wind Gust Ratio

Measures the strength of peak wind gusts relative to average wind speed.

### Season

Dates were grouped into Winter, Spring, Summer, and Fall to represent seasonal weather patterns directly.

## Statistical Inference

### January vs. July Temperature

A two-sample t-test found a substantial difference between average January and July temperatures.

| Month | Mean Temperature |
|---|---:|
| January | 30.65°F |
| July | 80.52°F |

**Estimated difference:** 49.87°F  
**95% CI:** 46.82°F – 52.93°F  
**p-value:** approximately 0

July was therefore nearly 50°F warmer than January on average, with strong statistical evidence of a seasonal temperature difference.

### Wind Gust Behavior on Rain Days

Rain days had an average wind gust ratio of **4.65**, compared with **4.10** on non-rain days.

**Difference:** 0.55  
**95% CI:** 0.03 – 1.08  
**p-value:** 0.0142

The difference was statistically significant, although relatively small in magnitude.

### Temperature Range and Wind Speed

The correlation between daily temperature range and average wind speed was:

**r = -0.19**

with a 95% confidence interval of **[-0.30, -0.07]**.

This indicates a weak negative relationship in which windier days tend to have slightly smaller differences between their daily high and low temperatures.

## Regression Modeling

A multiple linear regression model was developed to predict maximum daily temperature (`TMAX`) using:

- Minimum temperature
- Average wind speed
- Rain-day status
- Wind gust ratio

Predictors were standardized before modeling.

### Performance

| Metric | Result |
|---|---:|
| R² | **0.8838** |
| RMSE | **6.5473°F** |
| MAE | **5.2021°F** |

The model explained approximately **88% of the variation in maximum daily temperature**.

Minimum temperature was by far the strongest predictor. Rain-day status and wind gust ratio had smaller effects, while average wind speed contributed relatively little after accounting for the other variables.

## Rain Classification

A logistic regression model was trained to classify whether a day experienced rainfall.

The model used:

- Average temperature
- Temperature range
- Average wind speed
- Wind gust ratio

Precipitation itself was intentionally excluded because `rain_day` is derived directly from precipitation.

### Performance

| Metric | Result |
|---|---:|
| Accuracy | **76.36%** |
| ROC AUC | **0.7557** |

The confusion matrix showed:

- 33 correctly classified non-rain days
- 9 correctly classified rain days
- 2 false positives
- 11 false negatives

The model therefore performed reasonably well overall but was more likely to miss a rain event than incorrectly predict rain.

## Time-Series Analysis

Hourly precipitation observations were converted to a datetime index and examined for temporal structure.

The hourly data was highly sparse, consisting primarily of long dry periods interrupted by short precipitation events.

Aggregating observations to daily totals made individual rainfall events and longer wet or dry periods easier to identify.

### Seasonal Decomposition

Daily precipitation was decomposed using an additive model with a **365-day seasonal period**.

The decomposition revealed:

- A varying long-term precipitation trend
- A repeating annual seasonal component
- Short-lived precipitation events captured in the residual component

### Autocorrelation

ACF and PACF analysis showed the strongest meaningful dependence at **lag 1**, with autocorrelation around **0.15**.

Most later lags remained within the confidence bands, indicating that precipitation has relatively little long-term memory.

### Autoregressive Modeling

AR models from lag 1 through lag 10 were compared using AIC.

**AR(1)** produced the lowest AIC:

```text
AIC = -665.28
```

The fitted AR(1) coefficient was:

```text
AR(1) = 0.1531
```

This indicates weak positive persistence: precipitation on one day has a small influence on precipitation the following day, but rainfall events are largely episodic.

## Clustering Analysis

Unsupervised learning was used to identify common weather regimes.

The clustering features were:

- Average temperature (`TAVG`)
- Precipitation (`PRCP`)
- Average wind speed (`AWND`)
- Temperature range (`temp_range`)

All features were standardized before clustering.

Both **K-means** and **hierarchical clustering** were explored. The elbow method suggested **k = 5** for K-means.

### Identified Weather Regimes

| Cluster | Description |
|---|---|
| 0 | Mild, dry days with large temperature swings |
| 1 | Cool, dry, very windy days |
| 2 | Cold, slightly wet, calm days |
| 3 | Hot, dry, calm days |
| 4 | Warm, rainy days with moderate winds |

These clusters correspond to recognizable seasonal and weather conditions, ranging from calm summer days to windy cool-weather conditions and warm-season rain events.

## Key Findings

- Maximum temperature could be predicted effectively from other daily weather variables, with the regression model achieving **R² = 0.8838**.
- The rain classifier achieved **76.36% accuracy** and an **AUC of 0.7557** without directly using precipitation as an input.
- July averaged nearly **50°F warmer than January**, demonstrating the strong seasonal temperature cycle.
- Rain days had slightly higher wind gust ratios than non-rain days.
- Daily precipitation showed only weak short-term autocorrelation, with **AR(1)** providing the best autoregressive fit.
- Clustering identified five interpretable weather regimes corresponding to temperature, precipitation, wind, and seasonal conditions.

## Technologies

- Python
- pandas
- NumPy
- Matplotlib
- Seaborn
- SciPy
- scikit-learn
- statsmodels
- Jupyter Notebook

## Repository Structure

```text
bwi-weather-data-science-pipeline/
│
├── README.md
├── LICENSE
├── requirements.txt
├── weather_analysis.ipynb
│
└── data/
    ├── clean_daily_weather.csv
    └── rain_hourly_clean.csv
```

## Installation

Clone the repository:

```bash
git clone https://github.com/A-MOMOH/bwi-weather-data-science-pipeline.git
cd bwi-weather-data-science-pipeline
```

Install the required Python packages:

```bash
pip install -r requirements.txt
```

Open `weather_analysis.ipynb` to explore the complete analysis.
