# Prometheus_Capital_2026
The Weekly Quant Radar (WQR) is a recurring research series focused on portfolio construction under realistic conditions. Each instalment examines a core allocation framework, evaluates its theoretical appeal and tests its empirical performance using transparent methods.

A central theme throughout the series is the gap between in-sample optimality and out-of-sample robustness. By systematically comparing classical optimisation techniques with simple, implementable benchmarks, the radar aims to clarify when added model complexity genuinely improves outcomes and when it does not.

---
# WQR_1

Notebook `WQR_1.ipynb` accompanies the `WQR_1.pdf` documentation and provides a framework for comparing simple equal-weighted (1/n) portfolios with Markowitz mean-variance optimisation.

Using daily equity data, the notebook implements a fixed and expanding-window backtest in which portfolio weights are estimated on historical data and evaluated out-of-sample. The exercise highlights a key empirical result: while Markowitz optimisation can deliver strong in-sample performance, its out-of-sample results are often undermined by estimation error, particularly in expected returns, leaving the simple 1/n rule as a robust and competitive benchmark.

