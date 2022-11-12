---
title: Swap Curve
author:
  name: unknown
  link: https://github.com/QuhiQuhihi
date: 2022-10-29 12:00:00 +0800
categories: [FICC Quant]
tags: [investment, derivative]
render_with_liquid: false
use_math: true
math: true
---

This post is about swap curve. You might heard about swap and curve independtly. But swap curve is required to price interest rate swap(IRS) and cross currency swap(CCS).   

## What is Swap Curve
Swap rate is interest rate which is reflect credibility among banks. Since banks are major players of Over-The-Counter(OTC) market, special interest rate is needed.   

Each player in finance industry has different credibility. Credibility of US government, Korea government, JP Morgan bank , Korea industrial bank and me have different credibility. When these banks are trading derivative product in OTC market, special interest rate called swap rate curve which is different from treasury yield, corporate bond rate is needed.   

We need Libor rate, Euro dollar futures and swap rate to calculate swap curve. In here, we use QuantLib package to construct term structure.   

## Construct curve with QuantLib
#### TermStructure() = DepositRateHelper() + FuturesRateHelper() + SwapRateHelper()
DepositRateHelper = helper for short term LIBOR rate   
FuturesRateHelper = helper to extract interest rate implied in Euro Dollar futures contract   
SwapRateHelper = helper for swap rate longer than 2 years   

## Test Data
I downloaded from Bloomberg Terminal in excel form. If you want to apply below code in real world, you need to make excel type file. which contains futures, and swap rate info.   
[Data](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/market_data/swap_data.xlsx)


## Result

Result of swap curve calculation
```yaml
       discount_factor  zero_rate  forward_rate
Tenor                                          
3MO           0.999424   0.225410      0.225410
6MO           0.999006   0.225213      0.223262
9MO           0.998446   0.224097      0.221029
12MO          0.997891   0.222980      0.218796
15MO          0.997343   0.221863      0.216563
18MO          0.996800   0.220747      0.214329
21MO          0.996263   0.219630      0.212096
2Y            0.995599   0.218231      0.209299
3Y            0.993033   0.229976      0.265555
4Y            0.989204   0.267638      0.419189
5Y            0.983540   0.327478      0.628952
6Y            0.976198   0.396210      0.809997
7Y            0.967113   0.471725      1.003116
8Y            0.956853   0.544133      1.121227
9Y            0.945523   0.614450      1.248637
10Y           0.933515   0.679342      1.328857
11Y           0.921174   0.737188      1.373633
12Y           0.908723   0.787884      1.396338
15Y           0.871961   0.902431      1.475245
20Y           0.812882   1.023566      1.507719
25Y           0.765340   1.057177      1.223894
30Y           0.722038   1.073100      1.168532
40Y           0.659502   1.028424      0.849571
50Y           0.625947   0.925671      0.412459
```

## Let's code this idea
Full code can be found at below link.    
[CODE](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/6_swap_curve.ipynb)

## Full Code
```python
import os
import datetime
import numpy as np
import pandas as pd

import QuantLib as ql


def get_quote(today):
    quote = pd.read_excel(os.path.join(os.getcwd(), "market_data/swap_data.xlsx"),index_col='Tenor')
    # pre-processing dataframe
    quote['DaysToMaturity'] = np.nan
    quote['Maturity'] = pd.to_datetime(quote['Maturity']).dt.date

    for tenor in quote.index:
        quote.loc[tenor, 'DaysToMaturity'] = (quote.loc[tenor, 'Maturity'] - today).days
    return quote

# construct IRS curve
def swap_curve(today, quote):

    # divide dataframe into 3 parts
    depo = quote[quote['InstType'] == 'CASH']
    futures = quote[quote['InstType'] == 'FUTURES']
    swap = quote[quote['InstType'] == 'SWAP']

    # Set evaluation date
    todays_date = ql.Date(today.day, today.month, today.year)
    ql.Settings.instance().evaluationDate = todays_date

    #Market Conventions
    calendar = ql.UnitedStates()
    dayCounter = ql.Actual360()
    convention = ql.ModifiedFollowing
    settlementDays = 2
    frequency = ql.Semiannual

    ### Build Rate Helper ###

    # 1_Deposit rate helper
    depositHelpers = [
        ql.DepositRateHelper(
            ql.QuoteHandle(ql.SimpleQuote(rate/100)),
            ql.Period(int(day), ql.Days),
            settlementDays,
            calendar,
            convention,
            False,
            dayCounter
        )
        for day, rate in zip(depo['DaysToMaturity'], depo['Market.Mid'])
    ]

    # 2_ Futures rate helper
    futuresHelpers = []
    for i, price in enumerate(futures['Market.Mid']):
        iborStartDate = ql.Date(
            futures['Maturity'][i].day,
            futures['Maturity'][i].month,
            futures['Maturity'][i].year
        )
        
        futuresHelpers = ql.FuturesRateHelper(
            ql.QuoteHandle(ql.SimpleQuote(price)),
            iborStartDate,
            3,
            calendar,
            convention,
            False,
            dayCounter
        )
        futuresHelpers.append(futuresHelpers)

    # 3_ Swap rate helper
    swapHelpers = [ql.SwapRateHelper(
        ql.QuoteHandle(ql.SimpleQuote(rate/100)),
        ql.Period(int(day), ql.Days),
        calendar,
        frequency,
        convention,
        dayCounter,
        ql.Euribor3M()
        )
        for day, rate in zip(swap['DaysToMaturity'], swap['Market.Mid'])
    ]

    # Combine 1_2_3 with Piece wise linear zero method
    # Curve construction
    helpers = depositHelpers + futuresHelpers + swapHelpers
    depoFuturesSwapCurve = ql.PiecewiseLinearZero(todays_date, helpers, dayCounter)

    return depoFuturesSwapCurve

# use curve to compute discount factor and zero rate
# the curve is calcualted with module used while pricing treasury
def discount_factor(date, curve):
    date = ql.Date(date.day, date.month, date.year)
    return curve.discount(date)

def zero_rate(date, curve):
    date = ql.Date(date.day, date.month, date.year)
    day_counter = ql.Actual360()
    compounding = ql.Compounded
    freq = ql.Continuous
    zero_rate = curve.zeroRate(
        date,
        day_counter,
        compounding,
        freq
    ).rate()

    return zero_rate

# cacualte forward rate
def forward_rate(date, curve):
    date = ql.Date(date.day, date.month, date.year)
    day_counter = ql.Actual360()
    compounding = ql.Compounded
    freq = ql.Continuous
    forward_rate = curve.forwardRate(
        date,
        date,
        day_counter,
        compounding,
        freq,
        True
    ).rate()
    
    return forward_rate

##############################################
## Let's calculate swap curve


today = datetime.date(2020,10,9)
quote = get_quote(today=today)
curve = swap_curve(today=today, quote=quote)

# calculate discount factor/ zero rate/ forward rate
quote['discount_factor'], quote['zero_rate'], quote['forward_rate'] = np.nan, np.nan, np.nan

for tenor, date in zip(quote.index, quote['Maturity']):
    quote.loc[tenor, 'discount_factor'] = discount_factor(date, curve)
    quote.loc[tenor, 'zero_rate'] = zero_rate(date, curve) * 100
    quote.loc[tenor, 'forward_rate'] = forward_rate(date, curve) * 100

# print result
print(quote[['discount_factor', 'zero_rate', 'forward_rate']])
```


