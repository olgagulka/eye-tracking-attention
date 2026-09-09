---
layout: page
title: Synchrony and GLMM analysis
---

The supplied [`synchrony_visualization.ipynb`](https://github.com/olgagulka/eye-tracking-attention-/blob/main/notebooks/analysis/synchrony_visualization.ipynb) contains segment-level Pearson-correlation functions, the current synchrony calculation, GLMMs, average marginal effects, and visualization code.

## Segment-level gaze data

The intended input is one merged eye-tracking file per participant and movie, with rows already labeled by `segment_id`. According to the manuscript, segments span 3 seconds around each target frame: 1.5 seconds before through 1.5 seconds after target-frame onset. The notebook provided here consumes these matched files; the separate function used to create them has not yet been deposited.

## Pairwise Pearson correlations

Correlations are calculated separately for each movie. Only participants viewing the same movie and samples bearing the same segment identifier are compared.

For every participant pair and shared segment, the batch function:

1. selects the segment rows for both participants;
2. restricts both series to their common minimum length;
3. concatenates all x-coordinate samples followed by all y-coordinate samples for each participant;
4. calculates Pearson's correlation coefficient and p-value between the two stacked vectors; and
5. saves one row per segment, including participant identifiers, rows used, *r*, and *p*.

The batch function optionally excludes named segments and optionally caps the number of rows per segment. It writes a separate CSV for every participant pair within a movie.

## Synchrony measures

The current function is labeled `compute_group_and_pilot_sync` in the notebook and derives two values for each participant and segment:

- **Individual-to-group synchrony (IGS; `pilot_mean`)**: the mean of all pairwise correlations involving the participant of interest.
- **Group synchrony (GS; `group_mean`)**: the mean of correlations among all pairs that do not include the participant of interest.

GS therefore uses a leave-one-participant-out construction. It describes the scene-driven coherence of the remaining group's gaze without including the evaluated participant's gaze.

## Merging synchrony with behavior

The research workflow next merges IGS and GS, already aligned with movie segments, with trial-level behavioral accuracy and response-time data. The behavioral files must first contain the calculated YN and 2AFC response times. The separate notebook described as `prediciting_performance_analysis` was not supplied; the analysis notebook included here instead begins its modeling sections from previously merged CSV files.

## GLMM specification

The main models use `lme4::glmer` through the `rpy2` Jupyter interface with a binomial distribution and logit link. Trial correctness is the binary outcome, participant identity is a random intercept, and leave-one-out mean accuracy for the relevant task/trial is included as a trial-difficulty covariate.

The model formulas implemented for the decomposed predictors are:

```text
2AFC accuracy ~ Z1 + Z2 + leave-one-out 2AFC accuracy + (1 | participant)
YN accuracy   ~ Z1 + Z2 + leave-one-out YN accuracy   + (1 | participant)
```

Because IGS and GS were strongly collinear in the manuscript analysis (reported variance inflation factors of approximately 8), both were first z-scored and transformed as:

```text
Z1 = (z(IGS) + z(GS)) / 2
Z2 = (z(IGS) − z(GS)) / 2
```

`Z1` captures their shared component, and `Z2` captures the individual's deviation from the group component. The notebook uses the `bobyqa` optimizer for these models. It also retains non-decomposed model specifications for comparison.

Average marginal effects are calculated on the response-probability scale with `marginaleffects::avg_slopes`. The notebook stores estimates, confidence intervals, standard errors, and p-values and contains code for coefficient and predicted-probability plots.

## Signal-present and signal-absent YN analyses

YN trials are additionally modeled separately for:

- signal-present trials: hits and misses; and
- signal-absent trials: correct rejections and false alarms.

The notebook constructs condition-specific leave-one-out accuracy covariates, fits decomposed and non-decomposed GLMMs within each condition, and contains an interaction model testing whether the IGS association differs between signal-present and signal-absent trials. This separation is intended to distinguish a possible sensitivity benefit from a shift in response criterion.

The manuscript also describes a signal-detection criterion (*C*) analysis and a 2 × 2 repeated-measures ANOVA across low/high IGS and GS categories. No corresponding criterion/ANOVA script was included among the supplied files.
