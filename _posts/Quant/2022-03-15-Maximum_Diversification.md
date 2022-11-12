---
title: Maximum Diversification Portfolio Strategy
author:
  name: unknown
  link: https://github.com/QuhiQuhihi
date: 2022-10-03 12:00:00 +0800
categories: [Asset Allocation]
tags: [investment, portfolio]
render_with_liquid: false
mermaid: true
use_math: true
math: true
---
This post is about maximum diversification investment strategy. Unlike other post, this post includes visualization which helps you to understand theorm.

## What is Modern Portfolio Theory
Overall process of maximum diversification investment strategy is similar to mean variance portfolio(modern portfolio theory). Difference between maximum diversification portfolio and mean-variance optimized portfolio is object function.  

Modern Portfolio Theory (MPT) was designed by Harry Markowitz. Object function of this optimization process is maxmimze sharpe ratio. Sharpe ratio is calculated as return to standard deviation. So, by solving optimization problem, you can find optimal point where you can maximize return and reducing risk (maximum sharpe ratio point). 

On the other hand, Maximum diversification portfolio aims to minimize standard deviations. Optimization process is same as maximum sharpe portfolio. Only objective function is different. So, by solving this optimization problem, you can find most protective portfolio using certain investment universe. 


## simple and short math
Consider portfolio of N assets. $(x_1,x_2,x_3,x_4, ... ,x_N)$, where weight of assets $x_i$ is $w_i$, and standard deviation of the asset $x_i$ is $std_i$.

Let us note covariance matrix of the assets $X = (x_1, x_2, ..., x_N)$ by $\sum$  and risk contribution of each asset as $\sigma_i$. Expected return of portfolio = $\sum w_i x_i$ and variance of portfolio = $\sum \frac{x_i - \hat{x_i}}{n-1}$.

If we solve maximum diversification strategy, it will be 
minimize variance of portfolio = $\sum \frac{x_i - \hat{x_i}}{n-1}$.

This means that to solve this equation, inverse of covariance matrix is needed. This is problematic since correlation among asset classes sensitively react to small changes in volatility of stock markets. In addition, expected returns of each asset classes are in area of guess and depends on historical data. Unfortunately historical data might not repeat again in future or it never repeat again. So, methodology which sensitively reacts to historical data might not be efficient in real world.


## visualize these mathematical formula
![RANDOM](/assets/img/post_image/quant/max_variance/random_portfolio.png)

Scatter plots means randomly generated portfolio using market index data. Red line means efficient frontier, represent best portfolio choice. This means that portfolio gave the highest return given risk level. Maximum diversification portfolio is point where x-axis value is minimized.


## Data used and Benchmark for this strategy

I used index data from major index maker. Date from Jan 2006 to July 2022. All index are available in ETF(Exchange Traded Fund) form and can be traded easily. And benchmark for this strategy is 1/N portfolio.

```yaml
 US Equity Market : SPDR S&P 500 ETF Trust(SPY)
 Developed Market : iShares MSCI EAFE ETF(EAF)
 Emerging Market : iShares MSCI Emerging Markets ETF(EEM)
 Short Term Bond : iShares 1-3 Year Treasury Bond ETF(SHY)
 Long Term Bond : iShares 7-10 Year Treasury Bond ETF(IEF)
 Ultra Long Term Bond : Shares 20+ Year Treasury Bond ETF (TLT)
 IG Corporate Bond : iShares iBoxx Investment Grade Corporate Bond ETF(LQD)
```

## Backtesting Detail
Monthly rebalancing is assumed. To calculate portfolio, I used monthly data to derive covariance matrix(lookup period = 1 month).


## Strategy Overview
![MV](/assets/img/post_image/quant/max_variance/mv_1.png)


``` yaml
                           Strategy    Benchmark
-------------------------  ----------  -----------
Start Period               2006-01-03  2006-01-03
End Period                 2022-06-01  2022-06-01

Cumulative Return          85.11%      131.9%
CAGR﹪                     3.82%       5.26%
Sharpe                     0.68        0.60
```

## Full Code
You can see full code in here
[CODE](https://github.com/QuhiQuhihi/asset_allocation_strategy/blob/main/04_max_diversification.ipynb)

## Let's code this idea
Maximum diversification strategy via python code.

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
    bounds = ((0,1),(0,1),(0,1),(0,1),(0,1),(0,1),(0,1))
    initial = [1/7, 1/7,1/7,1/7,1/7,1/7,1/7] # 1/N weight for default

    optimizer = minimize(get_volatility, initial, method='SLSQP', bounds=bounds, constraints=cons)
    print(optimizer)
    print("Max Sharp portfolio consists of follows:")
    # in following orders
    # US Equity Market, Developed Market, Emerging Market, 
    # Short Term Bond, Long Term Bond, Ultra Long Term Bond, IG Corporate Bond 
    print(round(100*optimizer.x[0],2))

    return [
            round(optimizer.x[0],2),
            round(optimizer.x[1],2),
            round(optimizer.x[2],2),
            round(optimizer.x[3],2),
            round(optimizer.x[4],2),
            round(optimizer.x[5],2),
            round(optimizer.x[6],2)
            ]

def get_ret_vol_sharpe(weight):
    weight = np.array(weight)
    ret = np.sum(return_df.mean() * weight) * 252
    vol = np.sqrt(np.dot(weight.T, np.dot(return_df.cov()*252, weight)))
    # set risk free rate as 1.5%
    sharpe = (ret-0.015)/vol
    return {'return':ret, 'volatility':vol, 'sharpe':sharpe}

def get_volatility(weight):
    # to use convex optimization, change sign to minus to solve minimization problem.
    return get_ret_vol_sharpe(weight)['volatility'] * -1

# constraint : sum of weight should be less than equal to 1
def check_sum(weight):
    #return 0 if sum of the weights is 1
    return np.sum(weight)-1
```

## Full Result
```yaml
                           Strategy    Benchmark
-------------------------  ----------  -----------
Start Period               2006-01-03  2006-01-03
End Period                 2022-06-01  2022-06-01
Risk-Free Rate             0.0%        0.0%
Time in Market             100.0%      100.0%

Cumulative Return          85.11%      131.9%
CAGR﹪                     3.82%       5.26%

Sharpe                     0.68        0.6
Prob. Sharpe Ratio         99.67%      99.26%
Smart Sharpe               0.66        0.58
Sortino                    0.95        0.86
Smart Sortino              0.91        0.83
Sortino/√2                 0.67        0.61
Smart Sortino/√2           0.65        0.59
Omega                      1.14        1.14

Max Drawdown               -12.82%     -26.37%
Longest DD Days            440         831
Volatility (ann.)          5.78%       9.21%
R^2                        0.55        0.55
Information Ratio          -0.02       -0.02
Calmar                     0.3         0.2
Skew                       -0.49       -0.01
Kurtosis                   13.07       13.32

Expected Daily %           0.01%       0.02%
Expected Monthly %         0.31%       0.43%
Expected Yearly %          3.69%       5.07%
Kelly Criterion            4.77%       5.41%
Risk of Ruin               0.0%        0.0%
Daily Value-at-Risk        -0.58%      -0.93%
Expected Shortfall (cVaR)  -0.58%      -0.93%

Max Consecutive Wins       11          11
Max Consecutive Losses     9           10
Gain/Pain Ratio            0.14        0.12
Gain/Pain (1M)             0.8         0.75

Payoff Ratio               0.92        0.92
Profit Factor              1.14        1.12
Common Sense Ratio         1.12        1.07
CPC Index                  0.57        0.56
Tail Ratio                 0.99        0.95
Outlier Win Ratio          5.19        3.33
Outlier Loss Ratio         5.21        3.24

MTD                        -0.44%      -0.58%
3M                         -6.71%      -7.78%
6M                         -10.46%     -11.25%
YTD                        -10.8%      -12.23%
1Y                         -9.84%      -10.22%
3Y (ann.)                  1.8%        2.87%
5Y (ann.)                  2.86%       3.95%
10Y (ann.)                 3.98%       4.8%
All-time (ann.)            3.82%       5.26%

Best Day                   3.92%       6.84%
Worst Day                  -3.88%      -4.78%
Best Month                 5.7%        8.32%
Worst Month                -4.78%      -9.51%
Best Year                  12.8%       16.56%
Worst Year                 -10.8%      -12.76%

Avg. Drawdown              -0.92%      -1.11%
Avg. Drawdown Days         26          23
Recovery Factor            6.64        5.0
Ulcer Index                0.03        0.05
Serenity Index             2.1         1.47

Avg. Up Month              1.2%        1.79%
Avg. Down Month            -1.35%      -1.91%
Win Days %                 54.32%      54.73%
Win Month %                64.65%      64.14%
Win Quarter %              66.67%      68.18%
Win Year %                 76.47%      76.47%

Beta                       0.47        -
Alpha                      0.01        -
Correlation                74.19%      -
Treynor Ratio              182.81%     -
```
Distribution of the strategy
![MV](/assets/img/post_image/quant/max_variance/mv_2.png)
![MV](/assets/img/post_image/quant/max_variance/mv_3.png)

Weight flow of portfolio over investment horizon
![MV](/assets/img/post_image/quant/max_variance/weight_flow.png)

Monthly analysis of historical performance
![MV](/assets/img/post_image/quant/max_variance/mv_1.png)
![MV](/assets/img/post_image/quant/max_variance/mv_4.png)
![MV](/assets/img/post_image/quant/max_variance/mv_5.png)
![MV](/assets/img/post_image/quant/max_variance/mv_6.png)
![MV](/assets/img/post_image/quant/max_variance/mv_7.png)
![MV](/assets/img/post_image/quant/max_variance/mv_8.png)




## Strategy Evaluation
Evaluating investment strategy can be differ by investors and metrics. For high frequency trader, absoulte return is sole metric to evaluate the strategy. For long term investors, however other than absolute return should be considered. Such as risk, turnover, diversification ,,, and so on. Thus, less profitability doesn't mean that the strategy is not good.  

Modern portfolio theory suggested by Henry Markowitzcan be applied to build maximum diversification portfolio which returns more stable investment return. Since investment return and variance shows reverse relationship, minimizing variance might result in loose of potential investment returns.  

Optimization results showed better result than simple 1/N strategy in the aspect of sharpe ratio. Sharpe ratio is ratio of return to standard deviation. However, it showed less profitable returns than 1/N strategy.

So, careful consideration is required. Further calibration of parameter or constraints might enhance strategy such as lookup period, sup/inf constraints, turnover constraints. You can change parameter and find optimal solution. 

Unfortunately, mean-variance optimization method itself is sensitive to historical return and risk. So, calibration might not guarantee robustness of strategy.
