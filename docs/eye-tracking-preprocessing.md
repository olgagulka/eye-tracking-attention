---
layout: page
title: Eye-tracking preprocessing
---

The preprocessing record is preserved in [`preprocessing_Eyetracking.ipynb`](https://github.com/olgagulka/eye-tracking-attention-/blob/main/notebooks/preprocessing/preprocessing_Eyetracking.ipynb). The notebook contains multiple historical, current, and exploratory sections; the descriptions below identify the workflow reported in the research notes and the operations implemented in the notebook.



## Main gaze-preprocessing operations

For a selected participant, movie, and movie part, the current notebook code performs the following operations:

1. Read the Gazepoint all-gaze CSV and retain `FPOGX`, `FPOGY`, and `TIMETICK(f=10000000)`.
2. Convert `TIMETICK(f=10000000)` to seconds by dividing by 10,000,000.
3. Find the sample closest to the separately recorded movie-start time, discard earlier samples, and reset elapsed time to zero.
4. Read the movie duration with `pymediainfo` and retain samples through the end of the movie.
5. Mark normalized x or y values less than or equal to 0, or greater than 1, as missing (`NaN`). The research notes identify these values as primarily reflecting blinks or brief signal loss.
6. Linearly interpolate the missing x and y values in both directions using pandas.
7. Convert normalized coordinates to 1024 × 768 pixel coordinates:

   ```text
   x = FPOGX × 1024
   y = (1 − FPOGY) × 768
   ```

8. Retain `timestamp`, `x`, and `y`, perform printed range checks, and save a participant/movie-part CSV.

The notebook also contains a padding procedure for cases in which eye tracking began after movie onset. It estimates the sampling interval from the median timestamp difference and inserts missing rows before the first recorded gaze sample.

## Merging movie parts

Each movie was presented in two parts. The `merge_movie_parts` function reads the processed `run_01` and `run_02` files for one participant, adds a `movie_part` label to each, concatenates them, and saves one merged participant/movie file. These merged files are the inputs to the later movie-segment matching step.


## Behavioral preprocessing dependency

Before synchrony values are joined to behavior, the YN and 2AFC response times are calculated from the recorded absolute `perf_counter()` values. The research notes report that the timing check compared absolute values with `.flip()` timing, inspected drift/offset and its standard deviation, and confirmed agreement to approximately 0.0001 seconds. The processed files add `YN_RT` and `2AFC_RT` columns. The corresponding `RTs_for_YN_2AFC_tasks.ipynb` notebook has not yet been supplied to this repository.
