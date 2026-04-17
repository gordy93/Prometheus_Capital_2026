# Prometheus_Capital_2026
PROJECT OVERVIEW

This project builds a structured signal evaluation framework for systematic trading research. The notebook is designed to test whether a signal remains credible after transaction costs, repeated out-of-sample validation, overfitting checks, multiple-testing correction and risk-based diagnostics.

The core objective is to identify signals that are not only profitable in a static backtest, but also robust under realistic trading frictions, stable across time, resilient to changing validation splits and less likely to be false discoveries or backtest artefacts.

The framework combines signal generation, train/test backtesting, walk-forward analysis, combinatorial cross-validation, probability of backtest overfitting, false discovery rate control, downside-risk diagnostics and a final gated research leaderboard. It is therefore best understood as a research-quality filter for systematic signals rather than a simple performance report.

---

WHAT THE NOTEBOOK DOES

1. Loads market data and prepares a clean train/test split.
2. Generates a large library of systematic trading signals from multiple signal families.
3. Converts signals into tradeable return streams using realistic implementation assumptions.
4. Enforces a lookahead bias check.
5. Computes baseline train and test metrics for every signal.
6. Evaluates signal stability through rolling walk-forward testing.
7. Uses combinatorial purging and embargo cross-validation to reduce time-series leakage.
8. Estimates the probability that an apparently strong signal is actually overfit.
9. Applies false discovery rate control to reduce the chance of selecting false positives.
10. Measures downside risk using expected shortfall and drawdown-duration analysis.
11. Produces a final weighted scorecard and deployment gate to classify signals as Deploy, Monitor or Research.

---

DATA AND SAMPLE DESIGN

The notebook uses a market dataset loaded through an external signal module and then defines a fixed historical split:

Training period:
`2004-11-18 to 2018-12-31`

Test period:
`2019-01-01 to 2023-12-31`

Daily returns are computed from closing prices. The framework then evaluates every signal on the same underlying return series so that signal comparisons are consistent.

_Important implementation note:
Signal generation itself is done by the independent AI Quant Researcher. This notebook assumes that data loading and raw feature engineering already exist in cycle_1_signals. The notebook’s main contribution is therefore the evaluation, validation and signal-selection framework rather than raw data collection._

SIGNAL FAMILIES 

The framework evaluates signals across multiple systematic families.

This design allows the framework to compare different signal archetypes in a common evaluation environment rather than focusing on only one market hypothesis.

---

TRADING AND BACKTEST ASSUMPTIONS

The notebook converts a signal into realised strategy returns using a simple and transparent implementation rule.

Signal handling:

* Missing signals are filled with zero.
* Signals are clipped to the range [-1, 1].
* This means the framework supports flat, long, short and partially scaled positions.

Return construction:

* Gross strategy return is signal times market return.
* Net strategy return is gross return minus implementation costs.

Trading frictions:

* Transaction cost = 5 basis points
* Slippage = 2 basis points
* Total trading friction = 7 basis points times turnover

Turnover:

* Turnover is measured as the absolute change in position from one period to the next.

This setup is intentionally simple but realistic enough to penalise unstable or high-churn signals.

CORE METRICS COMPUTED

For each signal, the framework computes both training and test metrics.

Main performance metrics:

* Net Sharpe ratio
* Newey-West adjusted mean t-statistic
* Turnover
* Expected shortfall at the 5 percent tail
* Deflated Sharpe Ratio

Risk metrics:

* Maximum drawdown
* Drawdown duration distribution
* Expected shortfall curves across several tail probabilities

Statistical quality metrics:

* p-values for mean returns
* Deflated Sharpe Ratio to account for non-normality and multiple testing concerns
* False discovery rate significance flag
* Probability of backtest overfitting
* Cross-validated out-of-sample Sharpe measures

The idea is that a signal should not be promoted on Sharpe alone. It should also survive robustness, significance and downside-risk checks.

---

1. LOOKAHEAD BIAS CHECK

Before robustness rankings and checks, the notebook runs an explicit leakage screening on the signal universe.

Per signal the notebook evaluates:

* NaN coverage quality.
* Same-day exposure leakage, via correlation between signal and same-day returns.
* Execution-lag realism, via a Sharpe gap test comparing:
  * `Sharpe_t`: unrealistic same-day execution (`sig * ret`)
  * `Sharpe_t+1`: lagged execution (`sig.shift(1) * ret`)
  * `SharpeGap_t_minus_t+1`: uplift that may indicate lookahead contamination if too large.

The notebook then only keeps the signals that have passed this checkpoint for downstream evaluation.

2. BASELINE TRAIN/TEST EVALUATION

The second stage of the framework computes standard train and test metrics for every signal.

This stage answers the basic questions:

* Did the signal perform well in-sample?
* Did it retain performance out-of-sample?
* How much turnover does it require?
* What does its downside look like?
* Is the Sharpe ratio still meaningful after accounting for repeated backtests?

The notebook already includes implementation-cost-adjusted returns at this stage, so headline performance is not based on frictionless backtests.

3. WALK-FORWARD OPTIMISATION

The notebook then performs rolling walk-forward analysis.

Design:

* 5 years of training data
* 1 year of test data
* Window then rolls forward and repeats

For each split, the notebook measures:

* In-sample Sharpe
* Out-of-sample Sharpe
* Walk-Forward Efficiency

Walk-Forward Efficiency is conceptually:
`out-of-sample Sharpe divided by in-sample Sharpe`

Why this matters:
A strong signal should not only look good on one fixed test period. It should also show repeatable performance across multiple rolling historical regimes. Signals whose out-of-sample Sharpe collapses relative to in-sample Sharpe are treated with greater scepticism.

COMBINATORIAL PURGING AND EMBARGO CROSS-VALIDATION

A major feature of the notebook is the use of combinatorial purging and embargo cross-validation, following López de Prado (2018).

Configuration:

* 10 folds
* 2 test blocks per split
* 10-day embargo and purging
* minimum observation requirement per split

Why this matters:
Ordinary cross-validation is often unreliable for time-series trading strategies because nearby observations are dependent. If train and test windows are too close, the model can indirectly “see” future information. Purging and embargo reduce this leakage by removing observations around the test blocks from the training sample.

Outputs:

* Mean test Sharpe across combinatorial splits
* Median test Sharpe across splits
* Share of splits with positive out-of-sample Sharpe

This step gives a much more demanding picture of whether a signal is robust under alternative historical partitions.

4. PROBABILITY OF BACKTEST OVERFITTING

The framework also estimates the Probability of Backtest Overfitting (Bailey et al., 2015) using combinatorial symmetric cross-validation.

Concept:
For many balanced train/test partitions, the notebook selects the best-performing signal in-sample and then checks how that winner ranks out-of-sample. If the “winner” often performs poorly out-of-sample, this suggests the research process is overfitting noise.

Main outputs:

* Signal-level PBO
* Median rank ratio
* Performance degradation from in-sample to out-of-sample

Interpretation:

* Higher PBO means the signal is more likely to be a backtest artefact.
* Large degradation means the signal’s apparent edge weakens materially when moved out-of-sample.
* Better signals should rank well out-of-sample even when selected through many alternative partitions.

This section explicitly tests whether a signal survives the research-selection process itself.

5. FALSE DISCOVERY RATE CONTROL

Because many signals are tested simultaneously, the notebook applies Benjamini-Hochberg False Discovery Rate (Benjamini & Hochberg, 1995) control to test-period p-values.

Why this matters:
If many candidate signals are tested, some will appear significant purely by chance. False discovery rate control helps reduce the number of false positives among the signals that are selected.

Output:

* Raw p-value
* BH-adjusted p-value
* Boolean significance flag

Interpretation:
A signal that fails this step may still be interesting for further research, but it should not be treated as equally credible as one that remains significant after multiple-testing correction.

6. RISK PROFILE ANALYSIS

The notebook includes two additional risk diagnostics.

1. Expected Shortfall Curves
   For each signal, expected shortfall is calculated across multiple tail levels:
   1 percent, 2.5 percent, 5 percent, and 10 percent

This shows how the left tail behaves as the confidence level changes, which is often more informative than looking at only one fixed drawdown number.

2. Drawdown Duration Distribution
   The notebook measures how long each signal tends to remain underwater.

Why this matters:
Two signals with similar Sharpe ratios can feel very different in practice if one suffers long recovery periods and the other rebounds quickly. This is especially relevant for portfolio inclusion and real-world deployability.

7. FINAL RESEARCH QUALITY SCORECARD

After computing all diagnostics, the notebook builds a final signal leaderboard.

The scorecard includes:

* Test Sharpe
* Test Deflated Sharpe Ratio
* Walk-Forward Efficiency
* CPCV median test Sharpe
* Signal PBO
* Median rank ratio
* Performance degradation
* BH significance
* Maximum drawdown duration
* Expected shortfall

Scoring method:
Each metric is converted into a normalised sub-score on a 0 to 100 scale. Extreme values are winsorised before scaling so that one outlier does not dominate the ranking. Metrics where lower values are better, such as PBO or expected shortfall, are inverted appropriately.

The notebook then applies a weighted composite score. Higher weight is assigned to:

* Test Sharpe
* Test Deflated Sharpe Ratio
* CPCV median test Sharpe
* Walk-Forward Efficiency

Other important penalties and filters include:

* PBO
* degradation
* drawdown duration
* expected shortfall
* lack of false discovery significance

DEPLOYMENT GATES

The final ranking is not based on score alone. The notebook adds hard gates to prevent fragile signals from being promoted too easily.

A signal must pass:

* Test Sharpe >= 0.5
* CPCV median test Sharpe >= 0.0
* Test Deflated Sharpe Ratio >= 0.8
* Benjamini-Hochberg significance = True

Signals are then labelled as:

* Deploy
* Monitor
* Research

Interpretation:
Deploy means the signal passed the hard gates and also ranks highly on the total score.
Monitor means the signal is promising but not strong enough for immediate deployment.
Research means the signal remains exploratory or fails to meet the standard required for production consideration.

This separation is important because it creates a disciplined bridge between research and implementation.

---

WHY THIS FRAMEWORK IS USEFUL

This notebook is useful because it treats signal research as a full selection problem rather than a simple optimisation problem.

It helps answer:

* Is the signal profitable after costs?
* Is the signal stable over time?
* Does it survive different historical partitions?
* Is it likely to be overfit?
* Does it remain significant after multiple-testing adjustment?
* What does the left tail look like?
* Is the signal strong enough for deployment, or does it belong in further research?

In practice, this makes the framework suitable for:

* systematic signal triage
* research pipeline filtering
* pre-deployment validation
* identifying fragile backtests
* building a more disciplined quant research process

It also assumes that:

* raw data is already available
* the signal library has already been implemented
* all signals can be expressed as tradeable time series
* the user accepts a relatively simple transaction cost model

So this notebook should be viewed as the validation layer inside a broader quant research pipeline.

KEY IMPLEMENTATION DETAILS

Programming stack:

* Python
* NumPy
* pandas
* matplotlib
* seaborn
* SciPy

Key user-adjustable parameters:

* train and test dates
* transaction costs and slippage
* Newey-West lag length
* expected shortfall alpha levels
* number of CPCV folds
* number of CPCV test blocks
* embargo and purging length
* walk-forward train/test window lengths
* gating thresholds
* false discovery rate level

This makes the framework easy to adapt to different markets, signal universes and research standards.

OUTPUTS

The notebook produces:

* train and test metric tables for all signals
* walk-forward diagnostic tables
* CPCV out-of-sample robustness tables
* CSCV and PBO statistics
* FDR significance table
* expected shortfall charts by signal family
* drawdown duration distributions by signal family
* final leaderboard with total score, deployment gates, and final label

The final output is intended to support a research decision:
Which signals are robust enough to deploy, which deserve monitoring and which should remain in research.

PAPERS AND SOURCES REFERENCED

Bailey, D. H., and López de Prado, M. (2014). THE DEFLATED SHARPE RATIO: CORRECTING FOR SELECTION BIAS, BACKTEST OVERFITTING AND NON-NORMALITY 

Bailey, D. H., Borwein, J., López de Prado, M., and Zhu, Q. J. (2015). THE PROBABILITY OF BACKTEST OVERFITTING.

Benjamini, Y., and Hochberg, Y. (1995). Controlling the False Discovery Rate: A Practical and Powerful Approach to Multiple Testing.

López de Prado, M. (2018). Advances in Financial Machine Learning.
