# Fit weights using linear (GREG) calibration

Adjusts survey weights so that the weighted marginal totals of
calibration variables match known population values using linear
estimation — GREG (generalized regression, the survey name for linear
calibration). Uses `F(u) = 1 + u`, which is exact in a single Newton
step when `bounds = NULL`. When `bounds = c(L, U)` is specified,
switches to truncated-linear calibration where g-weights (the ratio of
calibrated to starting weight) are constrained to `[L, U]` using
Newton-Raphson iteration.

## Usage

``` r
calibrate_linear(
  data,
  targets,
  weights = NULL,
  wt_name = NULL,
  bounds = NULL,
  bounds_scale = c("multiplicative", "absolute"),
  unit_scale = NULL,
  type = c("prop", "count"),
  control = list(),
  reference_design = NULL
)
```

## Arguments

- data:

  A `survey_taylor`, `survey_nonprob`, or `survey_replicate`. Any other
  class -\> error. When `data` is a `survey_replicate`, calibration is
  applied independently to every replicate weight column using the same
  population `targets`. Replicate columns that fail calibration are kept
  at their original values and reported via
  `surveywts_warning_replicate_calibration_failed`; the full-sample
  calibration still completes normally.

- targets:

  Named list of population marginal targets (Format A) or a long data
  frame with columns `variable`, `level`, `target` (Format B).

  **Format A – named list:**

      list(
        age_group = c("18-34" = 0.30, "35-54" = 0.40, "55+" = 0.30),
        sex       = c("M" = 0.48, "F" = 0.52)
      )

  **Format B – long data frame:**

      data.frame(
        variable = c("age_group", "age_group", "sex", "sex"),
        level    = c("18-34", "35-54", "M", "F"),
        target   = c(0.40, 0.60, 0.49, 0.51)
      )

  Names must match column names in `data`. For `type = "prop"`: each
  element must sum to 1.0 within 1e-6. For `type = "count"`: all values
  must be strictly positive and all marginal sums must agree within
  1e-3.

- weights:

  \<[`tidy-select`](https://tidyselect.r-lib.org/reference/language.html)\>
  Weight column name (bare name). Auto-detected from survey object
  `@variables$weights`.

- wt_name:

  `NULL` (default) or a `character(1)`. When `NULL`, calibrated weights
  overwrite the existing weight column in place. When a character
  string, a new column is added and `@variables$weights` updated.

- bounds:

  `NULL` (default) or a length-2 numeric vector `c(L, U)`. `NULL`: plain
  unbounded linear calibration. G-weights are unconstrained and negative
  calibrated weights are possible. `c(L, U)`: truncated-linear
  calibration with g-weights constrained to the closed interval
  `[L, U]`. Interpretation depends on `bounds_scale`. Triggers
  `surveywts_error_bounds_invalid_calibration` on invalid values.

- bounds_scale:

  Character scalar. `"multiplicative"` (default): `bounds` constrain the
  g-weight ratio `g_k = w_k / d_k`. `"absolute"`: `bounds` constrain the
  final calibrated weight `w_k` directly. Matched with
  [`rlang::arg_match()`](https://rlang.r-lib.org/reference/arg_match.html).
  Ignored when `bounds = NULL`. For `"multiplicative"`: requires
  `L < 1 < U`. For `"absolute"`: requires `0 < L < U`.

- unit_scale:

  `NULL` (default) or a positive numeric vector of length `nrow(data)`.
  Per-unit scaling factors `q_k` for the calibration distance function
  (Deville & Sarndal 1992 eq. 2.2). `NULL` is equivalent to
  `rep(1, nrow(data))`. Triggers `surveywts_error_unit_scale_invalid` if
  not `NULL` and: not numeric, wrong length, contains `NA`, or contains
  non-positive values.

- type:

  Character scalar. `"prop"` (default): `targets` values are proportions
  summing to 1.0 per variable (within 1e-6 tolerance). `"count"`:
  `targets` values are population counts (all strictly positive).
  Matched with
  [`rlang::arg_match()`](https://rlang.r-lib.org/reference/arg_match.html).

- control:

  Named list of convergence parameters. Merged with defaults
  `list(maxit = 50, epsilon = 1e-7)`. Unrecognized keys trigger
  `surveywts_warning_control_param_ignored` per key. Valid keys:
  `maxit`, `epsilon`. For plain linear (`bounds = NULL`), `maxit` and
  `epsilon` are stored in the history entry but do not affect
  computation.

- reference_design:

  A `survey_taylor` object or `NULL`. When non-`NULL`, stored in the
  history entry with `targets_from_reference = TRUE`. Any non-`NULL`
  non-`survey_taylor` value triggers
  `surveywts_error_reference_design_not_taylor`.

## Value

- `survey_taylor` input -\> `survey_taylor` (class preserved); only
  `@variables$weights` (the weight column) and `@calibration` are
  modified. `@variables$ids`, `@variables$strata`, `@variables$fpc`, and
  the Taylor design structure are unchanged.

- `survey_nonprob` input -\> `survey_nonprob` (class preserved); weight
  column updated; history entry appended; `@calibration` slot populated.

- `survey_replicate` input -\> `survey_replicate` (class preserved);
  full-sample weight column updated; each replicate weight column
  calibrated independently. For `type = "count"` targets, each
  replicate's population totals are scaled:
  `rep_targets * (sum(rep_wt) / sum(base_wt))`. Failed replicates are
  kept at original values and reported via
  `surveywts_warning_replicate_calibration_failed`. `@calibration` slot
  populated including `replicate_converged`.

History entry `operation` field: `"calibrate_linear"`.

## Details

**When to use.** Choose linear calibration when speed matters: with
`bounds = NULL` the solution is exact in one step, with no iteration
(Deville, Sarndal & Sautory 1993). The weight adjustment is unbounded in
both directions, so a large gap between the sample and the targets can
produce negative weights. If negative weights are unacceptable, set
`bounds`, or use
[`calibrate_rake()`](https://jdenn0514.github.io/surveywts/reference/calibrate_rake.md)
or
[`calibrate_logit()`](https://jdenn0514.github.io/surveywts/reference/calibrate_logit.md).

## Algorithm

Linear calibration uses \\F(u) = 1 + u\\ (the GREG estimator). The
calibrated weights are \$\$w_k = d_k (1 + x_k^T \hat{\lambda})\$\$ where
the Lagrange multipliers satisfy \$\$\hat{\lambda} = \left(\sum\_{k \in
s} d_k x_k x_k^T\right)^{-1} \left(X_U - \sum\_{k \in s} d_k
x_k\right).\$\$ The solution is obtained in a single Newton step (no
iteration required) when `bounds = NULL`.

When `bounds = c(L, U)` is specified, g-weights are constrained to
`[L, U]` (truncated-linear calibration) via Newton-Raphson iteration.

## Convergence

For unbounded calibration (`bounds = NULL`), the solution is exact in
one step — no convergence check is performed.

For bounded calibration, Newton-Raphson terminates when the maximum
absolute change in \\\lambda\\ falls below `control$epsilon` (default
`1e-7`), or when `control$maxit` (default `50`) iterations are reached.
Calibration non-convergence raises an error.

## References

Deville, J.-C. and Sarndal, C.-E. (1992). Calibration estimators in
survey sampling. *Journal of the American Statistical Association*,
87(418), 376–382.

Deville, J.-C., Sarndal, C.-E. and Sautory, O. (1993). Generalized
raking procedures in survey sampling. *Journal of the American
Statistical Association*, 88(423), 1013–1020.

## See also

[`calibrate()`](https://jdenn0514.github.io/surveywts/reference/calibrate.md),
[`calibrate_rake()`](https://jdenn0514.github.io/surveywts/reference/calibrate_rake.md),
[`calibrate_logit()`](https://jdenn0514.github.io/surveywts/reference/calibrate_logit.md),
[`poststratify()`](https://jdenn0514.github.io/surveywts/reference/poststratify.md).
For the class system, the standard workflows, and a glossary of terms,
see the [Getting started
article](https://jdenn0514.github.io/surveywts/articles/getting-started.html).

Other calibration:
[`calibrate()`](https://jdenn0514.github.io/surveywts/reference/calibrate.md),
[`calibrate_logit()`](https://jdenn0514.github.io/surveywts/reference/calibrate_logit.md),
[`calibrate_rake()`](https://jdenn0514.github.io/surveywts/reference/calibrate_rake.md),
[`poststratify()`](https://jdenn0514.github.io/surveywts/reference/poststratify.md)

## Examples

``` r
ns_wave1_svy <- surveycore::as_survey_nonprob(ns_wave1, weights = weight)

targets_a <- list(
  sex   = c("Male" = 0.49, "Female" = 0.51),
  age_f3 = c("18-34" = 0.30, "35-54" = 0.33, "55+" = 0.37)
)

# Format A ---------------------------------------------------------------
result <- calibrate_linear(ns_wave1_svy, targets = targets_a)
summarize_weights(result)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv     min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl>   <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  6422       6422      0  1.00  1.36 0.00373 0.152 0.399  1.13  4.95 2248.

# Format B ---------------------------------------------------------------
targets_b <- data.frame(
  variable = c("sex", "sex", "age_f3", "age_f3", "age_f3"),
  level    = c("Male", "Female", "18-34", "35-54", "55+"),
  target   = c(0.49, 0.51, 0.30, 0.33, 0.37)
)
calibrate_linear(ns_wave1_svy, targets = targets_b)
#> # A calibrated survey design: 6,422 observations, 185 variables
#> # Variance: model-assisted (SRS assumption)
#> # IDs: ~1 | Strata: NULL | Weights: weight 
#> # Weighting history: 1 step 
#> #   Step 1 [2026-09-10]: calibrate_linear (variables: sex, age_f3) 
```
