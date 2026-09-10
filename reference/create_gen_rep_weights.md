# Generate generalized replication replicate weights

Generates Fay's generalized replication weights via
[`svrep::as_fays_gen_rep_design()`](https://bschneidr.github.io/svrep/reference/as_fays_gen_rep_design.html).
Produces deterministic replicate weights (sets of perturbed weight
columns used to compute standard errors); no randomness is involved.
Requires a `survey_taylor` design.

## Usage

``` r
create_gen_rep_weights(
  data,
  ...,
  variance_estimator = "SD2",
  max_replicates = Inf,
  balanced = TRUE,
  aux_var_names = NULL,
  mse = TRUE,
  seed = NULL
)
```

## Arguments

- data:

  A `survey_taylor` design.

- ...:

  Must be empty.

- variance_estimator:

  `character(1)`. Target variance estimator. Same options as
  [`create_gen_boot_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_boot_weights.md).
  Default `"SD2"`.

- max_replicates:

  `numeric(1)`, default `Inf`. Maximum number of replicates; `Inf` uses
  the natural count.

- balanced:

  `logical(1)`, default `TRUE`. Equal contribution of replicates to
  variance estimates.

- aux_var_names:

  `<tidy-select>` or `NULL`. Required for `"Deville-Tille"`.

- mse:

  `logical(1)`, default `TRUE`. Centers each replicate deviation on the
  full-sample estimate (`TRUE`; conservative) or on the mean of the
  replicate estimates (`FALSE`).

- seed:

  `integer(1)` or `NULL`. RNG seed for reproducibility. The construction
  is deterministic unless `max_replicates` is below the fully efficient
  replicate count; in that case the svrep back-end retains a random
  sample of replicates, and the seed makes that draw reproducible.

## Value

A `survey_replicate` with generalized replication weights and
`@variables$type = "other"` (the svrep back-end does not assign a
method-specific type for generalized replication).

## Details

**When to use.** Choose generalized replication when you want the
deterministic counterpart of the generalized bootstrap: it is built from
the same target variance estimators, but decomposes the target matrix
into fixed components instead of drawing random multipliers (Fay 1989).
There is no `replicates` argument — the construction sets the count, and
`max_replicates` caps it. For choosing `variance_estimator`, see the
**Choosing a target** section of
[`create_gen_boot_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_boot_weights.md);
note the defaults differ (`"SD2"` here, `"SD1"` there).

**Row order.** The default `"SD2"` reads the row order of the data. It
assumes the rows are still in the order the sample was drawn in (Ash
2014). Re-sorted rows give a different and incorrect answer, and no
error is raised. Sort the rows into selection order before you call this
function. `"SD1"` reads the row order in the same way; no other
`variance_estimator` does.

## Algorithm

Generalized replication (GR) is a BRR extension that removes the
requirement for exactly 2 PSUs per stratum. It constructs \\R \geq H\\
replicates (where \\H\\ is the number of strata) using a generalized
Hadamard matrix, assigning each stratum-PSU unit a weight that satisfies
the BRR variance formula \$\$\hat{V}\_{GR} = \frac{1}{R} \sum\_{r=1}^{R}
(\hat{\theta}^{(r)} - \hat{\theta})^2.\$\$ Delegates to
[`svrep::as_fays_gen_rep_design()`](https://bschneidr.github.io/svrep/reference/as_fays_gen_rep_design.html).

## Messages

The replicate weight creators re-emit the svrep back end's notices under
their own classes, so you can quiet one without quieting the rest. Which
message you see depends on the function and the arguments.

- `surveywts_message_row_order_assumed` — from
  [`create_gen_boot_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_boot_weights.md)
  and `create_gen_rep_weights()`, when `variance_estimator` is `"SD1"`
  or `"SD2"`. Both estimators read the row order of the data, and
  nothing here checks that order.

- `surveywts_message_replicates_rounded_up` — from
  [`create_sdr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_sdr_weights.md),
  when the Hadamard matrix order is above `replicates`. The result then
  carries more replicate columns than you asked for, and
  `use_normal_hadamard` selects which orders are reachable: at the
  default, `replicates = 100` gives 128.

- `surveywts_message_replicates_subsampled` — from
  `create_gen_rep_weights()`, when `max_replicates` is below the fully
  efficient replicate count. The back end keeps a random sample of the
  replicates, so set `seed` to make the draw reproducible.

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

Ash, S. (2014). Using successive difference replication for estimating
variances. *Survey Methodology, Statistics Canada*, 40(1), 47–59.

## See also

[`create_bootstrap_weights()`](https://jdenn0514.github.io/surveywts/reference/create_bootstrap_weights.md),
[`create_jackknife_weights()`](https://jdenn0514.github.io/surveywts/reference/create_jackknife_weights.md),
[`create_brr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_brr_weights.md),
[`create_gen_boot_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_boot_weights.md),
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
[`create_gen_boot_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_boot_weights.md),
[`create_jackknife_weights()`](https://jdenn0514.github.io/surveywts/reference/create_jackknife_weights.md),
[`create_replicate_weights()`](https://jdenn0514.github.io/surveywts/reference/create_replicate_weights.md),
[`create_sdr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_sdr_weights.md)

## Examples

``` r
# generalized replication with reproducible seed -----------------------
gss_svy <- surveycore::as_survey(
  gss_2024, weights = wtssps, strata = vstrat, ids = vpsu, nest = TRUE
)
# The construction is deterministic at the default `max_replicates = Inf`;
# `seed` only matters below that count, so it is defensive here.
gen_rep_design <- create_gen_rep_weights(gss_svy, seed = 42L)
#> ℹ `variance_estimator` is "SD2", which reads the row order of the data. It
#>   assumes the rows are still in the order the sample was drawn in.
#> ✔ Sort the rows into selection order before you call this function.
summarize_weights(gen_rep_design)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv   min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  3309       3309      0     1 0.979 0.151 0.384 0.717  1.15  8.76 1689.
# The confidence interval below is computed from the replicate columns
# rather than by Taylor linearization.
surveycore::get_means(gen_rep_design, age)
#> # A tibble: 1 × 4
#>    mean ci_low ci_high     n
#>   <dbl>  <dbl>   <dbl> <int>
#> 1  47.9   47.0    48.9  3208
```
