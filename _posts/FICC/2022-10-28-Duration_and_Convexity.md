---
title: "Duration, convexity and quote risk: what a bond hedge actually removes"
author: daham
date: 2022-10-25 12:00:00 +0800
last_modified_at: 2026-09-20 21:00:00 +0900
categories: [FICC Quant]
tags: [investment, derivatives, QuantLib]
render_with_liquid: false
math: true
---

A duration hedge can work well against a parallel rate move while leaving meaningful exposure to the shape of the curve. QuantLib makes that distinction visible when the bumped object, sign and unit are stated explicitly. The desk question is **which market move the hedge neutralizes, and what remains when another part of the curve moves**.

In the illustrative study, 1.009745 units of a seven-year payer swap hedge 100 face of the bond. The residual after a parallel +100 bp quote shock is approximately **−0.001330 USD**, but a local seven-year +100 bp shock leaves **+0.071261 USD**. These are instantaneous numerical scenarios under constructed curves, with positions frozen throughout.

## Start with the derivative, not its label

For a bond price written as a function of one yield $y$, modified duration and convexity are

$$D_{mod}=-\frac{1}{P}\frac{\partial P}{\partial y},\qquad
C=\frac{1}{P}\frac{\partial^2P}{\partial y^2}.$$

The local approximation is $\Delta P/P\approx-D_{mod}\Delta y+\tfrac12C(\Delta y)^2$. It requires a defined yield convention and a shock to that yield. It cannot automatically describe a change to every bootstrap quote or a localized tenor shock.

For a continuous zero spread $s$ applied to fixed cash flows, $P(s)=\sum_iCF_iD_i e^{-s t_i}$. Its signed currency sensitivity per positive basis point is

$$DV01_{zero}=-10^{-4}\sum_i t_iCF_iD_i.$$

The primary study instead bumps all discount and term-projection quotes and rebuilds both curves. Its signed quote DV01 is $[P(q+1bp)-P(q-1bp)]/2$. Under this convention a fixed-rate bond has negative DV01. A report using positive loss-for-a-rate-rise DV01 would display the opposite sign; the hedge equation must remain consistent with whichever definition is chosen.

## Calculate the hedge from frozen instruments

This excerpt runs from the maintained project root. `quote_risk` rebuilds the curves on both sides of the bump while retaining the original swap coupon. The model inputs are illustrative quotes dated 15 September 2026, and both instruments use 100-unit notionals before scaling the swap position.

```python
from research.pricing import valuation_date, fixture, curves, swap, quote_risk

with valuation_date():
    quotes = fixture()
    discount, projection, _ = curves(quotes)
    _, _, fixed_coupon = swap(discount, projection)
    signed_dv01, curvature = quote_risk(quotes, fixed_coupon)
    hedge_units = -signed_dv01[0] / signed_dv01[1]
    print(round(signed_dv01[0], 6), round(signed_dv01[1], 6))
    print(round(hedge_units, 6))
# Bond -0.062460; payer swap +0.061858 USD per positive bp
# 1.009745 swap units per 100 bond face
```

The independently differentiated continuous-zero-spread sensitivity is **−0.063951 USD/bp**. Its difference from quote DV01 is expected: rebuilding a calibrated curve changes cash-flow discounting differently from adding a constant zero spread. The notebook checks the zero-spread derivative against a central difference; the core study also compares 0.1, 1 and 5 bp quote bumps to inspect numerical stability.

![Exact residual profit and loss and quote-bucket approximation for the frozen bond and swap hedge.](/assets/post_image/renovated/ficc/05-bond-risk.png)
*USD P&L per 100 bond face. The fixed swap coupon and 1.009745 hedge units remain unchanged in every illustrative scenario.*

## One hedge removes one direction

| Quote scenario | Bond alone | Bond plus swap |
|---|---:|---:|
| Parallel +100 bp | −6.017852 | −0.001330 |
| Parallel −100 bp | +6.487715 | −0.001452 |
| Steepener | −1.016854 | +0.030756 |
| Local 7Y +100 bp | −6.097930 | +0.071261 |

The hedge ratio $h=-DV01_B/DV01_S$ neutralizes the chosen parallel first derivative at inception. It does not match the full vector of maturity-bucket exposures, separate discount/projection exposures, or curvature. The local seven-year shock bumps that tenor's OIS and term quotes together. The steepener applies a linear sequence from −50 bp at 1Y to +50 bp at 10Y to both quote strips; the flattener reverses those signs. The full [scenario implementation](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/research/run_study.py) retains all revaluations.

The notebook verifies $\Delta P_{hedged}=\Delta P_B+h\Delta P_S$ against saved revaluations. Quote-bucket first-order approximations explain shape residuals; a parallel second-order approximation captures the small curvature loss under symmetric parallel shocks. Neither approximation is claimed to replace full revaluation for arbitrary large scenarios.

No time passes in these shocks. Carry, roll-down, coupon payments, financing, transaction costs and hedge rebalancing are therefore absent. A live hedge study would need those components and observed instruments. The [quotes and relinkable handles lab](https://github.com/QuhiQuhihi/project_FICC_Quant/tree/main/topics/12-quotes-handles-risk) is a useful implementation companion: it shows how the same QuantLib instrument responds to input changes while its contract stays fixed.

[Read the topic note](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/topics/05-bond-risk/README.md) · [Explore the executed notebook](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/topics/05-bond-risk/study.ipynb) · [Browse all QuantLib desk examples](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/README.md)
