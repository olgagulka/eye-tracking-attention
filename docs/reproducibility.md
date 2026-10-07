---
layout: page
title: Reproducibility notes
---

## Code integrity

The uploaded notebooks were copied byte-for-byte. Their SHA-256 checksums are:

```text
0b1825cd79ab1d001663fa5325921092dd960c8428ee701cde0d775d835e1287  preprocessing_Eyetracking.ipynb
b4db9724af08f525250043212b6aa9b39588b5fd279c50eb475551ca450e16eb  synchrony_visualization.ipynb
```

This repository deliberately does not rewrite hard-coded paths, correct exploratory cells, remove stored output, or reorder the notebook. The surrounding documentation identifies the principal analysis sections while preserving the original record.

## Runtime information

 Python 3.12.7 and R 4.5.2 were used for the GLMM analysis. 

The notebooks use absolute paths from the original workstation. They will not run end-to-end on a new machine without recreating that directory structure or editing a working copy. 

## Documentation/code points to confirm

These items are recorded openly rather than silently resolved:

1. The manuscript wording says the stacked gaze vector has “x values following y's.” The implemented `combine_x_then_y` function concatenates all x values first and all y values second. This documentation follows the code.
2. The manuscript reports 183 time points per 3-second segment at 60 Hz. One demonstration cell in the supplied synchrony notebook uses `max_rows_per_segment = 180`, whereas the reusable batch function defaults to no cap and uses the common minimum row count. The exact sampling/row rule used for the reported results should be confirmed.
3. Study 2's demographic summary contains an unresolved `X` placeholder in the supplied manuscript methods.
4. The manuscript's criterion (*C*) equation is not present in the pasted text, and its analysis script was not among the supplied files.

## Files still to be deposited

The research notes name additional scripts or notebooks that are needed for a complete end-to-end archive:

- `script_sync_analysis/correlation_analysis_eyetracking`: participant-level mean-correlation quality checks across gaze and fixation measures;
- `per_trial_correlation`: matching target-frame windows to merged eye tracking and the full per-movie correlation/synchrony workflow (parts of the correlation workflow also appear in the supplied synchrony notebook);
- `prediciting_performance_analysis`: merging segment-level IGS/GS with behavioral performance;
- `RTs_for_YN_2AFC_tasks.ipynb`: response-time derivation and timing-drift checks;
- the signal-detection criterion and repeated-measures ANOVA analysis, if stored separately; and
- any final session/package lock file used during analysis.

## Data availability

No raw participant data or generated results were supplied for deposit. The public repository should contain only data that are permitted under the consent procedure, ethics approval, and institutional policy. If participant-level data cannot be shared, a de-identified example or simulated file plus a data dictionary can document the required schema without exposing research records.
