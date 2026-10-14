# Methods

## Workplace socioeconomic segregation

For each 1 km work grid p, aggregate sample counts for Q1–Q4. Let v_qp be group q's fraction of workers. Calculate S_p = (2/3) sum_q |v_qp - 1/4|. Absent groups contribute zero shares. The score ranges from 0 for equal representation to 1 for a single group. Retain work grids with at least 30 eligible workers. City workplace segregation is the unweighted median of retained grid scores; the unweighted mean is also exported with a distinct name.

## Worker-experienced socioeconomic segregation

Assign each worker the score of their work grid. Within each demographic or SES group w, calculate E_w = sum_p n_wp S_p / sum_p n_wp using retained grids only. Counts are sample worker counts, not expansion weights. The whole-city worker mean equals the worker-count-weighted mean of grid scores, not generally the grid median. Exported eligible and retained counts identify the analysis population.

## Housing-price interpolation and SES

Prepare community observations and residential 250 m centroids in the same city-specific projected CRS in metres. Remove missing/non-numeric coordinates and prices, non-positive prices and duplicate community-name/coordinate records. Use the 12 nearest observations, inverse squared distance weights and a 1e-10 m floor at zero distance; use all observations if fewer than 12 are available. No maximum search radius is imposed.

Rank home grids by interpolated price. Quartile boundaries are the lowest prices at which cumulative resident commuter counts reach 25%, 50% and 75% of the city total. Assign Q1 at or below the first boundary, Q2 at or below the second, Q3 at or below the third, and Q4 above it. Price ties stay together and group sizes need not be exactly equal. These groups measure relative SES, not measured income.

## Regression

Use one observation per retained work grid, with POI_count >= 10. Log-transform commuting distance, city-center distance, nearest-subway distance, road length, worker count and POI count using log(1+x). Include POI entropy and the three sectoral counts without log transformation. Standardize all ten predictors using the pooled eligible sample mean and sample standard deviation before dividing into East, Central, West and Northeast.

Fit each regional model by OLS with an intercept and city indicators. Report HC1 heteroskedasticity-robust covariance, t-based inference and residual degrees of freedom that include city effects. These are fixed-effects models, not random-effects models. Commuting distance in the regression is the prepared Euclidean-distance variable; network routing applies to the counterfactual stage.

The default missing-data policy is complete-case analysis. `missing_regression=legacy_zero` provides predictor-only zero imputation for compatibility; missing outcomes are always excluded. Explicitly select the policy when using research data. Collinear or undersized regional designs raise an error.

LMG/Shapley importance averages each predictor's incremental R-squared across all subsets after subtracting city means from outcome and predictors. The ten contributions sum to within-city R-squared. This decomposition describes explained variation and is not causal identification.

## Walking counterfactual

For each eligible home grid, evaluate Amap walking routes to every other 250 m grid in the supplied city domain. Route alternatives use minimum reported duration. Retain destinations at or below 900 s, storing distance in metres as well. Exclude the home grid. Supply GCJ-02 coordinates explicitly; no implicit datum conversion is performed.

Assign each individual independently and uniformly to a feasible destination. A multinomial draw for aggregate flow counts has exactly this distribution without materializing individual records. Preserve home, SES and demographic counts; map each simulated 250 m destination to its 1 km work grid and recompute segregation. Workplace capacities and occupational compatibility are not constrained.

The default is ten replicates with seeds 20261012 through 20261021. The replicate count, starting seed and walking threshold are configurable. Mean, standard deviation, minimum and maximum describe Monte Carlo assignment variation, not sampling confidence intervals. Retention sets can differ between observed and simulated scenarios after the 30-worker threshold; retained counts accompany each result.

## Parameter configuration

`config.json` contains minimum_workers=30, minimum_pois=10, idw_neighbors=12, idw_power=2, walking_seconds=900, seed=20261012, replicates=10 and missing_regression=drop. Functions expose their core numerical parameters. The package does not infer unspecified data-provider permissions or historic routing conditions.
