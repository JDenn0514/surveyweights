# Pew 2016 ATP opt-in sample

A 2,000-respondent random sample drawn from a 2016 Pew Research Center
study fielded simultaneously on three opt-in (non-probability) online
vendor panels alongside the probability-based American Trends Panel
(ATP). The original study (31,863 respondents) is used in Mercer, Lau
and Kennedy (2018) "For Weighting Online Opt-In Samples, What Matters
Most?" and provides 13 benchmark variables (shared with
[pew_2016_synth_pop](https://jdenn0514.github.io/surveywts/reference/pew_2016_synth_pop.md))
that can be validated against population truth estimates.
`pew_2016_optin` contains a random sample of 2,000 complete-case
respondents drawn after removing rows with `NA` in any of the 7
calibration variables (seed 42).

Value labels are preserved as named numeric attributes on each column
(accessible via `attr(pew_2016_optin$gender, "labels")`). Variable
labels are stored in the `"label"` attribute of each column.

To construct a `survey_nonprob` for analysis, use the promoted `weight`
column directly from the tibble:

    data(pew_2016_optin)
    pew_design <- surveycore::as_survey_nonprob(
      pew_2016_optin,
      weights = "weight",
      repweights = paste0("repwt_", 1:200)
    )

## Usage

``` r
pew_2016_optin
```

## Format

A data frame with 2,000 rows and 305 columns:

- rid:

  Character. Respondent ID with vendor prefix (e.g., `"V1_7"`).

- vendor:

  Numeric. Opt-in panel vendor: `1` = Vendor 1, `2` = Vendor 2, `3` =
  Vendor 3.

- enddate:

  Date. Date respondent completed the survey.

- age:

  Numeric (continuous). Age in years (range 18-110; values above ~95 are
  likely data entry artefacts or sentinel codes).

- presapp:

  Numeric. Presidential approval (Obama): `1` = Approve, `2` =
  Disapprove, `9` = Refused.

- happy:

  Numeric. General happiness: `1` = Very happy, `2` = Pretty happy, `3`
  = Not too happy, `9` = Refused.

- folgov:

  Numeric. Follows government/public affairs (ordinal): `1` = Most of
  the time, `2` = Some of the time, `3` = Only now and then, `4` =
  Hardly at all, `5` = Refused.

- votegen:

  Numeric. 2016 presidential vote intention (first choice).

- votegen2:

  Numeric. 2016 presidential vote intention (follow-up).

- votegen3:

  Numeric. Forced-choice vote preference (Trump vs Clinton).

- talk_cps:

  Numeric. Frequency of talking with neighbors (ordinal): `1` =
  Basically every day, `2` = A few times a week, `3` = A few times a
  month, `4` = Rarely, `5` = Not at all, `6` = Refused.

- trust_cps:

  Numeric. Trust in neighbors (ordinal): `1` = All of the people, `2` =
  Most, `3` = Some, `4` = None, `5` = Refused.

- comgrp_cps:

  Integer. Participated in community group in last 12 months: `1` = Yes,
  `0` = No, `NA` = Refused.

- vol1:

  Numeric. Volunteered through an organization in last 12 months: `1` =
  Yes, `2` = No, `9` = Refused.

- vol2:

  Numeric. Volunteered informally or for children's organizations: `1` =
  Yes, `2` = No, `9` = Refused.

- acaapp:

  Numeric. ACA approval: `1` = Approve, `2` = Disapprove, `9` = Refused.

- mrjlegal:

  Numeric. Marijuana legalization opinion: `1` = Should be legal, `2` =
  Should not be legal, `9` = Refused.

- discrima:

  Numeric. Discrimination against Blacks: `1` = A lot, `2` = Not a lot,
  `9` = Refused.

- discrimb:

  Numeric. Discrimination against gays/lesbians: `1` = A lot, `2` = Not
  a lot, `9` = Refused.

- discrimc:

  Numeric. Discrimination against Hispanics: `1` = A lot, `2` = Not a
  lot, `9` = Refused.

- folnews:

  Numeric. Follows the news: `1` = Most of the time, `2` = Sometimes,
  `3` = Hardly ever, `9` = Refused.

- newsclosea:

  Numeric. How closely follows international news.

- newscloseb:

  Numeric. How closely follows national news.

- newsclosec:

  Numeric. How closely follows local news.

- pair1:

  Numeric. Opinion pair statement 1 (government scope).

- pair2:

  Numeric. Opinion pair statement 2 (business regulation).

- pair3:

  Numeric. Opinion pair statement 3 (racial discrimination).

- owngun_gss:

  Integer. Gun in home: `1` = Yes, `0` = No, `NA` = Refused.

- evsmk_nhis:

  Numeric. Smoked 100+ cigarettes in lifetime: `1` = Yes, `2` = No, `9`
  = Refused.

- nowsmk_nhis:

  Numeric. Current smoking: `1` = Every day, `2` = Some days, `3` = Not
  at all, `9` = Refused.

- racerel:

  Numeric. Race relations trend: `1` = Getting better, `2` = Getting
  worse, `3` = About the same, `9` = Refused.

- pub_off_cps:

  Integer. Contacted public official in last 12 months: `1` = Yes, `0` =
  No, `NA` = Refused.

- prtypref_gss:

  Numeric. Party preference (raw GSS question): `1` = Republican, `2` =
  Democrat, `3` = Independent, `4` = Other, `5` = No preference, `9` =
  Refused.

- prtystrg_gss:

  Numeric. Strength of party ID: `1` = Strong, `2` = Not very strong,
  `9` = Refused.

- prtyind_gss:

  Numeric. Independent leaning: `1` = Republican, `2` = Democrat, `3` =
  Neither, `9` = Refused.

- polviews_gss:

  Numeric. Political views, 7-point scale (1 = Extremely liberal, 7 =
  Extremely conservative).

- tablet_cps:

  Integer. Uses tablet/e-reader: `1` = Yes, `0` = No, `NA` = Refused.

- textim_cps:

  Integer. Uses texting/instant messaging: `1` = Yes, `0` = No, `NA` =
  Refused.

- social_cps:

  Integer. Uses social networking: `1` = Yes, `0` = No, `NA` = Refused.

- adults_hh:

  Numeric. Number of adults (18+) in household.

- children_hh:

  Numeric. Number of children under 18 in household.

- home_acs:

  Numeric. Home ownership/rental status (raw ACS question).

- tenure_acs:

  Numeric. Lived in home one year ago: `1` = Yes, `2` = No – different
  address in same city, `3` = No – different city.

- gender:

  Numeric. `1` = Male, `2` = Female, `3` = Refused.

- educ_acs:

  Numeric. Highest degree completed (raw ACS question).

- marital_acs:

  Numeric. Marital status (ACS question).

- mil_acs:

  Numeric. Active-duty military service history (raw ACS).

- hisp_acs:

  Numeric. Hispanic or Latino origin (ACS question).

- race_acs_1:

  Numeric. Race indicator: White (`1` = selected).

- race_acs_2:

  Numeric. Race indicator: Black or African American.

- race_acs_3:

  Numeric. Race indicator: Asian.

- race_acs_4:

  Numeric. Race indicator: American Indian or Alaska Native.

- race_acs_5:

  Numeric. Race indicator: Native Hawaiian or Pacific Islander.

- race_acs_6:

  Numeric. Race indicator: Some other race.

- born_acs:

  Numeric. Born in the United States (ACS question).

- citizen:

  Numeric. U.S. citizenship status (raw ACS question).

- insure_nhis:

  Numeric. Health insurance coverage (NHIS question).

- fdall_nhanes:

  Numeric. Has food allergies (NHANES question): `1` = Yes, `2` = No,
  `9` = Refused.

- relig:

  Numeric. Present religion (raw question).

- relig_else:

  Character. Open-ended religion response ("other").

- chr:

  Numeric. Self-identifies as Christian: `1` = Yes, `2` = No, `9` =
  Refused.

- born:

  Numeric. Born-again or evangelical Christian: `1` = Yes, `2` = No, `9`
  = Refused.

- attend:

  Numeric. Religious service attendance frequency.

- relimp:

  Numeric. Importance of religion in life.

- pray:

  Numeric. Prayer frequency outside of religious services.

- fdstmp_cps:

  Integer. Household received food stamps in 2015: `1` = Yes, `0` = No,
  `NA` = Refused.

- wrkstat_gss:

  Numeric. Work status last week (GSS question).

- registered:

  Integer. Registered to vote: `1` = Yes, `0` = No, `NA` = Refused.

- pvote12a:

  Numeric. 2012 presidential election: voted or not.

- pvote12b:

  Numeric. 2012 presidential vote: Obama, Romney, or other.

- vote14:

  Integer. Voted in 2014 midterms: `1` = Voted, `0` = Did not vote, `NA`
  = Refused.

- faminc_cps:

  Numeric. Family income in past 12 months (raw CPS categories).

- ideo3:

  Numeric. Ideology 3-category recode of `polviews_gss`: `1` = Liberal,
  `2` = Moderate, `3` = Conservative, `4` = Refused.

- faminc5:

  Numeric. Family income, 5-category recode of `faminc_cps`.

- employed:

  Numeric. Employment status, 3-category recode of `wrkstat_gss`.

- citizen_rec:

  Numeric. Citizenship recode of `citizen` (removes "Not Asked"
  category).

- mil_acs_rec:

  Numeric. Military status, 2-category recode of `mil_acs`.

- home_acs_rec:

  Numeric. Home ownership, 3-category recode of `home_acs`.

- hhsizecat:

  Numeric. Household size category recode.

- hhsize:

  Numeric. Household size (sum of `adults_hh` and `children_hh`).

- childrencat:

  Numeric. Children category, 2-category recode of `children_hh`.

- agecat6:

  Numeric. 6-category age recode: `1` = 18-24, `2` = 25-34, `3` = 35-44,
  `4` = 45-54, `5` = 55-64, `6` = 65+.

- religcat:

  Numeric. Religion, 6-category recode incorporating `born`.

- relig_rec:

  Numeric. Religion recode incorporating `chr`.

- racethn:

  Numeric. Race/ethnicity: `1` = White non-Hispanic, `2` = Black
  non-Hispanic, `3` = Hispanic, `4` = Asian, `5` = Other race, `6` =
  Refused.

- educcat3:

  Numeric. 3-category education: `1` = HS or less, `2` = Some college,
  `3` = College grad, `4` = Refused.

- educcat5:

  Numeric. 5-category education: `1` = Less than HS, `2` = HS Grad, `3`
  = Some college, `4` = College grad, `5` = Postgraduate, `6` = Refused.

- partysum:

  Numeric. Party ID, 3-category recode of `partyscale5`.

- partyscale3:

  Numeric. Party ID, 3-category recode of `prtypref_gss`.

- partyscale5:

  Numeric. Party ID: `1` = Republican, `2` = Lean Republican, `3` =
  Ind/No Lean, `4` = Lean Democrat, `5` = Democrat.

- partyscale7:

  Numeric. 7-point party scale (Strong R to Strong D).

- smoker:

  Numeric. Current smoker recode of `evsmk_nhis` and `nowsmk_nhis`: `1`
  = Current smoker, `0` = Non-smoker.

- volsum:

  Integer. Volunteered in past year: `1` = Volunteered, `0` = Did not
  volunteer, `NA` = Refused.

- votesum:

  Numeric. Vote intention recode of `votegen` and `votegen3`.

- votescale:

  Numeric. 7-category vote scale recode of `votegen`, `votegen2`, and
  `votegen3`.

- state:

  Character. State of residence.

- division:

  Numeric. Census division, **alphabetical coding** (not standard Census
  order): `1` = East North Central, `2` = East South Central, `3` =
  Middle Atlantic, `4` = Mountain, `5` = New England, `6` = Pacific, `7`
  = South Atlantic, `8` = West North Central, `9` = West South Central.
  Coding matches
  [pew_2016_synth_pop](https://jdenn0514.github.io/surveywts/reference/pew_2016_synth_pop.md).

- region:

  Numeric. Census region: `1` = Midwest, `2` = Northeast, `3` = South,
  `4` = West.

- language:

  Numeric. Interview language: `1` = English, `2` = Spanish.

- sex:

  Factor. Derived from `gender`: `1` = `"Male"`, `2` = `"Female"`, `3`
  (Refused) = `NA`. Levels: `c("Male", "Female")`.

- race_f4:

  Factor. Derived from `racethn`: `1` = `"White"`, `2` = `"Black"`, `3`
  = `"Hispanic"`, `4`/`5` = `"Other"` (Asian collapsed), `6` = `NA`.
  Levels: `c("White", "Black", "Hispanic", "Other")`.

- edu_f3:

  Factor. Derived from `educcat5`: `1` = `"Less than HS"`, `2:3` =
  `"HS/Some college"`, `4:5` = `"College+"`, `6` = `NA`. Levels:
  `c("Less than HS", "HS/Some college", "College+")`.

- pid_f3:

  Factor. Derived from `partyscale3` (GSS-style 3-category party): `1` =
  `"Republican"`, `2` = `"Democrat"`, `3` = `"Independent"`. Levels:
  `c("Republican", "Independent", "Democrat")`.

- age_f3:

  Factor. Derived from `agecat6`: `1:2` = `"18-34"`, `3:4` = `"35-54"`,
  `5:6` = `"55+"`. Levels: `c("18-34", "35-54", "55+")`.

- weight:

  Numeric. Calibrated survey weight produced by raking to unweighted
  proportions from
  [pew_2016_synth_pop](https://jdenn0514.github.io/surveywts/reference/pew_2016_synth_pop.md)
  (Newton-Raphson, 7 marginal dimensions: `sex`, `age_f3`, `race_f4`,
  `edu_f3`, `division`, `pid_f3`, `ideo3`; 5th/95th percentile trim).
  All values are positive. Use with `weights = weight` when constructing
  a survey object.

- repwt_1:

  Numeric. Quasi-randomization bootstrap replicate weight 1. Pass all
  200 `repwt_*` columns to
  [`surveycore::as_survey_nonprob()`](https://jdenn0514.github.io/surveycore/reference/as_survey_nonprob.html)
  as `repweights` for variance estimation.

- repwt_2:

  Numeric. Quasi-randomization bootstrap replicate weight 2.

- repwt_3:

  Numeric. Quasi-randomization bootstrap replicate weight 3.

- repwt_4:

  Numeric. Quasi-randomization bootstrap replicate weight 4.

- repwt_5:

  Numeric. Quasi-randomization bootstrap replicate weight 5.

- repwt_6:

  Numeric. Quasi-randomization bootstrap replicate weight 6.

- repwt_7:

  Numeric. Quasi-randomization bootstrap replicate weight 7.

- repwt_8:

  Numeric. Quasi-randomization bootstrap replicate weight 8.

- repwt_9:

  Numeric. Quasi-randomization bootstrap replicate weight 9.

- repwt_10:

  Numeric. Quasi-randomization bootstrap replicate weight 10.

- repwt_11:

  Numeric. Quasi-randomization bootstrap replicate weight 11.

- repwt_12:

  Numeric. Quasi-randomization bootstrap replicate weight 12.

- repwt_13:

  Numeric. Quasi-randomization bootstrap replicate weight 13.

- repwt_14:

  Numeric. Quasi-randomization bootstrap replicate weight 14.

- repwt_15:

  Numeric. Quasi-randomization bootstrap replicate weight 15.

- repwt_16:

  Numeric. Quasi-randomization bootstrap replicate weight 16.

- repwt_17:

  Numeric. Quasi-randomization bootstrap replicate weight 17.

- repwt_18:

  Numeric. Quasi-randomization bootstrap replicate weight 18.

- repwt_19:

  Numeric. Quasi-randomization bootstrap replicate weight 19.

- repwt_20:

  Numeric. Quasi-randomization bootstrap replicate weight 20.

- repwt_21:

  Numeric. Quasi-randomization bootstrap replicate weight 21.

- repwt_22:

  Numeric. Quasi-randomization bootstrap replicate weight 22.

- repwt_23:

  Numeric. Quasi-randomization bootstrap replicate weight 23.

- repwt_24:

  Numeric. Quasi-randomization bootstrap replicate weight 24.

- repwt_25:

  Numeric. Quasi-randomization bootstrap replicate weight 25.

- repwt_26:

  Numeric. Quasi-randomization bootstrap replicate weight 26.

- repwt_27:

  Numeric. Quasi-randomization bootstrap replicate weight 27.

- repwt_28:

  Numeric. Quasi-randomization bootstrap replicate weight 28.

- repwt_29:

  Numeric. Quasi-randomization bootstrap replicate weight 29.

- repwt_30:

  Numeric. Quasi-randomization bootstrap replicate weight 30.

- repwt_31:

  Numeric. Quasi-randomization bootstrap replicate weight 31.

- repwt_32:

  Numeric. Quasi-randomization bootstrap replicate weight 32.

- repwt_33:

  Numeric. Quasi-randomization bootstrap replicate weight 33.

- repwt_34:

  Numeric. Quasi-randomization bootstrap replicate weight 34.

- repwt_35:

  Numeric. Quasi-randomization bootstrap replicate weight 35.

- repwt_36:

  Numeric. Quasi-randomization bootstrap replicate weight 36.

- repwt_37:

  Numeric. Quasi-randomization bootstrap replicate weight 37.

- repwt_38:

  Numeric. Quasi-randomization bootstrap replicate weight 38.

- repwt_39:

  Numeric. Quasi-randomization bootstrap replicate weight 39.

- repwt_40:

  Numeric. Quasi-randomization bootstrap replicate weight 40.

- repwt_41:

  Numeric. Quasi-randomization bootstrap replicate weight 41.

- repwt_42:

  Numeric. Quasi-randomization bootstrap replicate weight 42.

- repwt_43:

  Numeric. Quasi-randomization bootstrap replicate weight 43.

- repwt_44:

  Numeric. Quasi-randomization bootstrap replicate weight 44.

- repwt_45:

  Numeric. Quasi-randomization bootstrap replicate weight 45.

- repwt_46:

  Numeric. Quasi-randomization bootstrap replicate weight 46.

- repwt_47:

  Numeric. Quasi-randomization bootstrap replicate weight 47.

- repwt_48:

  Numeric. Quasi-randomization bootstrap replicate weight 48.

- repwt_49:

  Numeric. Quasi-randomization bootstrap replicate weight 49.

- repwt_50:

  Numeric. Quasi-randomization bootstrap replicate weight 50.

- repwt_51:

  Numeric. Quasi-randomization bootstrap replicate weight 51.

- repwt_52:

  Numeric. Quasi-randomization bootstrap replicate weight 52.

- repwt_53:

  Numeric. Quasi-randomization bootstrap replicate weight 53.

- repwt_54:

  Numeric. Quasi-randomization bootstrap replicate weight 54.

- repwt_55:

  Numeric. Quasi-randomization bootstrap replicate weight 55.

- repwt_56:

  Numeric. Quasi-randomization bootstrap replicate weight 56.

- repwt_57:

  Numeric. Quasi-randomization bootstrap replicate weight 57.

- repwt_58:

  Numeric. Quasi-randomization bootstrap replicate weight 58.

- repwt_59:

  Numeric. Quasi-randomization bootstrap replicate weight 59.

- repwt_60:

  Numeric. Quasi-randomization bootstrap replicate weight 60.

- repwt_61:

  Numeric. Quasi-randomization bootstrap replicate weight 61.

- repwt_62:

  Numeric. Quasi-randomization bootstrap replicate weight 62.

- repwt_63:

  Numeric. Quasi-randomization bootstrap replicate weight 63.

- repwt_64:

  Numeric. Quasi-randomization bootstrap replicate weight 64.

- repwt_65:

  Numeric. Quasi-randomization bootstrap replicate weight 65.

- repwt_66:

  Numeric. Quasi-randomization bootstrap replicate weight 66.

- repwt_67:

  Numeric. Quasi-randomization bootstrap replicate weight 67.

- repwt_68:

  Numeric. Quasi-randomization bootstrap replicate weight 68.

- repwt_69:

  Numeric. Quasi-randomization bootstrap replicate weight 69.

- repwt_70:

  Numeric. Quasi-randomization bootstrap replicate weight 70.

- repwt_71:

  Numeric. Quasi-randomization bootstrap replicate weight 71.

- repwt_72:

  Numeric. Quasi-randomization bootstrap replicate weight 72.

- repwt_73:

  Numeric. Quasi-randomization bootstrap replicate weight 73.

- repwt_74:

  Numeric. Quasi-randomization bootstrap replicate weight 74.

- repwt_75:

  Numeric. Quasi-randomization bootstrap replicate weight 75.

- repwt_76:

  Numeric. Quasi-randomization bootstrap replicate weight 76.

- repwt_77:

  Numeric. Quasi-randomization bootstrap replicate weight 77.

- repwt_78:

  Numeric. Quasi-randomization bootstrap replicate weight 78.

- repwt_79:

  Numeric. Quasi-randomization bootstrap replicate weight 79.

- repwt_80:

  Numeric. Quasi-randomization bootstrap replicate weight 80.

- repwt_81:

  Numeric. Quasi-randomization bootstrap replicate weight 81.

- repwt_82:

  Numeric. Quasi-randomization bootstrap replicate weight 82.

- repwt_83:

  Numeric. Quasi-randomization bootstrap replicate weight 83.

- repwt_84:

  Numeric. Quasi-randomization bootstrap replicate weight 84.

- repwt_85:

  Numeric. Quasi-randomization bootstrap replicate weight 85.

- repwt_86:

  Numeric. Quasi-randomization bootstrap replicate weight 86.

- repwt_87:

  Numeric. Quasi-randomization bootstrap replicate weight 87.

- repwt_88:

  Numeric. Quasi-randomization bootstrap replicate weight 88.

- repwt_89:

  Numeric. Quasi-randomization bootstrap replicate weight 89.

- repwt_90:

  Numeric. Quasi-randomization bootstrap replicate weight 90.

- repwt_91:

  Numeric. Quasi-randomization bootstrap replicate weight 91.

- repwt_92:

  Numeric. Quasi-randomization bootstrap replicate weight 92.

- repwt_93:

  Numeric. Quasi-randomization bootstrap replicate weight 93.

- repwt_94:

  Numeric. Quasi-randomization bootstrap replicate weight 94.

- repwt_95:

  Numeric. Quasi-randomization bootstrap replicate weight 95.

- repwt_96:

  Numeric. Quasi-randomization bootstrap replicate weight 96.

- repwt_97:

  Numeric. Quasi-randomization bootstrap replicate weight 97.

- repwt_98:

  Numeric. Quasi-randomization bootstrap replicate weight 98.

- repwt_99:

  Numeric. Quasi-randomization bootstrap replicate weight 99.

- repwt_100:

  Numeric. Quasi-randomization bootstrap replicate weight 100.

- repwt_101:

  Numeric. Quasi-randomization bootstrap replicate weight 101.

- repwt_102:

  Numeric. Quasi-randomization bootstrap replicate weight 102.

- repwt_103:

  Numeric. Quasi-randomization bootstrap replicate weight 103.

- repwt_104:

  Numeric. Quasi-randomization bootstrap replicate weight 104.

- repwt_105:

  Numeric. Quasi-randomization bootstrap replicate weight 105.

- repwt_106:

  Numeric. Quasi-randomization bootstrap replicate weight 106.

- repwt_107:

  Numeric. Quasi-randomization bootstrap replicate weight 107.

- repwt_108:

  Numeric. Quasi-randomization bootstrap replicate weight 108.

- repwt_109:

  Numeric. Quasi-randomization bootstrap replicate weight 109.

- repwt_110:

  Numeric. Quasi-randomization bootstrap replicate weight 110.

- repwt_111:

  Numeric. Quasi-randomization bootstrap replicate weight 111.

- repwt_112:

  Numeric. Quasi-randomization bootstrap replicate weight 112.

- repwt_113:

  Numeric. Quasi-randomization bootstrap replicate weight 113.

- repwt_114:

  Numeric. Quasi-randomization bootstrap replicate weight 114.

- repwt_115:

  Numeric. Quasi-randomization bootstrap replicate weight 115.

- repwt_116:

  Numeric. Quasi-randomization bootstrap replicate weight 116.

- repwt_117:

  Numeric. Quasi-randomization bootstrap replicate weight 117.

- repwt_118:

  Numeric. Quasi-randomization bootstrap replicate weight 118.

- repwt_119:

  Numeric. Quasi-randomization bootstrap replicate weight 119.

- repwt_120:

  Numeric. Quasi-randomization bootstrap replicate weight 120.

- repwt_121:

  Numeric. Quasi-randomization bootstrap replicate weight 121.

- repwt_122:

  Numeric. Quasi-randomization bootstrap replicate weight 122.

- repwt_123:

  Numeric. Quasi-randomization bootstrap replicate weight 123.

- repwt_124:

  Numeric. Quasi-randomization bootstrap replicate weight 124.

- repwt_125:

  Numeric. Quasi-randomization bootstrap replicate weight 125.

- repwt_126:

  Numeric. Quasi-randomization bootstrap replicate weight 126.

- repwt_127:

  Numeric. Quasi-randomization bootstrap replicate weight 127.

- repwt_128:

  Numeric. Quasi-randomization bootstrap replicate weight 128.

- repwt_129:

  Numeric. Quasi-randomization bootstrap replicate weight 129.

- repwt_130:

  Numeric. Quasi-randomization bootstrap replicate weight 130.

- repwt_131:

  Numeric. Quasi-randomization bootstrap replicate weight 131.

- repwt_132:

  Numeric. Quasi-randomization bootstrap replicate weight 132.

- repwt_133:

  Numeric. Quasi-randomization bootstrap replicate weight 133.

- repwt_134:

  Numeric. Quasi-randomization bootstrap replicate weight 134.

- repwt_135:

  Numeric. Quasi-randomization bootstrap replicate weight 135.

- repwt_136:

  Numeric. Quasi-randomization bootstrap replicate weight 136.

- repwt_137:

  Numeric. Quasi-randomization bootstrap replicate weight 137.

- repwt_138:

  Numeric. Quasi-randomization bootstrap replicate weight 138.

- repwt_139:

  Numeric. Quasi-randomization bootstrap replicate weight 139.

- repwt_140:

  Numeric. Quasi-randomization bootstrap replicate weight 140.

- repwt_141:

  Numeric. Quasi-randomization bootstrap replicate weight 141.

- repwt_142:

  Numeric. Quasi-randomization bootstrap replicate weight 142.

- repwt_143:

  Numeric. Quasi-randomization bootstrap replicate weight 143.

- repwt_144:

  Numeric. Quasi-randomization bootstrap replicate weight 144.

- repwt_145:

  Numeric. Quasi-randomization bootstrap replicate weight 145.

- repwt_146:

  Numeric. Quasi-randomization bootstrap replicate weight 146.

- repwt_147:

  Numeric. Quasi-randomization bootstrap replicate weight 147.

- repwt_148:

  Numeric. Quasi-randomization bootstrap replicate weight 148.

- repwt_149:

  Numeric. Quasi-randomization bootstrap replicate weight 149.

- repwt_150:

  Numeric. Quasi-randomization bootstrap replicate weight 150.

- repwt_151:

  Numeric. Quasi-randomization bootstrap replicate weight 151.

- repwt_152:

  Numeric. Quasi-randomization bootstrap replicate weight 152.

- repwt_153:

  Numeric. Quasi-randomization bootstrap replicate weight 153.

- repwt_154:

  Numeric. Quasi-randomization bootstrap replicate weight 154.

- repwt_155:

  Numeric. Quasi-randomization bootstrap replicate weight 155.

- repwt_156:

  Numeric. Quasi-randomization bootstrap replicate weight 156.

- repwt_157:

  Numeric. Quasi-randomization bootstrap replicate weight 157.

- repwt_158:

  Numeric. Quasi-randomization bootstrap replicate weight 158.

- repwt_159:

  Numeric. Quasi-randomization bootstrap replicate weight 159.

- repwt_160:

  Numeric. Quasi-randomization bootstrap replicate weight 160.

- repwt_161:

  Numeric. Quasi-randomization bootstrap replicate weight 161.

- repwt_162:

  Numeric. Quasi-randomization bootstrap replicate weight 162.

- repwt_163:

  Numeric. Quasi-randomization bootstrap replicate weight 163.

- repwt_164:

  Numeric. Quasi-randomization bootstrap replicate weight 164.

- repwt_165:

  Numeric. Quasi-randomization bootstrap replicate weight 165.

- repwt_166:

  Numeric. Quasi-randomization bootstrap replicate weight 166.

- repwt_167:

  Numeric. Quasi-randomization bootstrap replicate weight 167.

- repwt_168:

  Numeric. Quasi-randomization bootstrap replicate weight 168.

- repwt_169:

  Numeric. Quasi-randomization bootstrap replicate weight 169.

- repwt_170:

  Numeric. Quasi-randomization bootstrap replicate weight 170.

- repwt_171:

  Numeric. Quasi-randomization bootstrap replicate weight 171.

- repwt_172:

  Numeric. Quasi-randomization bootstrap replicate weight 172.

- repwt_173:

  Numeric. Quasi-randomization bootstrap replicate weight 173.

- repwt_174:

  Numeric. Quasi-randomization bootstrap replicate weight 174.

- repwt_175:

  Numeric. Quasi-randomization bootstrap replicate weight 175.

- repwt_176:

  Numeric. Quasi-randomization bootstrap replicate weight 176.

- repwt_177:

  Numeric. Quasi-randomization bootstrap replicate weight 177.

- repwt_178:

  Numeric. Quasi-randomization bootstrap replicate weight 178.

- repwt_179:

  Numeric. Quasi-randomization bootstrap replicate weight 179.

- repwt_180:

  Numeric. Quasi-randomization bootstrap replicate weight 180.

- repwt_181:

  Numeric. Quasi-randomization bootstrap replicate weight 181.

- repwt_182:

  Numeric. Quasi-randomization bootstrap replicate weight 182.

- repwt_183:

  Numeric. Quasi-randomization bootstrap replicate weight 183.

- repwt_184:

  Numeric. Quasi-randomization bootstrap replicate weight 184.

- repwt_185:

  Numeric. Quasi-randomization bootstrap replicate weight 185.

- repwt_186:

  Numeric. Quasi-randomization bootstrap replicate weight 186.

- repwt_187:

  Numeric. Quasi-randomization bootstrap replicate weight 187.

- repwt_188:

  Numeric. Quasi-randomization bootstrap replicate weight 188.

- repwt_189:

  Numeric. Quasi-randomization bootstrap replicate weight 189.

- repwt_190:

  Numeric. Quasi-randomization bootstrap replicate weight 190.

- repwt_191:

  Numeric. Quasi-randomization bootstrap replicate weight 191.

- repwt_192:

  Numeric. Quasi-randomization bootstrap replicate weight 192.

- repwt_193:

  Numeric. Quasi-randomization bootstrap replicate weight 193.

- repwt_194:

  Numeric. Quasi-randomization bootstrap replicate weight 194.

- repwt_195:

  Numeric. Quasi-randomization bootstrap replicate weight 195.

- repwt_196:

  Numeric. Quasi-randomization bootstrap replicate weight 196.

- repwt_197:

  Numeric. Quasi-randomization bootstrap replicate weight 197.

- repwt_198:

  Numeric. Quasi-randomization bootstrap replicate weight 198.

- repwt_199:

  Numeric. Quasi-randomization bootstrap replicate weight 199.

- repwt_200:

  Numeric. Quasi-randomization bootstrap replicate weight 200.

## Source

Derived from the Pew Research Center 2016 opt-in sample SPSS file. See
`data-raw/pew-2016-optin.R` for the preparation script.

## Details

Pew 2016 ATP opt-in sample

## See also

[pew_2016_synth_pop](https://jdenn0514.github.io/surveywts/reference/pew_2016_synth_pop.md)
