---
title: Regime Switching Model for Option Market 
author:
  name: unknown
  link: https://github.com/QuhiQuhihi
date: 2024-05-10 12:00:00 +0800
categories: [Financial Market]
tags: [regime model, statistics, derivatives]
render_with_liquid: false
use_math: true
math: true
---

This post is review of "Option Pricing with Markov Switching” by Cheng-Der Fuh, Kwok Wah Remus Ho, Inchi Hu and Ren-Her Wang. This paper introduce Regime Switching Model using Hidden Markov Model for option market. 

## What is Regime Switching Model
Financial market volatility is known to fluctuate over time, often exhibiting persistent changes known as volatility clustering. Empirical research has identified various stylized facts about volatility, leading to the development of models that aim to capture these dynamics. Recent events such as COVID-19, combined with periods of quantitative easing and tightening, have caused radical regime shifts in financial markets, resulting in extreme movements, especially in the options market.

Traditional models like ARCH, GARCH, and stochastic volatility models have been employed to capture changing volatility. However, **Regime-Switching Models**, particularly those using **Hidden Markov Models (HMMs)**, offer a robust framework for modeling the sudden shifts observed in financial markets.

Before the COVID-19 pandemic, the VIX index indicated a relatively stable volatility environment. However, post-pandemic, the market entered a high-volatility regime that persisted for an extended period.


## Literature Review: The Power of Regime-Switching Models
Regime-switching models were first introduced by **Hamilton (1988, 1989)** to incorporate time-varying volatility into financial modeling. Key contributions include:

- **Hamilton (1989)**: Developed a framework using Markov-switching models to capture changes in economic regimes.
- **Di Masi et al. (1994)**: Studied European call option pricing in a diffusion model where drift and volatility are governed by a two-state Markov process.
- **Guo (2001)**: Provided a closed-form solution for option pricing under regime-switching models.
- **Maghrebi et al. (2007)**: Demonstrated that Markov-switching models effectively adjust forecast errors and capture nonlinear volatility expectations, offering solutions to market overreactions and underreactions.

These models are particularly effective in capturing the nonlinear dynamics and regime shifts observed in financial markets.


## Integrating Black-Scholes with Hidden Markov Processes
In this research, we propose a model that merges the classic **Black-Scholes-Merton (BSM)** framework with a **Hidden Markov Model (HMM)** to address regime-dependent fluctuations in stock prices.

We consider a stock price process \( X_t \) that evolves according to:

$ dX_t = X_t \mu_{\varepsilon(t)} dt + X_t \sigma_{\varepsilon(t)} dW_t $

where:

- $ \mu_{\varepsilon(t)}$ and $ \sigma_{\varepsilon(t)}$ are the state-dependent drift and volatility parameters.
- $ \varepsilon(t) $ is a stochastic process representing the unobserved state of the business cycle.
- $ W_t $ is a standard Brownian motion.


## States of the Hidden Markov Model

We define a three-state HMM to capture different phases of the business cycle \varepsilon(t):

- $\varepsilon(t) = 0$ Contraction (Low Volatility), 
- $\varepsilon(t) = 1$ Transition (Moderate Volatility), 
- $\varepsilon(t) = 2$ Expansion (High Volatility) 

A three-state HMM effectively captures the empirical phenomena of financial time series, allowing for more nuanced modeling than a two-state model.



## Major Assumptions
The model is built on several key assumptions:

1. **Independence**: The state process $ \varepsilon(t) $ is independent of the Brownian motion $ W_t $.
2. **Fixed Asset Supply**: The total shares of the risky asset are fixed and normalized to 1.
3. **Risk-Free Rate**: The risk-free asset offers a constant instantaneous rate of return $ r $.
4. **Finite States**: The state process $ \varepsilon(t) $ is a Markov process with a finite number of states.
5. **Observability**: Each state has different volatility $ \sigma_{\varepsilon(t)} $, making $ \varepsilon(t) $ effectively observable through market data.
6. **Exponential Holding Times**: The time spent in each state follows an exponential distribution with rate $ \lambda_i $, i.e., $ P(\tau_i > t) = e^{-\lambda_i t} $ for state $ i $.



## Completing the Market: Risk-Neutral Measure

Since the market is incomplete due to the presence of unhedgeable risks from the Markov process $ \varepsilon(t) $, we introduce a **Completing-of-Securities (COS)** contract. This contract pays one dollar at the next state change of $ \varepsilon(t) $.

### Adjusting for Risk Neutrality

To ensure absence of arbitrage, we need to find an equivalent **risk-neutral measure** $ Q $. Under $ Q $, the adjusted dynamics of the stock price become:

$ dX_t = X_t r dt + X_t \sigma_{\varepsilon(t)} dW_t^Q $

where $ dW_t^Q $ is a Brownian motion under the risk-neutral measure.

The adjusted transition rates under $ Q $ are:

$\lambda_i^Q = \frac{r}{r + k_i} \lambda_i $

where $ k_i $ is the market price of risk associated with state $ i $.




## Pricing European Call Options Under Regime Switching

The primary goal is to derive the arbitrage-free price of a European call option in this regime-switching framework.

### Theoretical Framework

The arbitrage-free price $ V_i(T, K, r) $ of a European call option, given the initial state $ \varepsilon(0) = i $, is:

$ V_i(T, K, r) = e^{-rT} \mathbb{E}^Q\left[ (X_T - K)^+ \mid \varepsilon(0) = i \right] $


where:

- $ T $ is the time to maturity.
- $ K $ is the strike price.
- $ \mathbb{E}^Q $ denotes the expectation under the risk-neutral measure.

### Occupation Times

Let $ T_i $ be the occupation time in state $ i $ up to time $ T $. The option price can be expressed as:

$ V_i(T, K, r) = e^{-rT} \int_{0}^{T} \mathbb{E}^Q\left[ (X_T - K)^+ \mid T_i = t \right] f_i(t, T) dt $

where $ f_i(t, T) $ is the probability density function of $ T_i $.

### Stock Price Dynamics Under Risk Neutrality

Under the risk-neutral measure, the log-price $ \ln X_T $ is normally distributed with mean and variance:

$ m(t) &= \ln X_0 + \left( r - \frac{1}{2} \sigma_{\varepsilon(t)}^2 \right) T $ \\
$ v(t) &= \int_{0}^{T} \sigma_{\varepsilon(s)}^2 ds $



## Main Result: Closed-Form Option Pricing Formula

Combining the above, the arbitrage-free price of the European call option is:

$ V_i(T, K, r) = e^{-rT} \int_{0}^{T} \left[ X_0 e^{m(t) + \frac{1}{2} v(t)} N(d_1) - K N(d_2) \right] f_i(t, T) dt $

where:

- $ N(\cdot) $ is the cumulative distribution function of the standard normal distribution.
- $ d_1 $ and $ d_2 $ are given by:


$ d_1 &= \frac{m(t) - \ln K + v(t)}{\sqrt{v(t)}} $ \\
$ d_2 &= d_1 - \sqrt{v(t)} $


This formula generalizes the classic Black-Scholes formula to accommodate regime-dependent volatility.


### Case of Finite Number of States

For an HMM with \( N \) finite states, the option price becomes:

$ V_i(T, K, r) = e^{-rT} \int_{\mathcal{S}} \left[ X_0 e^{m(\mathbf{t}) + \frac{1}{2} v(\mathbf{t})} N(d_1) - K N(d_2) \right] f_i(\mathbf{t}, T) d\mathbf{t} $

where:

- $ \mathbf{t} = (t_0, t_1, \dots, t_N) $ are the occupation times in each state.
- $ \mathcal{S} = \{ \mathbf{t} \mid t_0 + t_1 + \dots + t_N = T \} $.
- $ m(\mathbf{t}) $ and $ v(\mathbf{t}) $ are computed based on the occupation times.


## Numerical Methods and Simulation

### Discrete Diffusion Method

This method discretizes the stochastic differential equation (SDE) governing \( X_t \):

$ X_{n+1} = X_n \exp\left( \left( r - \frac{1}{2} \sigma_{\varepsilon_n}^2 \right) \Delta t + \sigma_{\varepsilon_n} \sqrt{\Delta t} \eta_n \right) $


where $ \eta_n $ are independent standard normal random variables.

### Markovian Tree Method

This method constructs a recombining tree that accounts for both the stochastic process $ X_t $ and the Markov chain $ \varepsilon(t) $, capturing possible transitions between states at each time step.


## Simulation Results

We compared the simulation results with the analytical formula under various scenarios:

### 1. High vs. Low Volatility

Using empirical data to classify high and low volatility regimes, we observed:

- The discrete diffusion method closely approximates the analytical solution in both high and low volatility environments.
- The Markovian tree method performs comparably but may diverge under certain parameter settings.

### 2. Transition Rates

Different transition rate matrices $ Q $ impact the model's performance:

- When transition rates are similar across states, the discrete diffusion method excels.
- With varying transition rates, the Markovian tree method shows better accuracy.

### 3. Strike Price and Maturity (Volatility Surface)

The model effectively captures the volatility surface across different strike prices and maturities, aligning with observed market data during events like the COVID-19 pandemic.


## Sensitivity Analysis

A sensitivity analysis was conducted to assess the impact of model parameters $ \sigma $ (volatility) and $ \lambda $ (transition rates):

- **Volatility $(\sigma )$**: Higher volatility increases option prices, as expected.
- **Transition Rates $(\lambda)$**: Faster transitions between states lead to option prices converging across different initial states.


## Conclusion

This research enhances traditional option pricing models by integrating a regime-switching framework using Hidden Markov Models. The key contributions include:

- **Enhanced Modeling**: Captures volatility clustering and regime shifts more effectively than constant-volatility models.
- **Closed-Form Solution**: Provides an analytical formula for European call options under regime switching.
- **Numerical Methods**: Demonstrates the effectiveness of discrete diffusion and Markovian tree methods in approximating option prices.

### Future Research

Potential directions for future research include:

- **Model Calibration**: Using market data to calibrate the model parameters for real-world applications.
- **Extension to American Options**: Adapting the framework to price American options or other exotic derivatives.
- **Incorporating Additional Factors**: Integrating macroeconomic variables or other risk factors into the HMM.

By exploring the dynamics of volatility regime shifts through Hidden Markov Models, we provide a robust framework for understanding risk and return trade-offs in option markets. This approach offers valuable insights for both researchers and practitioners in finance.
