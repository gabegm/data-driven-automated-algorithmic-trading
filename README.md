# Data-Driven Automated Algorithmic Trading

> Bachelor thesis project exploring whether statistical machine learning and time series models can be used to predict stock prices and build a profitable automated trading strategy.

---

## Motivation

This project sets out to challenge two widely-held financial hypotheses:

- **Efficient Market Hypothesis (EMH)** — markets are perfectly efficient; no investor can consistently beat them.
- **Random Walk Hypothesis (RWH)** — stock prices move randomly; past prices carry no predictive information.

Using a combination of time series analysis, machine learning, and Bayesian statistics, the project tests whether historical price data can be used to forecast future movements and generate returns above a passive benchmark (S&P 500).

---

## Methodology

Three broad families of forecasting models were evaluated across five stocks: `MSFT`, `CDE`, `NAVB`, `HRG`, and `HL` over the period **2010–2017**.

### 1. Time Series Analysis (`tseries/`)

Classical econometric models were fitted to log returns and evaluated on in-sample fit and out-of-sample forecast quality using Sharpe ratio, AIC, and stationarity tests (ADF, KPSS, Shapiro-Wilk, etc.).

| Model | Result |
|---|---|
| Random Walk | Disproved — autocorrelation plots show past movement is related to future movement |
| OLS | Strong in-sample fit (>96% prediction rate) |
| AR | Poor fit; heavy tails in QQ plots; failed out-of-sample |
| MA | Marginal improvement over AR; still a poor fit |
| ARMA | Best of the classical models; lower Sharpe ratio divergence |
| ARIMA | Worse than ARMA; heavier tails |

### 2. Statistical Machine Learning (`mlearning/`)

Ten classifiers and regressors were trained on technical analysis features (SMA crossovers, lagged returns) and benchmarked against each other.

**Classification** (predicting price direction — up/down):

| Model | Notes |
|---|---|
| Decision Tree | >75% accuracy across all stocks |
| Boosted Decision Tree | Slight improvement over DT |
| SVM | Consistently strong results |
| Random Forest | Matched SVM accuracy |
| K-Nearest Neighbour | Inconsistent across stocks |
| Logistic Regression | <80% accuracy |
| Bernoulli Naive Bayes | Lower 70th percentile accuracy |
| Gaussian Naive Bayes | Similar to Bernoulli |
| Neural Network (MLP) | Good overall; some variability |
| Stochastic Gradient Descent | Unreliable; as low as 59% on one stock |

**Regression** (predicting price level):

| Model | Notes |
|---|---|
| Linear Regression | Best performer; >95% accuracy, low data loss |
| Neural Network (MLP) | Good but inconsistent across stocks |
| SGD | Solid in most cases; poor on CDE |
| Decision Tree | Failed to fit; predicted sharp declines |
| Boosted DT | Similar failure to DT |
| K-Nearest Neighbour | Poor fit |
| Random Forest | Poor fit; erratic out-of-sample forecasts |

### 3. Bayesian Statistics (`pmlearning/`)

Probabilistic models using PyMC3 were applied to log returns to estimate stochastic volatility.

| Model | Notes |
|---|---|
| Metropolis-Hastings | Poor in-sample fit; large Sharpe ratio divergence |
| NUTS (No-U-Turn Sampler) | Good in-sample fit; Sharpe ratios closely matched |

---

## Automated Trading Strategy (`strategies/`)

The top-performing ML models were integrated into a live backtesting engine built with [Zipline](https://github.com/quantopian/zipline). Two strategies were evaluated over 2010–2017 with commission costs of $0.013/share.

| Strategy | Long-only Return | Long + Short Return | With Stop-Loss (-20%) |
|---|---|---|---|
| ML Classifier (Random Forest) | +49.1% | -80.0% | +67.5% |
| ML Regressor (Random Forest) | +43.0% | -80.4% | **+83.7%** |
| S&P 500 Benchmark | — | — | +100.8% |

**Key finding:** Both strategies were profitable long-only with a stop-loss, but neither beat the S&P 500 benchmark over the test period — lending partial support to the EMH. However, evidence of repeating trends in some stocks suggests the RWH does not universally hold.

---

## Project Structure

```
├── functions.py               # Shared utilities (data loading, indicators, metrics, plots)
├── tseries/                   # Time series models (AR, MA, ARMA, ARIMA, OLS, Random Walk)
├── mlearning/                 # Supervised ML models (classifiers and regressors)
├── pmlearning/                # Bayesian models (Metropolis, NUTS via PyMC3)
├── strategies/                # Zipline backtesting strategies
│   ├── MachineLearningClassifier.py
│   └── MachineLearningRegressor.py
├── testing/                   # Model comparison scripts
├── sql/                       # Database schema for a securities master database
├── graduate-thesis/           # Full LaTeX thesis
└── journal-article/           # Condensed journal article version
```

---

## Setup

> ⚠️ This project was built with Python 3.6 and older library versions. Some APIs have since been deprecated (notably `pd.Panel`, `sklearn.preprocessing.Imputer`, `mlab.normpdf`). A modernised rewrite is planned.

### Dependencies

Key libraries used:

- `zipline` — backtesting engine
- `pymc3` — Bayesian modelling
- `scikit-learn` — machine learning models
- `statsmodels` — time series models (ARMA, ARIMA, SARIMAX)
- `arch` — GARCH / stationarity tests
- `pyfolio` — portfolio analytics
- `TA-Lib` — technical analysis indicators
- `quandl` — historical price data

### Data

Historical end-of-day OHLCV data is sourced from the [Quandl WIKI Prices](https://www.quandl.com/databases/WIKIP) dataset. Place the downloaded CSV at `data/WIKI_PRICES.csv`.

---

## Results Summary

- **Time series models** showed limited forecasting ability for stock returns; ARMA was the strongest classical model.
- **Machine learning classifiers** achieved reasonable directional accuracy (>75%), with SVM and Random Forest leading.
- **Machine learning regressors** performed well for price-level prediction, with Linear Regression being the most reliable.
- **Bayesian NUTS** provided the best probabilistic fit to return distributions.
- **Automated strategies** were profitable with a stop-loss but fell short of the passive S&P 500 benchmark.

---

## Future Work

- **Fundamental analysis features** — incorporating financial statements as additional model inputs.
- **Deep learning** — LSTM or Transformer-based models for sequence prediction.
- **Sentiment analysis** — using news/social data as a complementary signal.
- **Walk-forward validation** — replacing random train/test splits with proper time-ordered evaluation.
- **Modern stack** — migrating from deprecated Zipline to `vectorbt` or `backtrader`.

---

## Author

**Gabriel Gauci Maistre**  
Bachelor Thesis — *Data-Driven Automated Algorithmic Trading*
