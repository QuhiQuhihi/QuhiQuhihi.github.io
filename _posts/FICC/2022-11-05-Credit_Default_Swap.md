---
title: Credit Default Swap (CDS) (In Process)
author: Quantitative Boxer
date: 2022-11-05 12:00:00 +0800
categories: [FICC Quant]
tags: [investment, derivative]
mermaid: true
render_with_liquid: false
math: true
---

This post is about credit default swap. Though CDS is not familiar, CDS can be used to protect from credit risk and market risk. If you heavily invest on foreign currency based bond for long term, and the bond has default risk, CDS is absoultely what you are looking for.  

## What is Market risk
Market risk includes risk derived from interest rate, foreign currency exchnage rate fluctuation. These factors are mainly discussed in macro ecnoomy market. Forward Rate Agreement, Interest Rate Swap, and Cross Currency Swap are examples of financial instruments intened to protect from macro economic variable.    

## Whart is Credit risk
If you just invest in US treasury or treasury of soverign country and wriiten in local currency, you don't have to deal with credit risk. But, if you want to invest in corporate bond or sovereign bond of emerging country, you need to worry about default risk. CDS is the instrument to mitigate credit risk when default happens.   

## What is Credit Default Swap
CDS is an insurance contract, which you can receice payment when default happens. Suppose you are portfolio manager at Korea, you want to invest emerging bond in US with USDollar. What you need to worry about is default of company or country and fluctuation of interest rate.  Supoose you are investing on corporate bond in US, you need to worry about default of company too. Before CDS there was no adequate method to mitigate credit risk only, since corporate bond rate is sum of market risk and credit risk.


![CCS](/assets/post_image/FICC/CDS/CDS1.png)
![CCS](/assets/post_image/FICC/CDS/CDS2.png)
![CCS](/assets/post_image/FICC/CDS/CDS3.png)
![CCS](/assets/post_image/FICC/CDS/CDS4.png)
![CCS](/assets/post_image/FICC/CDS/CDS5.png)
![CCS](/assets/post_image/FICC/CDS/CDS6.png)
![CCS](/assets/post_image/FICC/CDS/CDS7.png)
![CCS](/assets/post_image/FICC/CDS/CDS8.png)


Below picture gives you what really consists of CDS. CDS decomposition helps you to understand how above frightening equation is derived from.    

## Buyer and seller of credit default swap (CDS)
Bid CDS (Protection Buy) = Sell corporate bond + Receive Interest Rate Swap + Buy Floating Rate Note     
Ask CDS (Protection Sell) = Buy corporate bond + Pay Interest Rate Swap + Buy Floating Rate Note     

## Summary
See below picture, if you are facing trouble in memorizing the CDS structure.
![CCS](/assets/post_image/FICC/CDS/CDS8.png)


## Prequisite
You need two curve to price CDS. Since it is long-term contract with underlying asset of bond, discount curve is needed. In here, swap_curve function is to discount future cash flow. In addition, you need to calculate harzard rate since it has possibility of default. In here, cds_curve function is to discount expected cashflow at maturity considering default probability.   

```python
from quant_lib.cds_curve import get_irs_quote, get_cds_quote, swap_curve, cds_curve
```
If you don't want to make your own swap curve library, go to this link and download and place it appripriate directory. 
[CDS_Curve_Code](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/quant_lib/cds_curve.py)    

## Result
Let's price credit default swap   
```yaml
price of CDS = 964810.2233
IR Delta = -236.2413
Credit Delta = -2693.3063
Theta = 75.4122
```


## Let's code this idea
Full code can be found at below link.   
[CODE](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/11_Credit_Default_Swap.ipynb)

## Full Code
```python
import os
import datetime
import numpy as np
import pandas as pd

import QuantLib as ql

from quant_lib.cds_curve import get_irs_quote, get_cds_quote, swap_curve, cds_curve

class CDS():
    def __init__(self, today, maturity_date, spread, recovery, notional, position):
        
        # initial setup
        self.date = today
        self.discount_curve_t0 = self.discount_curve(self.date)
        self.cds_curve_t0 = self.cds_curve(self.date)

        self.maturity_date = ql.Date(maturity_date.day, maturity_date.month, maturity_date.year)

        if position == 'long':
            self.position = ql.Protection.Buyer
        else:
            self.position = ql.Protection.Seller

        self.spread = spread
        self.notional = notional
        self.recovery_rate = recovery

        self.tenor = ql.Period(3, ql.Months)
        self.calendar =  ql.UnitedStates()
        self.convention = ql.ModifiedFollowing
        self.dateGeneration = ql.DateGeneration.CDS
        self.dayCount = ql.Actual360()
        self.endOfMonth = False

        # pricing result
        self.npv = self.pricing(self.discount_curve_t0, self.cds_curve_t0)
        self.ir_delta = self.ir_delta()
        self.credit_delta = self.credit_delta()
        self.theta = self.theta()

    def discount_curve(self, date):
        return swap_curve(date, get_irs_quote(date))
    
    def cds_curve(self, date):
        return cds_curve(date, get_cds_quote(date), swap_curve(date, get_irs_quote(date)))
    
    def pricing(self, discount_curve, cds_curve):
        # processing
        todays_date = ql.Date(self.date.day, self.date.month, self.date.year)
        discount_curve_handle = ql.YieldTermStructureHandle(discount_curve)

        schedule = ql.Schedule(todays_date,
                                self.maturity_date,
                                self.tenor,
                                self.calendar,
                                self.convention,
                                self.convention,
                                self.dateGeneration,
                                self.endOfMonth
                            )

        cds = ql.CreditDefaultSwap(self.position,
                                   self.notional,
                                   self.spread/1000,
                                   schedule,
                                   self.convention,
                                   self.dayCount
                                    )
        
        probability = ql.DefaultProbabilityTermStructureHandle(cds_curve)

        engine = ql.MidPointCdsEngine(probability=probability,
                                      recoveryRate=self.recovery_rate,
                                      discountCurve=discount_curve_handle
                                    )
        
        cds.setPricingEngine(engine)
        
        npv = cds.NPV()

        return npv
    
    def ir_delta(self):
        curve_handle = ql.YieldTermStructureHandle(self.discount_curve_t0)

        # 1bp
        basis_point = 0.0001

        # CDS price when 1bp up
        up_curve = ql.ZeroSpreadedTermStructure(curve_handle, ql.QuoteHandle(ql.SimpleQuote(basis_point)))
        up_cds = self.pricing(up_curve, self.cds_curve_t0)

        # CDS price when 1bp down
        down_curve = ql.ZeroSpreadedTermStructure(curve_handle, ql.QuoteHandle(ql.SimpleQuote(-basis_point)))
        down_cds = self.pricing(down_curve, self.cds_curve_t0)

        # interest rate delta
        return (up_cds - down_cds) / 2
    
    def credit_delta(self):
        _cds_quote = get_cds_quote(self.date)

        # CDS price when 1bp up
        _cds_quote['Market.Mid'] += 1
        up_curve = cds_curve(self.date, _cds_quote, self.discount_curve_t0)
        up_cds = self.pricing(self.discount_curve_t0, up_curve)

        # CDS price when 1bp down
        _cds_quote['Market.Mid'] -= 1
        down_curve = cds_curve(self.date, _cds_quote, self.discount_curve_t0)
        down_cds = self.pricing(self.discount_curve_t0, down_curve)

        # credit delta
        return (up_cds - down_cds) / 2

    def theta(self):
        price_t0 = self.pricing(self.discount_curve_t0, self.cds_curve_t0)

        discount_curve_t1 = self.discount_curve(self.date + datetime.timedelta(days=1))
        cds_curve_t1 = self.cds_curve(self.date + datetime.timedelta(days=1))
        price_t1 = self.pricing(discount_curve=discount_curve_t1, cds_curve=cds_curve_t1)

        theta = price_t1 - price_t0

        return theta

## build CDS contract information
todays_date = datetime.date(2020, 12, 11)
maturity_date = datetime.date(2025, 12, 11)

notional = 10000000
spread = 20.5241
recovery = 0.4
position = 'short'

# build CDS object
cds = CDS(
        today=todays_date,
        maturity_date=maturity_date,
        spread=spread,
        recovery=0.4,
        notional=notional,
        position=position
        )

# Print result
print("price of CDS = {}".format(round(cds.npv,4)))
print("IR Delta = {}".format(round(cds.ir_delta,4)))
print("Credit Delta = {}".format(round(cds.credit_delta,4)))
print("Theta = {}".format(round(cds.theta,4)))
```


