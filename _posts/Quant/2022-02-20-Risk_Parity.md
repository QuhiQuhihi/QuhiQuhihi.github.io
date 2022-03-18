---
title: Risk Parity strategy
author:
  name: unknown
  link: https://github.com/QuhiQuhihi
date: 2022-02-20 12:00:00 +0800
categories: [Asset Allocation]
tags: [quant, asset allocation, backtest, portfolio]
render_with_liquid: false
use_math: true
---

This post is about risk parity asset allocation strategy.

## What is Risk Parity strategy
Risk parity(Equally weighted risk contribution portfolio) is commonly used in asset allocation strategy.
Goal of this strategy is to allocate same amount of risk to each asset classes.

Consider portfolio of N assets. $(x_1,x_2,x_3,x_4, ... ,x_N)$, where weight of assets $x_i$ is $w_i$, and standard deviation of the asset $x_i$ is $std_i$.

Goal of this portfolio is not to find portfolio X which satisfy ($std_1$ = $std_2$ = ... = $std_N$). Covariance of each asset classes in portfolio should be considered. 

Let us note covariance matrix of the assets $X = (x_1, x_2, ..., x_N)$ by $\sum$  and risk contribution of each asset as $\sigma_i$. The volatility of the portfolio is defined as $\sigma(x) = \sum = \sqrt{w' \sum w}$  which is homogeneous of degree 1. Thus following Euler's theorm for homogeneous function.

$ \sigma_p(w) =\sum = \sqrt{w' \sum w}$ =  \sum{1=0}^N \sigma_i (w) $ \\
$ c(w) = \frac{w_i' \sigma w_i}{\sqrt{w'\sum w}} = \textsf{marginal risk contribution of asset i} $ \\
$ w_i = \frac{\sigma(w)^2}{(\sum w_i) \cdot N} $ \\

To sum up, we need to solve below formula to return realistic portfolio which contains constraints.

$ \underset{w}{\arg \min} \sum_{i=1}^N [\frac{\sqrt{w^T \Sigma w}}{N} - w_i \cdot c(w)_i]^2 $ \\



## How Risk Parity strategy works?

Let's check how this strategy works. 
First of all, I will compare how risk parity strategy will differ from traditional 60:40 strategy. Traditional 60:40 strategy derived from mean-variance optimization. 
While mean-variance optimization aims to maximize shapre ratio, Risk-parity optimization aims to build equal risk contribution.
See illustration below
![example](/assets/img/post_image/quant/risk_parity/risk_parity_example.png)


Benchmark for this investment strategy is all weather porfolio of Ray Dalio.
![bm](/assets/img/post_image/quant/risk_parity/all_weather.png)

Result of Risk Parity as follows, use previous 12 month data to generate portfolio.
Rebalancing at the end of each month. Data use index data of each representative asset classes.

## Data used (index)

#### I used index data from major index maker. All index are available in ETF(Exchange Traded Fund) form and can be traded easily.

```yaml
 Equity : MSCI world, MSCI Emerging
 Treasury : Bloomberg Barclays World AGG, Bloomberg Barclays EM AGG, Bloomberg Barclays US 1-3 Month Treasury, Bloomberg US 7-10 Treasury, Bloomberg US 20+ Treasury
 Credit :  Bloomberg Barclays US ABS, Bloomberg Barclays US High Yield, Bloomberg Barclays US Investment Grade, 
 TIPS : Bloomberg Barclays US Inflation Protected, Bloomberg Barclays US Inflation Protected 7+ 
 Alternative : Goldmansachs Commodity Index, S&P Metals and Mining Select Industry Index TR, S&P Oil Gas Exploration and Production Industry Index TR, S&P global infrastructure TR, MSCI World Real Estate
```


## Strategy Overview
![RP](/assets/img/post_image/quant/risk_parity/rp_summary.png)

``` yaml
                           Strategy    Benchmark
-------------------------  ----------  -----------
Start Period               2011-01-31  2011-01-31
End Period                 2021-06-30  2021-06-30

Cumulative Return          26.59%      120.12%
CAGR﹪                     2.29%       7.87%

Sharpe                     3.34        5.94
Max Drawdown               -5.0%       -6.86%
Expected Shortfall (cVaR)  -1.3%       -2.18%
```

## Full Code
You can see full code in here
[CODE](https://github.com/QuhiQuhihi/project_quant/blob/master/asset_allocation/01_risk_parity.ipynb)


## Let's code this idea
Risk parity strategy via python code.

```python
def RiskParity(covariance_matrix):

  # Calculate risk parity
  # :param covariance_matrix: covariance matrix of assets in universe
  # :return: [list] weight of risk parity investment strategy

  x0 = np.repeat(1/covariance_matrix.shape[1], covariance_matrix.shape[1])
  constraints = ({'type': 'eq', 'fun': SumConstraint},
                {'type': 'ineq', 'fun': LongOnly})
  options = {'ftol': 1e-20, 'maxiter': 2000}

  result = minimize(fun = RiskParityObjective,
                    args = (covariance_matrix),
                    x0 = x0,
                    method = 'SLSQP',
                    constraints = constraints,
                    options = options)
  return result.x

def RiskParityObjective(x, covariance_matrix) :
  # x means weight of portfolio
  variance = (x.T) @ (covariance_matrix) @ (x)
  sigma = np.sqrt(variance)
  mrc = 1/sigma * (covariance_matrix @ x)
  risk_contribution = x * mrc
  a = np.reshape(risk_contribution.to_numpy(), (len(risk_contribution), 1))

  # set marginal risk level of asset classes equal
  risk_diffs = a - a.T
  # np.ravel: convert n-dim to 1-dim
  sum_risk_diffs_squared = np.sum(np.square(np.ravel(risk_diffs)))
  return (sum_risk_diffs_squared)

# constraint 1 : sum of weight should be less than equal to 1
def SumConstraint(weight):
  return (weight.sum()-1.0)

# constraint 2 : long only portfolio should be consists of postive weight vectors
def LongOnly(weight):
  return(weight)

def RiskContribution(weight, covariance_matrix) :
  # to check whether given portfolio is equally distributed
  # :param weight: asset allocation weight of portfolio
  # :param covariance_matrix:
  # :return: risk contribution of each asset
  weight = np.array(weight)
  variance = np.dot(np.dot(weight.T, covariance_matrix) ,weight)
  sigma = np.sqrt(variance)
  mrc = 1/sigma * np.dot(covariance_matrix, weight)

  risk_contribution = weight * mrc
  risk_contribution = risk_contribution / risk_contribution.sum()
  return risk_contribution
```

## Full Result
```yaml
                           Strategy    Benchmark
-------------------------  ----------  -----------
Start Period               2011-01-31  2011-01-31
End Period                 2021-06-30  2021-06-30

Cumulative Return          26.59%      120.12%
CAGR﹪                     2.29%       7.87%

Sharpe                     3.34        5.94
Smart Sharpe               2.9         5.16
Sortino                    5.04        10.6
Smart Sortino              4.38        9.21
Sortino/√2                 3.56        7.5
Smart Sortino/√2           3.1         6.51
Omega                      1.83        1.83

Max Drawdown               -5.0%       -6.86%
Longest DD Days            731         488
Volatility (ann.)          14.42%      27.26%
R^2                        0.59        0.59
Calmar                     0.46        1.15
Skew                       -0.81       -0.31
Kurtosis                   5.79        0.57

Expected Daily %           0.19%       0.63%
Expected Monthly %         0.19%       0.63%
Expected Yearly %          2.17%       7.44%
Kelly Criterion            25.46%      40.56%
Risk of Ruin               0.0%        0.0%
Daily Value-at-Risk        -1.3%       -2.18%
Expected Shortfall (cVaR)  -1.3%       -2.18%

Gain/Pain Ratio            0.83        1.56
Gain/Pain (1M)             0.83        1.56

MTD                        0.67%       0.81%
3M                         -0.14%      1.06%
6M                         1.48%       6.24%
YTD                        0.51%       2.38%
1Y                         3.79%       13.13%
3Y (ann.)                  3.23%       10.97%
5Y (ann.)                  2.79%       8.53%
10Y (ann.)                 2.06%       7.54%
All-time (ann.)            2.29%       7.87%
```

![RP](/assets/img/post_image/quant/risk_parity/rp_full_1.png)
![RP](/assets/img/post_image/quant/risk_parity/rp_full_2.png)

## Strategy Evaluation
Evaluating investment strategy can be differ by investors and metrics.
For high frequency trader, absoulte return is sole metric to evaluate the strategy. For long term investors, however other than absolute return should be considered. Such as risk, turnover, diversification ,,, and so on. Thus, less profitability doesn't mean that the strategy is not good. 

Risk parity strategy show much more safer(risk averse) result. It is middele of traditional 60:40 and Bond investors. This is because most of risk contribution comes from equity investment. So, if risk contribution is equalized, bond exposure increase signigicantly and inevitably.

Interesting fact is that, it underperformed all weather portfolio. This underperformance contributes to risk parity strategy's extremely small exposure to equity. If investors prefer higher profit than fixed income investment, but want stationary investment return, risk parity can be a choice. By selecting risk parity strategy, investors can expect same return, same risk among asset classes. literally it looks great phrase, but it is very risk averse approach.

It is hard to predict the future performance of asset classes. Risk parity approach overcomes this shortcoming by building portfolios using only assets’ risk characteristics and correlation matrix. This idea give great relief to investors that they are suitably allocalte their wealth to major asset classes no matter how the asset classes perform.
