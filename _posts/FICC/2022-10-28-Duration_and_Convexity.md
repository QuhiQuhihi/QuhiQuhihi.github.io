---
title: Duration and Convexity
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

This post is about duration and convexit of bond. Duration and convexity of bond is essential information when you trade bonds. Data will be scaped from WSJ site and quantlib library will be used. 

## How to price bond in market?
Bond price can be calculted as below :   
$ V = Pe^{-y(T-t)} + \sum_{i=1}^{T} C_i e^{-y(t_i - t)} $
where where, $V$ is bond price, $P$ is notional amount, $y$ is market interest rate, $N$ is number of coupon payment, $C_i$ is $i_{th}$ coupon payment amount.   

## What is Duration?
Duration can be calucalted ad below :
$ Duration = -\frac{1}{V(y)} \frac{dy}{dV}$
where $\frac{dy}{dV}$ is change in bond price as interest rate changes.   

If you are not familiar with mathematical formula, you can understand duration as expected time needed to earn same amount of initial investment by dividend, interest, and payment at maturity. So, if duration is long, your portfolio will fluctuate by external shocks. On contrary, if your portfolio has short duration, it will be resilent to external shocks such as market interest rate shock. This can be understood in simple way, if time you will receive money back is short, then you can wait with patient and beleif. But if time you will receive money back is long, you will be impatient and suspicions, making stock or bond price to fall if external shock occurs.

## What is Convexity?
Convexity can be calculated as below :   
$ Convexity = \frac{1}{V(y)} \frac{d y^2}{d^2 V}$
where $\frac{d y^2}{d^2 V}$ is second order function related to price change of bond price to interest rate change.   

Convexit is second order function of duration. This might can reveal how duration will react in mathematical manner. In short, convexity can measure degree of the curve, in the relationship between bond prices and bond yields. So, how the duration of a bond changes as the interest rate changes can be expressed with convexity. It is essential for risk managers who are interested in ALM(Asset Liability Management). Typical ALM procedure pays great attention to duration. So, measuring how duration will change as interest rate change gives valuable information to risk managers who are in charge of ALM.


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
issueDate = 10/2/2022
maturityDate = 10/2/2032
tenor = semi-annual
coupon rate = 2.00%
face value = 100
settlement days = first date of month

Yield to maturity = 0.0272
Duration = 8.9461
Duration = 89.5908
```



## Let's code this idea
Full code can be found at below link.
[CODE](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/5_greeks_for_bond.ipynb)

## Full Code
```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import datetime
import requests

from bs4 import BeautifulSoup
import QuantLib as ql

from quant_lib import curve as cv


ref_date = cv.get_date()
quote = cv.get_quote(ref_date)
print(quote)
curve = cv.treasury_curve(ref_date, quote)

# convert into Engine
spotCurveHandle = ql.YieldTermStructureHandle(curve)
bondEngine = ql.DiscountingBondEngine(spotCurveHandle)

# Treasury Bond Specifiction
issueDate = ql.Date(2,10,2022)
maturityDate = ql.Date(2,10,2032)
tenor = ql.Period(ql.Semiannual)
calendar = ql.UnitedStates()
convention = ql.ModifiedFollowing
dateGeneration = ql.DateGeneration.Backward
monthEnd = False
schedule = ql.Schedule(
                issueDate,
                maturityDate,
                tenor,
                calendar,
                convention,
                convention,
                dateGeneration,
                monthEnd
                    )
dayCount = ql.ActualActual()
couponRate = [0.02]
settlementDays = 1
faceValue = 100

# set Fixed Rate bond pricing engine
fixedRateBond = ql.FixedRateBond(
    settlementDays,
    faceValue,
    schedule,
    couponRate,
    dayCount
)

# Conduct bond pricing
fixedRateBond.setPricingEngine(bondEngine)

# calculate YTM
targetPrice = fixedRateBond.cleanPrice()
ytm = ql.InterestRate(
    fixedRateBond.bondYield(targetPrice, dayCount, ql.Compounded, ql.Semiannual),
    dayCount,
    ql.Compounded,
    ql.Semiannual
)

print("Yield to maturity = {:.4f}".format(ytm.rate()))
print("Duration = {:.4f}".format(ql.BondFunctions.duration(fixedRateBond, ytm)))
print("Duration = {:.4f}".format(ql.BondFunctions.convexity(fixedRateBond, ytm)))
```


