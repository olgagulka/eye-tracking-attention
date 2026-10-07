# Notebooks

The notebooks in this directory are preserved analysis records. They were copied byte-for-byte from the supplied source files and have not been refactored, cleaned, or converted into new scripts.

## Organization

- [`preprocessing/preprocessing_Eyetracking.ipynb`](preprocessing/preprocessing_Eyetracking.ipynb): eye-tracking preprocessing, quality checks.
- [`preprocessing/match_movie_segments_to_eye_tracking.ipynb`](preprocessing/match_movie_segments_to_eye_tracking.ipynb): merging eye gaze-tracking with movie segments, concatenating movie parts, segment-level pairwise gaze correlations, individual and group synchrony.
- [`analysis/synchrony_visualization.ipynb`](analysis/synchrony_visualization.ipynb):  GLMMs, average marginal effects, and manuscript visualizations.

The notebooks include hard-coded paths and participant/movie settings from the original analysis environment. When running the the cells within each script the participant number/movie part should be adjusted accordingly. 


