# Convert a replicate design back to a Taylor design

Reconstructs a `survey_taylor` from a `survey_replicate` created by
`create_*_weights()`. The original Taylor structure (PSU IDs, strata,
FPC, nest flag) is read from the `"replicate_creation"` entry in the
weighting history. The replicate weights (sets of perturbed weight
columns used to compute standard errors) are dropped from `@data`.

## Usage

``` r
as_taylor_design(data)
```

## Arguments

- data:

  A `survey_replicate` created by `create_*_weights()`, or a
  `survey_taylor` (returns unchanged with a warning).

## Value

A `survey_taylor`.

## Details

The reconstructed design uses Taylor linearization (closed-form variance
from a linear approximation, the alternative to replication) for
variance estimation, and its restored structure includes the finite
population correction (a factor that shrinks the variance when the
sample is a large share of the population), stored as FPC, when the
original design carried one.

Returns `data` unchanged (with a warning) if `data` is already a
`survey_taylor`.

## Warnings

Every successful conversion warns that the replicate weight columns —
and the variance capability they carry — are discarded. The warning
marks an intentional loss of information, not a failure. When `data` is
already a `survey_taylor`, the function instead warns that it returned
the input unchanged.

## See also

[`create_bootstrap_weights()`](https://jdenn0514.github.io/surveywts/reference/create_bootstrap_weights.md),
[`create_jackknife_weights()`](https://jdenn0514.github.io/surveywts/reference/create_jackknife_weights.md),
[`create_brr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_brr_weights.md),
[`create_gen_boot_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_boot_weights.md),
[`create_gen_rep_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_rep_weights.md),
[`create_sdr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_sdr_weights.md),
[`create_replicate_weights()`](https://jdenn0514.github.io/surveywts/reference/create_replicate_weights.md).
For the class system, the standard workflows, and a glossary of terms,
see the [Getting started
article](https://jdenn0514.github.io/surveywts/articles/getting-started.html).

Other replicate-weights:
[`create_bootstrap_weights()`](https://jdenn0514.github.io/surveywts/reference/create_bootstrap_weights.md),
[`create_brr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_brr_weights.md),
[`create_gen_boot_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_boot_weights.md),
[`create_gen_rep_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_rep_weights.md),
[`create_jackknife_weights()`](https://jdenn0514.github.io/surveywts/reference/create_jackknife_weights.md),
[`create_replicate_weights()`](https://jdenn0514.github.io/surveywts/reference/create_replicate_weights.md),
[`create_sdr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_sdr_weights.md)

## Examples

``` r
# convert a bootstrap replicate design back to Taylor ------------------
gss_design <- surveycore::as_survey(
  gss_2024, weights = wtssps, strata = vstrat, ids = vpsu, nest = TRUE
)
rep_design <- create_bootstrap_weights(
  gss_design,
  replicates = 50L,
  seed = 1L
)
class(rep_design)[1]
#> [1] "surveycore::survey_replicate"
result <- as_taylor_design(rep_design)
#> Warning: ! Converting to <survey_taylor> discards replicate weights and variance
#>   capability.
# Warning: Converting to <survey_taylor> discards replicate weights and
# variance capability.
class(result)[1]
#> [1] "surveycore::survey_taylor"
```
