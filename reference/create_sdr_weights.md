# Generate successive difference replication weights

Generates successive difference replication weights — a form of
replicate weights (sets of perturbed weight columns used to compute
standard errors) — via
[`svrep::as_sdr_design()`](https://bschneidr.github.io/svrep/reference/as_sdr_design.html).
Requires a `survey_taylor` design.

## Usage

``` r
create_sdr_weights(
  data,
  replicates = 100L,
  ...,
  sort_var = NULL,
  use_normal_hadamard = FALSE,
  mse = TRUE
)
```

## Arguments

- data:

  A `survey_taylor` design. PSUs should be in systematic selection
  order, or use `sort_var`.

- replicates:

  `integer(1)`, default `100L`. Target replicate count (\>= 4). Actual
  count may be slightly larger due to Hadamard matrix sizing.

- ...:

  Must be empty.

- sort_var:

  \<[`tidy-select`](https://tidyselect.r-lib.org/reference/language.html)\>
  Bare column name giving the systematic selection order. Required for
  stratified designs (svrep \>= 0.9.1); for non-stratified designs row
  order is used as fallback.

- use_normal_hadamard:

  `logical(1)`, default `FALSE`. Selects which Hadamard orders the
  replicate count can take. `FALSE` gives orders that double from 4;
  `TRUE` gives a finer grid, so the count sits closer to `replicates`.
  The two settings give different variance estimates once the PSU count
  exceeds the smaller order — see the **Hadamard order and the column
  count** part of the Algorithm section.

- mse:

  `logical(1)`, default `TRUE`. Centers each replicate deviation on the
  full-sample estimate (`TRUE`; conservative) or on the mean of the
  replicate estimates (`FALSE`).

## Value

A `survey_replicate` with `@variables$type = "successive-difference"`.

## Details

**When to use.** Choose successive difference replication only when the
sample was drawn systematically from a sorted list and the rows still
carry that order (Ash 2014). The row order is part of the method:
re-sorted rows give a different and incorrect answer, and no error is
raised. Pass `sort_var` to pin the order.

This estimator targets the variance of a systematic random sample when
PSUs are in selection order (Ash, 2014; Fay & Train, 1995). See the
Algorithm section for when the match is exact.

## Algorithm

Successive difference replication (SDR) pairs adjacent PSUs in
systematic selection order. A Hadamard matrix of order \\R\\ assigns
each pair to a half-sample. The SDR variance estimator is:
\$\$\hat{V}\_{SDR} = \frac{4}{R} \sum\_{r=1}^{R} (\hat{\theta}^{(r)} -
\hat{\theta}\_{\text{full}})^2.\$\$ The match to SD2 is exact only while
the unit count does not exceed \\R\\. Above that the row assignment
recycles row pairs, so SDR approximates SD2 rather than reproducing it.
The bundled `cps_2023` example is in that regime. Delegates to
[`svrep::as_sdr_design()`](https://bschneidr.github.io/svrep/reference/as_sdr_design.html).

**Hadamard order and the column count.** The number of replicate columns
is the order of the Hadamard matrix, not `replicates`.
`use_normal_hadamard` selects which orders are reachable. At `FALSE`,
the default, the order doubles from 4 — 4, 8, 16, 32, 64, 128, 256, 512
and so on — and the smallest such order at or above `replicates` is the
one returned. At `TRUE` the order comes from
[`survey::hadamard()`](https://rdrr.io/pkg/survey/man/hadamard.html),
which supplies a finer grid: 20, 40, 56, 104 and 128 are all reachable,
so the count sits closer to `replicates`. A request the finer grid
cannot meet still rounds up — 52 returns 56. At `TRUE` some replicates
may be inactive: all of their replicate factors equal 1, so each equals
the full sample. The count of them rises as the PSU count falls relative
to the order, and it is not capped. An inactive replicate is valid. It
contributes a zero term to the variance sum, and the scale \\4/R\\
counts it, which is what keeps the estimator unbiased.

The check to run is your PSU count against the order you would land on.
While the PSU count does not exceed the smaller order, both settings
give the same answer and the smaller order is free. Above that the two
settings give different variance estimates, and the gap grows with the
PSU count. Measured at `replicates = 50` on a design of 480 rows in four
strata, the standard error moved by about 2% at 80 PSUs, 5% at 160 and
15% at 480. Both estimates are valid. Keep the default `FALSE` to
reproduce existing work.

Two further differences. At the same order and `mse = TRUE`, the
default, both settings give the same variance for a total, but a mean
can differ, because a mean is a ratio whose denominator varies by
replicate and the inactive replicates enter it. At `mse = FALSE` the two
settings differ even at the same order, for the same reason.

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
  `create_sdr_weights()`, when the Hadamard matrix order is above
  `replicates`. The result then carries more replicate columns than you
  asked for, and `use_normal_hadamard` selects which orders are
  reachable: at the default, `replicates = 100` gives 128.

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

Ash, S. (2014). Using successive difference replication for estimating
variances. *Survey Methodology, Statistics Canada*, 40(1), 47–59.

Fay, R.E. and Train, G.F. (1995). Aspects of survey and model-based
postcensal estimation of income and poverty characteristics for states
and counties. *Joint Statistical Meetings, Proceedings of the Section on
Government Statistics*, 154–159.

## See also

[`create_bootstrap_weights()`](https://jdenn0514.github.io/surveywts/reference/create_bootstrap_weights.md),
[`create_jackknife_weights()`](https://jdenn0514.github.io/surveywts/reference/create_jackknife_weights.md),
[`create_brr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_brr_weights.md),
[`create_gen_boot_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_boot_weights.md),
[`create_gen_rep_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_rep_weights.md),
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
[`create_gen_rep_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_rep_weights.md),
[`create_jackknife_weights()`](https://jdenn0514.github.io/surveywts/reference/create_jackknife_weights.md),
[`create_replicate_weights()`](https://jdenn0514.github.io/surveywts/reference/create_replicate_weights.md)

## Examples

``` r
# apply SDR to a Taylor-linearization design ---------------------------
# `cps_2023` carries no strata or PSU columns, so this design has neither.
cps_design <- surveycore::as_survey(cps_2023, weights = wtfinl)
# `replicates = 50L` returns 64 columns: the count is a Hadamard matrix
# order, and the default path doubles from 4 until it reaches 50.
sdr_design <- create_sdr_weights(cps_design, replicates = 50L)
#> ℹ `replicates` is 50, and the result has 64 replicate columns.
#> ℹ Successive difference replication takes the column count from the order of a
#>   Hadamard matrix, and `use_normal_hadamard` controls which orders are
#>   reachable.
summarize_weights(sdr_design)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv   min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  9999       9999      0 1890. 0.657  106.  966. 1667. 2637. 9475. 6985.
# The confidence interval below is computed from the 64 replicate
# columns rather than by Taylor linearization.
surveycore::get_means(sdr_design, age)
#> # A tibble: 1 × 4
#>    mean ci_low ci_high     n
#>   <dbl>  <dbl>   <dbl> <int>
#> 1  47.0   46.6    47.4  9999

# ask for a count closer to `replicates` --------------------------------
# The finer grid of Hadamard orders reaches 56, so the same request
# returns 56 columns rather than 64.
sdr_normal <- create_sdr_weights(
  cps_design,
  replicates = 50L,
  use_normal_hadamard = TRUE
)
#> ℹ `replicates` is 50, and the result has 56 replicate columns.
#> ℹ Successive difference replication takes the column count from the order of a
#>   Hadamard matrix, and `use_normal_hadamard` controls which orders are
#>   reachable.
length(sdr_normal@variables$repweights)
#> [1] 56
```
