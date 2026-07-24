# Data Setup

This project uses the public FreshRetailNet-LT retail dataset.

Raw source files are not included in this repository. Processed project files
are also excluded from version control so that the analysis can be reproduced
from the documented notebook pipeline.

## Expected Directory Structure

Place the source parquet files locally under:

```text
data/
└── raw/
    ├── train.parquet
    └── eval.parquet
```

The analysis notebooks generate:

```text
data/
└── processed/
    ├── study_portfolio.csv
    ├── recovered_demand.csv
    └── demand_forecast.csv
```

## Processing Pipeline

Run the notebooks sequentially.

### 1. Dataset Audit

```text
notebooks/01_dataset_audit.ipynb
```

This notebook audits the source data, applies the prespecified eligibility
criteria, selects Store 274, and fixes the 25-product study portfolio.

It creates:

```text
data/processed/study_portfolio.csv
```

### 2. Demand Analysis

```text
notebooks/02_demand_analysis.ipynb
```

This notebook develops and validates the stockout-aware demand-recovery
procedure, performs leakage-free forecast validation, evaluates the final
holdout period, and generates the seven-day demand forecast.

It creates:

```text
data/processed/recovered_demand.csv
data/processed/demand_forecast.csv
```

### 3. Replenishment Optimization

```text
notebooks/03_optimization.ipynb
```

This notebook consumes the processed demand forecast and reproduces the
multi-period replenishment optimization, fairness analysis, shadow-value
analysis, sensitivity experiments, and myopic-policy benchmark.

## Data Fields

The project uses only a subset of the source dataset fields.

See:

```text
docs/dataset_notes.md
```

for the variables used and their role in the analysis.

## Version-Control Policy

The following directories are intentionally excluded from Git tracking:

```text
data/raw/
data/processed/
```

This repository therefore contains the complete analysis code and
documentation but does not redistribute the source or derived datasets.