---
layout: page
title: Reproducibility notes
---

## Code integrity

The notebooks are the analysis record. They contain the code as it was run, including stored outputs and the original hard-coded paths. Their SHA-256 checksums at the time of this documentation update are:

```text
0bab511dcb2660b3128b63c05aee6af978fa48471105367bc50b95ac02fdd679  preprocessing_Eyetracking.ipynb
89310845c00bde36423fc1c89e9c167ffd75b56d2e807ae6ef6cac1f2d67a92a  match_movie_segments_to_eye_tracking.ipynb
04ba88022101e0a97cb64a247cedf00aebe5a188124ae01c1144931a14e8ef64  synchrony_visualization.ipynb
```

Hard-coded paths have not been rewritten, so they must be edited in a working copy. The surrounding documentation identifies the principal analysis sections. The checksums will change whenever a notebook is edited or re-run; the commit history records the versions.

## Runtime information

Python 3.12.7 and R 4.5.2 were used for the GLMM analysis.

The notebooks use absolute paths from the original workstation. They will not run end-to-end on a new machine without recreating that directory structure or editing a working copy. They are also not "run all" notebooks: the participant, movie, and movie part have to be set by hand in the relevant cells (see the [repository README](https://github.com/olgagulka/eye-tracking-attention-#how-to-run-the-notebooks)).

## Study-specific settings

The main experiment (YN-2AFC) and the control study (2AFC only) share all preprocessing and synchrony code. The settings that differ are:

| Setting | Where | Main experiment | Control study |
| --- | --- | --- | --- |
| Participant list (`CURRENT_PARTICIPANTS`) | `match_movie_segments_to_eye_tracking.ipynb` | main-experiment participants | `sub_01` to `sub_32` |
| Excluded participant/movie parts (`EXCLUDED_PARTS`) | `match_movie_segments_to_eye_tracking.ipynb` (defined in three cells) | main-experiment exclusions | control-study exclusions, as saved in the notebook |
| Input/output directories | all notebooks | main-experiment directories | the `2AFC_STUDY` directories, as saved in the notebook |
| Merged trial-level input file | `synchrony_visualization.ipynb` | `..._30_subjects_final.csv`, as saved in the notebook | control-study merged file |
| Models | `synchrony_visualization.ipynb` | YN and 2AFC GLMMs, plus YN signal-present/absent models | 2AFC GLMM only |

The notebooks as saved are configured as described in the "Control study" column for the matching notebook and the "Main experiment" column for the analysis notebook. The main-experiment exclusion list is not part of this repository.

Excluded segments are applied to all participants of a movie: `sprout` `seg_26` and `prayer` `seg_22`. The witness movie has a numbering exception: `seg_34` is matched to the movie with `frame_35_witness.jpg`, which corresponds to `frame_34_witness.jpg` in the behavioral task. The matched files store both names (`target_image` and `behavioral_target_image`).

## Documentation/code points to confirm

These items are recorded openly rather than silently resolved:

1. The manuscript wording says the stacked gaze vector has "x values following y's." The implemented code (`np.concatenate([x, y])`) places all x values first and all y values second. This documentation follows the code.
2. The manuscript reports 183 time points per 3-second segment at 60 Hz. The correlation function (`pearson_corr_per_segment`) uses the common minimum number of rows of the two participants in a segment and, in the saved calls, applies no cap (`max_rows_per_segment=None`). The row counts used for each pair are stored in the `rows_used` column of the pairwise-correlation files. The exact sampling/row rule used for the reported results should be confirmed against these files.
3. Study 2's demographic summary contains an unresolved `X` placeholder in the supplied manuscript methods.
4. The manuscript's criterion (*C*) equation is not present in the pasted text, and its analysis script was not among the supplied files.
5. The exclusion lists for participant/movie parts are defined in three separate cells of `match_movie_segments_to_eye_tracking.ipynb`. They have to be kept identical when edited.

## Files still to be deposited

The following steps are used in the workflow, but their code or inputs are not yet in the repository:

- merging the two movie parts of each participant, adding the `movie_part` and `part_timestamp` columns, and gaze-quality control, which together produce the `eye_tracking_filtered_qc` files read by the matching notebook;
- the movie annotation files (`annotations_<movie>_movie.csv`) containing the target-frame timestamps;
- `script_sync_analysis/correlation_analysis_eyetracking`: participant-level mean-correlation quality checks across gaze and fixation measures;
- `prediciting_performance_analysis`: merging segment-level IGS/GS with behavioral performance and computing the leave-one-out group accuracy that forms the input of the analysis notebook;
- `RTs_for_YN_2AFC_tasks.ipynb`: response-time derivation and timing-drift checks;
- the code that creates the signal-present counterpart of the signal-absent trial file;
- the signal-detection criterion and repeated-measures ANOVA analysis, if stored separately; and
- any final session/package lock file used during analysis.

## Data availability

The per-participant IGS and GS values for each movie segment are provided in `results/IGS_GS_synchrony/`. Raw and processed gaze and behavioral data are described in the repository's `data/README.md`. The public repository and any archive should contain only data that are permitted under the consent procedure, ethics approval, and institutional policy. If participant-level data cannot be shared, a de-identified example or simulated file plus a data dictionary can document the required schema without exposing research records.
