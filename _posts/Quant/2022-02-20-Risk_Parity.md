---
title: "Risk Parity: Equal Capital Is Not Equal Risk"
author: daham
date: 2022-06-30 12:00:00 +0800
last_modified_at: 2026-09-20 21:00:00 +0900
categories: [Asset Allocation]
tags: [asset allocation, risk parity, robust research]
render_with_liquid: false
use_math: true
math: true
---

An equally weighted portfolio can still be dominated by one source of risk. If equities fluctuate much more than short bonds, assigning each one third of the capital does little to equalize their influence on portfolio returns. Risk parity makes that imbalance explicit and turns the allocation problem into a risk-budget decision.

In the constructed example below, equities receive **33.33% of capital but 69.98% of portfolio variance contribution** under equal weight. Equal risk contribution brings each asset's risk share to one third, while assigning only 14.90% of capital to equities. The calculation explains the mechanism. Whether the resulting portfolio earns its complexity requires a separate historical comparison.

### Three different meanings of balance

Let $w$ contain portfolio weights and $\Sigma$ the covariance matrix of returns measured at a common horizon. Portfolio volatility is

$$\sigma_p=\sqrt{w^\top\Sigma w}.$$

The marginal change in volatility from increasing an asset's weight is $(\Sigma w)_i/\sigma_p$. Multiplying by its current weight gives its **component contribution**:

$$RC_i=w_i\frac{(\Sigma w)_i}{\sigma_p},\qquad \sum_i RC_i=\sigma_p.$$

Dividing by total volatility gives the risk share $q_i=w_i(\Sigma w)_i/(w^\top\Sigma w)$. Equal risk contribution, or ERC, targets $q_i=1/N$. This separates the marginal effect of a holding from its actual contribution at the chosen weight.

| Rule | What it equalizes | Information required |
|---|---|---|
| Equal weight | Capital weights | Eligible assets |
| Inverse volatility | Weight times individual volatility | Each asset's volatility |
| ERC | Contributions to total portfolio variance | Volatilities and correlations |

Inverse volatility uses $w_i\propto1/\sigma_i$. It agrees with ERC when assets are independent. General correlations change the contribution of a holding through its interaction with every other position.

### A controlled allocation example

The [worked notebook](https://github.com/QuhiQuhihi/project_Asset_Allocation/blob/main/topics/02-risk-parity/study.ipynb) supplies annual volatilities of 18%, 8% and 4% for fictional equity, long-bond and short-bond exposures. Equity–long-bond correlation is 0.20, equity–short-bond correlation is 0.10, and correlation between the bonds is 0.60. These are declared numerical assumptions, not current market estimates.

![Capital weights and variance contributions for three constructed allocation rules](/assets/post_image/renovated/allocation/risk-parity.png)

*Constructed covariance experiment. The left panel shows capital weights; the right panel shows shares of total portfolio variance. No historical performance is represented.*

| Exposure | Equal weight | Inverse volatility | ERC |
|---|---:|---:|---:|
| Equities | 33.33% | 12.90% | 14.90% |
| Long bonds | 33.33% | 29.03% | 27.34% |
| Short bonds | 33.33% | 58.06% | 57.76% |

Under inverse volatility, equity contributes 27.08% of variance, long bonds 37.50%, and short bonds 35.42%. The two bond exposures share more risk than their individual volatilities reveal. ERC adjusts their capital budgets until the contribution shares agree.

The maintained calculation solves a positive risk-budget objective,

$$\min_{x_i>0}\left(\tfrac12x^\top\Sigma x-\sum_i\log x_i\right),$$

then normalizes $x$ to fully invested weights. Its first-order condition makes $x_i(\Sigma x)_i$ equal across assets. The notebook checks the resulting budget, Euler decomposition and equal risk shares, and compares the independent-asset case against the inverse-volatility formula. Those checks establish numerical correctness under the specified covariance.

### Where robustness must enter

The risk budget is conditional on an estimate. A correlation change can destroy yesterday's balance, and low volatility does not imply low liquidity risk or limited tail losses. A large short-bond weight also changes expected return and interest-rate exposure. Without leverage, an ERC portfolio may simply take much less risk than an equally weighted comparator.

The project's [historical ETF comparison](https://github.com/QuhiQuhihi/project_Asset_Allocation/blob/main/docs/02-results.md) therefore includes equal weight scaled using prior risk information. Cash, holdings drift, execution delay and costs remain explicit. Reducing the control's exposure with zero assumed cash interest is economically different from owning a short-bond ETF, so even this comparison has limits.

For a future allocation decision, fix the covariance estimator and constraints before evaluation, perturb the inputs, inspect concentration, and retain unfavorable cost and timing results. The [estimation-risk chapter](https://github.com/QuhiQuhihi/project_Asset_Allocation/blob/main/topics/12-estimation-risk/README.md) demonstrates these perturbations. The [factor-risk chapter](https://github.com/QuhiQuhihi/project_Asset_Allocation/blob/main/topics/15-factor-risk-allocation/README.md) asks whether apparently balanced holdings still share the same economic exposure.

The methodological foundation is the [original ERC paper](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=1271972). Continue with the [detailed research note](https://github.com/QuhiQuhihi/project_Asset_Allocation/blob/main/topics/02-risk-parity/README.md), its [executed notebook](https://github.com/QuhiQuhihi/project_Asset_Allocation/blob/main/topics/02-risk-parity/study.ipynb), or the [allocation research collection](https://github.com/QuhiQuhihi/project_Asset_Allocation/blob/main/README.md).
