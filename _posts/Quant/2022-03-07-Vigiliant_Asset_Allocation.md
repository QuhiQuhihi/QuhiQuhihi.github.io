---
title: Vigiliant Asset Allocation with Relative Momentum
author:
  name: unknown
  link: https://github.com/QuhiQuhihi
date: 2022-10-01 12:00:00 +0800
categories: [Asset Allocation]
tags: [investment, portfolio]
render_with_liquid: false
mermaid: true
use_math: true
math: true
---

This post is about application of Relative(Cross-Sectional) Momentum in Vigiliant Asset Allocation to financial investors. 

## What is Momentum

Momentum is defined as product of the mass of a particle and its velocity. Momentum is a vector quantity; i.e., it has both magnitude and direction. 
From Newton’s second law it follows that, if a constant force acts on a particle for a given time, the product of force and the time interval (the impulse) is equal to the change in the momentum. Conversely, the momentum of a particle is a measure of the time required for a constant force to bring it to rest.
[Definition of Momentum](https://www.britannica.com/science/momentum)

In finance domain, momentum means price movement trend. Stock price literally do "work" by forces (probably by market participants). If forces act on stock price, it will move up or down. If stock price move(both with magnitude and quantity) by forces. So, Mid-term momentum means stock price movement trend for mid-term period and Long-term means stock price movement trend for long-term period. Direction can be either postive to negative.

## Define Momentum Score VAA

Rule is simple, VAA momentum score is time weighted momentum score. More weight to recent momentum and less weight to past.

Stock Price At Time(t) =  $ Y_t $   
12 Month Momentum= $\frac{Y_t}{Y_(t-12)} - 1$    
6 Month Momentum=  $\frac{Y_t}{Y_(t-6)} - 1$     
3 Month Momentum=  $\frac{Y_t}{Y_(t-3)} - 1$     
1 Month Momentum=  $\frac{Y_t}{Y_(t-1)} - 1$     

$ M_i $ = Vigiliant Asset Allocation Momentum Score     
$ M_i $ = 12x(1 Month Momentum) + 4x(3 Month Momentum) + 2x(6 Month Momentum) + 1x(12 Month Momentum)    

If VAA Momentum scores of all offensive asset classes are postive, build portfolio with offensive asset classes. If any of these score shows negative score, escape to Defensive Assets.

## Who invented this idea?
Prof. Wouter J. Keller invented this strategy at 2017. You can see original paper in SSRN. this is revised version of VAA strategy(2017).   
You can download orignal paper in below url.   
[paper](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3002624)


#### I used index data from major index maker. All index are available in ETF(Exchange Traded Fund) form and can be traded easily.

```yaml
# Offensive Assets
IVV : iShares Core S&P 500 ETF                              2000-05-15
VEA : Vanguard FTSE Developed Markets ETF                   2007-07-02
VWO : Vanguard FTSE Emerging Markets ETF                    2005-03-04
LQD : iShares iBoxx $ Investment Grade Corporate Bond ETF   2002-07-22

# Defensive Assets
SHY : iShares 1-3 Year Treasury Bond ETF                    2002-07-22
IEI : iShares 3-7 Year Treasury Bond ETF                    2007-01-05
BND : iShares 7-10 Year Treasury Bond ETF                   2002-07-22
TIP : iShares TIPS Bond ETF                                 2003-12-04
```

## Strategy Overview _ VAA
![VAA](/assets/img/post_image/quant/momentum/VAA/momentum_0.PNG)  

```yaml
                           Strategy    Benchmark
-------------------------  ----------  -----------
Start Period               2013-01-31  2013-01-31
End Period                 2022-03-31  2022-03-31

Cumulative Return          17.84%      88.37%
CAGR﹪                     1.81%       7.15%

Sharpe                     3.18        5.3
Expected Shortfall (cVaR)  -1.09%      -2.3%
Max Drawdown               -4.33%      -6.86%
```


## Full Code
You can see full code in here
[CODE](https://github.com/QuhiQuhihi/project_quant/blob/master/asset_allocation/06_vigilant_asset_allocation_with_momentum.ipynb)  

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
            # data set consists of weekly data, 52 weeks per year = 12 month per year
            if i > windows :
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
    select top 4 offensive sectors if condition satisfied.
    If failed to satisfy offensive condition, escape to Defensive strategy.

    Criteria: if any of offensive asset classes show negative momentum score, escape to defensive assets.

    returns: selected_tickers in list format. list with top 5 momentum in given period`
    """
    momentum_df = get_momentum(yld_df)

    selected_momentum = pd.DataFrame(
        columns=['momentum_1','momentum_2','momentum_3','momentum_4'],
        index=momentum_df.index
    )
    selected_ticker = pd.DataFrame(
        columns=['momentum_1','momentum_2','momentum_3','momentum_4'],
        index=momentum_df.index
    )

    print("S&P500 market signal : Total ",momentum_df.count()[0])
    print("postive signal : ", momentum_df[momentum_df['IVV'] >= 0]['IVV'].count())
    print("negative signal : ", momentum_df[momentum_df['IVV'] < 0]['IVV'].count())


    print("Developed market signal : Total ",momentum_df.count()[0])
    print("postive signal : ", momentum_df[momentum_df['VEA'] >= 0]['VEA'].count())
    print("negative signal : ", momentum_df[momentum_df['VEA'] < 0]['VEA'].count())

    print("Emerging market signal : Total ",momentum_df.count()[0])
    print("postive signal : ", momentum_df[momentum_df['VWO'] >= 0]['VWO'].count())
    print("negative signal : ", momentum_df[momentum_df['VWO'] < 0]['VWO'].count())

    print("Corporate bond market signal : Total ",momentum_df.count()[0])
    print("postive signal : ", momentum_df[momentum_df['LQD'] >= 0]['LQD'].count())
    print("negative signal : ", momentum_df[momentum_df['LQD'] < 0]['LQD'].count())

    for date in momentum_df.index:
        snp_momentum = momentum_df.loc[date,'IVV']
        dev_momentum = momentum_df.loc[date,'VEA']
        eme_momentum = momentum_df.loc[date,'VWO']
        bnd_momentum = momentum_df.loc[date,'LQD']

        short_momentum = momentum_df.loc[date,'SHY']
        mid_momentum = momentum_df.loc[date,'IEI']
        long_momentum = momentum_df.loc[date,'BND']
        tip_momentum = momentum_df.loc[date,'TIP']

        if snp_momentum >= 0 and dev_momentum >= 0 and eme_momentum >= 0 and bnd_momentum >= 0:
            selected_momentum.loc[date,'momentum_1'] = snp_momentum
            selected_ticker.loc[date,'momentum_1'] = 'IVV'

            selected_momentum.loc[date,'momentum_2'] = dev_momentum
            selected_ticker.loc[date,'momentum_2'] = 'VEA'

            selected_momentum.loc[date,'momentum_3'] = eme_momentum
            selected_ticker.loc[date,'momentum_3'] = 'VWO'

            selected_momentum.loc[date,'momentum_4'] = bnd_momentum
            selected_ticker.loc[date,'momentum_4'] = 'LQD'

        else:
            selected_momentum.loc[date,'momentum_1'] = short_momentum
            selected_ticker.loc[date,'momentum_1'] = 'SHY'

            selected_momentum.loc[date,'momentum_2'] = mid_momentum
            selected_ticker.loc[date,'momentum_2'] = 'IEI'

            selected_momentum.loc[date,'momentum_3'] = long_momentum
            selected_ticker.loc[date,'momentum_3'] = 'BND'

            selected_momentum.loc[date,'momentum_4'] = tip_momentum
            selected_ticker.loc[date,'momentum_4'] = 'TIP'

    return selected_ticker


def vaa_momentum(yld_df):
    """
    returns : market portfolio in pandas dataframe format.
    """
    mom_ticker_df = select_sector(yld_df)
    mp_table = pd.DataFrame(columns=yld_df.columns, index=yld_df.index)
    for date in mom_ticker_df.index:
        selected = mom_ticker_df.loc[date].tolist()
        for sel in selected:
            mp_table.loc[date, sel] = 1/4
    mp_table = mp_table.fillna(0)
    return mp_table

def trim_data(yld_df, mp_table, benchmark_yield_df):
   """
    since momentum strategy uses 12 month lagged momentum, first 12 month data cannot be used
    return : yld_df, mp_table, bm_yld in dataframe format
    """
    yld_df = yld_df.iloc[windows + 1:]
    mp_table = mp_table.iloc[windows + 1:]
    benchmark_yield_df = benchmark_yield_df.iloc[windows + 1:]
    return yld_df, mp_table, benchmark_yield_df
```

## Full Result
```yaml
                           Strategy    Benchmark
-------------------------  ----------  -----------
Start Period               2013-01-31  2013-01-31
End Period                 2022-03-31  2022-03-31
Risk-Free Rate             0.0%        0.0%
Time in Market             100.0%      100.0%

Cumulative Return          17.84%      88.37%
CAGR﹪                     1.81%       7.15%

Sharpe                     3.18        5.3
Smart Sharpe               3.1         5.17
Sortino                    5.26        9.23
Smart Sortino              5.13        8.99
Sortino/√2                 3.72        6.52
Smart Sortino/√2           3.62        6.36
Omega                      1.69        1.69

Max Drawdown               -4.33%      -6.86%
Longest DD Days            883         489
Volatility (ann.)          11.94%      27.91%
R^2                        0.0         0.0
Calmar                     0.42        1.04
Skew                       -0.12       -0.28
Kurtosis                   0.71        0.52

Expected Daily %           0.15%       0.57%
Expected Monthly %         0.15%       0.57%
Expected Yearly %          1.66%       6.54%
Kelly Criterion            32.52%      35.8%
Risk of Ruin               0.0%        0.0%
Daily Value-at-Risk        -1.09%      -2.3%
Expected Shortfall (cVaR)  -1.09%      -2.3%

Gain/Pain Ratio            0.69        1.32
Gain/Pain (1M)             0.69        1.32

Payoff Ratio               1.59        1.02
Profit Factor              1.69        2.32
Common Sense Ratio         2.3         2.9
CPC Index                  1.58        1.6
Tail Ratio                 1.36        1.25
Outlier Win Ratio          5.2         2.14
Outlier Loss Ratio         4.86        1.86

MTD                        0.43%       0.35%
3M                         -1.01%      6.13%
6M                         -2.08%      6.4%
YTD                        -1.41%      5.28%
1Y                         -0.99%      8.87%
3Y (ann.)                  4.01%       12.36%
5Y (ann.)                  2.92%       8.69%
10Y (ann.)                 1.81%       7.15%
All-time (ann.)            1.81%       7.15%

Best Day                   2.05%       4.86%
Worst Day                  -2.06%      -4.97%
Best Month                 2.05%       4.86%
Worst Month                -2.06%      -4.97%
Best Year                  7.56%       16.03%
Worst Year                 -3.08%      2.99%

Avg. Drawdown              -1.5%       -2.51%
Avg. Drawdown Days         262         130
Recovery Factor            4.12        12.87
Ulcer Index                0.01        0.02
Serenity Index             3.84        14.9

Avg. Up Month              0.61%       1.57%
Avg. Down Month            -0.38%      -1.54%
Win Days %                 58.56%      67.57%
Win Month %                58.56%      67.57%
Win Quarter %              64.86%      75.68%
Win Year %                 70.0%       100.0%
Beta                       -0.02       -
Alpha                      0.41        -
```

![VAA](/assets/img/post_image/quant/momentum/DAA/momentum_1.png)   
![VAA](/assets/img/post_image/quant/momentum/DAA/momentum_2.png)   
![VAA](/assets/img/post_image/quant/momentum/DAA/momentum_3.png)   
![VAA](/assets/img/post_image/quant/momentum/DAA/momentum_4.png)   
![VAA](/assets/img/post_image/quant/momentum/DAA/momentum_5.png)   
![VAA](/assets/img/post_image/quant/momentum/DAA/momentum_6.png)   
![VAA](/assets/img/post_image/quant/momentum/DAA/momentum_7.png)   
![VAA](/assets/img/post_image/quant/momentum/DAA/momentum_8.png)  

## Strategy Evaluation

As shown above, Vigiliant Asset Allocation strategy shows pretty unsatisfying returns from COVID-19. Also, unlike previous market correction, this strategy couldn't avoided COVID-19. This makes me suspicious about this strategy. You can find this in underwater plot which shows drawdown. Furthermore, VAA spends most of time in safe assets. But before 2020, this strategy showed above market performance. Considering the fact that the paper is released at 2017, the author of this strategy overfitted to market movement.

Some might say that VAA strategy is working well during 2021-2022 market correction. But this is first market correction after paper is released. So, credibility of this strategy is in question. Furthermore, this strategy shows dramatic turn over. suddenly turn to risky portfolio from risk hedged portfolio.

Some might wonder VAA spend most of their time in safe asset classes. Answer to this question can be found at bond market and emerging market. These two market showed constant underperformance and thus induced risk averse portfolio.
   
   
