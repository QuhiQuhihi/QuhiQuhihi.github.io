---
title: Research
icon: fas fa-flask
order: 3.5
permalink: /research/
---

These seven research collections connect an economic question to a mathematical model,
an implementation, and evidence that can challenge the result. The articles explain the
ideas; the linked repositories contain detailed notes and executed notebooks.

Revised **22 September 2026**. Historical empirical studies and constructed numerical examples
are identified separately throughout the articles.

## Robust asset allocation

Portfolio construction depends on estimates of returns, covariance, and changing market
conditions. This collection asks what each allocation rule assumes, how estimation errors
affect its weights, and what remains after sensible benchmarks, timing, and costs.

| Explore an idea | What the article develops |
| --- | --- |
| [Mean–variance allocation](/posts/Mean-Varance/) | Expected-return uncertainty, constraints, and why an optimizer can amplify a weak forecast |
| [Risk parity](/posts/Risk_Parity/) | Marginal risk contributions, correlations, and the difference between equal risk and equal money |
| [Hierarchical risk parity](/posts/Hierachical_Risk_Parity/) | Correlation distances, clustering, recursive allocation, and the limits of a stable-looking hierarchy |
| [Kelly allocation](/posts/Kelly_Rule/) | Log growth, leverage, and sizing a position when the estimated edge is uncertain |
| [Maximum diversification](/posts/Maximum_Diversification/) | The diversification-ratio objective and its sensitivity to the estimated covariance matrix |
| [Sector momentum](/posts/Sector_Momentum/) | Relative ranks, decision timing, and the cost of switching between sectors |
| [Vigilant allocation](/posts/Vigiliant_Asset_Allocation/) | Breadth signals and how a rule moves between offensive and defensive assets |
| [Defensive allocation](/posts/Defensive_Asset_Allocation/) | A separate canary universe and the consequences of discrete protection rules |
{: .research-index-table }

The [full allocation collection](https://github.com/QuhiQuhihi/project_Asset_Allocation)
also introduces sector reversal, Black–Litterman views, expected shortfall, factor exposures,
liability-driven allocation, volatility targeting, and CPPI. Each has a detailed note and
notebook, alongside chapters on purged validation, covariance uncertainty, and multiple testing.

## QuantLib for the FICC desk

A price is meaningful only alongside its dates, cash flows, curves, conventions, and model
assumptions. These hands-on examples use explicitly illustrative inputs and independently
check the quantities returned by QuantLib.

| Explore a product or task | What the article develops |
| --- | --- |
| [European options](/posts/Black_Scholes_Merton/) | Black–Scholes–Merton pricing, parity, and volatility assumptions |
| [Yield curves](/posts/Yield_Curve/) | Discount factors, zero rates, forward rates, and interpolation |
| [Bond valuation](/posts/Treasury_Pricing/) | Dated cash flows, settlement, accrued interest, and clean versus dirty prices |
| [Duration and convexity](/posts/Duration_and_Convexity/) | Signed risk, bump definitions, and approximation error |
| [Swap curves](/posts/Swap_Curve/) | Instrument helpers, repricing, and discount versus projection curves |
| [Interest-rate swaps](/posts/Interest_Rate_Swap/) | Floating-rate fixings, leg valuation, par rates, and hedge residuals |
| [Cross-currency swaps](/posts/Cross_Currency_Swap/) | FX quotation, notional exchanges, collateral assumptions, and basis limitations |
| [Credit default swaps](/posts/Credit_Default_Swap/) | Hazard curves, recovery, and premium versus protection legs |
{: .research-index-table }

The [QuantLib collection](https://github.com/QuhiQuhihi/project_FICC_Quant)
extends these articles with curve inversion, forward-rate agreements, FX forwards,
relinkable market-data handles, caps and floors, and European swaptions. A connected
curve-to-hedge study brings valuation and risk together.

## Market regimes: description, detection, and forecasting

Historical state charts are useful descriptions, but decision value requires a forecast
made with the information available at the time. The current study evaluates filtered
states against simple volatility baselines and reports the negative finding directly.

- [GMM and HMM regimes](/posts/Regime_Model%28Markov-Model%29/): filtering, uncertainty,
  chronological variance forecasts, and the comparison with EWMA.
- [Gaussian segmentation](/posts/Regime_Model%28Gausian-Segmentation%29/): historical
  boundaries, delayed alarms, and why detection need not locate a break accurately.
- [Regime-switching option models](/posts/Regime_Model-Option%28Markov-Model%29/): a paper
  discussion separating physical state probabilities from risk-neutral pricing assumptions.

Continue with the [regime research notebook and evidence](https://github.com/QuhiQuhihi/regime_model).
The option-pricing extension remains a conceptual discussion, not an empirically validated
pricing result from the forecasting study.

## SVD and portfolio risk

[SVD and PCA for systematic investing](/posts/SVD_and_Portfolio/) develops the matrix
geometry, explains variance shares and residual risk, and tests a constrained portfolio
rule against sample covariance and Ledoit–Wolf shrinkage. The small, unstable sector
effect and the bond-heavy multi-asset allocation both matter when interpreting the result.

The [sector and multi-asset notebooks](https://github.com/QuhiQuhihi/SVD_Portfolio_Strategy)
include timing, costs, forecast comparisons, and all declared sensitivities.

## Sector factor exposures

The State Street sector study asks whether regularizing Fama–French and momentum
exposures makes them more useful for explaining another month's returns. Charts use
sector names, and the longer nine-sector test is kept separate from the eleven-sector map.

- [Method and implementation](/posts/Factors_Sector_ETF_Code/): excess-return accounting,
  SVD geometry, ridge, PCR, and chronological evaluation.
- [Economic interpretation and findings](/posts/Factors_Sector_ETF_Result/): the exposure
  map, a small reconstruction gain, intercept uncertainty, and changing sector boundaries.

The [five-chapter factor collection](https://github.com/QuhiQuhihi/Famma-French-Factors-with-Sector-ETF)
provides short research notes and executed notebooks. Its evaluation uses realized factors;
the result measures conditional reconstruction rather than a before-month return forecast.

## Equity information in corporate bonds

Adding issuer equity signals makes a bond factor portfolio less volatile, but reducing
the original bond exposure offers a simpler alternative. This study asks whether the
additional information earns its place after that comparison and implementation costs.

[Do equity signals earn their place in a corporate bond portfolio?](/posts/Corporate_Bond_Factor_Strategy/)
follows the sign conventions, the comparison with a scaled bond portfolio, and the
opposing conditional results for equity momentum and value. The historical evidence
does not establish an advantage for the fixed combination; it points to a narrower
question about issuer information, credit risk, and stale bond prices.

The [full research collection](https://github.com/QuhiQuhihi/Factor-Strategy-for-Corporate-Bond-)
and [executed notebook](https://github.com/QuhiQuhihi/Factor-Strategy-for-Corporate-Bond-/blob/main/study.ipynb)
retain the replication, uncertainty estimates, dependent source checks, and the
explicitly unrun bond-level follow-up.

## Adaptive RFQ pricing

An RFQ quote trades execution probability against margin and conditional hedge cost.
[Adaptive RFQ pricing: when forgetting matters more than exploration](/posts/Adaptive_RFQ_Pricing/)
connects a monotone demand model to an eight-parameter online correction, then
separates chronological fill prediction from controlled policy experiments.

In the synthetic Treasury study, forgetting old feedback reduces regret after
simulated demand shifts, while discounted greedy outperforms discounted sampling
in that setting. The historical holdout does not establish an adaptation benefit.
The article includes original figures and downloadable result tables, with no
claim of observed market profitability.
