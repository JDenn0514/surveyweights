# Generate generalized bootstrap replicate weights

Generates generalized bootstrap replicate weights (sets of perturbed
weight columns used to compute standard errors) via
[`svrep::as_gen_boot_design()`](https://bschneidr.github.io/svrep/reference/as_gen_boot_design.html).
Requires a `survey_taylor` design.

## Usage

``` r
create_gen_boot_weights(
  data,
  replicates = 500L,
  ...,
  variance_estimator = "SD1",
  tau = 1,
  aux_var_names = NULL,
  mse = TRUE,
  seed = NULL
)
```

## Arguments

- data:

  A `survey_taylor` design.

- replicates:

  `integer(1)`, default `500L`. Number of bootstrap replicates.

- ...:

  Must be empty.

- variance_estimator:

  `character(1)`. Target variance estimator. One of `"SD1"` (default),
  `"SD2"`, `"Horvitz-Thompson"`, `"Yates-Grundy"`,
  `"Poisson Horvitz-Thompson"`, `"Stratified Multistage SRS"`,
  `"Ultimate Cluster"`, `"Deville-1"`, `"Deville-2"`, `"Deville-Tille"`,
  `"BOSB"`, or `"Beaumont-Emond"`.

- tau:

  `numeric(1)` or `"auto"`, default `1`. Rescaling constant to prevent
  negative replicate weights.

- aux_var_names:

  \<[`tidy-select`](https://tidyselect.r-lib.org/reference/language.html)\>
  or `NULL`. Auxiliary variable columns. Required when
  `variance_estimator = "Deville-Tille"`.

- mse:

  `logical(1)`, default `TRUE`. Centers each replicate deviation on the
  full-sample estimate (`TRUE`; conservative) or on the mean of the
  replicate estimates (`FALSE`).

- seed:

  `integer(1)` or `NULL`. RNG seed.

## Value

A `survey_replicate` with `@variables$type = "bootstrap"` (the stored
replicate type, not the method name).

## Details

**When to use.** Choose the generalized bootstrap when the design fits
none of the named methods — Poisson sampling, or an unusual multi-stage
structure — or when you must name the target variance estimator yourself
(Beaumont & Patak 2012). The choice of `variance_estimator` is the
central decision; see the **Choosing a target** section. Beaumont &
Patak recommend at least 750 replicates; the default is 500.

## Algorithm

The generalized bootstrap (Beaumont & Patak, 2012) generates replicate
weights using unit-level random multipliers: \$\$w_k^{(r)} = w_k \cdot
u_k^{(r)}\$\$ where \\u_k^{(r)}\\ are drawn from a distribution
calibrated to the design's first-order inclusion probabilities. Unlike
SRSWR bootstrap, the multipliers are chosen to satisfy \\E\[u_k\] = 1\\
and \\Var(u_k) = (1 - \pi_k) / \pi_k\\. Delegates to
[`svrep::as_gen_boot_design()`](https://bschneidr.github.io/svrep/reference/as_gen_boot_design.html).

## Choosing a target

`variance_estimator` names the variance formula the replicate weights
are built to reproduce. The mapped sources cover five of the options:

- `"SD1"` (the default) and `"SD2"`: the successive-difference
  estimators, which read the row order of the data (Ash 2014). They suit
  systematic samples.

- `"Horvitz-Thompson"`: valid for any design with computable inclusion
  probabilities, but for some designs its form implies a negative
  variance and the construction fails (Beaumont & Patak 2012).

- `"Yates-Grundy"`: requires a fixed sample size; not appropriate for
  Poisson sampling (Beaumont & Patak 2012).

- `"Poisson Horvitz-Thompson"`: the valid choice under Poisson sampling
  (Beaumont & Patak 2012).

The remaining options (`"Stratified Multistage SRS"`,
`"Ultimate Cluster"`, `"Deville-1"`, `"Deville-2"`, `"Deville-Tille"`,
`"BOSB"`, `"Beaumont-Emond"`) come from the svrep back end; see
[`svrep::as_gen_boot_design()`](https://bschneidr.github.io/svrep/reference/as_gen_boot_design.html)
for their definitions.

## Messages

The replicate weight creators re-emit the svrep back end's notices under
their own classes, so you can quiet one without quieting the rest. Which
message you see depends on the function and the arguments.

- `surveywts_message_row_order_assumed` — from
  `create_gen_boot_weights()` and
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

Beaumont, J.-F. and Patak, Z. (2012). On the generalized bootstrap for
sample surveys with special attention to Poisson sampling.
*International Statistical Review*, 80(1), 127–148.

Fay, R.E. (1984). Some properties of estimates of variance based on
replication methods. *Proceedings of the Section on Survey Research
Methods, American Statistical Association*, 495–500.

Dippo, C., Fay, R.E. and Morganstein, D. (1984). Computing variances
from complex samples with replicate weights. *Proceedings of the Section
on Survey Research Methods, American Statistical Association*, 489–494.

Bellhouse, D.R. (1985). Computing methods for variance estimation in
complex surveys. *Journal of Official Statistics*, 1(3), 323–329.

Ash, S. (2014). Using successive difference replication for estimating
variances. *Survey Methodology*, 40(1), 47–59.

## See also

[`create_bootstrap_weights()`](https://jdenn0514.github.io/surveywts/reference/create_bootstrap_weights.md),
[`create_jackknife_weights()`](https://jdenn0514.github.io/surveywts/reference/create_jackknife_weights.md),
[`create_brr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_brr_weights.md),
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
[`create_brr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_brr_weights.md),
[`create_gen_rep_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_rep_weights.md),
[`create_jackknife_weights()`](https://jdenn0514.github.io/surveywts/reference/create_jackknife_weights.md),
[`create_replicate_weights()`](https://jdenn0514.github.io/surveywts/reference/create_replicate_weights.md),
[`create_sdr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_sdr_weights.md)

## Examples

``` r
# generalized bootstrap with reproducible seed -------------------------
gss_svy <- surveycore::as_survey(
  gss_2024, weights = wtssps, strata = vstrat, ids = vpsu, nest = TRUE
)
# Real analyses use more replicates; 100 keeps `R CMD check` fast.
gen_boot_design <- create_gen_boot_weights(
  gss_svy,
  replicates = 100L,
  seed = 42L
)
#> ℹ `variance_estimator` is "SD1", which reads the row order of the data. It
#>   assumes the rows are still in the order the sample was drawn in.
#> ✔ Sort the rows into selection order before you call this function.
summarize_weights(gen_boot_design)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv   min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  3309       3309      0     1 0.979 0.151 0.384 0.717  1.15  8.76 1689.
# The confidence interval below is computed from the 100 replicate
# columns, which is what the replicate weights are for.
surveycore::get_means(gen_boot_design, age)
#> # A tibble: 1 × 4
#>    mean ci_low ci_high     n
#>   <dbl>  <dbl>   <dbl> <int>
#> 1  47.9   46.9    48.9  3208
```
