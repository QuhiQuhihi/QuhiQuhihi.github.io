---
title: "Defensive Allocation: Separate the Warning Signal from the Holdings"
author: daham
date: 2022-10-02 12:00:00 +0800
last_modified_at: 2026-09-20 21:00:00 +0900
categories: [Asset Allocation]
tags: [asset allocation, defensive allocation, momentum]
render_with_liquid: false
use_math: true
math: true
---

An asset can be useful to monitor without being the asset a strategy wants to hold. Defensive Asset Allocation separates those roles: momentum chooses the investment candidates, while a designated canary universe controls the amount of defensive exposure.

That separation has a concrete consequence. In the constructed study here, weakening one canary moves the portfolio to **50% defense** even though every risky candidate is unchanged. When the risky candidates deteriorate but the canaries remain positive, the same rule stays fully exposed. Both responses follow the specified detector. Whether they improve investment outcomes is a separate empirical question.

### Give each universe a clear job

The design has three logical roles:

| Universe | Role in the decision | Constructed example |
|---|---|---|
| Risky | Supplies prospective holdings | US equity, global equity, real estate and gold |
| Canary | Determines the exposure budget | Emerging-equity and aggregate-bond warning paths |
| Defensive | Receives the defensive budget | Bills and Treasury exposure |

The [original DAA paper](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3212862) introduced a separate warning universe as an extension of VAA. The project uses disjoint sets to make those roles visible. Disjointness is a property of this exercise; an instrument can serve multiple roles in other specifications.

Every asset receives the same price-momentum score:

$$M_t=12R_{1,t}+4R_{3,t}+2R_{6,t}+R_{12,t},\qquad R_{h,t}=\frac{P_t}{P_{t-h}}-1.$$

If $b_t$ of the two canaries have non-positive scores, the example sets the defensive fraction to

$$d_t=\min\left(1,\frac{b_t}{2}\right).$$

The remaining capital is divided equally between the two highest-ranked risky assets. The strongest defensive asset receives the defensive budget. Risky selection is based on relative rank: a weak risky score does not independently veto a holding while the canaries remain healthy.

The threshold of two and this fractional mixing rule are declared educational choices. They are not universal DAA parameters or a complete replication of the paper's portfolio-slot conventions. The canaries echo emerging-equity and aggregate-bond economic roles, but every input path here is constructed rather than downloaded ETF history.

### Change the warning while holding the candidates fixed

The [worked notebook](https://github.com/QuhiQuhihi/project_Asset_Allocation/blob/main/topics/08-defensive-allocation/study.ipynb) creates four controlled scenarios. They isolate the difference between observing the risky universe and observing canaries.

![Defensive allocations from canary and risky-universe counts under four constructed scenarios](/assets/post_image/renovated/allocation/defensive-allocation.png)

*Constructed signal comparison. Both detectors use a breadth threshold of two. The different allocations arise from monitoring different sets of assets, not from an estimated performance advantage.*

| Scenario | Weak risky assets | Weak canaries | Canary-rule defense | Risky-universe-rule defense |
|---|---:|---:|---:|---:|
| All positive | 0 | 0 | 0% | 0% |
| One weak canary | 0 | 1 | 50% | 0% |
| Two weak canaries | 0 | 2 | 100% | 0% |
| Risk weak; canaries positive | 4 | 0 | 0% | 100% |

In the first scenario, US and global equity receive 50% each. Weakening one canary leaves their relative ranking unchanged but reduces their weights to 25% each, with the other 50% in the selected Treasury exposure. With two weak canaries the portfolio is fully defensive.

The final scenario is equally instructive. All four risky scores turn negative, yet the canary detector remains positive. The rule still allocates to the two strongest risky candidates, because the strategy delegated the exposure decision to a different universe. Adding an absolute veto now would create another strategy variant and should be recorded as such.

The notebook checks the score against independent constant-growth arithmetic, verifies the nonnegative budget, confirms that changing a canary leaves risky rankings unchanged, and ensures that future prices cannot change the original decision. These are controlled mechanism checks, not a backtest of crisis protection.

### Treat the canary choice as a model choice

A canary may weaken before a loss in the investment universe, after that loss, or without a relevant loss occurring. It can also stay negative while investment candidates recover. A useful evaluation needs false alarms, missed losses, recovery participation and turnover, rather than only a dramatic episode that the detector happened to anticipate.

Choosing canaries after reviewing many crises is itself a search process. A small final pair can conceal a large family of earlier alternatives. Correlated canaries may provide less independent evidence than their count suggests, and changes in their economic exposures can alter the relationship being relied upon.

Defensive holdings require equal attention. A bond canary and a Treasury defensive holding can both react to rising rates; moving into the defensive sleeve does not remove duration risk. Compare the actual replacement assets, execution delay and costs under the same dates and constraints.

The [backtest-selection laboratory](https://github.com/QuhiQuhihi/project_Asset_Allocation/blob/main/topics/11-backtest-selection/README.md) makes search bias explicit. For another meaning of protection, [CPPI](https://github.com/QuhiQuhihi/project_Asset_Allocation/blob/main/topics/18-cppi/README.md) sets risky exposure from the cushion above a wealth floor and demonstrates gap risk. [Liability-driven allocation](https://github.com/QuhiQuhihi/project_Asset_Allocation/blob/main/topics/16-liability-driven-allocation/README.md) starts from dated obligations rather than a momentum warning.

Read the [DAA research note](https://github.com/QuhiQuhihi/project_Asset_Allocation/blob/main/topics/08-defensive-allocation/README.md), [executed notebook](https://github.com/QuhiQuhihi/project_Asset_Allocation/blob/main/topics/08-defensive-allocation/study.ipynb), and [allocation collection](https://github.com/QuhiQuhihi/project_Asset_Allocation/blob/main/README.md). The examples explain the choices and failure conditions while leaving an empirical claim for a separately specified test.
