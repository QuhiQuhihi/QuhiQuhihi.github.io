---
title: "CDS in QuantLib: calibrate survival and reconcile the credit legs"
author: daham
date: 2022-11-05 12:00:00 +0800
last_modified_at: 2026-09-20 21:00:00 +0900
categories: [FICC Quant]
tags: [investment, derivatives, QuantLib]
render_with_liquid: false
math: true
---

A credit default swap exchanges a running premium for protection against specified credit events. A useful implementation must connect the quoted spreads to survival probabilities and then explain the premium and protection legs of the actual ticket. The approximation “spread equals loss-given-default times hazard” is a helpful starting intuition, but it is not a substitute for contract-level valuation.

This example calibrates **illustrative 1Y, 3Y and 5Y spreads of 80, 110 and 140 bp**, assumes 40% recovery, and prices a separate five-year protection-buyer ticket with a 100 bp running premium. Its value is **+1.758672 per 100 notional** under the constructed model. Independently summed midpoint cash flows reproduce both legs and the total QuantLib value.

## Follow the contingent cash flows

Let $Q(t)$ be survival probability and $R$ recovery. A simplified premium leg includes scheduled coupons weighted by survival, plus accrued premium payable upon default. The protection leg values the loss payment $N(1-R)$ over default-time probabilities. With discount factor $D(t)$, the continuous-time schematic is

$$PV_{protection}=N(1-R)\int_0^T D(t)\,[-dQ(t)].$$

The buyer's value is protection received minus premium paid. A higher running premium therefore lowers value when the curves and other terms are unchanged. Recovery is an explicit model assumption in this example; setting it to zero simply for convenience would alter the calibration and the payment being valued.

Hazard rates here belong to the pricing measure inferred from assumed spreads. They are not estimated physical default forecasts. Credit protection also does not remove every bond exposure: interest-rate, liquidity, contractual basis and counterparty risks require their own treatment.

## Build and check the survival curve

The valuation date is **15 September 2026**. Discounting uses the project's constructed OIS curve. The calibration helpers use zero settlement days, quarterly premiums, Following adjustment, Forward schedule generation and Actual/360. A piecewise-flat hazard curve is bootstrapped against those helper contracts.

Run this excerpt from the maintained project root. The call to survival probability triggers the lazy bootstrap; asking for implied helper quotes before calculation would obscure that dependency.

```python
import QuantLib as ql
from research.pricing import DATE, CAL, valuation_date, curves

with valuation_date():
    discount, _, _ = curves()
    discount_handle = ql.YieldTermStructureHandle(discount)
    tenors, spreads = [1, 3, 5], [.008, .011, .014]
    helpers = [ql.SpreadCdsHelper(
        s, ql.Period(y, ql.Years), 0, CAL,
        ql.Quarterly, ql.Following, ql.DateGeneration.Forward,
        ql.Actual360(), .40, discount_handle)
        for y, s in zip(tenors, spreads)]
    hazard = ql.PiecewiseFlatHazardRate(
        DATE, helpers, ql.Actual365Fixed())
    hazard.survivalProbability(DATE + ql.Period("5Y"))
    errors = [h.impliedQuote() - s for h, s in zip(helpers, spreads)]
    assert max(abs(e) for e in errors) < 1e-8
    survival = [hazard.survivalProbability(DATE + ql.Period(y, ql.Years))
                for y in range(6)]
    assert all(0 < q <= 1 for q in survival)
    assert all(a >= b for a, b in zip(survival, survival[1:]))
    print([round(1e4 * h.impliedQuote(), 6) for h in helpers])
# [80.0, 110.0, 140.0]
```

The [executed notebook](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/topics/11-credit-default-swaps/study.ipynb) then constructs `CreditDefaultSwap` and attaches `MidPointCdsEngine`. It exposes the complete ticket arguments, including accrued premium on default, payment at default, protection from the reference date, no accrual rebate and zero cash-settlement days.

![Calibrated risk-neutral survival and premium versus protection leg values for an illustrative CDS.](/assets/post_image/renovated/ficc/11-credit-default-swaps.png)
*Constructed spreads and 40% recovery; the positive protection-buyer PV belongs to a deliberately off-market 100 bp ticket.*

## Reconcile the ticket, not just the helpers

| Five-year ticket quantity | Result per 100 notional |
|---|---:|
| Premium paid, including accrued-on-default | 4.406400 |
| Protection received | 6.165073 |
| Buyer value | 1.758672 |
| Ticket fair running spread | 139.911772 bp |

The displayed values are rounded independently. The notebook reconstructs scheduled survival-weighted premiums, accrued premium at the midpoint of each default interval, and loss-given-default payments at that midpoint. Each leg and total PV agree with the engine within $10^{-10}$. Repricing the same ticket at its own fair spread gives zero value within that tolerance.

The fair ticket spread differs slightly from the five-year helper's 140 bp. The separate ticket's complete conventions are not asserted to equal every helper convention. That distinction prevents an unexplained difference from being mislabeled a calibration failure or a credit trading opportunity.

The [QuantLib midpoint engine](https://github.com/lballabio/QuantLib/blob/master/ql/pricingengines/credit/midpointcdsengine.cpp) approximates default timing within each accrual interval. This exercise validates that declared calculation, not the [ISDA Standard Model](https://www.cdsmodel.com/) or standard IMM contracts. All quotes remain illustrative; there is no issuer-specific credit inference or executable bid/ask comparison. The next substantive extension would need actual contract conventions, sourced credit quotes and a benchmark model comparison before claiming market accuracy.

[Read the topic note](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/topics/11-credit-default-swaps/README.md) · [Explore the executed notebook](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/topics/11-credit-default-swaps/study.ipynb) · [Browse all QuantLib desk examples](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/README.md)
