# Code Review & Critique

## Code Quality

**1. God module (`functions.py`)**
`functions.py` is 700+ lines handling data fetching, statistics, plotting, ML metrics, and time series — all mixed together. It should be split into focused modules (`data.py`, `stats.py`, `plotting.py`, etc.).

**2. Bare `except` clauses**
Throughout model-selection loops (`get_best_arma_model`, etc.):
```python
except:
    continue
```
This silently swallows *all* exceptions including `KeyboardInterrupt`. Use `except Exception:` at minimum, and log what failed.

**3. NaN check / copy-paste bug in `MachineLearningClassifier`**
```python
if np.isnan(self.recent_open_price[security]).any():  # checks open...
    self.recent_close_price[security] = ...           # ...but replaces close
```
The checks and replacements are swapped. Also, `np.isnan()` on a Python list will fail — needs `np.array(...)` first.

**4. Shared model across securities**
In `MachineLearningClassifier.handle_data()`, a single `self.mdl` is fitted in a loop over all securities. By the time predictions are made, the model only reflects the *last* security fitted. Each security needs its own model instance.

**5. Non-reproducible train/test splits**
```python
split = rand.uniform(0.60, 0.80)
```
Using a random split means results can't be reproduced. Fix with a seed (`random.seed(42)`) or a fixed ratio. Also, for time series data, a proper walk-forward or time-ordered split is more appropriate than a random one.

**6. Magic numbers / no configuration**
Values like `0.80` (stop-loss), `100` (data points), `0.013` (commission), `252` (trading days) are scattered and hardcoded. These should be named constants or a config file.

**7. Global variable usage in strategy classes**
`tickers` and `log` are referenced as globals inside `MachineLearningClassifier` and `MachineLearningRegressor`. They should be parameters, not implicit globals.

**8. Wrong percentage calculation**
```python
percent = ((last - first) / first) * 10  # should be * 100
```

**9. Identity comparison on string**
```python
if title is not '':  # should be: if title != ''
```
`is not` checks object identity, not equality.

---

## Deprecated / Broken APIs

| Issue | Location |
|---|---|
| `pd.Panel` removed in pandas 1.0 | `MachineLearningClassifier.py`, `MachineLearningRegressor.py` |
| `sklearn.preprocessing.Imputer` → `SimpleImputer` | Both strategy files |
| `LinearRegression(normalize=True)` removed in sklearn 1.0+ | `mlearning/LinearRegression.py` |
| `mlab.normpdf` removed in matplotlib 3+ | `functions.py` |
| Python 3.6 only (`bin/python3.6`) | `bin/` |

---

## Project Structure / Repository Hygiene

**10. Virtualenv committed to git**
The entire `bin/` directory is a Python 3.6 virtual environment. It should be in `.gitignore` and replaced with a `requirements.txt` or `pyproject.toml`.

**11. `__pycache__` and `.DS_Store` committed**
Both are in the repo and should be gitignored. The `.gitignore` appears to be present but not working for these.

**12. No `requirements.txt`**
There's no way to reproduce the environment. Given the number of dependencies (`zipline`, `pymc3`, `talib`, `pyfolio`, `arch`, `quandl`), this is a significant gap.

**13. README is empty**
The README only contains the project title. No setup instructions, no description of the models, no how-to-run.

---

## SQL

**14. Typo and invalid column name in `sql/script.sql`**
```sql
synbol_id INT REFERENCES symbol (id)  -- typo: synbol → symbol
ex-dividend decimal(19,4) NULL         -- hyphen in column name is invalid without quoting
```

**15. Hardcoded password in SQL script**
```sql
CREATE USER sec_user WITH PASSWORD 'Password';
```
Even for a thesis, this is a bad habit and could be committed to a public repo.

---

## ML / Finance Methodology

**16. Refitting the model every bar on growing data**
The model is retrained from scratch each bar with an ever-growing training window. This is computationally expensive and is a subtle form of look-ahead bias for out-of-sample evaluation.

**17. Only SMA features**
The regressor uses only SMA-15 and SMA-50 as features. More robust feature engineering (e.g., RSI, volume trends, lagged returns) would improve predictive power — the `get_technical_analysis_features` function in `functions.py` already computes many of these but isn't used by the strategies.

**18. No walk-forward validation**
For financial time series, standard k-fold cross-validation is inappropriate due to temporal dependency. Walk-forward (rolling-window) validation is the correct approach.

**19. No tests**
`testing/` contains comparison scripts, not unit tests. There are no tests for data transforms, metric calculations, or strategy logic.

---

## Summary of Priority Fixes

| Priority | Issue |
|---|---|
| 🔴 High | Fix NaN check bug & shared model in classifier |
| 🔴 High | Add `requirements.txt`, remove virtualenv from git |
| 🔴 High | Replace deprecated pandas/sklearn APIs |
| 🟡 Medium | Replace bare `except` with specific exceptions |
| 🟡 Medium | Split `functions.py` into modules |
| 🟡 Medium | Fix SQL typos and remove hardcoded password |
| 🟢 Low | Add README, fix magic numbers, add reproducible seeds |
