# Multi-Period Replenishment Optimization Model

## 1. Scope

The 2026 extension formulates a deterministic seven-day replenishment problem
for the 25-product portfolio selected at Store 274. Demand inputs are the
stockout-aware forecasts produced in `02_demand_analysis.ipynb`.

The model focuses on replenishment rather than price optimization. The public
dataset does not provide observed procurement costs, absolute selling prices,
inventory quantities, replenishment quantities, or experimentally identified
price-demand response. Capacity and inventory-carryover parameters introduced
below are therefore treated as explicit planning-scenario assumptions rather
than retailer-specific estimates.

The final decision rule uses a two-stage lexicographic linear program:

1. maximize total fulfilled forecast demand;
2. among maximum-service solutions, maximize the minimum weekly product-level
   fill rate.

A candidate inventory-minimization tie-breaker was evaluated separately but
was found to be redundant under the base scenario and is not part of the final
decision rule.

## 2. Sets and Indices

- \(i \in \mathcal{I}\): products, with \(|\mathcal{I}| = 25\)
- \(t \in \mathcal{T}\): planning days, with \(|\mathcal{T}| = 7\)

The planning horizon covers July 14–20, 2025.

## 3. Demand Inputs

Let

\[
d_{it} \ge 0
\]

denote the forecast demand for product \(i\) on day \(t\).

These demand values are generated in `02_demand_analysis.ipynb` using the
four-week weekday-mean forecasting rule selected through leakage-free
rolling-origin validation. The forecasting rule was fixed before evaluation
on the final July 14–20 holdout period. The forecast is based on
stockout-aware recovered demand rather than raw observed sales alone.

Demand is treated as deterministic within each optimization scenario.

## 4. Decision Variables

For each product \(i\) and day \(t\):

\[
x_{it} \ge 0
\]

is the replenishment quantity arriving at the beginning of day \(t\),

\[
s_{it} \ge 0
\]

is fulfilled demand during day \(t\), and

\[
I_{it} \ge 0
\]

is usable end-of-day inventory.

The Stage 2 fairness model additionally introduces

\[
0 \le z \le 1,
\]

where \(z\) is the minimum weekly product-level fill rate guaranteed across
the portfolio.

## 5. Scenario Parameters

The optimization uses the following planning parameters:

- \(\alpha \in [0,1]\): fraction of end-of-day inventory remaining usable on
  the following day;
- \(R_t \ge 0\): total replenishment capacity available on day \(t\);
- \(I_{i0} \ge 0\): usable initial inventory for product \(i\).

The dataset does not directly identify these quantities. They are therefore
specified transparently and varied through sensitivity analysis.

## 6. Scenario Calibration

Daily replenishment-capacity scenarios are anchored to empirical quantiles of
historical portfolio-level recovered demand using only dates for which all 25
study products are observed. This balanced calibration period contains 193
days spanning July 11, 2024 through July 13, 2025.

The four capacity scenarios are:

| Scenario | Historical Quantile | Daily Capacity |
|---|---:|---:|
| Tight | 50th percentile | 53.367548 |
| Base | 60th percentile | 56.867736 |
| Adequate | 75th percentile | 62.850277 |
| Relaxed | 90th percentile | 70.481525 |

These values provide reproducible planning scales. They are not estimates of
the retailer's actual replenishment capacity.

Inventory-carryover scenarios are:

| Scenario | \(\alpha\) |
|---|---:|
| High perishability | 0.60 |
| Base | 0.80 |
| No decay benchmark | 1.00 |

The base planning experiment therefore uses

\[
R_t = 56.867736
\qquad
\text{for all } t,
\]

\[
\alpha = 0.80,
\]

and

\[
I_{i0} = 0
\qquad
\text{for all } i.
\]

The zero-initial-inventory condition is a planning assumption rather than a
claim about the retailer's actual inventory position.

## 7. Inventory Dynamics

Replenishment \(x_{it}\) is assumed to arrive at the beginning of day \(t\)
and is therefore available to satisfy demand on that day.

For the first planning day,

\[
I_{i1}
=
I_{i0}
+
x_{i1}
-
s_{i1}.
\]

For subsequent days,

\[
I_{it}
=
\alpha I_{i,t-1}
+
x_{it}
-
s_{it},
\qquad
t=2,\ldots,T.
\]

The carryover factor \(\alpha\) represents the fraction of previous
end-of-day inventory that remains usable on the following day.

## 8. Demand-Fulfillment Constraints

Fulfilled demand cannot exceed forecast demand:

\[
0
\le
s_{it}
\le
d_{it},
\qquad
\forall i,t.
\]

No backlogging is allowed. Unfulfilled demand on one day is not transferred
to later days.

## 9. Shared Replenishment-Capacity Constraint

Total replenishment on each day cannot exceed the daily planning capacity:

\[
\sum_{i \in \mathcal{I}}
x_{it}
\le
R_t,
\qquad
\forall t.
\]

This shared constraint creates competition for replenishment capacity across
products and links the allocation problem to the intertemporal inventory
dynamics.

## 10. Stage 1: Maximum Total Service

The first optimization stage maximizes total fulfilled demand:

\[
S^*
=
\max
\sum_{i \in \mathcal{I}}
\sum_{t \in \mathcal{T}}
s_{it},
\]

subject to the inventory-balance, demand, capacity, and nonnegativity
constraints.

This objective identifies the maximum aggregate service level achievable
under the planning scenario.

Because the objective depends only on aggregate fulfilled demand, multiple
product-level allocations may attain the same value \(S^*\).

## 11. Stage 2: Max-Min Product Fairness

Define weekly forecast demand for product \(i\) as

\[
D_i
=
\sum_{t \in \mathcal{T}}
d_{it}.
\]

The second stage preserves the Stage 1 optimum:

\[
\sum_{i \in \mathcal{I}}
\sum_{t \in \mathcal{T}}
s_{it}
=
S^*.
\]

For every product,

\[
\sum_{t \in \mathcal{T}}
s_{it}
\ge
z D_i.
\]

The Stage 2 objective is

\[
z^*
=
\max z.
\]

Because

\[
z
\le
\frac{\sum_t s_{it}}{\sum_t d_{it}}
\]

for every product, maximizing \(z\) maximizes the minimum weekly product-level
fill rate.

The final decision rule therefore follows the lexicographic priority

\[
\text{maximum aggregate service}
\;\longrightarrow\;
\text{maximum minimum product service}.
\]

Aggregate efficiency is never sacrificed to improve fairness.

## 12. Candidate Inventory-Minimization Tie-Breaker

A candidate third objective was evaluated after fixing both

\[
\sum_{i,t}s_{it}=S^*
\]

and

\[
z=z^*.
\]

The proposed tie-breaker was

\[
\min
\sum_{i,t}
I_{it}.
\]

Under the base scenario, this objective did not reduce aggregate carried
inventory.

Summing the inventory-balance equations across all products and planning days
gives

\[
\sum_i\sum_t x_{it}
=
\sum_i\sum_t s_{it}
+
(1-\alpha)
\sum_i\sum_{t=1}^{T-1}
I_{it}
+
\sum_i I_{iT}
-
\sum_i I_{i0}.
\]

In the base solution, all seven daily replenishment-capacity constraints are
binding, so aggregate replenishment is fixed. Aggregate service is also fixed
at the Stage 1 optimum, initial inventory is zero, and aggregate terminal
inventory is zero. The aggregate inventory-carryover term is therefore fixed
as well.

The candidate objective consequently cannot reduce total carried inventory.
It selects a different product-day allocation from the same optimal set rather
than improving the aggregate inventory profile. It is retained as a model
diagnostic but omitted from the final optimization rule.

## 13. Capacity Shadow Values

Capacity sensitivity is analyzed using the Stage 1 maximum-service model.

For the constraint

\[
\sum_i x_{it}
\le
R_t,
\]

the capacity shadow value measures the local change in maximum fulfilled
demand associated with a small increase in \(R_t\).

Because the numerical solver minimizes the negative Stage 1 service objective,
the reported service shadow value is the negative of the solver marginal.

These values are interpreted only as local marginal quantities within the
specified deterministic planning scenario.

The solver dual values are also checked through finite-difference
perturbations of individual daily capacity limits.

## 14. Sensitivity Analysis

The service model is re-solved over the Cartesian product of the four
capacity scenarios and three carryover scenarios.

The primary sensitivity outcomes are:

- total fulfilled demand;
- portfolio fill rate;
- unmet forecast demand.

Allocation-specific inventory quantities are not treated as identified
sensitivity outcomes because the service-only Stage 1 objective may admit
multiple optimal replenishment and inventory allocations once maximum service
has been achieved.

The sensitivity analysis therefore focuses on service-side consequences of
capacity scarcity and inventory carryover rather than on arbitrary
solver-selected allocations.

## 15. Myopic Policy Benchmark

The optimized multi-period plan is compared with a proportional same-day
replenishment policy.

For day \(t\), define total forecast demand as

\[
D_t
=
\sum_i d_{it}.
\]

The myopic allocation factor is

\[
q_t
=
\min
\left(
1,
\frac{R_t}{D_t}
\right).
\]

The benchmark fulfills

\[
s^{\text{myopic}}_{it}
=
q_t d_{it}.
\]

The policy therefore allocates capacity proportionally across current-day
demand but does not build inventory in anticipation of future demand.

This benchmark is designed to isolate the value of intertemporal planning
rather than compare the optimization model with an intentionally weak policy.

## 16. Interpretation Boundary

All demand, replenishment, inventory, and capacity quantities are expressed
in normalized planning units.

The model does not estimate or claim:

- actual Dingdong procurement costs;
- actual retail prices or profits;
- actual replenishment capacity;
- actual starting inventory;
- actual spoilage or shelf-life parameters;
- deployed retailer decisions.

The carryover and capacity values are explicit scenario assumptions used to
study the structure and implications of a replenishment decision problem.

Optimization improvements relative to the myopic benchmark are therefore
model-implied results under the stated planning scenario rather than observed
business-performance gains.