# Notebooks

The notebooks in this directory are the analysis record. They contain hard-coded paths and participant/movie settings from the original analysis environment, so they are meant to be run **cell by cell on a working copy**, not as "run all" scripts. See the top-level [README](../README.md#how-to-run-the-notebooks) for a step-by-step description of each notebook, the parameters to set, and how the main experiment (YN-2AFC) and the control study (2AFC only) differ.

## Organization (run in this order)

1. [`preprocessing/preprocessing_Eyetracking.ipynb`](preprocessing/preprocessing_Eyetracking.ipynb): eye-tracking preprocessing and quality checks. Run once per participant and movie part; set `folder`, `participant_id`, `movie_type`, `video_name`, and `start` (movie-onset timestamp from the separate time-log file).
2. [`preprocessing/match_movie_segments_to_eye_tracking.ipynb`](preprocessing/match_movie_segments_to_eye_tracking.ipynb): labelling gaze with the 3-second movie segments, segment-level pairwise gaze correlations, and individual (IGS) and group (GS) synchrony. Set `movie` and `participant_id` (and `movie_part`) before running the matching cells; call the correlation and synchrony functions once per movie.
3. [`analysis/fitting_models_and_main_figures.ipynb`](analysis/fitting_models_and_main_figures.ipynb): GLMMs, average marginal effects, visualisations in the main paper.
4. [`analysis/Supplementary_figures.ipynb`](analysis/Supplementay_figures.ipynb): plotting the figures founding the supplementary material.
   When running the cells and replace the absolute paths.
