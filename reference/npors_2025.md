# Pew NPORS 2025 probability sample with harmonized demographic columns

The 2025 Pew National Public Opinion Reference Survey (NPORS), obtained
from
[`surveycore::pew_npors_2025`](https://jdenn0514.github.io/surveycore/reference/pew_npors_2025.html).
All 5,022 respondents are retained. The original `gender` column is kept
as-is (numeric). Six derived columns are added: `sex` (factor from
`gender`), `age_f3`, `race_f4`, `edu_f3`, `pid_f3`, and `wt_pop`.

NPORS is a national address-based probability sample with stratified
random sampling and differential probabilities of selection across
strata. Always include `strata = stratum` when constructing a survey
design from this data. To construct a survey design for standard
estimation:

    data(npors_2025_clean)
    npors_design <- surveycore::as_survey(
      npors_2025_clean, weights = weight, strata = stratum
    )

For IPW use, construct a reference design using `wt_pop`:

    ref <- surveycore::as_survey(
      npors_2025_clean, weights = wt_pop, strata = stratum
    )
    ipw(ns_wave1, ref,
        selection = ~sex + age_f3 + race_f4 + edu_f3,
        missing_method = "omit")

Approximately 0.5% of rows have `NA` in each derived column (from `99` /
"Refused" codes). Use
[npors_2025_clean](https://jdenn0514.github.io/surveywts/reference/npors_2025_clean.md)
to avoid listwise-deletion warnings when passing to
[`ipw()`](https://jdenn0514.github.io/surveywts/reference/ipw.md).

## Usage

``` r
npors_2025
```

## Format

### `npors_2025`

A data frame with 5,022 rows and 71 columns. All 65 original columns
from
[`surveycore::pew_npors_2025`](https://jdenn0514.github.io/surveycore/reference/pew_npors_2025.html)
are retained, including `gender` as numeric. Six new derived columns are
added (`sex`, `age_f3`, `race_f4`, `edu_f3`, `pid_f3`, `wt_pop`):

- respid:

  Character. Respondent identifier.

- mode:

  Numeric. Interview mode.

- language:

  Numeric. Interview language.

- languageinitial:

  Numeric. Language at initial contact.

- stratum:

  Numeric. Sampling stratum.

- interview_start:

  Character. Interview start date/time.

- interview_end:

  Character. Interview end date/time.

- econ1mod:

  Numeric. Economic conditions assessment.

- econ1bmod:

  Numeric. Economic conditions (follow-up).

- comtype2:

  Numeric. Community type.

- unity:

  Numeric. National unity opinion.

- crimesafe:

  Numeric. Crime and safety opinion.

- govprotct:

  Numeric. Government protection opinion.

- moregunimpact:

  Numeric. Opinion on gun impact.

- fin_sit:

  Numeric. Financial situation.

- vet1:

  Numeric. Veteran status.

- vol12_cps:

  Numeric. Volunteered in last 12 months.

- eminuse:

  Numeric. Email use.

- intmob:

  Numeric. Internet mobile use.

- intfreq:

  Numeric. Internet frequency.

- intfreq_collapsed:

  Numeric. Internet frequency (collapsed).

- home4nw2:

  Numeric. Home internet access.

- bbhome:

  Numeric. Broadband at home.

- smuse_fb:

  Numeric. Uses Facebook.

- smuse_yt:

  Numeric. Uses YouTube.

- smuse_x:

  Numeric. Uses X (formerly Twitter).

- smuse_ig:

  Numeric. Uses Instagram.

- smuse_sc:

  Numeric. Uses Snapchat.

- smuse_wa:

  Numeric. Uses WhatsApp.

- smuse_tt:

  Numeric. Uses TikTok.

- smuse_rd:

  Numeric. Uses Reddit.

- smuse_bsk:

  Numeric. Uses Bluesky.

- smuse_th:

  Numeric. Uses Threads.

- smuse_ts:

  Numeric. Uses Tumblr.

- radio:

  Numeric. Listens to radio.

- device1a:

  Numeric. Device ownership.

- smart2:

  Numeric. Smartphone ownership.

- nhisll:

  Numeric. Health insurance.

- relig:

  Numeric. Religion.

- religcat1:

  Numeric. Religion category.

- born:

  Numeric. Born-again Christian.

- attendper:

  Numeric. Religious attendance (in-person).

- attendonline2:

  Numeric. Religious attendance (online).

- relimp:

  Numeric. Importance of religion.

- pray:

  Numeric. Prayer frequency.

- educcat:

  Numeric. Education category (raw): `1` = College+, `2` = HS/Some
  college, `3` = Less than HS, `99` = Refused.

- registration:

  Numeric. Voter registration status.

- party:

  Numeric. Political party identification.

- partyln:

  Numeric. Party leaning (for independents).

- partysum:

  Numeric. Party summary.

- hisp:

  Numeric. Hispanic origin: `1` = Yes, `2` = No.

- racecmb:

  Numeric. Race (combined).

- racethn:

  Numeric. Race/ethnicity (raw): `1` = White, `2` = Black, `3` =
  Hispanic, `4` = Other, `5` = Asian, `99` = Refused.

- agegrp:

  Numeric. Age group (raw): `1`-`13` = fine-grained age categories, `99`
  = Refused.

- agecat:

  Numeric. Age category (alternative grouping).

- birthplace:

  Numeric. Born in the United States.

- gender:

  Numeric. Interview gender: `1` = Male, `2` = Female, `3` = Non-binary,
  `99` = Refused.

- adults:

  Numeric. Number of adults in household.

- voted2024:

  Numeric. Voted in 2024.

- votegen_post:

  Numeric. 2024 vote choice.

- inc_sdt1:

  Numeric. Income.

- cregion:

  Numeric. Census region.

- metro:

  Numeric. Metropolitan area.

- basewt:

  Numeric. Base weight before calibration.

- weight:

  Numeric. Final normalized survey weight (mean approximately 1). Use
  for standard estimation. For IPW use `wt_pop` instead.

- sex:

  Factor. Derived from `gender`: `1` = `"Male"`, `2` = `"Female"`, `3`
  (Non-binary) and `99` (Refused) recoded to `NA`. Levels:
  `c("Male", "Female")`. Approximately 0.5% `NA`.

- age_f3:

  Factor. Derived from `agegrp`: `1:3` = `"18-34"`, `4:7` = `"35-54"`,
  `8:13` = `"55+"`, `99` = `NA`. Levels: `c("18-34", "35-54", "55+")`.
  Approximately 0.5% `NA`.

- race_f4:

  Factor. Derived from `racethn`: `1` = `"White"`, `2` = `"Black"`, `3`
  = `"Hispanic"`, `4`/`5` = `"Other"` (Asian collapsed), `99` = `NA`.
  Levels: `c("White", "Black", "Hispanic", "Other")`. Approximately 0.5%
  `NA`.

- edu_f3:

  Factor. Derived from `educcat`: `3` = `"Less than HS"`, `2` =
  `"HS/Some college"`, `1` = `"College+"`, `99` = `NA`. Levels:
  `c("Less than HS", "HS/Some college", "College+")`. Approximately 0.5%
  `NA`.

- pid_f3:

  Factor. Derived from `partysum`: `1` = `"Republican"`, `2` =
  `"Democrat"`, `9` = `"Independent"`, other = `NA`. Levels:
  `c("Republican", "Independent", "Democrat")`. Approximately 0.5% `NA`.

- wt_pop:

  Numeric. Population-scaled weight:
  `weight * (260000000 / nrow(npors_2025))`. Use for IPW reference
  design construction (sums to approximately 260 million, the 2024 US
  adult population estimate).

## Source

Derived from
[`surveycore::pew_npors_2025`](https://jdenn0514.github.io/surveycore/reference/pew_npors_2025.html).
See `data-raw/npors-2025.R` for the construction script.

## See also

[npors_2025_clean](https://jdenn0514.github.io/surveywts/reference/npors_2025_clean.md),
[ns_wave1](https://jdenn0514.github.io/surveywts/reference/ns_wave1.md)
