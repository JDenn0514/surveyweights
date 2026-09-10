# Adjust weights to match population totals

A thin dispatcher that routes to
[`calibrate_rake()`](https://jdenn0514.github.io/surveywts/reference/calibrate_rake.md),
[`calibrate_linear()`](https://jdenn0514.github.io/surveywts/reference/calibrate_linear.md),
or
[`calibrate_logit()`](https://jdenn0514.github.io/surveywts/reference/calibrate_logit.md)
based on `method`. All arguments are forwarded unchanged; all validation
and error handling occur in the dispatched function.

## Usage

``` r
calibrate(
  data,
  targets,
  weights = NULL,
  wt_name = NULL,
  type = c("prop", "count"),
  reference_design = NULL,
  ...,
  method = c("rake", "linear", "logit")
)
```

## Arguments

- data:

  A `survey_nonprob`, `survey_taylor`, or `survey_replicate`. Forwarded
  unchanged to the dispatched function. For `survey_replicate` inputs,
  calibration is applied to every replicate weight column using the same
  `targets`; see the dispatched function for replicate weight handling
  details.

- targets:

  Target specification. Forwarded to the dispatched function. Two
  formats are accepted:

  **Format A — named list** (one element per calibration variable):

      list(
        sex   = c("Male" = 0.49, "Female" = 0.51),
        age_f3 = c("18-34" = 0.30, "35-54" = 0.33, "55+" = 0.37)
      )

  **Format B — long data frame** with columns `variable`, `level`,
  `target`:

      data.frame(
        variable = c("sex", "sex", "age_f3", "age_f3", "age_f3"),
        level    = c("Male", "Female", "18-34", "35-54", "55+"),
        target   = c(0.49, 0.51, 0.30, 0.33, 0.37)
      )

  Format B is auto-detected and converted to Format A before dispatch.
  See
  [`calibrate_rake()`](https://jdenn0514.github.io/surveywts/reference/calibrate_rake.md),
  [`calibrate_linear()`](https://jdenn0514.github.io/surveywts/reference/calibrate_linear.md),
  or
  [`calibrate_logit()`](https://jdenn0514.github.io/surveywts/reference/calibrate_logit.md)
  for per-method target validation rules.

- weights:

  \<[`tidy-select`](https://tidyselect.r-lib.org/reference/language.html)\>
  Weight column (bare name). Forwarded to the dispatched function.
  `NULL` (the default) auto-detects the weight column from survey object
  `@variables$weights`.

- wt_name:

  `NULL` (the default) or a `character(1)`. When `NULL`, calibrated
  weights overwrite the existing weight column in place. When a
  character string, a new column is added and `@variables$weights`
  updated. Forwarded to the dispatched function.

- type:

  `character(1)`. `"prop"` (the default): `targets` values are
  proportions. `"count"`: `targets` values are population counts.
  Forwarded to the dispatched function.

- reference_design:

  A `survey_taylor` or `NULL` (the default). Stored in the weighting
  history for provenance. Forwarded to the dispatched function.

- ...:

  Additional arguments forwarded as-is to the dispatched function. See
  [`calibrate_rake()`](https://jdenn0514.github.io/surveywts/reference/calibrate_rake.md),
  [`calibrate_linear()`](https://jdenn0514.github.io/surveywts/reference/calibrate_linear.md),
  or
  [`calibrate_logit()`](https://jdenn0514.github.io/surveywts/reference/calibrate_logit.md)
  for available arguments (e.g., `algorithm`, `bounds`, `cap`,
  `control`).

- method:

  `character(1)`. Calibration method: `"rake"` (the default),
  `"linear"`, or `"logit"`. Matched with
  [`rlang::arg_match()`](https://rlang.r-lib.org/reference/arg_match.html).

  - `"rake"`: multiplicative raking via
    [`calibrate_rake()`](https://jdenn0514.github.io/surveywts/reference/calibrate_rake.md).
    Weights remain strictly positive. Two algorithms available via
    `algorithm` in `...`.

  - `"linear"`: GREG estimator via
    [`calibrate_linear()`](https://jdenn0514.github.io/surveywts/reference/calibrate_linear.md).
    Exact in one step; may produce negative weights for large
    discrepancies.

  - `"logit"`: logit-bounded calibration via
    [`calibrate_logit()`](https://jdenn0514.github.io/surveywts/reference/calibrate_logit.md).
    G-weight ratios constrained to an open interval `(L, U)` via
    `bounds` in `...`.

## Value

An object of the same class as `data`, as returned by the dispatched
function. See
[`calibrate_rake()`](https://jdenn0514.github.io/surveywts/reference/calibrate_rake.md),
[`calibrate_linear()`](https://jdenn0514.github.io/surveywts/reference/calibrate_linear.md),
or
[`calibrate_logit()`](https://jdenn0514.github.io/surveywts/reference/calibrate_logit.md)
for class-specific return value details and weighting history
guarantees.

## Details

All three methods implement the Deville-Sarndal calibration framework:
each adjusts survey weights so that weighted auxiliary totals match
known population totals. The methods share a variance estimator and
differ in the weight-ratio function \\F\\ applied during calibration
(Deville & Sarndal 1992; Deville, Sarndal & Sautory 1993). The value of
\\F\\ is the g-weight (the ratio of calibrated to starting weight).

**Raking** (`method = "rake"`, the default) uses the multiplicative
function \\F(u) = \exp(u)\\, which keeps all calibrated weights strictly
positive. For marginal targets, raking reduces to classical iterative
proportional fitting (Deville, Sarndal & Sautory 1993). Two algorithms
are available via `algorithm` (passed through `...`): `"classic_ipf"`
(the default; chi-square variable selection ported from the ANES raking
procedure, DeBell & Krosnick 2009) and `"nr"` (Newton-Raphson). The
weight ratio \\w_k / d_k\\ is unbounded above.

**Linear** (`method = "linear"`) uses \\F(u) = 1 + u\\, equivalent to
the generalized regression (GREG) estimator. The solution is exact in a
single step — no iteration required — making it the fastest method
(Deville & Sarndal 1992). The weight ratio is unbounded in both
directions; large sample-to-population discrepancies can produce
negative calibrated weights.

**Logit** (`method = "logit"`) constrains the weight ratio \\w_k / d_k\\
to the open interval \\(L, U)\\ via a logit-bounded \\F\\ function
(Deville & Sarndal 1992; Deville, Sarndal & Sautory 1993). Pass `bounds`
via `...` to control the interval (default `c(1e-6, 1e6)`). Note that
bounds apply to the ratio of calibrated to design weight, not to
calibrated weights directly.

Post-stratification does not route through `calibrate()`. When you have
population values for every joint cell of the stratification variables —
a full cross-tabulation, not separate margins — use
[`poststratify()`](https://jdenn0514.github.io/surveywts/reference/poststratify.md),
which matches those cells exactly in one pass.

For full algorithm documentation, convergence criteria, and
replicate-weight handling, see
[`calibrate_rake()`](https://jdenn0514.github.io/surveywts/reference/calibrate_rake.md),
[`calibrate_linear()`](https://jdenn0514.github.io/surveywts/reference/calibrate_linear.md),
and
[`calibrate_logit()`](https://jdenn0514.github.io/surveywts/reference/calibrate_logit.md).

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

[`calibrate_rake()`](https://jdenn0514.github.io/surveywts/reference/calibrate_rake.md),
[`calibrate_linear()`](https://jdenn0514.github.io/surveywts/reference/calibrate_linear.md),
[`calibrate_logit()`](https://jdenn0514.github.io/surveywts/reference/calibrate_logit.md),
[`poststratify()`](https://jdenn0514.github.io/surveywts/reference/poststratify.md).
For the class system, the standard workflows, and a glossary of terms,
see the [Getting started
article](https://jdenn0514.github.io/surveywts/articles/getting-started.html).

Other calibration:
[`calibrate_linear()`](https://jdenn0514.github.io/surveywts/reference/calibrate_linear.md),
[`calibrate_logit()`](https://jdenn0514.github.io/surveywts/reference/calibrate_logit.md),
[`calibrate_rake()`](https://jdenn0514.github.io/surveywts/reference/calibrate_rake.md),
[`poststratify()`](https://jdenn0514.github.io/surveywts/reference/poststratify.md)

## Examples

``` r
ns_wave1_svy <- surveycore::as_survey_nonprob(ns_wave1, weights = weight)

targets_a <- list(
  sex    = c("Male" = 0.49, "Female" = 0.51),
  age_f3 = c("18-34" = 0.30, "35-54" = 0.33, "55+" = 0.37)
)

# Format A + rake (default) --------------------------------------------
result <- calibrate(ns_wave1_svy, targets = targets_a)
summarize_weights(result)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv     min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl>   <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  6422       6422      0     1  1.36 0.00373 0.152 0.399  1.13  4.96 2248.

# Format A + linear ----------------------------------------------------
calibrate(ns_wave1_svy, targets = targets_a, method = "linear")
#> # A calibrated survey design: 6,422 observations, 185 variables
#> # Variance: model-assisted (SRS assumption)
#> # IDs: ~1 | Strata: NULL | Weights: weight 
#> # Weighting history: 1 step 
#> #   Step 1 [2026-09-10]: calibrate_linear (variables: sex, age_f3) 

# Format A + logit -----------------------------------------------------
calibrate(ns_wave1_svy, targets = targets_a, method = "logit")
#> # A calibrated survey design: 6,422 observations, 185 variables
#> # Variance: model-assisted (SRS assumption)
#> # IDs: ~1 | Strata: NULL | Weights: weight 
#> # Weighting history: 1 step 
#> #   Step 1 [2026-09-10]: calibrate_logit (variables: sex, age_f3) 

# Format B + rake ------------------------------------------------------
targets_b <- data.frame(
  variable = c("sex", "sex", "age_f3", "age_f3", "age_f3"),
  level    = c("Male", "Female", "18-34", "35-54", "55+"),
  target   = c(0.49, 0.51, 0.30, 0.33, 0.37)
)
calibrate(ns_wave1_svy, targets = targets_b)
#> # A calibrated survey design: 6,422 observations, 185 variables
#> # Variance: model-assisted (SRS assumption)
#> # IDs: ~1 | Strata: NULL | Weights: weight 
#> # Weighting history: 1 step 
#> #   Step 1 [2026-09-10]: raking (targets: sex, age_f3) 
```
