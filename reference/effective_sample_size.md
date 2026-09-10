# Estimate Kish's effective sample size of weighted data

The effective sample size (ESS) measures how much statistical precision
the weighted sample retains relative to an equal-sized simple random
sample. Higher weight variability reduces the ESS, resulting in higher
variance for weighted estimates. Rows with zero weights (typically
produced by
[`adjust_nonresponse()`](https://jdenn0514.github.io/surveywts/reference/adjust_nonresponse.md))
are excluded before computing ESS.

## Usage

``` r
effective_sample_size(x, weights = NULL)
```

## Arguments

- x:

  A `survey_taylor`, `survey_nonprob`, or `survey_replicate`. The weight
  column is auto-detected from `@variables$weights`.

- weights:

  Bare name (NSE). Weight column. Auto-detected from survey object
  `@variables$weights`.

## Value

A named numeric scalar: `c(n_eff = <value>)`. The name `"n_eff"` is part
of the API contract.

## Algorithm

\$\$ESS = \frac{(\sum w)^2}{\sum w^2}\$\$

## References

Kish, L. (1965). *Survey Sampling*. New York: John Wiley & Sons.

## See also

[`weight_variability()`](https://jdenn0514.github.io/surveywts/reference/weight_variability.md),
[`summarize_weights()`](https://jdenn0514.github.io/surveywts/reference/summarize_weights.md).
For the class system, the standard workflows, and a glossary of terms,
see the [Getting started
article](https://jdenn0514.github.io/surveywts/articles/getting-started.html).

Other diagnostics:
[`summarize_weights()`](https://jdenn0514.github.io/surveywts/reference/summarize_weights.md),
[`weight_variability()`](https://jdenn0514.github.io/surveywts/reference/weight_variability.md)

## Examples

``` r
ns_wave1_svy <- surveycore::as_survey_nonprob(ns_wave1, weights = weight)
effective_sample_size(ns_wave1_svy)
#>    n_eff 
#> 2254.539 
#>    n_eff
#> 2254.539
```
