# Loyalty Agent Model

An agent-based model (ABM) of a competitive cafe market used to study how different
**loyalty program designs** affect customer behavior, cafe profitability, and market
share. The model is built in [NetLogo](https://ccl.northwestern.edu/netlogo/) 7.0.4,
and BehaviorSpace experiment results are analyzed with a Python data-science stack
(pandas / scikit-learn / statsmodels).

## Repository structure

| Path | Description |
|---|---|
| `loyalty-cafes.nlogox` | **Current/canonical model.** Unifies all loyalty-program and network variants behind sliders/choosers so a single model can be swept via BehaviorSpace. |
| `archive/` | Older/deprecated model versions and one-off experiment outputs. |
| `loyalty-cafes-simple-experiment-*.csv` | Raw BehaviorSpace output for the "Simple experiment" (spreadsheet/table/lists/stats formats). Generated locally, **git-ignored**. |
| `process_experiments_results.py` | Standalone script version of the sensitivity analysis (Random Forest + permutation importance). |
| `behaviorspace_sensitivity_analysis.ipynb` | Notebook analyzing the **Spreadsheet/Table** BehaviorSpace export; computes permutation importance per KPI and saves CSV summaries to the repo root. |
| `behaviorspace_table_sensitivity_analysis.ipynb` | More complete notebook analyzing the **Table** export: descriptive stats, per-parameter sensitivity plots, Random Forest permutation importance, standardized linear regression coefficients, and OLS p-values. Saves figures/tables to `analysis_outputs/`. |
| `analysis_outputs/` | Charts (`parameter_importance.png`, `standardized_coefficients.png`) and CSV summaries produced by the notebooks. |
| `netlogo-style-guide.skill` | Claude Code skill bundle with NetLogo coding conventions used when editing `.nlogox` code. |

## The model

Five `cafes` compete for `customers` who decide daily how many coffees to buy and
which cafe to visit.

**Cafes** (`cafes-own`): `business-model` (`mass-market` vs `premium`, with different
price/unit-cost/attractiveness/capacity), and a `loyalty-type` (one per cafe, 0–4):

- `0` — no loyalty program
- `1` — bonus points
- `2` — cashback
- `3` — free 6th cup
- `4` — tiered rewards (rate increases after 10 visits)

**Customers** (`customers-own`) are assigned a `segment` at creation, each with
different sensitivities to price, attractiveness, loyalty rewards, and habit:
`price-sensitive`, `quality-oriented`, `bonus-hunter`, `habit-loyal`, `random`.

Each tick (`go`), every customer decides how many cups to buy
(`purchase-probability`, driven by `base-purchase-probability` and `satisfaction`),
scores all cafes with capacity left (`cafe-score`: attractiveness, price, loyalty
reward, habit, and word-of-mouth recommendation), and picks one of the top 3.
Visiting a cafe updates that cafe's revenue/profit/loyalty-cost and the customer's
`satisfaction`, and may trigger a `recommend-cafe` to their friends over the
customer social network.

**Customer social network** (`setup-customers` / `network-type` chooser):
- `no network` — customers are unconnected (no word-of-mouth).
- `preferential attachment` — `nw:generate-preferential-attachment`.
- `small world` — `nw:generate-watts-strogatz`.

Key sweepable parameters (sliders in the model, and the BehaviorSpace inputs used by
the Python analysis): `number-of-customers`, `base-purchase-probability`,
`satisfaction-purchase-factor`, `loyalty-effect-multiplier`,
`recommendation-probability`, `recommendation-boost`, `mass-market-capacity`,
`premium-capacity`, `network-type`.

Two BehaviorSpace experiments are defined inside `loyalty-cafes.nlogox` (via
NetLogo's Tools → BehaviorSpace): **"Full"** (wider parameter grid) and **"Simple
experiment"** (smaller grid, used for the CSVs in this repo), both running 365 ticks
with 10 repetitions per parameter combination.

## Running the model

1. Open `loyalty-cafes.nlogox` in NetLogo 7.0.4 (requires the bundled `nw` extension).
2. Click `setup`, then `go` to run interactively, or
3. Tools → BehaviorSpace → run the **"Simple experiment"** (or **"Full"**) experiment
   and export as **Table** output — this produces
   `loyalty-cafes-simple-experiment-table.csv`, matching what the notebooks expect.

## Python analysis

Dependencies: `pandas`, `numpy`, `matplotlib`, `scikit-learn`, `statsmodels`,
`openpyxl`.

```bash
pip install pandas numpy matplotlib scikit-learn statsmodels openpyxl
```

Then open `behaviorspace_table_sensitivity_analysis.ipynb` (the primary/most complete
notebook) in Jupyter. It expects the BehaviorSpace **Table** CSV
(`loyalty-cafes-simple-experiment-table.csv`, set via `DATA_PATH`) in the same
directory, and walks through:

1. Loading and validating the BehaviorSpace table export (data starts at row 7).
2. Descriptive statistics and KPI distributions.
3. Per-parameter sensitivity plots and grouped summaries (e.g. by `network-type`).
4. Random Forest + permutation importance per KPI.
5. Standardized linear regression coefficients and OLS p-values.
6. Exporting figures and summary tables to `analysis_outputs/`.

KPIs analyzed include total/per-cafe `profit`, `revenue`, `loyalty-cost`, `visits`,
`lost-visits`, and mean customer `satisfaction`.

`behaviorspace_sensitivity_analysis.ipynb` and `process_experiments_results.py` are
earlier/lighter versions of the same analysis (Random Forest permutation importance
only) and can be used as quick reference or a plain-script alternative to the notebook.
