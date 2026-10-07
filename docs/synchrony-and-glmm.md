---
layout: page
title: Synchrony and GLMM analysis
---

The segment-matching, correlation, and synchrony code is in [`match_movie_segments_to_eye_tracking.ipynb`](https://github.com/olgagulka/eye-tracking-attention-/blob/main/notebooks/preprocessing/match_movie_segments_to_eye_tracking.ipynb). The models, average marginal effects, and figures are in [`synchrony_visualization.ipynb`](https://github.com/olgagulka/eye-tracking-attention-/blob/main/notebooks/analysis/synchrony_visualization.ipynb).

The same synchrony procedure is used for the main experiment (YN-2AFC) and the control study (2AFC only). The two studies differ only in the models fitted at the end (see [GLMM specification](#glmm-specification)).

## Segment-level gaze data

The inputs are the preprocessed gaze files (see [Eye-tracking preprocessing](eye-tracking-preprocessing.md)) and one annotation file per movie (`annotations_<movie>_movie.csv`) with the timestamp (`match_timestamp_sec`) at which each target frame occurs in the movie.

The function `match_movie_segments_to_eye_tracking(movie, participant_id, output_file, movie_part=None)` labels every gaze sample that lies within 1.5 seconds before or after a target-frame timestamp, giving 3-second segments. Each labelled row receives a `segment_id` (`seg_1`, `seg_2`, ..., numbered across the whole movie, so that part 2 continues the numbering of part 1), the `target_image` used to find the movie timestamp, and the `behavioral_target_image` used for the behavioral merge.

- Matching uses `part_timestamp` (time within each movie part), because the annotation timestamps restart at zero in each part, whereas the merged `timestamp` includes the duration of part 1 for part 2.
- With `movie_part=None` the merged file of both parts is used. With `"01"` or `"02"`, a single part file is used, which is how participants with only one usable part are processed.
- If two segment windows overlap, the later segment overwrites the earlier label, and a warning is printed.
- Participant/movie parts listed in `EXCLUDED_PARTS` and movie segments listed in `EXCLUDED_SEGMENTS` are not labelled. The exclusion lists differ between the main experiment and the control study.

The participant and movie are set by hand for single runs. A batch cell loops over the participants for one movie and has to be re-run for each of the four movies.

## Pairwise Pearson correlations

Correlations are calculated separately for each movie with `pearson_corr_per_segment(movie_name, participants)`. Only participants viewing the same movie and samples bearing the same segment identifier are compared.

For every participant pair and shared segment, the function:

1. selects the segment rows for both participants, sorted by `part_timestamp`;
2. restricts both series to their common minimum length (optionally capped by `max_rows_per_segment`; the saved calls apply no cap);
3. concatenates all x-coordinate samples followed by all y-coordinate samples for each participant;
4. calculates Pearson's correlation coefficient and p-value between the two stacked vectors; and
5. saves one row per segment, including the movie, segment, participant identifiers, `rows_used`, *r*, and *p*.

Excluded participant/movie parts and rejected segments (`DEFAULT_REJECT_SEGMENTS`) are removed before the correlations are computed. The function writes a separate CSV for every participant pair within a movie. `plot_average_segment_correlations(movie_name)` summarizes these files per segment (Fisher-*z*-averaged *r*, median, and interquartile range across pairs), saves a summary CSV, and plots it as a quality check.

## Synchrony measures

The function `compute_group_and_pilot_sync(movie, participants, reject_segments)` derives two values for each participant and segment from the pairwise files of one movie:

- **Individual-to-group synchrony (IGS; `pilot_mean`)**: the mean of all pairwise correlations involving the participant of interest.
- **Group synchrony (GS; `group_mean`)**: the mean of correlations among all pairs that do not include the participant of interest.

Both are means of the raw Pearson *r* values. GS therefore uses a leave-one-participant-out construction. It describes the scene-driven coherence of the remaining group's gaze without including the evaluated participant's gaze.

The function saves `participants_sync_metrics_<movie>_final_all_subjects.csv` with the columns `segment_num`, `participant`, `pilot_mean`, and `group_mean`. These files are provided in `results/IGS_GS_synchrony/<movie>/`. `reject_segments` takes segment numbers; segments already rejected in the correlation step are absent from the pairwise files.

## Merging synchrony with behavior

IGS and GS, aligned with movie segments, are merged with trial-level behavioral accuracy and response-time data, and the leave-one-out group accuracy is added for each trial. The behavioral files must first contain the calculated YN and 2AFC response times. The code of this merging step has not yet been deposited: the analysis notebook starts its modeling sections from the previously merged CSV file.

The merged file contains one row per participant and trial, with at least `participant_id`, `movie`, `segment_num`, `pilot_mean` (IGS), `group_mean` (GS), `correct_response` and `mean_correct_2AFC_LOO` (2AFC accuracy and leave-one-out group accuracy) and, for the main experiment, `correct_response_occlusion`, `response_occlusion`, and `mean_correct_YN_LOO` (YN accuracy, response, and leave-one-out group accuracy).

## GLMM specification

The models use `lme4::glmer` through the `rpy2` Jupyter interface (R with the `lme4` and `marginaleffects` packages) with a binomial distribution and logit link. Trial correctness is the binary outcome, participant identity is a random intercept, and leave-one-out mean accuracy for the relevant task is included as a trial-difficulty covariate. The `bobyqa` optimizer is used.

The model formulas implemented for the decomposed predictors are:

```text
2AFC accuracy ~ Z1 + Z2 + leave-one-out 2AFC accuracy + (1 | participant)    main experiment and control study
YN accuracy   ~ Z1 + Z2 + leave-one-out YN accuracy   + (1 | participant)    main experiment only
```

Because IGS and GS were strongly collinear in the manuscript analysis (reported variance inflation factors of approximately 8), both were first z-scored and transformed as:

```text
Z1 = (z(IGS) + z(GS)) / 2
Z2 = (z(IGS) − z(GS)) / 2
```

`Z1` captures their shared component, and `Z2` captures the individual's deviation from the group component.

Average marginal effects are calculated on the response-probability scale with `marginaleffects::avg_slopes`. The notebook saves estimates, confidence intervals, standard errors, and p-values as CSV files and plots them as forest plots (change in predicted % correct with 95% confidence intervals).

### Main experiment and control study

| | Main experiment (YN-2AFC) | Control study (2AFC only) |
| --- | --- | --- |
| 2AFC GLMM and AME | yes | yes |
| YN GLMM and AME | yes | no (no YN task) |
| YN vs. 2AFC forest plot | yes | no |
| YN signal-present/absent models | yes | no |

For the control study, the merged CSV file path in the notebook has to point to the control-study data, and only the 2AFC cells are run.

## Signal-present and signal-absent YN analyses (main experiment only)

YN trials are additionally modeled separately for:

- signal-present trials: hits and misses; and
- signal-absent trials: correct rejections and false alarms.

The notebook includes the code that identifies signal-absent trials and computes their condition-specific leave-one-out accuracy (on a 0 to 100 scale); the code that creates the signal-present file is not included. In each condition, a GLMM with the z-scored IGS (`pilot_mean_z`), the z-scored GS (`group_mean_z`), the condition-specific leave-one-out accuracy, and a random participant intercept is fitted, and the average marginal effects are plotted side by side. These models use IGS and GS as separate predictors rather than the Z1/Z2 decomposition. This separation is intended to distinguish a possible sensitivity benefit from a shift in response criterion.

The manuscript also describes a signal-detection criterion (*C*) analysis and a 2 × 2 repeated-measures ANOVA across low/high IGS and GS categories. These analyses are not part of the deposited notebooks.
