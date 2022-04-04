---
title: Regime Switching Model
author:
  name: unknown
  link: https://github.com/QuhiQuhihi
date: 2022-02-20 12:00:00 +0800
categories: [Financial Market]
tags: [quant, market, statistics]
render_with_liquid: false
use_math: true
---

This post is about Regime Switching Model for financial market

## What is Regime Switching Model
Financial market tend to show different behavior and pattern if market change their state. State is called "regime" in financial market. Easy example of regime switch is quantitative easying after COVID-19 outbreak in 2020, start of quantitative tightening in 2022.
If we can determine what the regimes are, we can understand how a portfolio might react to various regime. This can enhance model's predictability and reduce effort to build all-time effective killer model.

## How to build regime switch model
Data-driven approach is letting historical data on assets and/or market risks delineate the regimes. 
A specific example of this approach is a Gaussian Mixture Model (GMM), which is a type of unsupervised learning method. The GMM uses various Gaussian distributions (another word for a normal, bell curve distribution) to model different parts of the data. 
Another example of this approach is Greedy Gausian Segmentation algorithm. This algorithm segment multivariate time series via unsupervised learning method. GGS is heuristic method of complex dynamic programming which can reveal local optimum points.

## Why Gausian Mixture Model for regime switch model

Gausian Mixture Model(GMM) is an especially helpful method for modeling financial assets, as their return distributions can often exhibit skew with a meaningful number of observations in the tails.