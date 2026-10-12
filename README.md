# Portfolio Optimization of Macedonian Stocks — SLSQP

A quantitative finance project applying constrained portfolio optimization, statistical analysis, and robust regression to historical Macedonian stock market data. The project compares an optimized five-stock portfolio with an equally weighted portfolio and the MBI-10 benchmark.

## Overview

The project investigates the historical return-to-risk characteristics of Macedonian equities using Python.

The analysis includes:

* **Portfolio optimization:** SciPy's Sequential Least Squares Programming (SLSQP) algorithm.
* **Portfolio comparison:** Optimized weights versus equal weights and the MBI-10 index.
* **Return and risk analysis:** Annualized returns, volatility, return-to-risk ratios, quartiles, and historical 5th-percentile returns.
* **Correlation analysis:** Spearman correlations between assets and Pearson correlations including the constructed portfolios.
* **Performance evaluation:** Cumulative performance, hypothetical investment growth, and maximum drawdown.
* **Regression analysis:** OLS and Huber robust regression for estimating market beta.

## Assets and Data

The optimized portfolio contains five Macedonian stocks:

| Stock   | Portfolio asset |
| ------- | --------------- |
| Replek  | Equity          |
| TTK     | Equity          |
| Makstil | Equity          |
| Tikves  | Equity          |
| Okta    | Equity          |

The MBI-10 index is used as the market benchmark.

The input datasets are:

* `Vremenska_serija.xlsx` — historical stock prices.
* `MBI10_mesecni.xlsx` — monthly MBI-10 returns in the `Prosecen_prinos` column.

The analysis assumes that the datasets contain corresponding monthly observations in chronological order. The code aligns observations by row index rather than merging on dates, so this assumption must be verified before interpreting the results. Benchmark returns must also use units consistent with the stock returns.

## Methodology

### 1. Return Calculation

Stock log returns are calculated from consecutive prices:

$$
r_t = \ln\left(\frac{P_t}{P_{t-1}}\right)
$$

For portfolio construction, log returns are converted into simple returns:

$$
R_t = e^{r_t}-1
$$

The weighted simple returns are then converted back into portfolio log returns:

$$
r_{p,t} = \ln\left(1+\sum_{i=1}^{n}w_iR_{i,t}\right)
$$

This allows the code to calculate portfolio wealth by compounding the resulting monthly log returns.

### 2. Constrained Portfolio Optimization

SLSQP is used to maximize the historical portfolio return-to-volatility ratio:

$$
\max_{\mathbf{w}}
\frac{\mathbf{w}^{T}\boldsymbol{\mu}}
{\sqrt{\mathbf{w}^{T}\Sigma\mathbf{w}}}
$$

where:

* **w** is the vector of portfolio weights.
* **μ** is the vector of historical mean monthly log returns.
* **Σ** is the covariance matrix of monthly log returns.

The optimization applies the following constraints:

$$
\sum_{i=1}^{n}w_i=1
$$

$$
0\leq w_i\leq0.45
$$

The portfolio is fully invested, short selling is prohibited, and each stock is capped at 45% of the portfolio.

The objective is a return-to-volatility ratio, not a conventional Sharpe ratio, because no risk-free rate is subtracted from returns.

### 3. Portfolio Performance

Three portfolios are evaluated:

1. **SLSQP Optimized Portfolio** — weights selected by the optimization algorithm.
2. **Equally Weighted Portfolio** — 20% allocated to each stock.
3. **MBI-10 Stock Index** — the market benchmark.

The performance analysis includes annualized arithmetic approximations of mean log returns, annualized compounded returns derived from mean monthly log returns, annualized volatility, and annualized return-to-volatility ratios.

Annualized volatility is calculated as monthly standard deviation multiplied by \(\sqrt{12}\).

### 4. Distribution and Risk Analysis

Monthly return distributions are examined using:

* First quartile (Q1).
* Third quartile (Q3).
* The 5th-percentile monthly return, used as a historical lower-tail risk threshold.

The reported `VaR_5%_monthly` is the 5th-percentile return expressed as a percentage. It is not converted into a positive loss figure.

### 5. Cumulative Performance and Drawdown

A hypothetical initial investment of 1,000 MKD is used to illustrate the historical growth of each portfolio.

Cumulative log returns are summed and exponentiated to obtain wealth multiples. The resulting wealth series is used to calculate hypothetical portfolio values and cumulative performance.

Drawdown measures the percentage decline in portfolio value from its previous peak. Maximum drawdown is the most negative drawdown observed over the sample.

### 6. OLS and Huber Regression

OLS and Huber robust regression are used to estimate the relationship between asset or portfolio returns and MBI-10 returns.

The analysis reports:

* OLS beta.
* Huber beta.
* The difference between the Huber and OLS beta estimates.

The comparison provides a way to examine how robust regression responds to observations that may exert greater influence on an ordinary least squares estimate.

These regressions describe historical relationships and do not establish causality.

## Visualizations

The script generates and saves the following figures in the `images/` directory:

| File                         | Description                                                                    |
| ---------------------------- | ------------------------------------------------------------------------------ |
| `correlation_spearman.png`   | Spearman correlation matrix for the benchmark and individual stocks            |
| `correlation_pearson.png`    | Pearson correlation matrix including the optimized and equal-weight portfolios |
| `drawdowns.png`              | Historical portfolio drawdowns                                                 |
| `cumulative_performance.png` | Cumulative portfolio performance                                               |
| `hypothetical_profit.png`    | Hypothetical value of an initial 1,000 MKD investment                          |

## Technologies

* Python
* pandas
* NumPy
* SciPy
* Matplotlib
* Seaborn
* statsmodels
* Excel data import with `openpyxl`

## Project Structure

```text
portfolio-optimization-macedonian-stocks-SLSQP-2026/
├── README.md
├── portfolio_optimization.py
├── Data/
│   ├── Vremenska_serija.xlsx
│   └── MBI10_mesecni.xlsx
├── images/
│   ├── correlation_spearman.png
│   ├── correlation_pearson.png
│   ├── drawdowns.png
│   ├── cumulative_performance.png
│   └── hypothetical_profit.png
└── results/
    ├── weights.csv
    ├── returns_and_risk.csv
    ├── q1_q3_var.csv
    ├── profit_drawdown_cumulative.csv
    └── betas_ols_vs_huber.csv
```

The structure above describes the intended organization of the project. Generated files appear after the script runs.

## Running the Project

### 1. Install dependencies

```bash
pip install pandas numpy seaborn statsmodels matplotlib scipy openpyxl
```

### 2. Configure the data directory

Update `DATA_FOLDER` in the Python script to point to the directory containing both Excel datasets.

For example:

```python
DATA_FOLDER = "Data/"
```

Ensure that the directory contains `Vremenska_serija.xlsx` and `MBI10_mesecni.xlsx`.

### 3. Run the script

Execute the script in a Python environment with access to the input files:

```bash
python portfolio_optimization.py
```

The script prints the portfolio weights, return and risk statistics, distribution measures, hypothetical investment results, and regression estimates. It saves figures to `images/` and CSV tables to `results/`.

## Limitations

* The analysis uses a limited historical sample of monthly observations.
* The optimization is performed in-sample and does not include out-of-sample or walk-forward validation.
* The datasets are aligned by row index rather than by explicit dates.
* Results depend on consistent return units and correct benchmark alignment.
* Transaction costs, taxes, liquidity constraints, and market impact are not modeled.
* Historical optimization results do not establish that the selected weights will perform well in the future.
* The return-to-volatility objective does not account explicitly for a risk-free rate.

## Disclaimer

This project is intended for educational and analytical purposes only. It does not constitute investment advice or a recommendation to buy or sell any security.
