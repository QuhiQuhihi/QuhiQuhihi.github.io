---
title: Black-Scholes Merton model
author:
  name: unknown
  link: https://github.com/QuhiQuhihi
date: 2022-10-15 12:00:00 +0800
categories: [FICC Quant]
tags: [investment, derivative]
render_with_liquid: false
use_math: true
math: true
---

This post is about Black Scholes Merton (BSM) model which is used for option pricing. Comming series will use QuantLib library for python. 

## What is Black Scholes Merton model?
Black Scholes Merton model is used for pricing options. Options are one of derivative products which are widely used to mitigate unnecessary risk to other party. For example, one might have crops, or stock compensations which will be delivered 3 month later. Prices of these are subject to volatilie change. But you want to sell at today's price as soon as these products are at your hand. Then you might consider put option which grant you right to sell the underlying product at designated price. Someone who want to buy your underlying product at designated price might be willing to sell that option. 

Intereting feature of options is that option buyer might not to exercise option if price level at excercise time point is not profitable. And some features (American Option) allow option buyer to exercise prior to maturity if it is profitable. These make quant difficult to valuate options.

Fischer Black, Myron Scholes and Robert C. Merton invented equation and model to valuate European-style options and wrote academic paper. The main principle behind the model is to hedge the option by buying and selling the underlying asset in a specific way to eliminate risk. This type of hedging is called "continuously revised delta hedging" and is the basis of more complicated hedging strategies such as those engaged in by investment banks and hedge funds.

## Is Black-Scholes model can be applied to real world directly?
The answer to this question is no. The Black–Scholes formula has only one parameter that cannot be directly observed in the market: the average future volatility of the underlying asset, though it can be found from the price of other options. Since the option value (whether put or call) is increasing in this parameter, it can be inverted to produce a "volatility surface" that is then used to calibrate other models. Since options are used to control unexpected volatility of market, the fact that we cannot observe "volatility" in market to price options is quite disappointing. (I first thought that I can valaute every derivative options in market correctly)

## Black-Scholes formula

$t$ is time in year, $T$ is time of option expiration ,and $r$ is risk-free interest rate.  
$S(t)$ is underlying asset price at time $t$, and $K$ is strike price of the option.  
$C(S,t)$ is price of European call option and $P(S,t)$ is price of European put option.  
$N(x)$ denotes the standard normal cumulative distribution function.   

boundary conditions for black-scholes formula are follows
$C(0,t) = 0$ for all t,    
$C(S,t) --> S-K$ as $S --> \infty$     
$C(S,T) = max{S-K, 0}$    

Price of Call option is :
$C(S_t,t) = N(d_1)S_t - N(d_2)Ke^{-r(T-t)}$   

Price of Put option is :
$P(S_t,t) = N(-d_2)Ke^{-r(T-t)} - N(-d_1)S_t$
        $ = Ke^{-r(T-t)} -S_t + C(S_t, t)$    
where 
$d_1 = \frac{\sigma \sqrt{T-t}}{ln\frac{S_t}{K} + (r + \frac{2}{\sigma^2}(T-t))}$   
$d_2 = d_1 - \sigma \sqrt{T-t} $

## Black Sholes with Code
[CODE](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/1_black_scholes.ipynb)

## Full COde
```python
import QuantLib as ql

# set valuation date
valuationDate = ql.Date(13,6,2022)
ql.Settings.instance().evaluationDate = valuationDate

# set calendar
calendar = ql.UnitedStates()
dayCount = ql.ActualActual()

# simple quote objects
underlying_qt = ql.SimpleQuote(3900)    # underlying price
dividend_qt = ql.SimpleQuote(0.0175)      # Dividend Yield
riskfreerate_qt = ql.SimpleQuote(0.02479)  # Risk-free Rate
volatility_qt = ql.SimpleQuote(0.25275)    # Volatility

# Quoto Handle Objects
u_qhd = ql.QuoteHandle(underlying_qt)
q_qhd = ql.QuoteHandle(dividend_qt)
r_qhd = ql.QuoteHandle(riskfreerate_qt)
v_qhd = ql.QuoteHandle(volatility_qt)

# Term Structure Objects
"""
Dividend yield, riskfree rate and volatility requires term structure across
These values change over time and thus requires detailed insturction
Dividend yield, riskfree rate --> FlatForward() : certain value at maturity 
Volatility --> BlackConstantVol() : 
"""
r_ts = ql.FlatForward(valuationDate, r_qhd, dayCount)
d_ts = ql.FlatForward(valuationDate, q_qhd, dayCount)
v_ts = ql.BlackConstantVol(valuationDate, calendar, v_qhd, dayCount)

# Term-Structure Handle Objects
r_thd = ql.YieldTermStructureHandle(r_ts)
d_thd = ql.YieldTermStructureHandle(d_ts)
v_thd = ql.BlackVolTermStructureHandle(v_ts)

# process and engine
process = ql.BlackScholesMertonProcess(u_qhd, d_thd, r_thd, v_thd)
engine = ql.AnalyticEuropeanEngine(process)

# option objects
option_type = ql.Option.Call
strike_price = 3900
expiry_date = ql.Date(30,12,2022)
exercise = ql.EuropeanExercise(expiry_date)
payoff = ql.PlainVanillaPayoff(option_type, strike_price)
option = ql.VanillaOption(payoff, exercise)

# Pricing
option.setPricingEngine(engine)

# price and greek results
print("option premium = ", round(option.NPV(), 2))
print("option delta = ", round(option.delta(), 4))
print("option gamma = ", round(option.gamma(), 4))
print("option theta = ", round(option.thetaPerDay(), 4))
print("option vega = ",  round(option.vega() / 100, 4))
print("option rho = " ,  round(option.rho() / 100, 4))
```

## Result
```yaml
option premium =  297.29
option delta =  0.5408
option gamma =  0.0005
option theta =  -0.7325
option vega =  11.414
option rho =  10.0765
```
