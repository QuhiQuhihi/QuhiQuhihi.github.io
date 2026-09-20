---
title: "Cross-currency swaps: a cash-flow ledger before the pricing engine"
author: daham
date: 2022-11-02 12:00:00 +0800
last_modified_at: 2026-09-20 21:00:00 +0900
categories: [FICC Quant]
tags: [investment, derivatives, QuantLib]
render_with_liquid: false
math: true
---

A cross-currency swap combines two interest-rate exposures with an exchange-rate exposure. Before choosing a curve engine, it helps to write the actual currency ledger, including principal exchanges. A position can have zero value at inception and still acquire substantial value when spot changes.

The maintained example uses a **five-year fixed-for-fixed USD/EUR swap** to isolate that mechanism. The remaining USD leg is worth 110 and the remaining EUR leg is worth 100 in their respective currencies. At spot 1.10 USD per EUR they offset. Increasing the conversion spot to 1.20, while holding curves and contractual cash flows fixed, gives the remaining position a value of **−10 USD**.

## State the direction and currency units

The holder receives EUR 100 and pays USD 110 at inception. During the contract, the holder pays EUR coupons and receives USD coupons. At maturity, EUR 100 is returned and USD 110 is received. Those are contractual amounts; a scenario does not reset them to a new spot rate.

| Date | USD received | EUR received |
|---|---:|---:|
| Inception | −110.000000 | +100.000000 |
| Each year, 1–4 | +4.489185 | −2.531512 |
| Year 5, including principal | +114.489185 | −102.531512 |

These cash flows use **illustrative flat continuous USD 4% and EUR 2.5% rates**, annual exact-year payments, zero basis and compatible collateral assumptions. They are not a market cross-currency basis quote set. The construction deliberately avoids resettable notionals and floating-index conventions so the principal and conversion signs remain visible.

For each currency $c$, a par annual coupon follows from

$$k_c=\frac{1-D_c(T)}{\sum_{i=1}^{5}D_c(i)}.$$

Immediately after the initial exchange, the remaining USD value is

$$V(S)=PV_{USD\ leg}-S\,PV_{EUR\ leg},$$

where $S$ is USD per EUR. Initial exchanges balance at inception and are not reinserted into a later remaining-cash-flow valuation.

## Use QuantLib discount factors to check the ledger

The topic notebook evaluates the exact exponential discount factors directly. This compact adaptation uses QuantLib flat curves for the same assumptions and reconciles them with that formula. Run it from the maintained project root; the date context restores the global setting afterward. Times are exact year fractions here, so no currency holiday schedules are implied.

```python
import numpy as np
import QuantLib as ql
from research.pricing import DATE, valuation_date

with valuation_date():
    dc = ql.Actual365Fixed()
    usd = ql.FlatForward(DATE, .04, dc)
    eur = ql.FlatForward(DATE, .025, dc)
    times = np.arange(1, 6, dtype=float)
    d_usd = np.array([usd.discount(float(t)) for t in times])
    d_eur = np.array([eur.discount(float(t)) for t in times])
    assert np.allclose(d_usd, np.exp(-.04 * times), atol=1e-12, rtol=0)
    assert np.allclose(d_eur, np.exp(-.025 * times), atol=1e-12, rtol=0)
    k_usd = (1 - d_usd[-1]) / d_usd.sum()
    k_eur = (1 - d_eur[-1]) / d_eur.sum()
    pv_usd = 110 * (k_usd * d_usd.sum() + d_usd[-1])
    pv_eur = 100 * (k_eur * d_eur.sum() + d_eur[-1])
    print(round(pv_usd - 1.10 * pv_eur, 6),
          round(pv_usd - 1.20 * pv_eur, 6))
# 0.0 -10.0
```

![Remaining cross-currency cash-flow value as USD per EUR spot changes.](/assets/post_image/renovated/ficc/10-cross-currency-swaps.png)
*Fixed coupons, notionals and curves remain unchanged. The plot is a spot-conversion scenario for the remaining legs, not a historical return series.*

## Separate currency risk from a basis calibration

The annual par coupons are **4.081077% USD** and **2.531512% EUR**. Both legs independently price at par. A finite-difference spot derivative agrees with $\partial V/\partial S=-PV_{EUR}=-100$, measured as USD value per unit change in USD/EUR. The example also increases the contractual EUR coupon by 10 bp: the resulting change is **−0.510578 USD**. That is an annuity calculation, not an estimated market basis spread.

The [FX-forward chapter](https://github.com/QuhiQuhihi/project_FICC_Quant/tree/main/topics/09-fx-forwards) provides a related consistency check. For a forward receiving one EUR at maturity and paying $K$ USD, $PV=S D_{EUR}-K D_{USD}$ and $F=S D_{EUR}/D_{USD}$. Its separate OIS-based illustration produces a one-year forward of **1.114175 USD/EUR** from spot 1.10. It uses a different USD curve from this flat-rate CCS exercise, so the numbers should not be mixed.

Real cross-currency valuation requires collateral currency, basis quotes, settlement, fixing rules, principal resets and funding conventions. This simple ledger does not calibrate those objects or establish executable covered-interest arbitrage. Its contribution is a validated set of signs, units and cash-flow identities that can be retained when a richer instrument is introduced. The [project source register](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/research/SOURCES.md) identifies the reference material and the boundaries of the constructed inputs.

[Read the topic note](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/topics/10-cross-currency-swaps/README.md) · [Explore the executed notebook](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/topics/10-cross-currency-swaps/study.ipynb) · [Browse all QuantLib desk examples](https://github.com/QuhiQuhihi/project_FICC_Quant/blob/main/README.md)
