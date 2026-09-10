# Clip weights to a bounded interval

Clips (trims) survey weights to `[lower, upper]` and redistributes the
trimmed excess equally across untrimmed units, preserving the total
weight sum. Bounds can be absolute weight values or percentiles. Applies
to main weights and — for inputs carrying replicate weight columns
(`survey_replicate` or `survey_nonprob` with `repweights`) — all
replicate columns.

## Usage

``` r
trim_weights(
  data,
  weights = NULL,
  lower = NULL,
  upper = NULL,
  k = 5,
  type = c("absolute", "percentile"),
  strict = FALSE,
  wt_name = NULL
)
```

## Arguments

- data:

  A `survey_taylor`, `survey_nonprob`, or `survey_replicate`. For inputs
  carrying replicate weight columns, see the **Replicate Weights**
  section.

- weights:

  \<[`tidy-select`](https://tidyselect.r-lib.org/reference/language.html)\>
  Weight column. Auto-detected from survey object `@variables$weights`.

- lower:

  `numeric(1)` or `NULL`. Lower bound. `NULL` means no lower trimming.
  When `type = "percentile"`, interpreted as a quantile in \[0, 1\].
  Must not be `NA`.

- upper:

  `numeric(1)` or `NULL`. Upper bound. `NULL` with `type = "absolute"`
  (the default) computes the cutoff as `median(w) + k * IQR(w)`. `NULL`
  is incompatible with `type = "percentile"`. When `type = "absolute"`,
  must be strictly positive. When `type = "percentile"`, must be in \[0,
  1\]. Must not be `NA`.

- k:

  `numeric(1)`. IQR multiplier used only when `upper = NULL` and
  `type = "absolute"`. Default `5` (Potter & Zheng 2015). Must be
  positive. Ignored when `upper` is specified.

- type:

  `character(1)`. `"absolute"` (default): bounds are raw weight values.
  `"percentile"`: bounds are quantiles on \[0, 1\].

- strict:

  `logical(1)`. When `FALSE` (default), one clip-and-redistribute pass
  is applied. When `TRUE`, the loop repeats until all main weights fall
  within `[lower_abs, upper_abs]`. Not applied to replicate columns.

- wt_name:

  `NULL` (default) or a `character(1)`. When `NULL`, trimmed weights
  overwrite the existing weight column in place. When a character
  string, a new column is added and `@variables$weights` updated.

## Value

An object of the same class as `data` with trimmed weights. A new entry
with `operation = "trim_weights"` is appended to the weighting history.

## Details

The clip-and-redistribute logic is adapted from
[`survey::trimWeights()`](https://rdrr.io/pkg/survey/man/trimWeights.html)
(Thomas Lumley). The default upper cutoff follows Potter & Zheng (2015):
`median(w) + k * IQR(w)` with `k = 5`.

## Replicate Weights

When the input carries replicate weight columns (either
`survey_replicate` or `survey_nonprob` with `repweights`), the same
absolute bounds `[lower_abs, upper_abs]` derived from the main weights
are applied to each replicate column using the same
clip-and-redistribute logic. The `strict` loop is not applied to
replicate columns; each replicate column receives exactly one
clip-and-redistribute pass.

## Algorithm

Bounds are computed from the main weights:

- `type = "absolute"` with `upper = NULL`:
  `upper_abs = median(w) + k * IQR(w)`; `lower_abs = lower` (or `0` if
  `NULL`).

- `type = "absolute"` with `upper` specified: `upper_abs = upper`,
  `lower_abs = lower` (or `0` if `NULL`).

- `type = "percentile"`: `lower_abs = quantile(w, lower)`,
  `upper_abs = quantile(w, upper)`.

Within each clip-and-redistribute pass, weights above `upper_abs` are
clipped to `upper_abs`; weights below `lower_abs` are clipped to
`lower_abs`. The clipped mass is redistributed equally across untrimmed
observations so that the total weight sum is preserved. When
`strict = TRUE`, the pass repeats until all weights satisfy
`[lower_abs, upper_abs]`.

## References

Potter, F. and Zheng, Y. (2015). Methods and issues in trimming extreme
weights in sample surveys. *Proceedings of the Joint Statistical
Meetings, Section on Survey Research Methods*, 2707–2719.

## See also

[`rescale_weights()`](https://jdenn0514.github.io/surveywts/reference/rescale_weights.md).
For the class system, the standard workflows, and a glossary of terms,
see the [Getting started
article](https://jdenn0514.github.io/surveywts/articles/getting-started.html).

Other utilities:
[`rescale_weights()`](https://jdenn0514.github.io/surveywts/reference/rescale_weights.md)

## Examples

``` r
ns_wave1_svy <- surveycore::as_survey_nonprob(ns_wave1, weights = weight)

# IQR default (k = 5) ---------------------------------------------------
summarize_weights(ns_wave1_svy)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv     min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl>   <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  6422       6422      0  1.00  1.36 0.00382 0.153 0.400  1.13  4.78 2255.
result_iqr <- trim_weights(ns_wave1_svy)
#> Warning: ! No weights were trimmed: all main weights already fall within [-Inf,
#>   5.29094227304786].
summarize_weights(result_iqr)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv     min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl>   <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  6422       6422      0  1.00  1.36 0.00382 0.153 0.400  1.13  4.78 2255.
# Warns: "No weights were trimmed: all main weights already fall within
# [-Inf, 5.29094227304786]." The IQR-default upper bound sits above every
# weight, so there is nothing to trim.

# explicit percentile bounds --------------------------------------------
result_pct <- trim_weights(
  ns_wave1_svy, lower = 0.05, upper = 0.95, type = "percentile"
)
summarize_weights(result_pct)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv     min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl>   <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  6422       6422      0  1.00  1.36 0.00542 0.153 0.400  1.13  4.75 2255.

# absolute bounds -------------------------------------------------------
result_abs <- trim_weights(
  ns_wave1_svy, lower = 0.3, upper = 3.0, type = "absolute"
)
summarize_weights(result_abs)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv   min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  6422       6422      0  1.00 0.934   0.3   0.3 0.586  1.32  3.18 3429.

# survey_replicate — bounds auto-applied to all replicate columns -------
cps_svy <- surveycore::as_survey_replicate(
  cps_2023,
  weights    = "wtfinl",
  repweights = paste0("repwtp", 1:160),
  type       = "successive-difference",
  scale      = 4 / 160,
  rscales    = rep(1, 160)
)
result_rep <- trim_weights(
  cps_svy, lower = 0.05, upper = 0.95, type = "percentile"
)
summarize_weights(result_rep)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv   min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  9999       9999      0 1890. 0.594  270. 1008. 1709. 2679. 4301. 7392.
```
