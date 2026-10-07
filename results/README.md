# Results

## Included

- [`IGS_GS_synchrony/<movie>/participants_sync_metrics_<movie>_final_all_subjects.csv`](IGS_GS_synchrony/): individual-to-group synchrony (`pilot_mean`, IGS) and leave-one-out group synchrony (`group_mean`, GS) per participant (`participant`) and movie segment (`segment_num`), for the movies `guest`, `prayer`, `sprout`, and `witness`. These are the output of the final cells of [`match_movie_segments_to_eye_tracking.ipynb`](../notebooks/preprocessing/match_movie_segments_to_eye_tracking.ipynb).

## Not deposited

The notebooks also write other outputs that are not stored here, including:

- cleaned participant/movie-part gaze CSV files;
- merged run-01/run-02 gaze files;
- participant-pair Pearson-correlation CSV files for each movie;
- merged trial-level synchrony and behavioral files;
- GLMM average-marginal-effect tables; and
- manuscript figures and diagnostic plots.

The repository `.gitignore` excludes generated files in this directory by default. Selected aggregate tables and figures can be added later after confirming which outputs correspond to the final manuscript analysis.
