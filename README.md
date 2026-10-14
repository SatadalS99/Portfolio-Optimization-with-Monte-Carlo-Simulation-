# Portfolio Optimization with Monte Carlo Simulation

A Monte Carlo simulation framework for building and stress-testing equity
portfolios. Using five years of real daily data for seven large-cap U.S. stocks,
the project simulates thousands of possible futures, compares **Monte Carlo–based
portfolio optimization** against the classical **mean–variance (Markowitz)**
method, and measures downside risk with **Value-at-Risk (VaR)** and **Conditional
Value-at-Risk (CVaR)**.

> Final project for *On the Theory and Practice of Monte Carlo Simulations*,
> TU Dortmund, Winter Semester 2025/26.

---

## What it does

1. **Simulates portfolio returns** from real stock data using a correlated
   multivariate Monte Carlo engine (Box–Muller / Marsaglia-polar generators with
   Cholesky factorization).
2. **Compares two optimizers** — Markowitz mean–variance vs. a Monte Carlo
   random-search optimizer (with common random numbers).
3. **Analyzes tail risk** — VaR and CVaR at the 95% and 99% levels, under several
   scenario models, with out-of-sample backtesting.

## The nine-phase pipeline

The whole analysis lives in one module, `portfolio_mc.py`, organized into nine
namespaces (`p1`–`p9`):

| Phase | Namespace | What it does |
|------|-----------|--------------|
| 1 | `p1` | Data loading, log returns, 80/20 split, EDA (QQ-plots, kurtosis, volatility clustering) |
| 2 | `p2` | Parameter estimation; correlated MVN simulation engine; fan chart |
| 3 | `p3` | Scenario models: MVN, historical bootstrap, regime-switching, Student-t copula |
| 4 | `p4` | Monte Carlo convergence diagnostics and confidence intervals |
| 5 | `p5` | Markowitz frontier + Monte Carlo optimization |
| 6 | `p6` | VaR / CVaR estimation across models |
| 7 | `p7` | Variance reduction: antithetic, control variates, importance sampling |
| 8 | `p8` | Estimation risk: bootstrap + Bayesian (Gibbs) parameter uncertainty |
| 9 | `p9` | Out-of-sample validation, Kupiec/Christoffersen backtests, stress test |

## Repository layout

```
.
├── portfolio_mc.py                 # the full pipeline (p1..p9) in one module
├── analysis.ipynb                  # call-only notebook that runs every phase
├── analysis_executed.ipynb         # the same notebook, with all outputs and figures
├── final_7_stocks_2020_2024.csv    # cleaned dataset (see "Data" below)
├── requirements.txt
├── report/                         # written report (PDF + LaTeX source)
└── docs/                           # plain-language explanation of the project
```

## Quick start

```bash
git clone https://github.com/satadal-santra/portfolio-mc-optimization.git
cd portfolio-mc-optimization

python -m venv .venv && source .venv/bin/activate   # optional
pip install -r requirements.txt

jupyter notebook analysis.ipynb
```

> **Run the notebook top to bottom.** Each phase hands its results to the next
> through `phaseN_outputs.pkl` files, which are build artifacts and are not in the
> repository. Jumping straight to a later phase on a fresh clone raises
> `FileNotFoundError: phase2_outputs.pkl`. Run every cell in order, from the
> repository root (all data and output paths are relative).

The notebook's first cell pulls in every phase from the module:

```python
from portfolio_mc import p1, p2, p3, p4, p5, p6, p7, p8, p9
```

Or use the module directly in a script:

```python
from portfolio_mc import p1, p2
cfg = p1.Phase1Config(train_split=0.8, trading_days=252)
prices, returns, R_train, R_test, tickers = p1.load_and_validate(
    "final_7_stocks_2020_2024.csv", cfg)
mu, sigma, params = p2.estimate_params(R_train, trading_days=252)
```

## Selected results

- Across all seven stocks, normality is rejected (heavy tails, volatility
  clustering), so variance alone is an incomplete risk measure.
- Mean–variance and Monte Carlo search **agree** on the max-Sharpe portfolio
  (mostly AAPL + MSFT); they **diverge** when the objective is min-CVaR, which
  shifts toward defensive names (WMT, JNJ, XOM).
- The scenario model matters most in the far tail: the plain MVN model is the
  least conservative at the 99% level.
- Out-of-sample, the predicted VaR/CVaR track realized losses, with exceedance
  rates close to nominal.

Full details, tables, and discussion are in [`report/report.pdf`](report/report.pdf).
A plain-language walkthrough is in
[`docs/project_explained.pdf`](docs/project_explained.pdf).

## Data

The dataset is a cleaned extract of the open source dataset
*"US Stock Market Giants: Top Companies Stocks Data"*
7 assets, daily, 2020-01-02 → 2024-11-07 —
1,222 trading days per company, 8,554 rows, no missing values.

> **Note on redistribution:** check the source dataset's license before committing
> the CSV. If redistribution isn't permitted, remove the CSV from the repo and
> document the download/cleaning steps instead.

## Reproducibility

All simulations use a fixed random seed (default `42`), so results are
reproducible run-to-run. Tail estimates (VaR/CVaR) need a large simulation
budget (N ≈ 10⁴–10⁵) to stabilize; see Phase 4 for the convergence analysis.

## License

Released under the MIT License — see [`LICENSE`](LICENSE).

## Author

Satadal Santra  — TU Dortmund.
