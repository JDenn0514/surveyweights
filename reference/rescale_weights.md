# Rescale survey weights to a target mean or sum

Rescales weights so they sum to the sample size `n` (globally) or to the
group sample size within each group (when `by` is specified). Relative
weights within the sample are preserved exactly. Applies to main weights
and — for inputs carrying replicate weight columns (`survey_replicate`
or `survey_nonprob` with `repweights`) — all replicate columns.

## Usage

``` r
rescale_weights(data, weights = NULL, by = NULL, wt_name = NULL)
```

## Arguments

- data:

  A `survey_taylor`, `survey_nonprob`, or `survey_replicate`. For inputs
  carrying replicate weight columns, see the **Replicate Weights**
  section.

- weights:

  \<[`tidy-select`](https://tidyselect.r-lib.org/reference/language.html)\>
  Weight column. Auto-detected from survey object `@variables$weights`.

- by:

  \<[`tidy-select`](https://tidyselect.r-lib.org/reference/language.html)\>
  Grouping variable(s). Rescaling is performed within each group
  (weights in group `h` sum to `n_h`). `NULL` → global rescaling (all
  weights sum to `n`).

- wt_name:

  `NULL` (default) or a `character(1)`. When `NULL`, rescaled weights
  overwrite the existing weight column in place. When a character
  string, a new column is added and `@variables$weights` updated.

## Value

An object of the same class as `data` with rescaled weights. A new entry
with `operation = "rescale_weights"` is appended to the weighting
history.

## Details

The scale factor is `n / sum(w)` globally, or `n_h / W_h` per group `h`.
This operation preserves ratio estimators (means, proportions) but
changes population total estimates by the factor `n / W`.

## Replicate Weights

When the input carries replicate weight columns (either
`survey_replicate` or `survey_nonprob` with `repweights`), all replicate
columns are scaled by the same factor(s) derived from the main weights —
globally `n / sum(w)` or per group `n_h / W_h`. Each row's replicate
values are multiplied by the same scalar (global) or per-group scalar
that was applied to the main weight.

## See also

[`trim_weights()`](https://jdenn0514.github.io/surveywts/reference/trim_weights.md).
For the class system, the standard workflows, and a glossary of terms,
see the [Getting started
article](https://jdenn0514.github.io/surveywts/articles/getting-started.html).

Other utilities:
[`trim_weights()`](https://jdenn0514.github.io/surveywts/reference/trim_weights.md)

## Examples

``` r
ns_wave1_svy <- surveycore::as_survey_nonprob(ns_wave1, weights = weight)

# Rescale weights to unit mean (default) -----------------------------------
summarize_weights(ns_wave1_svy)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv     min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl>   <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  6422       6422      0  1.00  1.36 0.00382 0.153 0.400  1.13  4.78 2255.

result <- rescale_weights(ns_wave1_svy)
summarize_weights(result)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv     min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl>   <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  6422       6422      0     1  1.36 0.00382 0.153 0.400  1.13  4.78 2255.

# Rescale within groups using by = -----------------------------------------
result_by <- rescale_weights(ns_wave1_svy, by = ns_region)
summarize_weights(result_by, by = ns_region)
#> # A tibble: 4 × 12
#>   ns_region     n n_positive n_zero  mean    cv     min   p25   p50   p75   max
#>   <fct>     <int>      <int>  <int> <dbl> <dbl>   <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1 West       1387       1387      0     1  1.33 0.00351 0.132 0.387  1.15  4.39
#> 2 South      2399       2399      0     1  1.35 0.00377 0.157 0.394  1.13  4.72
#> 3 Midwest    1469       1469      0     1  1.35 0.00414 0.172 0.434  1.14  5.14
#> 4 Northeast  1167       1167      0     1  1.41 0.00397 0.143 0.388  1.05  4.93
#> # ℹ 1 more variable: ess <dbl>

# survey_replicate: the replicate columns scale by the same factor ----------
cps_svy <- surveycore::as_survey_replicate(
  cps_2023,
  weights    = "wtfinl",
  repweights = paste0("repwtp", 1:160),
  type       = "successive-difference",
  scale      = 4 / 160,
  rscales    = rep(1, 160)
)
result_rep <- rescale_weights(cps_svy)
summarize_weights(result_rep)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv    min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl>  <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  9999       9999      0     1 0.657 0.0560 0.511 0.882  1.40  5.01 6985.

# The first replicate column moved by the same factor as the main weight.
mean(surveycore::survey_data(cps_svy)$repwtp1)
#> [1] 1886.274
mean(surveycore::survey_data(result_rep)$repwtp1)
#> [1] 0.9977692
```
