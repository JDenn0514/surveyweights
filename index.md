# surveywts

Part of the [surveyverse](https://github.com/JDenn0514), surveywts is
the one-stop shop for survey weighting. It covers the full weight
adjustment workflow: calibrating to population benchmarks, handling unit
nonresponse, weighting non-probability samples via inverse probability
weighting, generating replicate weights for variance estimation, and
diagnosing weight quality, all with a full audit trail recorded on the
result object.

## Installation

``` r

# From GitHub (development version)
pak::pak("JDenn0514/surveywts")

# From r-universe (pre-built binaries, no GitHub PAT needed)
install.packages("surveywts", repos = "https://jdenn0514.r-universe.dev")
```

## The surveyverse

surveywts is the weighting layer of the surveyverse ecosystem.
[surveycore](https://jdenn0514.github.io/surveycore/) is the foundation,
representing sampling designs and enabling analysis. surveywts operates
on the survey objects surveycore creates, adjusting and calibrating
their weights. [surveytidy](https://jdenn0514.github.io/surveytidy/)
rounds out the ecosystem with tidy data manipulation for survey objects.

## Functions

### Calibration

| Function | Purpose |
|----|----|
| [`calibrate()`](https://jdenn0514.github.io/surveywts/reference/calibrate.md) | Calibrate to population `targets`; choose method with `method = "rake"`, `"linear"`, or `"logit"` |
| [`calibrate_linear()`](https://jdenn0514.github.io/surveywts/reference/calibrate_linear.md) | Linear (GREG) calibration |
| [`calibrate_rake()`](https://jdenn0514.github.io/surveywts/reference/calibrate_rake.md) | Raking (iterative proportional fitting) |
| [`calibrate_logit()`](https://jdenn0514.github.io/surveywts/reference/calibrate_logit.md) | Logit-bounded calibration (guaranteed positive weights) |
| [`poststratify()`](https://jdenn0514.github.io/surveywts/reference/poststratify.md) | Exact cell-level post-stratification |
| [`calibrate_to_survey()`](https://jdenn0514.github.io/surveywts/reference/calibrate_to_survey.md) | Calibrate against a control survey, propagating its sampling uncertainty |
| [`calibrate_to_estimate()`](https://jdenn0514.github.io/surveywts/reference/calibrate_to_estimate.md) | Calibrate against external estimates with a known variance-covariance matrix |

### Nonresponse adjustment

| Function | Purpose |
|----|----|
| [`adjust_nonresponse()`](https://jdenn0514.github.io/surveywts/reference/adjust_nonresponse.md) | Nonresponse adjustment via weighting class, propensity cell, or propensity score |
| [`redistribute_weights()`](https://jdenn0514.github.io/surveywts/reference/redistribute_weights.md) | Low-level weight redistribution between any two groups |

### Non-probability samples

| Function | Purpose |
|----|----|
| [`ipw()`](https://jdenn0514.github.io/surveywts/reference/ipw.md) | Inverse probability weighting via logistic regression (pseudo-likelihood or calibration GEE) |

### Replicate weights

| Function | Purpose |
|----|----|
| [`create_replicate_weights()`](https://jdenn0514.github.io/surveywts/reference/create_replicate_weights.md) | Dispatcher for all replicate weight methods |
| [`create_bootstrap_weights()`](https://jdenn0514.github.io/surveywts/reference/create_bootstrap_weights.md) | Bootstrap replicate weights (Rao-Wu-Yue-Beaumont; quasi-randomization for NPS) |
| [`create_jackknife_weights()`](https://jdenn0514.github.io/surveywts/reference/create_jackknife_weights.md) | Jackknife replicate weights (`type = "jk1"`, `"jkn"`, or `"grouped"`) |
| [`create_brr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_brr_weights.md) | Balanced repeated replication (standard or Fay’s variant) |
| [`create_gen_boot_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_boot_weights.md) | Generalized bootstrap weights |
| [`create_gen_rep_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_rep_weights.md) | Generalized replication weights (Fay’s method) |
| [`create_sdr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_sdr_weights.md) | Successive difference replication |
| [`as_taylor_design()`](https://jdenn0514.github.io/surveywts/reference/as_taylor_design.md) | Convert a replicate design back to Taylor linearization |

### Diagnostics

| Function | Purpose |
|----|----|
| [`effective_sample_size()`](https://jdenn0514.github.io/surveywts/reference/effective_sample_size.md) | Kish effective sample size |
| [`weight_variability()`](https://jdenn0514.github.io/surveywts/reference/weight_variability.md) | Coefficient of variation of the weights |
| [`summarize_weights()`](https://jdenn0514.github.io/surveywts/reference/summarize_weights.md) | Full distributional summary, optionally by group |

### Utilities

| Function | Purpose |
|----|----|
| [`trim_weights()`](https://jdenn0514.github.io/surveywts/reference/trim_weights.md) | Clip extreme weights with mass redistribution |
| [`rescale_weights()`](https://jdenn0514.github.io/surveywts/reference/rescale_weights.md) | Rescale weights to unit mean |

## Usage

### Weighting a non-probability sample

[`ipw()`](https://jdenn0514.github.io/surveywts/reference/ipw.md)
constructs weights for a non-probability sample by fitting a
participation propensity model against a probability reference survey.
surveywts ships with a harmonized online panel (`ns_wave1`) and
probability reference surveys (`gss_2024`, `npors_2025_clean`,
`cps_2023`) to illustrate the workflow. Survey designs are constructed
from the tibbles with
[`surveycore::as_survey()`](https://jdenn0514.github.io/surveycore/reference/as_survey.html)
or
[`surveycore::as_survey_nonprob()`](https://jdenn0514.github.io/surveycore/reference/as_survey_nonprob.html).

``` r

library(surveywts)

# Construct a reference design from the npors_2025_clean tibble
npors_ref <- surveycore::as_survey(
  npors_2025_clean,
  weights = wt_pop,
  strata = stratum
)

# Drop the 7 rows with NA partisanship (pid_f3) — the benchmark variable
# in the calibrate-to-survey step below.
ns_complete <- ns_wave1[!is.na(ns_wave1$pid_f3), ]

nps_wts <- ipw(
  ns_complete,
  npors_ref,
  predictors = c("sex", "age_f3", "race_f4", "edu_f3"),
  missing_method = "omit",
  estimating_eq = "mle" # keeps the per-replicate refits below stable
)
#> Warning: ! 120 row(s) in `data` dropped: NA in race_f4.
#> ℹ Rows with any NA in a `selection` variable are excluded when `missing_method
#>   = "omit"`.
#> ✔ Use `missing_method = "separate"` or `missing_method = "impute"` to retain
#>   rows with NA.

summarize_weights(nps_wts)
#> # A tibble: 1 × 11
#>       n n_positive n_zero   mean    cv    min    p25    p50    p75     max   ess
#>   <int>      <int>  <int>  <dbl> <dbl>  <dbl>  <dbl>  <dbl>  <dbl>   <dbl> <dbl>
#> 1  6295       6295      0 39944. 0.936 13052. 17535. 26423. 42933. 370516. 3355.
effective_sample_size(nps_wts)
#>    n_eff 
#> 3355.106
```

### Calibrating to population benchmarks

[`calibrate()`](https://jdenn0514.github.io/surveywts/reference/calibrate.md)
adjusts weights to match known population `targets`. Here we rake the
IPW-weighted panel to census marginals for a doubly robust estimate.

``` r

targets <- list(
  sex = c("Male" = 0.49, "Female" = 0.51),
  age_f3 = c("18-34" = 0.28, "35-54" = 0.37, "55+" = 0.35)
)

calibrated <- calibrate(
  nps_wts,
  targets = targets,
  method = "rake"
)

summarize_weights(calibrated)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv   min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  6295       6295      0     1 0.893 0.339 0.507 0.610  1.11  8.56 3503.
```

### Calibrating to a reference survey

[`calibrate_to_survey()`](https://jdenn0514.github.io/surveywts/reference/calibrate_to_survey.md)
calibrates a primary design to match estimates from a control survey,
propagating the control’s own sampling uncertainty into the final
variance estimates. Both designs must carry replicate weights. Here the
IPW-weighted panel is benchmarked to the NPORS estimate of partisanship
(`pid_f3`).

``` r

# Primary: the IPW-weighted panel with quasi-randomization bootstrap
# replicates. Each replicate resamples the panel and refits the IPW model.
nps_rep <- create_bootstrap_weights(
  nps_wts,
  type = "quasi-randomization",
  replicates = 100L,
  seed = 1
)

# Control: the NPORS reference survey with bootstrap replicate weights.
npors_rep <- create_bootstrap_weights(npors_ref, seed = 1)

calibrated_to_ref <- calibrate_to_survey(
  nps_rep,
  npors_rep,
  variables = c(pid_f3)
)
```

### Generating replicate weights

[`create_bootstrap_weights()`](https://jdenn0514.github.io/surveywts/reference/create_bootstrap_weights.md)
adds replicate weight columns for variance estimation. For
non-probability samples it applies a quasi-randomization bootstrap.

``` r

rep_design <- create_bootstrap_weights(
  calibrated,
  type       = "quasi-randomization",
  replicates = 100L,
  seed       = 1
)
```

## Learn more

Full documentation is available at
<https://jdenn0514.github.io/surveywts/>.
