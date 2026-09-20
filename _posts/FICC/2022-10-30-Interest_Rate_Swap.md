---
title: "Interest rate swaps in QuantLib: cash flows, fair coupons and hedging"
author: daham
date: 2022-10-30 12:00:00 +0800
last_modified_at: 2026-09-20 21:00:00 +0900
categories: [FICC Quant]
tags: [investment, derivatives, QuantLib]
render_with_liquid: false
math: true
---

An interest rate swap becomes easier to reason about when “pay fixed” and “receive floating” are written as signed cash flows. Its initial fair coupon is a consequence of those flows, their schedules, the floating-index projection and discounting assumptions. It is not a property of maturity alone.

The illustrative seven-year payer swap in this project has a fair fixed coupon of **3.959824%**. Independent floating-coupon forecasts and a fixed-leg annuity reproduce that rate and its zero initial value. Increasing the contractual fixed coupon by 10 bp, with curves unchanged, reduces value by **0.618575 per 100 notional**. This offers a simple, inspectable check on signs and price scale.

## Which exposure is being exchanged?

A payer swap pays fixed and receives floating. A borrower paying a matching floating-rate liability can use that floating receipt to offset the liability's floating component, leaving a fixed payment, subject to schedule, index and spread differences. Conversely, an investor receiving a floating-rate asset and wanting fixed receipts would generally examine the receiver direction. “Buying a swap” is less precise than stating the two legs.

For fixed-leg accrual fractions $\alpha_i$ and discount factors $D_i$, define $A=\sum_i\alpha_iD_i$. Let $B$ be the discounted projected floating payments per unit notional. Then

$$K^*=\frac{B}{A},\qquad PV_{payer}=N(B-KA).$$

The formula makes the sign transparent: raising the coupon paid reduces payer value. A conventional single-currency interest rate swap exchanges coupon amounts without an initial or terminal exchange of principal; the notional scales those amounts.

## Make the conventions visible

The constructed valuation date is **15 September 2026**. The swap starts one business day later and lasts seven years. It pays fixed annually and receives a synthetic USD 3M rate quarterly, using Actual/360, the US GovernmentBond calendar and Modified Following. Notional is 100. The floating index has zero fixing lag and a next-business-day start, avoiding a hidden dependence on historical fixings in this example.

Projection comes from the illustrative term curve and discounting from the OIS curve. Neither the term quotes nor the OIS quotes are live market observations. The project sets these assumptions explicitly so the coupon calculation can be investigated without a vendor terminal.

This runnable excerpt uses the maintained QuantLib builders from the project root. `flows` is the independently reconstructed signed ledger, not a second engine NPV.

```python
from research.pricing import valuation_date, curves, swap

with valuation_date():
    discount, projection, _ = curves()
    instrument, flows, fair = swap(discount, projection)
    fixed = flows.query('instrument == "fixed"')
    floating = flows.query('instrument == "float"')
    annuity = (fixed.accrual * fixed.discount).sum()
    manual_rate = floating.pv.sum() / (100 * annuity)
    assert abs(manual_rate - fair) < 1e-10
    assert abs(flows.pv.sum() - instrument.NPV()) < 1e-8
    changed, _, _ = swap(discount, projection, fair + .001)
    assert abs(changed.NPV() + 100 * .001 * annuity) < 1e-8
    print(round(100 * fair, 6), round(changed.NPV(), 6))
# Fair coupon 3.959824 percent; higher-coupon payer PV -0.618575
```

![Equal and opposite fixed and floating leg present values for the par payer swap.](/assets/post_image/renovated/ficc/08-interest-rate-swaps.png)
*Present values per 100 notional under illustrative curves. Equal aggregate values do not mean individual coupon amounts or dates match.*

## What the independent comparison catches

The fixed annuity is **6.185755**. Multiplying it by 100 notional and a 0.001 coupon increment gives the predicted PV loss. The primary swap's total engine-versus-independent PV difference is approximately $3.28\times10^{-14}$ currency units.

Floating coupons need careful dates. The projection ratio uses the index's value and maturity dates and its accrual convention; the cash amount then uses the actual coupon accrual. Holiday adjustments can make those intervals differ. A shortcut that assumes they are always identical can hide a convention error despite a plausible total price.

For hedging, the fixed coupon must stay fixed after inception. Recomputing a fair coupon following every market shock replaces the contract with a different par swap and erases the exposure being measured. The [bond risk exercise](https://github.com/QuhiQuhihi/project_FICC_Quant/tree/main/topics/05-bond-risk) freezes both coupon and hedge units before comparing parallel, slope and local-tenor scenarios. It finds that a single swap removes one first-order curve direction while leaving shape and curvature risk.

Several nearby desk questions extend the same ingredients. A [forward rate agreement](https://github.com/QuhiQuhihi/project_FICC_Quant/tree/main/topics/07-forward-rate-agreements) isolates one period. [Caps and floors](https://github.com/QuhiQuhihi/project_FICC_Quant/tree/main/topics/13-caps-and-floors) place optionality on floating payments. A [European swaption](https://github.com/QuhiQuhihi/project_FICC_Quant/tree/main/topics/14-european-swaptions) makes entering a future swap optional and prices that choice using a forward swap rate and annuity. Their executed notebooks provide additional QuantLib objects and independent checks, with the same separation between illustrative assumptions and market evidence.

[Read the topic note](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/topics/08-interest-rate-swaps/README.md) · [Explore the executed notebook](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/topics/08-interest-rate-swaps/study.ipynb) · [Browse all QuantLib desk examples](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/README.md)
