# Generate bootstrap replicate weights

Generates bootstrap replicate weights (sets of perturbed weight columns
used to compute standard errors) for probability-sample designs via
[`svrep::as_bootstrap_design()`](https://bschneidr.github.io/svrep/reference/as_bootstrap_design.html).
For non-probability samples (`survey_nonprob`), it generates
quasi-randomization bootstrap (each replicate refits the weighting
model) replicate weights via an internal resample-reweight algorithm.

## Usage

``` r
create_bootstrap_weights(
  data,
  replicates = NULL,
  ...,
  type = c("Rao-Wu-Yue-Beaumont", "Rao-Wu", "Antal-Tille", "Preston", "Canty-Davison",
    "quasi-randomization", "hybrid"),
  reference_sample = NULL,
  mse = c("mse", "chrostowski", "uncentered"),
  seed = NULL
)
```

## Arguments

- data:

  A `survey_taylor` or `survey_nonprob` design object.
  `survey_replicate` and `data.frame` → error. A `survey_nonprob` is
  accepted with a probability-sample `type` as well; it is then wrapped
  as a simple random sample — see `@details`.

- replicates:

  `integer(1)` or `NULL`. Number of bootstrap replicates. Default `NULL`
  resolves to `200L` for `type = "quasi-randomization"` and
  `type = "hybrid"`, and `500L` for all probability-sample types. Must
  be at least 2. Whole-number doubles are coerced to integer silently.

- ...:

  Must be empty. Forces all subsequent arguments to be named.

- type:

  `character(1)`. Bootstrap variant. For probability-sample designs:
  `"Rao-Wu-Yue-Beaumont"` (default), `"Rao-Wu"`, `"Antal-Tille"`,
  `"Preston"`, or `"Canty-Davison"` — passed to
  [`svrep::as_bootstrap_design()`](https://bschneidr.github.io/svrep/reference/as_bootstrap_design.html).
  For non-probability samples: `"quasi-randomization"`
  (resample-reweight bootstrap) or `"hybrid"` (error stub; requires
  `mass_imputation()`, not yet implemented). See
  [`svrep::as_bootstrap_design()`](https://bschneidr.github.io/svrep/reference/as_bootstrap_design.html)
  for how the five probability-sample variants differ.

- reference_sample:

  `survey_taylor` or `NULL`. Reference probability sample for NPS types.
  When non-`NULL`, takes precedence over any reference design stored in
  `@metadata@weighting_history`. Ignored (with a warning) when `type` is
  a probability-sample type. `survey_replicate` → error.

- mse:

  `character(1)`. Variance formula for bootstrap variance. `"mse"`
  (default): mean squared deviation from the full-sample estimate,
  \\(1/B) \sum (\hat{\theta}^{(b)} - \hat{\theta})^2\\. `"chrostowski"`:
  \\(1/(B-1)) \sum (\hat{\theta}^{(b)} - \hat{\theta})^2\\ (NPS types
  only; errors for probability-sample types). `"uncentered"`: standard
  Bessel-corrected variance centered on the bootstrap mean. For
  probability-sample types, `"mse"` maps to `TRUE` and `"uncentered"`
  maps to `FALSE` in the `svrep` call. **Legacy note:** `mse = TRUE` or
  `mse = FALSE` (logical) is no longer accepted and emits
  `surveywts_error_mse_not_character`.

- seed:

  `integer(1)` or `NULL`. RNG seed. For NPS types,
  [`set.seed()`](https://rdrr.io/r/base/Random.html) is called once
  immediately before the bootstrap loop (or before the `svrep`
  pre-computation for Level B). The caller's global RNG state is **not**
  restored. For probability-sample types, the seed is applied via
  [`withr::local_seed()`](https://withr.r-lib.org/reference/with_seed.html)
  and the caller's state is restored.

## Value

- Probability-sample types → `survey_replicate` with `replicates` new
  `rep_1...rep_N` columns and a `"replicate_creation"` history entry.
  This holds for `survey_nonprob` input too: a `survey_nonprob` with a
  probability-sample `type` returns a `survey_replicate`.

- `type = "quasi-randomization"` → `survey_nonprob` with `replicates`
  new `repwt_1...repwt_B` columns in `@data`, `@variables$repweights`
  populated, and a `"bootstrap_weights"` history entry.

## Details

**When to use.** Choose the bootstrap when you estimate a median or
another quantile — the jackknife standard error for a quantile does not
improve as the sample grows (Elliott & Valliant 2017) — or when your
estimator has no textbook variance formula (Wu 2022). For a
non-probability sample with a reference probability sample, use
`type = "quasi-randomization"`, which refits the weighting model inside
every replicate.

A `survey_nonprob` passed with a probability-sample `type` (the default
included) is silently wrapped as a simple random sample: the replicates
resample rows with the base weights and ignore the propensity-estimation
step. To capture that step in the variance, use
`type = "quasi-randomization"` instead. The `survey_replicate` returned
by the wrapped path cannot be converted back with
[`as_taylor_design()`](https://jdenn0514.github.io/surveywts/reference/as_taylor_design.md)
— that function refuses an object whose source design was a
non-probability sample.

## Algorithm

For probability-sample designs (`survey_taylor`), delegates to
[`svrep::as_bootstrap_design()`](https://bschneidr.github.io/svrep/reference/as_bootstrap_design.html)
with the specified `type`. The variance estimator for resampling type
`"Rao-Wu-Yue-Beaumont"` is: \$\$\hat{V}\_{boot} = \frac{1}{B}
\sum\_{b=1}^{B} (\hat{\theta}^{(b)} - \hat{\theta})^2\$\$ when
`mse = "mse"`, with `B = replicates`.

For non-probability samples (`type = "quasi-randomization"`), each
bootstrap replicate resamples respondents with replacement (SRSWR), then
re-runs the original IPW fitting on the resampled data, producing
replicate weights that reflect the variability of the propensity
estimation step.

The refit returns weights in resample order, so each replicate column is
mapped back to original-unit order before it is stored. A unit the draw
picked more than once carries the sum of the weights of its copies, and
a unit the draw did not pick carries 0. Every `repwt_*` column therefore
holds about 37% zeros, and an estimator applied to the column reproduces
the estimate that draw produced.

## Limitations

Bootstrap standard errors from `type = "quasi-randomization"` likely
understate true sampling variability because SRSWR resampling cannot
replicate the original NPS recruitment mechanism (AAPOR 2022, §4). This
understatement is not reduced by increasing `replicates`.

## Messages

When the design carries a calibration, the replay re-runs it inside
every replicate. Replicates that already meet their margins are counted
and reported in one summary line, not announced one by one. A count
close to `replicates` means the replay changed little, so the variance
estimate may be near zero and deserves a look.

The replicate weight creators re-emit the svrep back end's notices under
their own classes, so you can quiet one without quieting the rest. Which
message you see depends on the function and the arguments.

- `surveywts_message_row_order_assumed` — from
  [`create_gen_boot_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_boot_weights.md)
  and
  [`create_gen_rep_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_rep_weights.md),
  when `variance_estimator` is `"SD1"` or `"SD2"`. Both estimators read
  the row order of the data, and nothing here checks that order.

- `surveywts_message_replicates_rounded_up` — from
  [`create_sdr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_sdr_weights.md),
  when the Hadamard matrix order is above `replicates`. The result then
  carries more replicate columns than you asked for, and
  `use_normal_hadamard` selects which orders are reachable: at the
  default, `replicates = 100` gives 128.

- `surveywts_message_replicates_subsampled` — from
  [`create_gen_rep_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_rep_weights.md),
  when `max_replicates` is below the fully efficient replicate count.
  The back end keeps a random sample of the replicates, so set `seed` to
  make the draw reproducible.

- `surveywts_message_backend_note` — from any of them, when the back end
  emits a message this package does not recognise. The text is
  unchanged.

To quiet one message and leave the rest alone:

    withCallingHandlers(
      create_sdr_weights(design, replicates = 100L),
      surveywts_message_replicates_rounded_up = function(cnd) {
        invokeRestart("muffleMessage")
      }
    )

## References

Elliott, M.R. and Valliant, R. (2017). Inference for nonprobability
samples. *Statistical Science* **32**(2), 249–264.

Wu, C. (2022). Statistical inference with non-probability survey
samples. *Survey Methodology* **48**(2), 283–311.

Chrostowski, L., Chlebicki, P. and Beresewicz, M. nonprobsvy — An R
package for modern methods for non-probability surveys.

## See also

[`create_jackknife_weights()`](https://jdenn0514.github.io/surveywts/reference/create_jackknife_weights.md),
[`create_brr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_brr_weights.md),
[`create_gen_boot_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_boot_weights.md),
[`create_gen_rep_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_rep_weights.md),
[`create_sdr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_sdr_weights.md),
[`create_replicate_weights()`](https://jdenn0514.github.io/surveywts/reference/create_replicate_weights.md),
[`as_taylor_design()`](https://jdenn0514.github.io/surveywts/reference/as_taylor_design.md).
For the class system, the standard workflows, and a glossary of terms,
see the [Getting started
article](https://jdenn0514.github.io/surveywts/articles/getting-started.html).

Other replicate-weights:
[`as_taylor_design()`](https://jdenn0514.github.io/surveywts/reference/as_taylor_design.md),
[`create_brr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_brr_weights.md),
[`create_gen_boot_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_boot_weights.md),
[`create_gen_rep_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_rep_weights.md),
[`create_jackknife_weights()`](https://jdenn0514.github.io/surveywts/reference/create_jackknife_weights.md),
[`create_replicate_weights()`](https://jdenn0514.github.io/surveywts/reference/create_replicate_weights.md),
[`create_sdr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_sdr_weights.md)

## Examples

``` r
# default SRSWR bootstrap on a probability survey -----------------------
gss_svy <- surveycore::as_survey(
  gss_2024, weights = wtssps, strata = vstrat, ids = vpsu, nest = TRUE
)
# Real analyses use more replicates; 100 keeps `R CMD check` fast.
boot_rep <- create_bootstrap_weights(gss_svy, replicates = 100L, seed = 1L)
summarize_weights(boot_rep)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv   min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  3309       3309      0     1 0.979 0.151 0.384 0.717  1.15  8.76 1689.
# The confidence interval below is computed from the 100 replicate
# columns, which is what the replicate weights are for.
surveycore::get_means(boot_rep, age)
#> # A tibble: 1 × 4
#>    mean ci_low ci_high     n
#>   <dbl>  <dbl>   <dbl> <int>
#> 1  47.9   46.9    49.0  3208

# quasi-randomization bootstrap for a calibrated non-probability sample ----
targets_a <- list(
  sex    = c("Male" = 0.49, "Female" = 0.51),
  age_f3 = c("18-34" = 0.30, "35-54" = 0.33, "55+" = 0.37)
)
ns_wave1_svy <- surveycore::as_survey_nonprob(ns_wave1, weights = weight)
ns_wave1_cal <- calibrate_rake(ns_wave1_svy, targets = targets_a)
# This path refits the weighting model inside every replicate, so it costs
# far more per replicate than the probability bootstrap above. Real
# analyses use more; 10 keeps `R CMD check` fast.
nps_rep <- create_bootstrap_weights(
  ns_wave1_cal,
  type       = "quasi-randomization",
  replicates = 10L,
  seed       = 1L
)
summarize_weights(nps_rep)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv     min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl>   <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  6422       6422      0     1  1.36 0.00373 0.152 0.399  1.13  4.96 2248.
```
