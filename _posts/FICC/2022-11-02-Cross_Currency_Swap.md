---
title: Cross Currency Swap (CCS)
author:
  name: unknown
  link: https://github.com/QuhiQuhihi
date: 2022-11-02 12:00:00 +0800
categories: [FICC Quant]
tags: [investment, derivative]
mermaid: true
render_with_liquid: false
math: true
---

This post is about cross currency swap. Though CCS is not familiar, IRS can offer interest rate risk - currency rate risk mitigation. If you heavily invest on foreign currency based asset classes for long term, CCS is absoultely what you are looking for.  

## What is Cross Currency Swap
Suppose you are portfolio manager at Korea, you want to invest certain asset in US with USDollar. Some might think FX swap as a solution to mitigate currency risk since FX swap combines spot and forward agreement. However, typical FX swap contracts have maturity under 1 year. So, if use forward aggrement in rolling manner, you cannot mitigate risk completely.   

Cross Currency Swap is used in this situation. CCS is OTC contract which participants exchange interest of each currency to use foreing currency. For example, one pay KRW interest to use KRW currency loan and the others pay USD interest to use USD loan. Typically, notional amount (loan amount) is exchanged at the beginning of contract with spot exchange rate.

Typically, every 6 month, interest are payed. And participants who have KRW & use USD  pays fixed interest and the other participant who have USD & use KRW pays floating interest to others. See below image to help your understanding.   
![CCS](/assets/img/post_image/FICC/CCS/ccs1.png)

## How to valuate Cross Currency Swap

Swap can be caluclated by summing present value of expected cashflows. Since interest rate of each currency is different, we need to build two individual interest rate swap curve for each currency. Equation is like below.    
$CCS  Swap  Value = {Receive} - {Pay}$    
where,   
$Receive = N \times e_{t_0} \times [1-\frac{1}{({1+z_{t_n}}^{KRW})^{t_n}} + \sum_{i=1}^{n} \frac{{s_{t_0}}^{KRW} \delta_{i-1}}{({1+z_{t_i}}^{KRW})^{t_i}}] $ 
$Pay = N \times [1-\frac{1}{(1+z_{t_n}^{USD})^{t_n}} - \sum_{i=1}^{n} \frac{L_{t_{i-1}}^{USD} \delta_{i-1}}{(1+z_{t_i}^{USD})^{t_i}}]$    


Below picture gives you what really consists of CCS. CCS decomposition helps you to understand how above frightening equation is derived from.    
![CCS](/assets/img/post_image/FICC/CCS/ccs2.png)


## Buyer and seller of cross currency swap (CCS)
Bid CCS (CCS pay) = expect exchage rate rise, KRW interest rate rise, USD interest rate fall    
Ask CCS (CCS receive) = expect exchange rate fall, KRW interest rate fall, USD interest rate fall    
![CCS](/assets/img/post_image/FICC/CCS/ccs3.png)





## Summary
See below picture, if you are facing trouble in memorizing the CCS structure.
![CCS](/assets/img/post_image/FICC/CCS/ccs1.png)


## Prequisite 1
```python
pip install QuantLib==1.18
pip install QuantExt-Python==1.8.3.3.5
```   
We use extended QuantLib package. So, you need to install stated version to calculate CCS.    
Since original quantlib cannot incorporate multiple curve, which is essential for cross currency products, We have to use extended QuantLib package. Please follow above code.

## Prequisite 2
```python
from quant_lib.fx_swap_curve import get_quote, usdirs_curve, krwccs_curve
```
If you don't want to make your own swap curve library, go to this link and download and place it appripriate directory.   
[FX_Swap_Curve_Code](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/quant_lib/fx_swap_curve.py) 


## Result
Let's price cross currency swap   
```yaml
price of FXF = 19914.3747
FX Delta = -4296.0136
USD IR Delta = -6301.2805
KRW IR Delta = 4523955.1303
Theta = -23596.1045
```



## Let's code this idea
Full code can be found at below link.   
[CODE](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/10_Cross_Currency_Swap.ipynb)

## Full Code
```python
import os
import datetime
import numpy as np
import pandas as pd

import QuantExt as qe

from quant_lib.fx_swap_curve import get_quote, usdirs_curve, krwccs_curve

class CCS():
    def __init__(self, today, effective_date, maturity_date, ccs_rate, fx_spot, usd_notional, position):
        
        # initial setup
        self.date = today
        self.usd_curve = self.usd_curve(self.date)
        self.krw_curve = self.krw_curve(self.date)
        self.fx_spot = fx_spot

        self.effective_date = qe.Date(effective_date.day, effective_date.month, effective_date.year)
        self.maturity_date = qe.Date(maturity_date.day, maturity_date.month, maturity_date.year)

        self.ccs_rate = ccs_rate

        self.usd = qe.KRWCurrency()
        self.krw = qe.USDCurrency()
        self.usd_notional = usd_notional
        self.krw_notional = usd_notional * fx_spot

        self.day_count = qe.ActualActual()

        if position == 'long':
            self.position = qe.VanillaSwap.Payer
        else:
            self.position = qe.VanillaSwap.Receiver

        self.length = 2
        self.spread = 0.0
        self.convention = qe.ModifiedFollowing
        self.calendar = qe.JointCalendar(qe.SouthKorea(), qe.UnitedStates())
        self.tenor = qe.Period(6, qe.Months)

        self.fixed_day_count = qe.Actual365Fixed()
        self.float_day_count = qe.Actual360()
        self.dateGeneration = qe.DateGeneration.Backward()

        # pricing result
        self.npv = self.PRICING(self.usd_curve, self.krw_curve, self.fx_spot)
        self.fx_delta = self.FX_DELTA()
        self.usd_ir_delta = self.USD_IR_DELTA()
        self.krw_ir_delta = self.KRW_IR_DELTA()
        self.theta = self.THETA()
    
    def usd_curve(self, date):
        return usdirs_curve(date, get_quote(date, 'USD'))
    
    def krw_curve(self, date):
        return krwccs_curve(date, get_quote(date, 'KRW'))


    def PRICING(self, usd_curve, krw_curve, fx_spot):
        
        # Handles of Market variables
        usd_curve_handle = qe.YieldTermStructureHandle(usd_curve)
        krw_curve_handle = qe.YieldTermStructureHandle(krw_curve)
        fx_spot_handle = qe.QuoteHandle(qe.SimpleQuote(fx_spot))

        # Reference Rate
        usd_6m_libor = qe.USDLibor(qe.Period(6, qe.Months), usd_curve_handle)

        # Fixed Schedule
        fixed_schedule = qe.Schedule(self.effective_date,
                                     self.maturity_date,
                                     self.tenor,
                                     self.calendar,
                                     self.convention,
                                     self.convention,
                                     self.dateGeneration,
                                     False
                                )
        
        float_schedule = qe.Schedule(self.effective_date,
                                     self.maturity_date,
                                     self.tenor,
                                     self.calendar,
                                     self.convention,
                                     self.convention,
                                     self.dateGeneration,
                                     False
                                )
        
        ccs = qe.CrossCcyFixFloatSwap(self.position,
                                      self.krw_notional,
                                      self.krw,
                                      fixed_schedule,
                                      self.ccs_rate,
                                      self.fixed_day_count,
                                      self.convention,
                                      self.length,
                                      self.calendar,
                                      self.usd_notional,
                                      self.usd,
                                      float_schedule,
                                      usd_6m_libor,
                                      self.spread,
                                      self.convention,
                                      self.length,
                                      self.calendar
                                )
        
        # Price Engine
        engine = qe.CrossCcySwapEngine(self.krw,
                                       krw_curve_handle,
                                       self.usd,
                                       usd_curve_handle,
                                       fx_spot_handle
                                )

        # conduct prcing
        ccs.setPricingEngine(engine)

        # net present value
        npv = ccs.NPV()

        return npv

    
    def FX_DELTA(self):

        percentage = 0.01

        # CCS price when 1% up
        up_fx = self.fx_spot * (1 + percentage)
        up_ccs = self.PRICING(self.usd_curve, self.krw_curve, up_fx)

        # CCS price when 1% down
        down_fx = self.fx_spot * (1 - percentage)
        down_ccs = self.PRICING(self.usd_curve, self.krw_curve, down_fxf)

        return  (up_ccs - down_ccs) / 2




    def USD_IR_DELTA(self):
        # Handle of USD curve
        curve_handle = qe.YieldTermStructureHandle(self.usd_curve)

        # 1 bp
        basis_point = 0.0001

        # ccs price when 1bp up
        up_curve = qe.ZeroSpreadedTermStructure(curve_handle, qe.QuoteHandle(qe.SimpleQuote(basis_point)))
        up_ccs = self.PRICING(up_curve, self.krw_curve, self.fx_spot)

        # ccs price when 1bp down
        down_curve = qe.ZeroSpreadedTermStructure(curve_handle, qe.QuoteHandle(qe.SimpleQuote(-basis_point)))
        down_ccs = self.PRICING(down_curve, self.krw_curve, self.fx_spot)       

        return (up_ccs - down_ccs) / 2
    
    
    def KRW_IR_DELTA(self):
        # Handle of KRW curve
        curve_handle = qe.YieldTermStructureHandle(self.krw_curve)

        # 1 bp
        basis_point = 0.0001

        # ccs price when 1bp up
        up_curve = qe.ZeroSpreadedTermStructure(curve_handle, qe.QuoteHandle(qe.SimpleQuote(basis_point)))
        up_ccs = self.PRICING(self.usd_curve, up_curve, self.fx_spot)

        # ccs price when 1bp down
        down_curve = qe.ZeroSpreadedTermStructure(curve_handle, qe.QuoteHandle(qe.SimpleQuote(basis_point)))
        down_ccs = self.PRICING(self.usd_curve, down_curve, self.fx_spot)

        return (up_ccs - down_ccs) / 2



    def THETA(self):
        # theta is change in value if one unit time passes.
        # in here, unit time is 1 day
        # since derivative product have time value, time to maturity is major variable in pricing derivatives
        price_t0 = self.PRICING(self.usd_curve, self.krw_curve, self.fx_spot)

        # ccsprice at t1
        usd_curve_t1 = self.usd_curve(self.date + datetime.datetime(days=1))
        krw_curve_t1 = self.krw_curve(self.date + datetime.datetime(days=1))

        price_t1 = self.PRICING(usd_curve_t1, krw_curve_t1, self.fx_spot)

        return price_t1 - price_t0


## build CCS contract information
todays_date = datetime.date(2020, 10, 8)
effective_date = datetime.date(2021, 11, 1)
maturity_date = datetime.date(2025, 11, 1)

position = 'long'
fx_spot = 1133.85

ccs_rate = 0.002438
usd_notional = 10000000

# build CCS object
ccs = CCS(today=todays_date,
        effective_date=effective_date,
        maturity_date=maturity_date,
        ccs_rate=ccs_rate,
        fx_spot=fx_spot,
        usd_notional=usd_notional,
        position=position
)

# Print result
print("price of FXF = {}".format(round(ccs.npv,4)))
print("FX Delta = {}".format(round(ccs.fx_delta,4)))
print("USD IR Delta = {}".format(round(ccs.usd_ir_delta,4)))
print("KRW IR Delta = {}".format(round(ccs.krw_ir_delta,4)))
print("Theta = {}".format(round(ccs.theta,4)))
```


