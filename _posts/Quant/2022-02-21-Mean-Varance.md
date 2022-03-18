---
title: Markowitz Mean Variance Strategy
author:
  name: unknown
  link: https://github.com/QuhiQuhihi
date: 2022-02-21 12:00:00 +0800
categories: [Asset Allocation]
tags: [quant, asset allocation, backtest, portfolio]
render_with_liquid: false
use_math: true
---

This post is about mean variance strategy(modern portfolio theory). Unlike other post, this post includes visualization which helps you to understand theorm.

## What is Modern Portfolio Theory

Modern Portfolio Theory (MPT) was designed by Harry Markowitz. Key idea of this theorm is that more diversification, less risk of investment. In addition to this simple idea, the Markowitz propose that you can find optimal points where you can maximize investment return given risk levels. So, by solving optimization problem, you can find optimal point where you can maximize return and reducing risk (maximum sharpe ratio point).


## simple and short math
Consider portfolio of N assets. $(x_1,x_2,x_3,x_4, ... ,x_N)$, where weight of assets $x_i$ is $w_i$, and standard deviation of the asset $x_i$ is $std_i$.

Let us note covariance matrix of the assets $X = (x_1, x_2, ..., x_N)$ by $\sum$  and risk contribution of each asset as $\sigma_i$. Expected return of portfolio = $\sum w_i x_i$ and variance of portfolio = $\sum \frac{x_i - \hat{x_i}}{n-1}$.

If we solve mean variance strategy via optimization problem, it will be 
$ \max_w[w^t \mu - \frac{\lambda}{2} w^t \sum w] $ \\
Solution for this problem is below :
$ w = (\lambda \sum)^(-1)\mu $ \\

This means that to solve this equation, inverse of covariance matrix is needed. 
This is problematic since correlation among asset classes sensitively react to small changes in volatility of stock markets. In addition, expected return of each asset classes is in area of guess and depends on historical which might not repeat again in future. So alternative approach such as Hierachical Risk Parity arise.

... To be updated

## visualize math formula
![RANDOM](/assets/img/post_image/quant/mean_variance/random_portfolio.png)

scatter plots means randomly generated portfolio using market index data. Red line means efficient frontier, represent best portfolio choice. This means that portfolio gave the highest return given risk level. 

## Data used (index)

#### I used index data from major index maker. All index are available in ETF(Exchange Traded Fund) form and can be traded easily.

```yaml
 Developed Market : MSCI world
 Emerging Market : MSCI Emerging
 Fixed Income : Bloomberg Barclays World AGG
 Alternative :  MSCI World Real Estate
```
#### Benchmark for this strategy is all weather porfolio of Ray Dalio. 

```yaml
 Equity: MSCI All Country 30%
 Long Fixed Income : Bloomberg US 20+ Treasury 40%
 Mid-Short Fixed Income : Bloomberg US 7-10 Treasury 15%
 Commodity : Gold Index 7.5%
 Commodity :  Goldmansachs Commodity Index 7.5%
```

## Strategy Overview
![MV](/assets/img/post_image/quant/mean_variance/mv_plot_rebal.png)

``` yaml
                           Strategy    Benchmark
-------------------------  ----------  -----------
Start Period               2010-01-13  2010-01-13
End Period                 2021-06-30  2021-06-30

Cumulative Return          92.63%      149.49%
CAGR﹪                     5.88%       8.3%

Sharpe                     1.02        1.0
Smart Sharpe               0.99        0.97
Expected Shortfall (cVaR)  -0.39%      -0.57%
```

## Full Code
You can see full code in here
[CODE](https://github.com/QuhiQuhihi/project_quant/blob/master/asset_allocation/00_mean_variance.ipynb)

## Let's code this idea
Mean variance strategy via python code.

```python
def generate_random_port():
    num_ports = 10000
    all_weights = np.zeros((num_ports, len(return_df.columns)))
    ret, vol, sharpe = np.zeros(num_ports), np.zeros(num_ports), np.zeros(num_ports)

    for x in range(num_ports):
        # 4 means number of asset in universe. 
        # If you want to add more vehicle to universe, change this number
        weights = np.array(np.random.random(4))
        weights = weights/np.sum(weights)
    return weights

def max_sharpe_portfolio(weight):
    cons = (
        {'type':'eq',
         'fun':check_sum}
    )
    bounds = ((0,1),(0,1),(0,1),(0,1))
    initial = [0.25, 0.25, 0.25, 0.25] # 1/N weight for default

    optimizer = minimize(neg_sharpe, initial, method='SLSQP', bounds = bounds, constraints = cons)
    print(optimizer)
    print("Max Sharp portfolio consists of follows:")
    print("{}% of dm stock".format(round(100*optimizer.x[0],2)))
    print("{}% of em stock".format(round(100*optimizer.x[1],2)))
    print("{}% of fixed income".format(round(100*optimizer.x[2],2)))
    print("{}% of real estate".format(round(100*optimizer.x[3],2)))

    return [
            round(optimizer.x[0],2),
            round(optimizer.x[1],2),
            round(optimizer.x[2],2),
            round(optimizer.x[3],2)
            ]
def get_ret_vol_sharpe(weight):
    weight = np.array(weight)
    ret = np.sum(return_df.mean() * weight) * 252
    vol = np.sqrt(np.dot(weight.T, np.dot(return_df.cov()*252, weight)))
    # set risk free rate as 1.5%
    sharpe = (ret-0.015)/vol
    return {'return':ret, 'volatility':vol, 'sharpe':sharpe}

def neg_sharpe(weight):
    # to use convex optimization, change sign to minus to solve minimization problem.
    return get_ret_vol_sharpe(weight)['sharpe'] * -1

# constraint : sum of weight should be less than equal to 1
def check_sum(weight):
    #return 0 if sum of the weights is 1
    return np.sum(weight)-1
```

## Full Result
```yaml
                           Strategy    Benchmark
-------------------------  ----------  -----------
Start Period               2010-01-13  2010-01-13
End Period                 2021-06-30  2021-06-30
Risk-Free Rate             0.0%        0.0%
Time in Market             72.0%       69.0%

Cumulative Return          92.63%      149.49%
CAGR﹪                     5.88%       8.3%

Sharpe                     1.02        1.0
Smart Sharpe               0.99        0.97
Sortino                    1.45        1.4
Smart Sortino              1.42        1.37
Sortino/√2                 1.03        0.99
Smart Sortino/√2           1.0         0.97
Omega                      1.24        1.24

Max Drawdown               -9.33%      -14.1%
Longest DD Days            385         485
Volatility (ann.)          3.95%       5.68%
R^2                        0.73        0.73
Calmar                     0.63        0.59
Skew                       -0.72       -0.95
Kurtosis                   15.52       14.38

Expected Daily %           0.02%       0.02%
Expected Monthly %         0.48%       0.66%
Expected Yearly %          5.62%       7.92%
Kelly Criterion            9.48%       9.1%
Risk of Ruin               0.0%        0.0%
Daily Value-at-Risk        -0.39%      -0.57%
Expected Shortfall (cVaR)  -0.39%      -0.57%

Gain/Pain Ratio            0.24        0.23
Gain/Pain (1M)             1.59        1.68

Payoff Ratio               0.97        0.93
Profit Factor              1.24        1.23
Common Sense Ratio         1.33        1.22
CPC Index                  0.67        0.64
Tail Ratio                 1.08        0.99
Outlier Win Ratio          6.47        4.57
Outlier Loss Ratio         4.12        2.68

MTD                        1.02%       2.25%
3M                         3.73%       6.82%
6M                         1.23%       2.94%
YTD                        0.96%       2.49%
1Y                         6.62%       11.01%
3Y (ann.)                  8.44%       11.29%
5Y (ann.)                  5.6%        8.1%
10Y (ann.)                 5.53%       7.89%
All-time (ann.)            5.88%       8.3%

Best Day                   2.22%       2.61%
Worst Day                  -2.97%      -4.66%
Best Month                 3.74%       4.66%
Worst Month                -2.51%      -3.66%
Best Year                  13.31%      18.55%
Worst Year                 -2.31%      -3.4%

Avg. Drawdown              -0.68%      -1.04%
Avg. Drawdown Days         20          22
Recovery Factor            9.93        10.6
Ulcer Index                0.02        0.03
Serenity Index             3.9         4.38

Avg. Up Month              1.26%       1.74%
Avg. Down Month            -1.01%      -1.43%
Win Days %                 55.47%      56.27%
Win Month %                66.67%      66.67%
Win Quarter %              80.43%      76.09%
Win Year %                 91.67%      83.33%
Beta                       0.59        -
Alpha                      0.01        -
```

![MV](/assets/img/post_image/quant/mean_variance/mv_full_1.png)
![MV](/assets/img/post_image/quant/mean_variance/mv_full_2.png)
![MV](/assets/img/post_image/quant/mean_variance/mv_full_3.png)
![MV](/assets/img/post_image/quant/mean_variance/mv_full_4.png)



## Strategy Evaluation
Evaluating investment strategy can be differ by investors and metrics.
For high frequency trader, absoulte return is sole metric to evaluate the strategy. For long term investors, however other than absolute return should be considered. Such as risk, turnover, diversification ,,, and so on. Thus, less profitability doesn't mean that the strategy is not good. 

Markowitz Mean Variance Portfolio shows more stable investment return. However, investment returns heavily relies on return of equity. To sum up, fixed income investment portion offers stability to portfolio and equity investment offers profitability to portfolio.