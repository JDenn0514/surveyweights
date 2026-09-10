# Reweight to externally estimated population totals

Adjusts the full-sample and replicate weights (sets of perturbed weight
columns used to compute standard errors) of `design` so that its
estimated totals for the variables named in `targets` match the supplied
count totals. The `vcov_estimate` matrix propagates the uncertainty of
those external estimates into the calibrated design's variance
estimates.

## Usage

``` r
calibrate_to_estimate(
  design,
  targets,
  vcov_estimate,
  method = c("rake", "linear", "logit"),
  bounds = c(-Inf, Inf),
  unit_scale = NULL,
  reference_design = NULL,
  control = list()
)
```

## Arguments

- design:

  A `survey_replicate` or `survey_nonprob` object with replicate
  weights. The design to be calibrated. Must have replicate weights
  populated.

- targets:

  A named list of population count totals. Each element is a named
  numeric vector whose names are the level labels for that variable in
  `design@data`. All values must be strictly positive. The list name is
  the variable name in `design@data`.

- vcov_estimate:

  A numeric matrix of dimension `k x k`, where
  `k = length(unlist(targets))`. The variance-covariance matrix of the
  population estimates. Must be symmetric (tolerance `1e-8`) and
  positive definite.

- method:

  Calibration method. One of `"rake"` (default), `"linear"`, or
  `"logit"`. Matched with
  [`rlang::arg_match()`](https://rlang.r-lib.org/reference/arg_match.html).

- bounds:

  Numeric vector of length 2: `c(lower, upper)`. Bounds on the
  calibrated-to-starting-weight ratio. Default `c(-Inf, Inf)`. Note:
  per-unit `bounds_scale` is not supported; use scalar bounds only.

- unit_scale:

  `NULL` or a positive numeric vector of length `nrow(design@data)`.
  Passed to svrep as the `variance` argument (per-unit variance
  scaling). `NULL` means no scaling.

- reference_design:

  A `survey_taylor` object or `NULL`. Stored in the history entry for
  provenance only; not used in computation.

- control:

  Named list of calibration control parameters. Known keys:

  - `maxit`: maximum iterations (default `50L`)

  - `epsilon`: convergence tolerance (default `1e-7`)

  - `col_selection`: passed to svrep for reproducible replicate
    selection; not stored in history. Unknown keys trigger a
    `surveywts_warning_control_param_ignored` warning.

## Value

A calibrated design object whose class matches the class of `design`: a
`survey_nonprob` input returns a `survey_nonprob`; a `survey_replicate`
input returns a `survey_replicate`. Updated full-sample and replicate
weights are written back. A new entry with
`operation = "calibrate_to_estimate"` is appended to the weighting
history.

## Details

**When to use.** Use `calibrate_to_estimate()` when you have only point
estimates and their variance-covariance matrix, for example published
totals from a statistical agency. When you have the control survey
itself with replicate weights, use
[`calibrate_to_survey()`](https://jdenn0514.github.io/surveywts/reference/calibrate_to_survey.md),
which estimates the targets and their uncertainty directly.

`design` must carry replicate weights — either as a `survey_replicate`
object or as a `survey_nonprob` object to which replicate weights have
been added. Taylor-linearization designs are not sufficient for this
purpose because replicate weights are required to propagate the
uncertainty of the external estimates (captured in `vcov_estimate`) into
the variance of the calibrated design.

The `targets` list is converted to a named vector via `unlist(targets)`,
and a calibration formula is built from `names(targets)`. Ensure that
the names of each `targets` element exactly match the levels in the
corresponding column of `design@data` (for factor columns:
[`levels()`](https://rdrr.io/r/base/levels.html); for character columns:
[`unique()`](https://rdrr.io/r/base/unique.html)).

When `method = "linear"`, any negative full-sample weights are clipped
to `.Machine$double.eps` with a warning; replicate weights are not
clipped.

## References

Fuller, W.A. (1998). Replication variance estimation for two-phase
samples. *Statistica Sinica*, 8, 1153–1164.

Opsomer, J.D. and Erciulescu, A. (2021). Replication variance estimation
after sample-based calibration. *Survey Methodology*, 47, 265–277.

## See also

[`calibrate_to_survey()`](https://jdenn0514.github.io/surveywts/reference/calibrate_to_survey.md).
For the class system, the standard workflows, and a glossary of terms,
see the [Getting started
article](https://jdenn0514.github.io/surveywts/articles/getting-started.html).

Other sample-calibration:
[`calibrate_to_survey()`](https://jdenn0514.github.io/surveywts/reference/calibrate_to_survey.md)

## Examples

``` r
# calibrate GSS 2024 pid_f3 to NPORS population estimates ---------------

# build primary replicate design from GSS (JKn on complete pid_f3 rows)
gss_pop <- surveycore::as_survey(
  gss_2024[!is.na(gss_2024$pid_f3), ],
  weights = wt_pop,
  strata  = vstrat,
  ids     = vpsu,
  nest    = TRUE
) |>
  create_jackknife_weights(type = "jkn")

# build NPORS control design. `vcov_estimate` must be a covariance matrix
# across the levels of a factor, and only the survey package makes one.
npors_pop <- survey::svydesign(
  ids     = ~1,
  strata  = ~stratum,
  weights = ~wt_pop,
  data    = npors_2025_clean
)

# derive targets from control survey
pid_f3_est    <- survey::svytotal(~pid_f3, npors_pop)
pid_f3_totals <- setNames(coef(pid_f3_est), levels(npors_2025_clean$pid_f3))
vcov_pid_f3   <- vcov(pid_f3_est)

# svrep draws the perturbed replicate columns at random, so seed the call
set.seed(1)
result <- calibrate_to_estimate(
  gss_pop,
  targets       = list(pid_f3 = pid_f3_totals),
  vcov_estimate = vcov_pid_f3
)
#> Selection of replicate columns whose control totals will be perturbed will be done at random.
#> For tips on reproducible selection, see `help('calibrate_to_estimate')`

summarize_weights(result)
#> # A tibble: 1 × 11
#>       n n_positive n_zero   mean    cv   min    p25    p50    p75     max   ess
#>   <int>      <int>  <int>  <dbl> <dbl> <dbl>  <dbl>  <dbl>  <dbl>   <dbl> <dbl>
#> 1  3185       3185      0 78471.  1.14 3673. 25706. 52665. 91037. 882307. 1386.
```
