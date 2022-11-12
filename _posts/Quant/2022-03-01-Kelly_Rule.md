---
title: Kelly Criterion (Multiple Assets)
author:
  name: unknown
  link: https://github.com/QuhiQuhihi
date: 2022-07-10 12:00:00 +0800
categories: [Asset Allocation]
tags: [investment, portfolio]
render_with_liquid: false
use_math: true
math: true
---

This post is about application of kelly criterion to financial investors. 

## What is Kelly Criterion

Kelly Criterion was developed to assist AT&T with its long-distance telephone signal noise issues and published as "A New Interpretation of Information Rate" in 1956. However, this idea could be easily adopted to gambling field. This idea enabled gamblers to maximize the size of their bankroll over the long term. Today, many people use it as a general money management system for gambling as well as investing.

Applying this therom to multi asset classes portfolio is equivalent to mean-variance portfolio optimization process. Thus maximum sharpe ratio solution (which requires inverse of correlation matrix) is needed. Detailed derivation can be found in below link.

[Mathematics for Kelly Criterim](https://medium.com/raposa-technologies/how-to-use-python-and-the-kelly-criterion-to-optimize-your-stock-portfolio-bb6e43df50c2)

## Simple Math
Kelly began by analyzing games with a binary win-lose outcome. The key variables are:
- b: The odds define the amount won for a \\$1 bet. Odds = 5/1 implies a \\$5 gain if the bet wins, plus recovery of the \\$1 capital.
- p: The probability defines the likelihood of a favorable outcome.
- f: The share of the current capital to bet.
- V: The value of the capital as a result of betting.

The Kelly rule aims to maximize the value's growth rate, G, of infinitely-repeated bets.

$ G=\lim_{N\rightarrow\infty}=\frac{1}{N}\log\frac{V_N}{V_0} $

If we apply this formula to multiple assets, the result is equivalent to the (potentially levered) maximum Sharpe ratio portfolio from the mean-variance optimization. The computation involves the dot product of the precision matrix, which is the inverse of the covariance matrix, and the return matrix:



## Data used (index)

#### I used index data from major index maker. All index are available in ETF(Exchange Traded Fund) form and can be traded easily.

```yaml
IVV	: iShares Core S&P 500 ETF                              
VEA	: Vanguard FTSE Developed Markets ETF                    
VWO	: Vanguard FTSE Emerging Markets ETF                    
TLT	: iShares 
TIP	: iShares TIPS Bond ETF                                 
LQD	: iShares iBoxx $ Investment Grade Corporate Bond ETF   
DBC	: Invesco DB Commodity Index Tracking Fund              
XAR	: SPDR® S&P® Aerospace & Defense ETF                    
XLB	: The Materials Select Sector SPDR® Fund                
XLE	: The Energy Select Sector SPDR® Fund                   
XLF	: The Financial Select Sector SPDR® Fund                
XLI	: The Industrial Select Sector SPDR® Fund               
XLK	: The Technology Select Sector SPDR® Fund               
XME	: SPDR® S&P® Metals & Mining ETF                        
XLP	: The Consumer Staples Select Sector SPDR® Fund         
XLY	: The Consumer Discretionary Select Sector SPDR® Fund   
XLU	: The Utilities Select Sector SPDR® Fund                
XLV	: The Health Care Select Sector SPDR® Fund              
XPH	: SPDR® S&P® Pharmaceuticals ETF                        
XBI	: SPDR® S&P® Biotech ETF                                
RWR	: SPDR® Dow Jones® REIT ETF                             
RWX	: SPDR® Dow Jones® International Real Estate ETF        

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
![HP](/assets/img/post_image/quant/kelly/kelly_plot.png)

```yaml
                             Strategy    Benchmark
--------------------------  ----------  -----------
Start Period                2013-06-30  2013-06-30
End Period                  2022-02-28  2022-02-28

Cumulative Return           143.67%     75.73%
CAGR﹪                      10.82%      6.72%

Sharpe                        4.24        4.83
Expected Shortfall (cVaR)    -4.7%       -2.44%
Max Drawdown                -21.52%     -6.86%
```

## Full Code
You can see full code in here
[CODE](https://github.com/QuhiQuhihi/project_quant/blob/master/asset_allocation/03_kelly_criteriom.ipynb)

## Let's code this idea
Kelly Criterion in multiple asset classes via python code.

```python
def kelly_allocation_mp(asset_ylds):
    cov = asset_ylds.cov()
    # np.linalg.inv(cov) returns inverse matrix in numpy matrix type
    precision_matrix = pd.DataFrame(np.linalg.inv(cov), index=asset_ylds.columns.tolist(), columns=asset_ylds.columns.tolist())

    kelly_allocation = asset_ylds.mean().dot(precision_matrix)

    # constraints : long position only, short signal will be map to zero
    kelly_allocation[kelly_allocation < 0] = 0
    # normalize target postion to 100
    kelly_allocation = kelly_allocation.div(kelly_allocation.sum())

    return kelly_allocation
```

## Full Result
``` yaml
                           Strategy    Benchmark
-------------------------  ----------  -----------
Start Period               2013-06-30  2013-06-30
End Period                 2022-02-28  2022-02-28
Risk-Free Rate             0.0%        0.0%
Time in Market             100.0%      100.0%

Cumulative Return          143.67%     75.73%
CAGR﹪                     10.82%      6.72%

Sharpe                     4.24        4.83
Smart Sharpe               4.12        4.69
Sortino                    6.41        8.36
Smart Sortino              6.23        8.12
Sortino/√2                 4.54        5.91
Smart Sortino/√2           4.4         5.74
Omega                      2.12        2.12

Max Drawdown               -21.52%     -6.86%
Longest DD Days            336         488
Volatility (ann.)          54.12%      28.94%
R^2                        0.43        0.43
Calmar                     0.5         0.98
Skew                       -1.08       -0.23
Kurtosis                   6.02        0.23

Expected Daily %           0.85%       0.54%
Expected Monthly %         0.85%       0.54%
Expected Yearly %          9.32%       5.8%
Kelly Criterion            31.85%      30.27%
Risk of Ruin               0.0%        0.0%
Daily Value-at-Risk        -4.7%       -2.44%
Expected Shortfall (cVaR)  -4.7%       -2.44%

Gain/Pain Ratio            1.12        1.14
Gain/Pain (1M)             1.12        1.14

Payoff Ratio               0.86        1.02
Profit Factor              2.12        2.14
Common Sense Ratio         2.67        2.6
CPC Index                  1.24        1.42
Tail Ratio                 1.26        1.21
Outlier Win Ratio          2.85        4.46
Outlier Loss Ratio         2.34        4.41

MTD                        -3.25%      -1.7%
3M                         -1.68%      0.82%
6M                         1.16%       1.95%
YTD                        -1.73%      -1.35%
1Y                         17.99%      5.15%
3Y (ann.)                  15.98%      12.26%
5Y (ann.)                  12.15%      9.41%
10Y (ann.)                 10.82%      6.72%
All-time (ann.)            10.82%      6.72%

Best Day                   10.63%      4.86%
Worst Day                  -15.97%     -4.97%
Best Month                 10.63%      4.86%
Worst Month                -15.97%     -4.97%
Best Year                  22.48%      16.01%
Worst Year                 -3.64%      -2.96%

Avg. Drawdown              -3.82%      -2.24%
Avg. Drawdown Days         79          106
Recovery Factor            6.68        11.03
Ulcer Index                0.05        0.02
Serenity Index             11.85       14.34

Avg. Up Month              2.82%       1.78%
Avg. Down Month            -3.29%      -1.74%
Win Days %                 68.57%      64.76%
Win Month %                68.57%      64.76%
Win Quarter %              72.22%      72.22%
Win Year %                 80.0%       60.0%
Beta                       1.23        -
Alpha                      0.58        -
```

![KELLY](/assets/img/post_image/quant/kelly/kelly_full_1.png)
![KELLY](/assets/img/post_image/quant/kelly/kelly_full_2.png)
![KELLY](/assets/img/post_image/quant/kelly/kelly_full_3.png)


## Strategy Evaluation

To be updated ...
