# Fit weights using post-stratification

Adjusts survey weights so that the weighted cell counts (or proportions)
match known population values for every joint combination of
stratification variables. Unlike
[`calibrate_linear()`](https://jdenn0514.github.io/surveywts/reference/calibrate_linear.md)
and
[`calibrate_rake()`](https://jdenn0514.github.io/surveywts/reference/calibrate_rake.md),
which match marginal totals, `poststratify()` matches exact
cross-tabulation cells in a single pass.

## Usage

``` r
poststratify(
  data,
  targets,
  weights = NULL,
  wt_name = NULL,
  type = c("prop", "count"),
  reference_design = NULL
)
```

## Arguments

- data:

  A `survey_nonprob`, `survey_taylor`, or `survey_replicate`. Any other
  class -\> error. When `data` is a `survey_replicate`,
  post-stratification is applied independently to every replicate weight
  column using the same population `targets`. Replicate columns where
  any cell has zero or negative total weight fail calibration; a
  `surveywts_warning_replicate_calibration_failed` warning is emitted
  and the original replicate weights are kept.

- targets:

  A `data.frame` with one column per stratification variable (column
  names must match column names in `data`), plus a column named
  `"target"`, and one row per unique cell combination. The
  stratification variables are automatically identified as
  `setdiff(names(targets), "target")`.

  Unlike
  [`calibrate_linear()`](https://jdenn0514.github.io/surveywts/reference/calibrate_linear.md)
  and
  [`calibrate_rake()`](https://jdenn0514.github.io/surveywts/reference/calibrate_rake.md),
  `targets` must be a `data.frame` — named lists are not accepted
  (`surveywts_error_margins_format_invalid`). The `targets` data frame
  must have at least one non-`"target"` column
  (`surveywts_error_no_strata_variables` if zero).

  For `type = "count"`: values in `target` must be strictly positive.
  For `type = "prop"`: values in `target` must sum to 1.0 (within
  `1e-6`).

- weights:

  \<[`tidy-select`](https://tidyselect.r-lib.org/reference/language.html)\>
  Weight column name (bare name). `NULL` -\> auto-detected from survey
  object `@variables$weights`.

- wt_name:

  `NULL` (default) or a character scalar. When `NULL`, calibrated
  weights overwrite the existing weight column in place. When a
  character string, a new column is added and `@variables$weights`
  updated.

- type:

  Character scalar. `"prop"` (default): `target` values are proportions
  summing to 1.0. `"count"`: `target` values are population counts.
  Consistent with
  [`calibrate_linear()`](https://jdenn0514.github.io/surveywts/reference/calibrate_linear.md)
  and
  [`calibrate_rake()`](https://jdenn0514.github.io/surveywts/reference/calibrate_rake.md).

- reference_design:

  A `survey_taylor` object or `NULL`. The probability survey from which
  `targets` were estimated. When non-`NULL`, stored in the history entry
  with `targets_from_reference = TRUE`. Any non-`NULL`
  non-`survey_taylor` value triggers
  `surveywts_error_reference_design_not_taylor`.

## Value

- `survey_taylor` or `survey_nonprob` input -\> same class as input
  (class preserved); `@calibration` slot is populated

- `survey_replicate` input -\> `survey_replicate` (class preserved);
  `@calibration` slot is populated, including `replicate_converged`

The weight column in the output contains post-stratified weights. A
history entry with `operation = "poststratify"` is appended to
`weighting_history`. For `survey_replicate` inputs, each replicate
weight column is also post-stratified. Failed replicates retain their
original weights and are recorded in
`output@calibration$replicate_converged` as `FALSE`.

## Details

**When to use.** Choose post-stratification when you have population
values for every joint cell of the stratification variables and every
cell contains sample members. A populated cell with no sample members
makes the adjustment undefined (Deville & Sarndal 1992). With several
variables the cells thin out quickly; when cells run empty, match
margins instead via
[`calibrate()`](https://jdenn0514.github.io/surveywts/reference/calibrate.md).

## Algorithm

Within each cell \\h\\ defined by the joint combination of
stratification variables, the calibration factor is \$\$c_h =
\frac{T_h}{W_h}\$\$ where \\T_h\\ is the target cell total (population
count or proportion scaled to population size) and \\W_h = \sum\_{k \in
h} w_k\\ is the sum of current weights in cell \\h\\. The calibrated
weight for each unit in cell \\h\\ is \\w_k^\* = c_h \cdot w_k\\. The
solution is exact in one pass — no iteration is required.

## References

Valliant, R. (1993). Poststratification and conditional variance
estimation. *Journal of the American Statistical Association*, 88(421),
89–96.

Deville, J.-C. and Sarndal, C.-E. (1992). Calibration estimators in
survey sampling. *Journal of the American Statistical Association*,
87(418), 376–382.

Rao, J. N. K., Yung, W. and Hidiroglou, M. A. (2002). Estimating
equations for the analysis of survey data using poststratification
information. *Sankhya*, 64(2), 364–378.

Deville, J.-C., Sarndal, C.-E. and Sautory, O. (1993). Generalized
raking procedures in survey sampling. *Journal of the American
Statistical Association*, 88(423), 1013–1020.

Rao, J. N. K., Wu, C. F. J. and Yue, K. (1992). Some recent work on
resampling methods for complex surveys. *Survey Methodology*, 18(2),
209–217.

## See also

[`calibrate()`](https://jdenn0514.github.io/surveywts/reference/calibrate.md),
[`calibrate_rake()`](https://jdenn0514.github.io/surveywts/reference/calibrate_rake.md),
[`calibrate_linear()`](https://jdenn0514.github.io/surveywts/reference/calibrate_linear.md),
[`calibrate_logit()`](https://jdenn0514.github.io/surveywts/reference/calibrate_logit.md).
For the class system, the standard workflows, and a glossary of terms,
see the [Getting started
article](https://jdenn0514.github.io/surveywts/articles/getting-started.html).

Other calibration:
[`calibrate()`](https://jdenn0514.github.io/surveywts/reference/calibrate.md),
[`calibrate_linear()`](https://jdenn0514.github.io/surveywts/reference/calibrate_linear.md),
[`calibrate_logit()`](https://jdenn0514.github.io/surveywts/reference/calibrate_logit.md),
[`calibrate_rake()`](https://jdenn0514.github.io/surveywts/reference/calibrate_rake.md)

## Examples

``` r
ns_wave1_svy <- surveycore::as_survey_nonprob(ns_wave1, weights = weight)

# joint cell proportions (sex x age_f3, 6 cells, sum = 1.000) --------
ps_cells <- data.frame(
  sex    = rep(c("Male", "Female"), each = 3),
  age_f3 = rep(c("18-34", "35-54", "55+"), times = 2),
  target = c(0.1470, 0.1617, 0.1813, 0.1530, 0.1683, 0.1887)
)
result <- poststratify(ns_wave1_svy, targets = ps_cells)
summarize_weights(result)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv     min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl>   <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  6422       6422      0  1.00  1.36 0.00362 0.152 0.397  1.13  5.20 2245.

# type = "count" with US adult population counts (260 million) ------------
ps_counts <- data.frame(
  sex    = rep(c("Male", "Female"), each = 3),
  age_f3 = rep(c("18-34", "35-54", "55+"), times = 2),
  target = c(0.1470, 0.1617, 0.1813, 0.1530, 0.1683, 0.1887) * 260000000
)
poststratify(ns_wave1_svy, targets = ps_counts, type = "count")
#> # A calibrated survey design: 6,422 observations, 185 variables
#> # Variance: model-assisted (SRS assumption)
#> # IDs: ~1 | Strata: NULL | Weights: weight 
#> # Weighting history: 1 step 
#> #   Step 1 [2026-09-10]: poststratify (strata: sex, age_f3) 
```
