# Prometheus_Capital_2026
The Weekly Quant Radar (WQR) is a recurring research series focused on portfolio construction under realistic conditions. Each instalment examines a core allocation framework, evaluates its theoretical appeal and tests its empirical performance using transparent methods.

A central theme throughout the series is the gap between in-sample optimality and out-of-sample robustness. By systematically comparing classical optimisation techniques with simple, implementable benchmarks, the radar aims to clarify when added model complexity genuinely improves outcomes and when it does not.

---
### WQR_1

Notebook `WQR_1.ipynb` accompanies the `WQR_1.pdf` documentation and provides a framework for comparing simple equal-weighted (1/n) portfolios with Markowitz mean-variance optimisation.

Using daily equity data, the notebook implements a fixed and expanding-window backtest in which portfolio weights are estimated on historical data and evaluated out-of-sample. The exercise highlights a key empirical result: while Markowitz optimisation can deliver strong in-sample performance, its out-of-sample results are often undermined by estimation error, particularly in expected returns, leaving the simple 1/n rule as a robust and competitive benchmark.

---
### WQR_2

Notebook `WQR_2.ipynb` accompanies the `WQR_2.pdf` documentation and provides a framework for comparing simple equal-weighted (1/n) portfolios with robust Markowitz mean-variance optimisation. 

The notebook evaluates several regularisation techniques, including mean shrinkage, covariance shrinkage, Bayesian-style priors and explicit weight constraints, all designed to reduce the "error-maximising" behaviour of classical Markowitz optimisation.

Using daily equity data, the notebook implements a fixed and expanding-window backtest in which portfolio weights are estimated on historical data and evaluated out-of-sample. The exercise highlights a key empirical result: although robust Markowitz variants often deliver stronger in-sample Sharpe ratios, their out-of-sample advantage is far less reliable. Indicating that successful active allocation is not driven by optimisation alone, but by the ability to control estimation error and produce stable realised performance.
