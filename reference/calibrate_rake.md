# Fit weights using raking

Iterative proportional fitting (raking) that adjusts survey weights to
match multiple marginal population totals simultaneously. Supports two
algorithms: the `"classic_ipf"` method (chi-square variable selection,
improvement-based convergence, ported from the anesrake package) and the
`"nr"` method (Newton-Raphson solver from Deville, Sarndal & Sautory
1993, using the multiplicative `F(u) = exp(u)` function).

## Usage

``` r
calibrate_rake(
  data,
  targets,
  weights = NULL,
  wt_name = NULL,
  type = c("prop", "count"),
  algorithm = c("classic_ipf", "nr"),
  cap = NULL,
  control = list(),
  reference_design = NULL
)
```

## Arguments

- data:

  A `survey_taylor`, `survey_nonprob`, or `survey_replicate`. Any other
  class -\> error. When `data` is a `survey_replicate`, raking is
  applied independently to every replicate weight column using the same
  population `targets`. Replicate columns that fail raking are kept at
  their original values and reported via a
  `surveywts_warning_replicate_calibration_failed` warning; the
  full-sample raking still completes normally.

- targets:

  Named list or data frame specifying population margin targets.

  **Format A – named list:**

      list(
        age_group = c("18-34" = 0.28, "35-54" = 0.37, "55+" = 0.35),
        sex       = c("M" = 0.49, "F" = 0.51)
      )

  Each element can be a named numeric vector or a data frame with
  columns `level` and `target` (formats can be mixed within the list).

  **Format B – long data frame** with columns `variable`, `level`,
  `target`:

      data.frame(
        variable = c("age_group", "age_group", "sex", "sex"),
        level    = c("18-34", "35-54", "M", "F"),
        target   = c(0.40, 0.60, 0.49, 0.51)
      )

  Format B is auto-detected and converted to Format A before use.

- weights:

  \<[`tidy-select`](https://tidyselect.r-lib.org/reference/language.html)\>
  Weight column name (bare name). Auto-detected from survey object
  `@variables$weights`.

- wt_name:

  `NULL` (default) or a `character(1)`. When `NULL`, raked weights
  overwrite the existing weight column in place. When a character
  string, a new column is added and `@variables$weights` updated.

- type:

  Character scalar. `"prop"` (default): `targets` values are
  proportions. `"count"`: `targets` values are counts.

- algorithm:

  Character scalar. `"classic_ipf"` (default): chi-square discrepancy
  variable selection with improvement-based convergence, as in the
  `anesrake` package. `"nr"`: Newton-Raphson solver using the
  multiplicative F-function `F(u) = exp(u)`, corresponding to Deville,
  Sarndal & Sautory (1993) generalized raking.

- cap:

  Numeric or `NULL`. Cap on the weight ratio `w / mean(w)`. Any weight
  exceeding `cap * mean(w)` is set to `cap * mean(w)`. Applied after
  each per-margin adjustment step (not post-hoc). `NULL` (default) means
  no cap. Not compatible with `algorithm = "nr"` (-\> error).

- control:

  Named list of algorithm parameters. Merged with algorithm-specific
  defaults. Omitted keys retain their defaults.

  **`algorithm = "classic_ipf"` defaults:**

  - `maxit = 1000`: maximum full sweeps

  - `improvement = 0.01`: percentage improvement convergence threshold

  - `pval = 0.05`: chi-square p-value threshold for variable selection

  - `min_cell_n = 0L`: minimum unweighted observations per cell (0 = no
    min)

  - `variable_select = "total"`: chi-square aggregation for ranking
    (`"total"`, `"max"`, or `"average"`)

  **`algorithm = "nr"` defaults:**

  - `maxit = 50L`: maximum Newton-Raphson iterations

  - `epsilon = 1e-7`: maximum relative margin error convergence
    threshold

  Passing `"classic_ipf"`-specific keys when `algorithm = "nr"` (or vice
  versa) triggers `surveywts_warning_control_param_ignored` per ignored
  key.

- reference_design:

  A `survey_taylor` object or `NULL`. The probability survey from which
  `targets` were estimated. When non-`NULL`, stored in the history entry
  with `targets_from_reference = TRUE`. Any non-`NULL`
  non-`survey_taylor` value triggers an error.

## Value

- `survey_taylor` or `survey_nonprob` input -\> same class as input
  (class is preserved); `@calibration` slot is populated

- `survey_replicate` input -\> `survey_replicate` (class preserved);
  `@calibration` slot is populated, including `replicate_converged`

The weight column in the output contains raked weights. A history entry
with `operation = "calibrate_rake"` is appended to `weighting_history`.
For `survey_replicate` inputs, each replicate weight column is also
raked. Failed replicates retain their original weights and are recorded
in `output@calibration$replicate_converged` as `FALSE`.

For `algorithm = "nr"`, `@calibration$lambda` contains the converged
Lagrange multiplier vector. For `algorithm = "classic_ipf"`,
`@calibration$lambda` is `NULL`.

## Details

**When to use.** Raking is the default calibration method. Choose it
when your targets are separate margins (for example sex, age group, and
region) and the weights must stay positive (Deville, Sarndal & Sautory
1993). The weight ratio has no upper bound, so a margin far from the
sample can produce extreme weights; use `cap`, or
[`trim_weights()`](https://jdenn0514.github.io/surveywts/reference/trim_weights.md)
afterward, to limit them.

## Algorithm

Both algorithms implement the raking calibration function \\F(u) =
\exp(u)\\, which keeps all calibrated weights strictly positive.

**Classic IPF (`algorithm = "classic_ipf"`, the default)**

At each sweep, variables are ranked by chi-square discrepancy between
weighted and target margins (controlled by `control$variable_select`,
default `"chi_square"`). Variables with any cell below
`control$min_cell_n` (default `5L`) unweighted observations are excluded
entirely; variables whose chi-square p-value exceeds `control$pval`
(default `0.01`) are skipped for that sweep. Within each selected
variable, weights are scaled by `target_k / weighted_k` for each level
`k`. If all variables pass or are excluded in sweep 1, a message is
emitted indicating the data is already calibrated.

**Newton-Raphson (`algorithm = "nr"`)**

Solves the calibration score equations via Newton-Raphson iteration.
Calibrated weights satisfy \$\$w_k = d_k \exp(x_k^T \hat{\lambda})\$\$
where \\\hat{\lambda}\\ is found by iterating on \$\$\sum\_{k \in s} d_k
\exp(x_k^T \lambda) x_k = X_U.\$\$ Step-halving guards against
non-finite g-weights at each iteration.

## Convergence

**Classic IPF:** Terminates when the percentage improvement in total
chi-square discrepancy between successive sweeps falls below
`control$improvement` (default `0.01`%), or when `control$maxit`
(default `1000L`) full sweeps are completed.

**Newton-Raphson:** Terminates when \\\max(\|\text{misfit}\| / (1 +
\|\text{population}\|)) \< \epsilon\\, where \\\epsilon\\ is
`control$epsilon` (default `1e-7`), or when `control$maxit` (default
`50L`) iterations are completed.

Calibration non-convergence raises an error.

## References

DeBell, M. and Krosnick, J.A. (2009). Computing Weights for American
National Election Study Survey Data. ANES Technical Report series, no.
nes012427. Ann Arbor, MI, and Palo Alto, CA: American National Election
Studies.

Deville, J.-C. and Sarndal, C.-E. (1992). Calibration estimators in
survey sampling. *Journal of the American Statistical Association*,
87(418), 376–382.

Deville, J.-C., Sarndal, C.-E. and Sautory, O. (1993). Generalized
raking procedures in survey sampling. *Journal of the American
Statistical Association*, 88(423), 1013–1020.

Kott, P.S. (2003). An overview of calibration weighting. 2003 Joint
Statistical Meetings — Section on Survey Research Methods.

## See also

[`calibrate()`](https://jdenn0514.github.io/surveywts/reference/calibrate.md),
[`calibrate_linear()`](https://jdenn0514.github.io/surveywts/reference/calibrate_linear.md),
[`calibrate_logit()`](https://jdenn0514.github.io/surveywts/reference/calibrate_logit.md),
[`poststratify()`](https://jdenn0514.github.io/surveywts/reference/poststratify.md).
For the class system, the standard workflows, and a glossary of terms,
see the [Getting started
article](https://jdenn0514.github.io/surveywts/articles/getting-started.html).

Other calibration:
[`calibrate()`](https://jdenn0514.github.io/surveywts/reference/calibrate.md),
[`calibrate_linear()`](https://jdenn0514.github.io/surveywts/reference/calibrate_linear.md),
[`calibrate_logit()`](https://jdenn0514.github.io/surveywts/reference/calibrate_logit.md),
[`poststratify()`](https://jdenn0514.github.io/surveywts/reference/poststratify.md)

## Examples

``` r
ns_wave1_svy <- surveycore::as_survey_nonprob(ns_wave1, weights = weight)

targets_a <- list(
  sex   = c("Male" = 0.49, "Female" = 0.51),
  age_f3 = c("18-34" = 0.30, "35-54" = 0.33, "55+" = 0.37)
)

# Format A + classic_ipf (default) ------------------------------------
result <- calibrate_rake(ns_wave1_svy, targets = targets_a)
summarize_weights(result)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv     min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl>   <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  6422       6422      0     1  1.36 0.00373 0.152 0.399  1.13  4.96 2248.

# Format A + Newton-Raphson algorithm ---------------------------------
calibrate_rake(ns_wave1_svy, targets = targets_a, algorithm = "nr")
#> # A calibrated survey design: 6,422 observations, 185 variables
#> # Variance: model-assisted (SRS assumption)
#> # IDs: ~1 | Strata: NULL | Weights: weight 
#> # Weighting history: 1 step 
#> #   Step 1 [2026-09-10]: raking (targets: sex, age_f3) 

# Format B ------------------------------------------------------------
targets_b <- data.frame(
  variable = c("sex", "sex", "age_f3", "age_f3", "age_f3"),
  level    = c("Male", "Female", "18-34", "35-54", "55+"),
  target   = c(0.49, 0.51, 0.30, 0.33, 0.37)
)
calibrate_rake(ns_wave1_svy, targets = targets_b)
#> # A calibrated survey design: 6,422 observations, 185 variables
#> # Variance: model-assisted (SRS assumption)
#> # IDs: ~1 | Strata: NULL | Weights: weight 
#> # Weighting history: 1 step 
#> #   Step 1 [2026-09-10]: raking (targets: sex, age_f3) 
```
