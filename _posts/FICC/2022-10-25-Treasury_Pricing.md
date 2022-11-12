---
title: Treasury Pricing
author:
  name: unknown
  link: https://github.com/QuhiQuhihi
date: 2022-10-25 12:00:00 +0800
categories: [FICC Quant]
tags: [investment, derivative]
render_with_liquid: false
use_math: true
math: true
---

This post is about pricing treasury in bond market. Data will be scaped from WSJ site and quantlib library will be used. 

## What is Treasury and how to price it?
Treasury price can be calculated with yield curve. When you buy treasury, you can see how much money they will pay and when they will pay. Since credit score of tresury is AAA, you don't have to worry about default probability. Government is most credible institution in countries. So just discount future cashflow is needed. 

How to valuate it is simple. it is just discount expected cashflow with interest rate. Coupon is cash payments which are given before maturity. And notional amount is cash payment which are given at the time of maturity. Price of treasury is discounted value of expected coupons and notional amount. 

## Prequisite
If you have time to follow this series, please go to 
[Yield_Curve_Post](https://quhiquhihi.github.io/posts/Yield_Curve)
to make own yield curve module. This notebook simply import curve from library.
```python
from quant_lib import curve as cv
```
If you don't want to make your own library, go to this link and download and place it appripriate directory. 
[Yield_Curve_Code](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/quant_lib/curve.py) 



## Result with real market data
market data fof treasury. Using discount factor and yield curve, you can make pricing for treasury.
```yaml
             days  price  coupon   discount factor   zero rate
maturity                       
2022-10-25     27   2.53    0.0     0.998133        0.025425
2022-12-29     92  3.301    0.0     0.992677        0.029375
2023-03-30    183  3.914    0.0     0.982763        0.034982
2023-09-07    344  3.803    0.0     0.964756        0.038435
2024-09-30    733  4.204   4.25     0.918859        0.042629
2025-09-15   1083  4.216    3.5     0.902767        0.034806
2027-09-30   1828  4.013  4.125     0.814004        0.041538
2029-09-30   2559  3.911  3.875     0.764772        0.038650
2032-08-15   3609  3.773   2.75     0.772329        0.026319
2052-08-15  10914  3.707    3.0     0.407590        0.030263
```

for example, let's price treasury. Information of treasury is below.
```yaml
issueDate = 30/9/2022
maturityDate = 15/8/2032
tenor = semi-annual
coupon rate = 1.75%
face value = 100
settlement days = first date of month

Bond Price = 91.7212

Projected cashflow
 Date          CashFlow
 15/2/2023     0.661644
 15/8/2023     0.867808
 15/2/2024     0.881602
 15/8/2024     0.870219
 18/2/2025     0.894754
 15/8/2025     0.853425
 17/2/2026     0.891781
 17/8/2026     0.867808
 16/2/2027     0.877397
 16/8/2027     0.867808
 15/2/2028     0.876808
 15/8/2028     0.870219
 15/2/2029     0.880371
 15/8/2029     0.867808
 15/2/2030     0.882192
 15/8/2030     0.867808
 18/2/2031     0.896575
 15/8/2031     0.853425
 17/2/2032     0.891165
 16/8/2032     0.865437
 16/8/2032   100.000000
```



## Let's code this idea
Full code can be found at below link.
[CODE](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/4_treasury_pricing.ipynb)

## Full Code
```python
import numpy as np
import pandas as pd
import QuantLib as ql

from calendar import calendar
from sqlite3 import Date

from quant_lib import curve as cv

ref_date = cv.get_date()
quote = cv.get_quote(ref_date)
print(quote)
curve = cv.treasury_curve(ref_date, quote)

# Convert into Engine
spotCurveHandle=ql.YieldTermStructureHandle(curve)
bondEngine=ql.DiscountingBondEngine(spotCurveHandle)

issueDate=ql.Date(30,9,2022)
maturityDate=ql.Date(15,8,2032)
tenor=ql.Period(ql.Semiannual)
calendar=ql.UnitedStates()
convention=ql.ModifiedFollowing
dateGeneration=ql.DateGeneration.Backward
monthEnd=False
schedule=ql.Schedule(issueDate, maturityDate, tenor,
                    calendar, convention, convention,
                    dateGeneration, monthEnd)
dayCount=ql.ActualActual()
couponRate=[0.0175]
settlementDays=1
faceValue=100

fixedRateBond=ql.FixedRateBond(settlementDays, faceValue, schedule, couponRate, dayCount)

# conduct pricing
fixedRateBond.setPricingEngine(bondEngine)

# Result
print("Bond Price = {}".format(round(fixedRateBond.NPV(),4)))
for cf in fixedRateBond.cashflows():
    print('%20s %12f' % (cf.date(), cf.amount()))
```


