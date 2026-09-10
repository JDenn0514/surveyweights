# Measure how unequal the survey weights are

The coefficient of variation (CV) measures how spread out the weights
are relative to their mean. A CV near zero indicates near-uniform
weights; higher values signal greater variability and a correspondingly
larger design effect (variance inflation relative to a simple random
sample of the same size). Rows with zero weights (typically produced by
[`adjust_nonresponse()`](https://jdenn0514.github.io/surveywts/reference/adjust_nonresponse.md))
are excluded before computing CV.

## Usage

``` r
weight_variability(x, weights = NULL)
```

## Arguments

- x:

  A `survey_taylor`, `survey_nonprob`, or `survey_replicate`. The weight
  column is auto-detected from `@variables$weights`.

- weights:

  Bare name (NSE). Weight column. Auto-detected from survey object
  `@variables$weights`.

## Value

A named numeric scalar: `c(cv = <value>)`. The name `"cv"` is part of
the API contract.

## Algorithm

`cv(w) = sd(w) / mean(w)`

## References

Kish, L. (1965). *Survey Sampling*. New York: John Wiley & Sons.

## See also

[`effective_sample_size()`](https://jdenn0514.github.io/surveywts/reference/effective_sample_size.md),
[`summarize_weights()`](https://jdenn0514.github.io/surveywts/reference/summarize_weights.md).
For the class system, the standard workflows, and a glossary of terms,
see the [Getting started
article](https://jdenn0514.github.io/surveywts/articles/getting-started.html).

Other diagnostics:
[`effective_sample_size()`](https://jdenn0514.github.io/surveywts/reference/effective_sample_size.md),
[`summarize_weights()`](https://jdenn0514.github.io/surveywts/reference/summarize_weights.md)

## Examples

``` r
ns_wave1_svy <- surveycore::as_survey_nonprob(ns_wave1, weights = weight)
weight_variability(ns_wave1_svy)
#>       cv 
#> 1.359692 
#>       cv
#> 1.359692
```
