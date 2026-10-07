---
layout: home
title: Home
---

This research repository documents two studies examining whether similarity in participants' eye movements while viewing narrative movies predicts later recognition memory performance:

- the **main experiment** (Study 1), with a sequential Yes/No (YN) judgment followed by a two-alternative forced-choice (2AFC) judgment on every trial; and
- the **control study** (Study 2), with the 2AFC judgment only.

Eye-tracking preprocessing and the synchrony calculation are identical in the two studies. They differ only in the final statistical analysis (YN and 2AFC accuracy in the main experiment; 2AFC accuracy only in the control study).

## What is documented

- [Study design](study-design.md): participants, apparatus, stimuli, and the distinction between the main experiment (Study 1) and the control study (Study 2).
- [Eye-tracking preprocessing](eye-tracking-preprocessing.md): movie alignment, trimming, invalid-sample handling, interpolation, pixel conversion, and merging.
- [Synchrony and GLMM analysis](synchrony-and-glmm.md): segment matching, pairwise correlations, IGS/GS construction, and trial-level mixed-effects models.
- [Reproducibility notes](reproducibility.md): software, code provenance, study-specific settings, known documentation/code differences, and files still to be deposited.

## Notebooks (run in this order)

1. [Eye-tracking preprocessing notebook](https://github.com/olgagulka/eye-tracking-attention-/blob/main/notebooks/preprocessing/preprocessing_Eyetracking.ipynb): gaze alignment, cleaning, interpolation, and pixel conversion for one participant and movie part at a time.
2. [Movie-segment matching, pairwise correlation, and synchrony notebook](https://github.com/olgagulka/eye-tracking-attention-/blob/main/notebooks/preprocessing/match_movie_segments_to_eye_tracking.ipynb): labelling the 3-second segments around each target frame, pairwise segment-level gaze correlations, and individual-to-group (IGS) and group (GS) synchrony per movie.
3. [Synchrony, GLMM, and visualization notebook](https://github.com/olgagulka/eye-tracking-attention-/blob/main/notebooks/analysis/synchrony_visualization.ipynb): GLMMs, average marginal effects, and figures.

The notebooks are not "run all" scripts. The participant, movie, and movie part must be set by hand in the relevant cells, and the absolute paths from the original workstation must be replaced. Step-by-step instructions, including which cells to run for each study, are in the [repository README](https://github.com/olgagulka/eye-tracking-attention-#how-to-run-the-notebooks).

## Results

Per-participant IGS and GS values for each movie segment are provided in [`results/IGS_GS_synchrony/`](https://github.com/olgagulka/eye-tracking-attention-/tree/main/results/IGS_GS_synchrony).
