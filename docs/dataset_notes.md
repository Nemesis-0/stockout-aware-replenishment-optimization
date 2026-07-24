# Dataset Notes

## Source

This project uses the public **FreshRetailNet-LT** dataset released by
Dingdong-Inc.

FreshRetailNet-LT contains daily and hourly sales information for perishable
retail products together with hourly stockout annotations and contextual
covariates.

The project uses the dataset's two parquet splits:

```text
data/raw/train.parquet
data/raw/eval.parquet
```

The raw files are stored locally and excluded from version control.

## Dataset Scale

The source dataset contains:

- 7,869,549 rows in the training split;
- 70,000 rows in the evaluation split;
- 1,057 stores;
- 18 cities;
- 576 perishable products;
- more than 20,000 store-product time series.

Each row represents one store-product-date observation.

The project verifies the analytical structure and integrity of the source data
in `notebooks/01_dataset_audit.ipynb` before constructing the study
population.

## Fields Used Directly in the Analysis

### `store_id`

Encoded store identifier.

It is used to define store-level portfolios and ultimately to select the fixed
study store used in the downstream analysis.

### `product_id`

Encoded product identifier.

The primary analytical series is defined at the store-product level.

### `dt`

Calendar date of the observation.

It is used for:

- temporal coverage checks;
- weekday construction;
- chronological train-validation splits;
- rolling-origin forecast validation;
- the final seven-day planning horizon.

### `sale_amount`

Daily sales amount after the dataset's global normalization.

The project treats this quantity as observed sales rather than automatically
interpreting it as latent demand. When stockouts occur, observed sales may
understate the quantity that would have been sold under full availability.

All downstream demand, inventory, replenishment, and optimization quantities
therefore remain in normalized units.

### `hours_sale`

Length-24 sequence of normalized hourly sales.

The project verifies that the sum of the 24 hourly values agrees with
`sale_amount` up to floating-point precision.

Hourly sales are used to construct the stockout-aware demand-recovery
procedure.

### `stock_hour6_22_cnt`

Number of stockout-labeled hours between 06:00 and 22:00.

This field is used to measure operating-period stockout severity and to
distinguish:

- fully uncensored days;
- moderately censored days;
- severely censored days.

The final demand-recovery rule uses hourly-share recovery for days with
1–12 stockout-labeled operating hours and a historical fallback under more
severe censoring.

### `hours_stock_status`

Length-24 sequence of hourly stockout indicators.

The project uses the 06:00–22:00 portion of this sequence as the operating
period for stockout-aware reconstruction.

The notebook verifies that the operating-period sum of
`hours_stock_status` agrees with `stock_hour6_22_cnt`.

Stockout-labeled hours are treated as indicators of potential demand
censoring rather than as direct observations of missing latent demand.

## Product-Category Fields

The source data also contain encoded product hierarchy fields:

- `management_group_id`;
- `first_category_id`;
- `second_category_id`;
- `third_category_id`.

These fields are used descriptively during dataset auditing and study
portfolio characterization.

They are not treated as economic product attributes and do not enter the
final replenishment objective.

## Location Field

### `city_id`

Encoded city identifier.

This field is retained for dataset-structure auditing and descriptive checks.
The final optimization experiment is conducted within a single selected store,
so city is not an optimization decision variable.

## Contextual Fields Available in the Dataset

FreshRetailNet-LT also provides:

- `discount`;
- `holiday_flag`;
- `activity_flag`;
- `precpt`;
- `avg_temperature`;
- `avg_humidity`;
- `avg_wind_level`.

`discount` is expressed as a multiplier, where 1.0 indicates no discount and
0.9 indicates a 10% discount.

These variables are available as contextual covariates but are not used as
decision variables in the final 2026 replenishment optimization model.

In particular, `discount` is not interpreted as an absolute selling price.
The dataset does not provide the absolute prices, procurement costs, or
identified price-demand response required for a defensible empirical pricing
optimization model.

## Stockout Interpretation

Observed sales and latent demand are distinguished throughout the project.

On a fully available day, observed sales provide the cleanest available demand
measurement.

On a stockout day, observed sales may be censored because customers cannot
purchase unavailable inventory.

The project therefore constructs a stockout-aware recovered-demand proxy
before forecasting.

The recovered quantity is an estimated analytical target, not directly
observed ground-truth latent demand.

## Operating Period

The stockout-recovery analysis focuses on the 16 hourly positions from
06:00 through 21:59, corresponding to indices:

```text
6:22
```

in the 24-element hourly arrays.

Sales outside this operating period are retained as observed rather than
reconstructed.

## Fixed Study Population

The downstream analysis uses a study population selected in
`01_dataset_audit.ipynb`.

Eligibility is determined before demand-recovery and forecasting outcomes are
examined.

The final portfolio consists of:

```text
Store 274
25 products
```

All eligible products at the selected store are retained; products are not
subsequently removed based on forecasting or optimization performance.

The selected portfolio is written to:

```text
data/processed/study_portfolio.csv
```

## Processed Demand Outputs

`02_demand_analysis.ipynb` generates:

```text
data/processed/recovered_demand.csv
```

which contains the stockout-aware historical demand proxy, and:

```text
data/processed/demand_forecast.csv
```

which contains the final 25-product by 7-day demand forecast used by the
optimization model.

These processed files are generated locally and excluded from Git tracking.

## Interpretation Boundary

The dataset does not directly provide:

- absolute retail prices;
- procurement costs;
- starting inventory quantities;
- replenishment quantities;
- replenishment-capacity limits;
- product-specific spoilage rates or shelf lives.

These quantities are therefore not presented as observed retailer data.

Parameters introduced later in the optimization model, including
replenishment capacity and inventory carryover, are explicitly defined as
planning-scenario assumptions.

See `docs/model_specification.md` for the mathematical model and the complete
optimization interpretation boundary.