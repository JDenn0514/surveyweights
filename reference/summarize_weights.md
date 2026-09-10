# Report summary statistics for the weight distribution

Returns a tibble of distribution statistics for the weight column; the
full column set is listed under **Value**. Rows with a zero weight
(typically produced by
[`adjust_nonresponse()`](https://jdenn0514.github.io/surveywts/reference/adjust_nonresponse.md))
are excluded before the statistics are computed. Pass `by` to compute
statistics separately within each subgroup defined by one or more
grouping variables.

## Usage

``` r
summarize_weights(x, weights = NULL, by = NULL)
```

## Arguments

- x:

  A `survey_taylor`, `survey_nonprob`, or `survey_replicate`. The weight
  column is auto-detected from `@variables$weights`.

- weights:

  Bare name (NSE). Weight column. Auto-detected from survey object
  `@variables$weights`.

- by:

  \<[`tidy-select`](https://tidyselect.r-lib.org/reference/language.html)\>
  Optional grouping variables. When `NULL` (the default), a single-row
  summary over all observations is returned. When specified, one row is
  returned per unique group combination.

## Value

A tibble with columns `n` (rows summarized), `n_positive` (rows with a
weight above zero), `n_zero` (rows with a weight of exactly zero),
`mean`, `cv`, `min`, `p25`, `p50`, `p75`, `max`, and `ess` (Kish
effective sample size). Because zero-weight rows are excluded before the
summary, `n_zero` is `0` and `n_positive` equals `n`. When `by` is
non-`NULL`, the group columns precede the summary columns.

## See also

[`effective_sample_size()`](https://jdenn0514.github.io/surveywts/reference/effective_sample_size.md),
[`weight_variability()`](https://jdenn0514.github.io/surveywts/reference/weight_variability.md).
For the class system, the standard workflows, and a glossary of terms,
see the [Getting started
article](https://jdenn0514.github.io/surveywts/articles/getting-started.html).

Other diagnostics:
[`effective_sample_size()`](https://jdenn0514.github.io/surveywts/reference/effective_sample_size.md),
[`weight_variability()`](https://jdenn0514.github.io/surveywts/reference/weight_variability.md)

## Examples

``` r
ns_wave1_svy <- surveycore::as_survey_nonprob(ns_wave1, weights = weight)

# overall summary --------------------------------------------------------
summarize_weights(ns_wave1_svy)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv     min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl>   <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  6422       6422      0  1.00  1.36 0.00382 0.153 0.400  1.13  4.78 2255.
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv     min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl>   <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  6422       6422      0  1.00  1.36 0.00382 0.153 0.400  1.13  4.78 2255.

# grouped by sex ---------------------------------------------------------
summarize_weights(ns_wave1_svy, by = sex)
#> # A tibble: 2 × 12
#>   sex        n n_positive n_zero  mean    cv     min   p25   p50   p75   max
#>   <fct>  <int>      <int>  <int> <dbl> <dbl>   <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1 Male    3632       3632      0 0.877  1.44 0.00382 0.138 0.353 0.893  4.78
#> 2 Female  2790       2790      0 1.16   1.26 0.00382 0.173 0.494 1.47   4.77
#> # ℹ 1 more variable: ess <dbl>
#> # A tibble: 2 × 12
#>   sex        n n_positive n_zero  mean    cv     min   p25   p50   p75   max
#>   <fct>  <int>      <int>  <int> <dbl> <dbl>   <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1 Male    3632       3632      0 0.877  1.44 0.00382 0.138 0.353 0.893  4.78
#> 2 Female  2790       2790      0 1.16   1.26 0.00382 0.173 0.494 1.47   4.77
#> # ℹ 1 more variable: ess <dbl>
```
