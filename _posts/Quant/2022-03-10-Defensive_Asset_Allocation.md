---
title: Defensive Asset Allocation with Relative Momentum
author:
  name: unknown
  link: https://github.com/QuhiQuhihi
date: 2022-03-10 12:00:00 +0800
categories: [Asset Allocation]
tags: [quant, asset allocation, backtest, portfolio]
render_with_liquid: false
use_math: true
mermaid: true
---

This post is about application of Relative(Cross-Sectional) Momentum in Defensive Asset Allocation to financial investors. 

## What is Momentum

Momentum is defined as product of the mass of a particle and its velocity. Momentum is a vector quantity; i.e., it has both magnitude and direction. 
From Newton’s second law it follows that, if a constant force acts on a particle for a given time, the product of force and the time interval (the impulse) is equal to the change in the momentum. Conversely, the momentum of a particle is a measure of the time required for a constant force to bring it to rest.
[Definition of Momentum](https://www.britannica.com/science/momentum)

In finance domain, momentum means price movement trend. Stock price literally do "work" by forces (probably by market participants). If forces act on stock price, it will move up or down. If stock price move(both with magnitude and quantity) by forces. So, Mid-term momentum means stock price movement trend for mid-term period and Long-term means stock price movement trend for long-term period. Direction can be either postive to negative.

## Define Momentum Score DAA

Rule is simple, DAA momentum score is time weighted momentum score. More weight to recent momentum and less weight to past. 

Interesting point is that, there is canaria asset which detect anormaly of market in advance.

Stock Price At Time(t) =  $ Y_t $ \\
12 Month Momentum= $\frac{Y_t}{Y_(t-12)} - 1$ \\
6 Month Momentum=  $\frac{Y_t}{Y_(t-6)} - 1$ \\
3 Month Momentum=  $\frac{Y_t}{Y_(t-3)} - 1$ \\
1 Month Momentum=  $\frac{Y_t}{Y_(t-1)} - 1$ 

$ M_i $ = Defensive Asset Allocation Momentum Score \\
$ M_i $ = 12x(1 Month Momentum) + 4x(3 Month Momentum) + 2x(6 Month Momentum) + 1x(12 Month Momentum) 

If Canaria asset shows negative momentum score, the strategy escape to safe heaven. Others build 1/N portfolio with risk assets. 

```yaml
# Ofensive Assets
IVV :  iShares Core S&P 500 ETF                              2000-05-15
QQQ :  Invesco QQQ trust Nasdaq 100                          1999-03-10
VEA :  Vanguard FTSE Developed Markets ETF                   2007-07-02
VWO :  Vanguard FTSE Emerging Markets ETF                    2005-03-04
TLT :  iShares 20+ Year Treasury Bond ETF                    2002-07-22
RWR :  SPDR® Dow Jones® REIT ETF                             2001-04-23
GSG :  iShares S&P GSCI Commodity-Indexed Trust              2006-07-10

# Defensive Assets
SHY :  iShares 1-3 Year Treasury Bond ETF                    2002-07-22
IEI :  iShares 3-7 Year Treasury Bond ETF                    2007-01-05

# Canaria Assets
VWO :  Vanguard FTSE Emerging Markets ETF                    2005-03-04
BND :  iShares 7-10 Year Treasury Bond ETF                   2002-07-22
```

#### Bechmark for this strategy is all weather 
```yaml
 Equity             : MSCI All Country 30%
 Long Fixed Income  : Bloomberg US 20+ Treasury 40%
 Mid  Fixed Income  : Bloomberg US 7-10 Treasury 15%
 Commodity 1        : Gold Index 7.5%
 Commodity 2        :  Goldmansachs Commodity Index 7.5%
```

## Strategy Overview _ DAA
![Sector](/assets/img/post_image/quant/momentum/DAA/momentum_1.PNG)

```yaml
                           Strategy    Benchmark
-------------------------  ----------  -----------
Start Period               2009-09-30  2009-09-30
End Period                 2022-03-31  2022-03-31

Cumulative Return          9.92%       185.66%
CAGR﹪                     0.76%       8.75%

Sharpe                     0.85        6.12
Expected Shortfall (cVaR)  -2.14%      -2.34%
Max Drawdown               -14.45%     -6.86%
```


## Full Code
You can see full code in here
[CODE](https://github.com/QuhiQuhihi/project_quant/blob/master/asset_allocation/07_defensive_asset_allocation_with_momentum_monthly.ipynb)

## Let's code this idea
Vigiliant Asset Allocation with relative Momentum Criterion in multiple asset classes via python code.

```python
def get_momentum(yld_df):
    """
    calculate momentum of sectors. using 12 month, 6 month, 3 month market price of asset
    input
    yiled_df : dataframe with weekly yield of asset classes

    returns : momentum in pandas dataframe format. momentum of each asset classes for give date
    """
    momentum = pd.DataFrame(columns = yld_df.columns, index = yld_df.index)

    for asset in  yld_df.columns:
        i = 0
        for date in yld_df.index:
            # data set consists of weekly data
            # 52 weeks per year = 12 month per year
            if i > 12 :
                # first 12 month data (52 data points) cannot be used since 12 month lagged returns is required
                current = yld_df[asset].iloc[i]
                before_1m = yld_df[asset].iloc[i-1]
                before_3m = yld_df[asset].iloc[i-3]
                before_6m = yld_df[asset].iloc[i-6]
                before_12m = yld_df[asset].iloc[i-12]

                momentum.loc[date, asset] = 12*(current/before_1m - 1) + 4*(current/before_3m - 1) \
                                            + 2*(current/before_6m - 1) + 1*(current/before_12m - 1)
            else:
                momentum.loc[date, asset] = 0
            i = i + 1

    # abnormal data processing
    momentum = momentum.replace([np.inf], 1000)
    momentum = momentum.replace([-np.inf], -1000)
    momentum = momentum.replace([np.nan], 0)
    return momentum

def select_sector(yld_df):
    """
    select top scored two offensive sectors if condition satisfied(canaria not dead).
    If failed to save canaria, escape to Defensive strategy.

    Criteria: if any of canaria asset classes show negative momentum score, escape to canaria assets.

    returns: selected_tickers in list format. list with top 5 momentum in given period`
    """
    momentum_df = get_momentum(yld_df)

    selected_momentum = pd.DataFrame(
        columns=['momentum_1','momentum_2'],
        index=momentum_df.index
    )
    selected_ticker = pd.DataFrame(
        columns=['momentum_1','momentum_2'],
        index=momentum_df.index
    )

    selectable_asset = [
    "IVV",  # iShares Core S&P 500 ETF                              2000-05-15
    "QQQ",  # Invesco QQQ trust Nasdaq 100                          1999-03-10
    "VEA",  # Vanguard FTSE Developed Markets ETF                   2007-07-02
    "VWO",  # Vanguard FTSE Emerging Markets ETF                    2005-03-04
    "TLT",  # iShares 20+ Year Treasury Bond ETF                    2002-07-22
    "RWR",  # SPDR® Dow Jones® REIT ETF                             2001-04-23
    "GSG",  # iShares S&P GSCI Commodity-Indexed Trust              2006-07-10
    ]
    print("emerging market signal : Total ",momentum_df.count()[0])
    print("postive signal : ", momentum_df[momentum_df['VWO'] >= 0]['VWO'].count())
    print("negative signal : ", momentum_df[momentum_df['VWO'] < 0]['VWO'].count())


    print("bond market signal : Total ",momentum_df.count()[0])
    print("postive signal : ", momentum_df[momentum_df['BND'] >= 0]['BND'].count())
    print("negative signal : ", momentum_df[momentum_df['BND'] < 0]['BND'].count())


    for date in momentum_df.index:
        emerging_momentum = momentum_df.loc[date,'VWO']
        bnd_momentum = momentum_df.loc[date,'BND']

        short_treasury_momentum = momentum_df.loc[date,'SHY']
        mid_treasury_momentum = momentum_df.loc[date,'IEI']

        sorted_momentum = momentum_df[selectable_asset].loc[date].sort_values(ascending=False)

        if emerging_momentum >= 0 and bnd_momentum >= 0:
            selected_momentum.loc[date,'momentum_1'] = sorted_momentum[0]
            selected_ticker.loc[date,'momentum_1'] = sorted_momentum.index[0]

            selected_momentum.loc[date,'momentum_2'] = sorted_momentum[1]
            selected_ticker.loc[date,'momentum_2'] = sorted_momentum.index[1]

        else:
            selected_momentum.loc[date,'momentum_1'] = short_treasury_momentum
            selected_ticker.loc[date,'momentum_1'] = 'SHY'

            selected_momentum.loc[date,'momentum_2'] = mid_treasury_momentum
            selected_ticker.loc[date,'momentum_2'] = 'IEI'

    return selected_ticker


def daa_momentum(yld_df):
    """
    returns : market portfolio in pandas dataframe format.
    """
    mom_ticker_df = select_sector(yld_df)
    mp_table = pd.DataFrame(columns=yld_df.columns, index=yld_df.index)
    for date in mom_ticker_df.index:
        selected = mom_ticker_df.loc[date].tolist()
        for sel in selected:
            mp_table.loc[date, sel] = 1/2
    mp_table = mp_table.fillna(0)

    return mp_table

def trim_data(yld_df, mp_table, benchmark_yield_df):
   """
    since momentum strategy uses 12 month lagged momentum, first 12 month data cannot be used
    return : yld_df, mp_table, bm_yld in dataframe format
    """
    yld_df = yld_df.iloc[12 + 1:]
    mp_table = mp_table.iloc[12 + 1:]
    benchmark_yield_df = benchmark_yield_df.iloc[12 + 1:]
    return yld_df, mp_table, benchmark_yield_df
```

## Full Result
```yaml
                           Strategy    Benchmark
-------------------------  ----------  -----------
Start Period               2009-09-30  2009-09-30
End Period                 2022-03-31  2022-03-31
Risk-Free Rate             0.0%        0.0%
Time in Market             100.0%      100.0%

Cumulative Return          9.92%       185.66%
CAGR﹪                     0.76%       8.75%

Sharpe                     0.85        6.12
Smart Sharpe               0.74        5.37
Sortino                    1.07        11.03
Smart Sortino              0.94        9.68
Sortino/√2                 0.75        7.8
Smart Sortino/√2           0.66        6.84
Omega                      1.25        1.25

Max Drawdown               -14.45%     -6.86%
Longest DD Days            3804        489
Volatility (ann.)          21.39%      29.44%
R^2                        0.02        0.02
Calmar                     0.05        1.28
Skew                       -2.7        -0.28
Kurtosis                   22.41       1.43

Expected Daily %           0.06%       0.7%
Expected Monthly %         0.06%       0.7%
Expected Yearly %          0.68%       7.79%
Kelly Criterion            8.84%       44.34%
Risk of Ruin               0.0%        0.0%
Daily Value-at-Risk        -2.14%      -2.34%
Expected Shortfall (cVaR)  -2.14%      -2.34%

Gain/Pain Ratio            0.25        1.67
Gain/Pain (1M)             0.25        1.67

Payoff Ratio               0.84        1.27
Profit Factor              1.25        2.67
Common Sense Ratio         1.83        3.69
CPC Index                  0.61        2.33
Tail Ratio                 1.47        1.38
Outlier Win Ratio          7.2         2.71
Outlier Loss Ratio         7.07        3.61

MTD                        0.04%       0.35%
3M                         -1.23%      6.13%
6M                         -2.52%      6.4%
YTD                        -1.41%      5.28%
1Y                         -2.59%      8.87%
3Y (ann.)                  2.22%       12.36%
5Y (ann.)                  0.74%       8.69%
10Y (ann.)                 0.31%       7.67%
All-time (ann.)            0.76%       8.75%

Best Day                   5.24%       7.04%
Worst Day                  -9.34%      -5.92%
Best Month                 5.24%       7.04%
Worst Month                -9.34%      -5.92%
Best Year                  5.8%        16.03%
Worst Year                 -4.85%      1.76%

Avg. Drawdown              -3.65%      -2.06%
Avg. Drawdown Days         827         103
Recovery Factor            0.69        27.05
Ulcer Index                0.08        0.02
Serenity Index             0.12        43.38

Avg. Up Month              0.7%        1.88%
Avg. Down Month            -0.82%      -1.48%
Win Days %                 58.28%      68.87%
Win Month %                58.28%      68.87%
Win Quarter %              64.71%      78.43%
Win Year %                 50.0%       100.0%
Beta                       0.1         -
Alpha                      0.0         -
```

![Sector](/assets/img/post_image/quant/momentum/DAA/momentum_1.PNG)
![Sector](/assets/img/post_image/quant/momentum/DAA/momentum_2.PNG)
![Sector](/assets/img/post_image/quant/momentum/DAA/momentum_3.PNG)

## Strategy Evaluation

As shown above, Defensive Asset Allocation strategy shows underperformance. Though original paper suggest overperformance in profitability and stability, my research shows contrasting results. This means that alpha decayed and thus no longer applicable in real world. The idea of time weighted momentum score and early warning system by canaria assets are great. But Regime of market changed significantly and this no longer works. 

Emerging market works as canaria in this strategy. If any canaria gives warning signal, the strategy escape from market. Emerging market showed significant underperformance in past 10 years. Rise of technology stock in developed market was signal for underperformace of emerging market equity. Some contend that this is due to entry barrier of technology sector, so rise of big tech in developed market seized more investors'attention. This phenomenon triggered escape signal too often. 

This means that emerging market movement triggered the strategy to escape from risky but opportunistic market, meaning that huge opportunity cost from rising developed equity market. Thus the strategy spend most of their time in save heaven(short term treasury). So, direct comparison to rebalanced all weather strategy is not adequate.

Below table summarize how many times canaria gave early alerts. If alerts activated, the strategy evacuate to safe universe.

| Monthly Signal | Emerging | Fixed_Income |
|:-------------|:------:|:----------:|
| Postive | 46 | 46 |
| Negative| 117 | 117 |
| Total | 163 | 163 |

Though above number looks same, emerging market stock and fixed income gave signal in different time. This is due to their negative correlation effects. When uncertainty in market rise, investors tend to prefer bond market compare to stock market. 46 times postive and 117 times negative is coincident. Below table which shows total offensive time and defensive time shows that signal worked in different time

Below table summarize how many times Early WARNING system alerts and strategy evacuated to safe universe.

| Monthly Signal | Offensive | Defensive | Total |
|:---------------|:---------:|:---------:|:-----:|
|Month|16|147|163|
|Percentage|10%|90%|100%|
