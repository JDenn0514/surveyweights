# test(replicate): bind the SDR scale to 4 / R

**Date**: 2026-09-08
**Branch**: JDenn0514/no-test-binds-variables-scale-to-4-r-for-success
**Issue**: #126

## Changes

- Add one `test_that()` block that holds `create_sdr_weights()` to the SDR
  variance scale factor. PR #124 corrected the number in the help page, from
  `1 / (2R)` to `4 / R`, but no test tied the printed formula to the object
  the function returns
- The block asserts the scale two ways: a fixed pin, `4 / 32` on a 20-PSU
  design at `replicates = 20L`, and the relation
  `scale == 4 / length(repweights)` over both settings of
  `use_normal_hadamard` and three more replicate counts
- `R` is the full column count, inactive replicates included. An inactive
  column adds a zero term to the variance sum, so the scale counts it
- No production code changed

## Verification

- New block: 13 expectations, 0 failed
- `tests/testthat/test-replicate-weights.R`: 627 passed, 0 failed, 66 skipped
- `air format`: no change
- `devtools::check()`: 0 errors, 0 warnings

## Files Modified

- `tests/testthat/test-replicate-weights.R` — the new scale block, next to the
  column-count and inactive-count blocks that read the same contract
- `NEWS.md` — entry under `## Internal`
