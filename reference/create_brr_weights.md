# Generate BRR (Fay) replicate weights

Generates BRR (balanced repeated replication) or Fay's BRR replicate
weights (sets of perturbed weight columns used to compute standard
errors) via
[`survey::as.svrepdesign()`](https://rdrr.io/pkg/survey/man/as.svrepdesign.html).
Requires a paired-PSU design: exactly 2 PSUs (primary sampling units:
the first units the design selects, such as counties or schools) per
stratum.

## Usage

``` r
create_brr_weights(data, ..., rho = 0, mse = TRUE)
```

## Arguments

- data:

  A `survey_taylor` with exactly 2 PSUs per stratum.

- ...:

  Must be empty.

- rho:

  `numeric(1)`, default `0`. Fay damping coefficient. `rho = 0` gives
  standard BRR; `rho > 0` gives Fay's BRR variant with factors `rho` and
  `2 - rho`. Must satisfy `0 <= rho < 1`.

- mse:

  `logical(1)`, default `TRUE`. Centers each replicate deviation on the
  full-sample estimate (`TRUE`; conservative) or on the mean of the
  replicate estimates (`FALSE`).

## Value

A `survey_replicate` with `@variables$type` of `"BRR"` or `"Fay"`.

## Details

**When to use.** Choose BRR when the design holds exactly two PSUs in
every stratum, and especially when you estimate a quantile or another
nonlinear statistic — BRR is proven for those, and no jackknife variant
is (Valliant, Dever & Kreuter 2018, Section 15.4). Set `rho > 0` when
you estimate a ratio: with the default `rho = 0`, one PSU per stratum
drops to zero weight in each replicate, which can leave a ratio
undefined (Dippo, Fay & Morganstein 1984).

## Algorithm

BRR creates \\R\\ half-sample replicates from a paired-PSU design
(exactly 2 PSUs per stratum). A Hadamard matrix of order \\R\\
determines which PSU in each stratum belongs to each half-sample. Within
replicate \\r\\, PSU 1 receives weight \\2(1-\rho)\\ and PSU 2 receives
weight \\2\rho\\ (or vice versa). The BRR variance estimator is:
\$\$\hat{V}\_{BRR} = \frac{1}{R(1-\rho)^2} \sum\_{r=1}^{R}
(\hat{\theta}^{(r)} - \hat{\theta})^2.\$\$ When `rho = 0`, this
simplifies to standard BRR. The Fay variant (`rho > 0`) reduces variance
instability from extreme replicate estimates.

## Messages

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

Fay, R.E. (1984). Some properties of estimates of variance based on
replication methods. *Proceedings of the Section on Survey Research
Methods, American Statistical Association*, 495–500.

Fay, R.E. (1989). Theory and application of replicate weighting for
variance calculations. *Proceedings of the Section on Survey Research
Methods, American Statistical Association*.

Dippo, C., Fay, R.E. and Morganstein, D. (1984). Computing variances
from complex samples with replicate weights. *Proceedings of the Section
on Survey Research Methods, American Statistical Association*, 489–494.

Valliant, R., Dever, J. and Kreuter, F. (2018). *Practical Tools for
Designing and Weighting Survey Samples*, 2nd edition. New York:
Springer.

## See also

[`create_bootstrap_weights()`](https://jdenn0514.github.io/surveywts/reference/create_bootstrap_weights.md),
[`create_jackknife_weights()`](https://jdenn0514.github.io/surveywts/reference/create_jackknife_weights.md),
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
[`create_bootstrap_weights()`](https://jdenn0514.github.io/surveywts/reference/create_bootstrap_weights.md),
[`create_gen_boot_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_boot_weights.md),
[`create_gen_rep_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_rep_weights.md),
[`create_jackknife_weights()`](https://jdenn0514.github.io/surveywts/reference/create_jackknife_weights.md),
[`create_replicate_weights()`](https://jdenn0514.github.io/surveywts/reference/create_replicate_weights.md),
[`create_sdr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_sdr_weights.md)

## Examples

``` r
# standard BRR on a stratified 2-PSU-per-stratum design ----------------
gss_svy <- surveycore::as_survey(
  gss_2024, weights = wtssps, strata = vstrat, ids = vpsu, nest = TRUE
)
brr_design <- create_brr_weights(gss_svy)
summarize_weights(brr_design)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv   min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  3309       3309      0     1 0.979 0.151 0.384 0.717  1.15  8.76 1689.
# The confidence interval below is computed from the BRR half-sample
# columns rather than by Taylor linearization.
surveycore::get_means(brr_design, age)
#> # A tibble: 1 × 4
#>    mean ci_low ci_high     n
#>   <dbl>  <dbl>   <dbl> <int>
#> 1  47.9   47.0    48.9  3208

# Fay's BRR with damping coefficient -----------------------------------
fay_design <- create_brr_weights(gss_svy, rho = 0.5)
summarize_weights(fay_design)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv   min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  3309       3309      0     1 0.979 0.151 0.384 0.717  1.15  8.76 1689.
```
