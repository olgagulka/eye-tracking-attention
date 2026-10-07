# Eye-tracking synchrony and episodic memory

This repository accompanies a manuscript investigating whether inter-individual eye-gaze synchrony during movie viewing predicts subsequent recognition-memory performance. The project contains

- the **main experiment**, in which each trial had a sequential Yes/No (YN) recognition judgment followed by a two-alternative forced-choice (2AFC) judgment (the YN-2AFC task); and
- a **control study**, which used the 2AFC task only.

Both studies used the same movies, eye-tracker, and **identical eye-tracking preprocessing and synchrony computation**. They differ only in the final statistical analysis: the main experiment models YN and 2AFC accuracy, whereas the control study models 2AFC accuracy only (see [Main experiment vs. control study](#main-experiment-vs-control-study)). (The pages in [`docs/`](docs/index.md) call these Study 1 and Study 2, respectively.)

## Repository contents

| Location | Description |
| --- | --- |
| [`notebooks/preprocessing/preprocessing_Eyetracking.ipynb`](notebooks/preprocessing/preprocessing_Eyetracking.ipynb) | **Step 1.** Eye-tracking alignment to movie onset, trimming, blink handling, interpolation, and conversion to pixel coordinates (one participant × movie part per run) |
| [`notebooks/preprocessing/match_movie_segments_to_eye_tracking.ipynb`](notebooks/preprocessing/match_movie_segments_to_eye_tracking.ipynb) | **Step 2.** Labelling gaze samples with the 3-second movie segments around each target frame, pairwise segment-level gaze correlations, and individual-to-group (IGS) and group (GS) synchrony |
| [`notebooks/analysis/synchrony_visualization.ipynb`](notebooks/analysis/synchrony_visualization.ipynb) | **Step 3.** GLMMs, average marginal effects, and manuscript figures |
| [`results/IGS_GS_synchrony/`](results/) | IGS/GS values per participant and movie segment (output of Step 2), one CSV per movie |
| [`docs/`](docs/index.md) | Study design, processing workflow, statistical analysis, and reproducibility notes |
| [`environment/`](environment/README.md) | Python and R dependency information |
| [`data/`](data/README.md) | Data availability and expected-input documentation |

## Analysis workflow

```text
raw Gazepoint export ──► Step 1 ──► filtered gaze per participant/movie part
                                          │  (merge the two movie parts, QC)
                                          ▼
movie annotation files ─► Step 2a: label 3-s segments ──► Step 2b: pairwise Pearson r
                                                               │
                                                               ▼
                                              Step 2c: IGS and GS per segment
                                                               │  (merge with behavior)
                                                               ▼
                                          Step 3: GLMMs + average marginal effects + figures
```

1. Record gaze position during movie encoding and behavioral responses during the subsequent recognition task.
2. Calculate YN and 2AFC response times from the raw behavioral files.
3. **Step 1:** Align each gaze recording to movie onset, trim it to the movie duration, mark invalid normalized coordinates, interpolate missing samples, and convert coordinates to pixels. Merge the two parts of each movie.
4. **Step 2a:** Match the 3-second windows (1.5 s before and after target-frame onset) to each participant's merged gaze data.
5. **Step 2b:** Within each movie and segment, calculate Pearson correlations for every participant pair using the gaze-coordinate time series.
6. **Step 2c:** Derive individual-to-group synchrony (IGS) and leave-one-out group synchrony (GS), then merge them with trial-level behavioral outcomes.
7. **Step 3:** Fit binomial generalized linear mixed-effects models (GLMMs) predicting trial accuracy, with participant identity as a random intercept and leave-one-out trial accuracy as a covariate.

More detail is given in [preprocessing](docs/eye-tracking-preprocessing.md) and [synchrony/GLMM](docs/synchrony-and-glmm.md).

## How to run the notebooks

> **Important:** the notebooks are not "run all" scripts. Each one has cells where you must **set the participant and the movie (and, where relevant, the movie part) by hand** and then run the cells again, once per participant × movie × part. Several cells also contain hard-coded absolute paths from the original workstation (`/Users/olgagulka/Desktop/IBS/...`) that must be changed to your own directory structure.

The four movies are `guest`, `prayer`, `sprout`, and `witness`. Each movie was shown in two parts (`01` and `02`), so every participant has eight gaze recordings (4 movies × 2 parts).

### Step 1: `preprocessing_Eyetracking.ipynb`

Run once **per participant and per movie part**. Edit the parameters at the top of the main code cell:

| Parameter | Meaning | Example |
| --- | --- | --- |
| `folder` | Movie folder name in the exported eye-tracking data | `"guest"` |
| `participant_id` | Participant identifier | `"pilot_07"` |
| `movie_type` | Movie and part label used in the export folder and in the output filename | `"guest_run_02"` |
| `video_name` | File name of the movie part, used to read its exact duration | `"NO_CAL_GUEST_part02.mp4"` |
| `start` | Gazepoint timestamp (seconds) at which the movie started. This value is recorded separately during the experiment in the `time_log_movies` file and must be looked up for every participant and movie part | `750493.835` |

Also set `eye_data_file` (the Gazepoint *all gaze* export, `User 0_all_gaze.csv`), `output_dir`, and the movie directory used by `MediaInfo.parse`.

What the cell does:

1. Reads `FPOGX`, `FPOGY`, and `TIMETICK(f=10000000)` and converts the tick counter to seconds.
2. Finds the sample closest to `start`, discards earlier samples, and resets time to 0 at movie onset.
3. Reads the movie duration with `pymediainfo` and discards samples after the end of the movie.
4. Treats normalized coordinates that are ≤ 0 or > 1 (mainly blinks and signal loss) as missing, then fills them by linear interpolation.
5. Converts to pixels for the 1024 × 768 screen: `x = FPOGX × 1024`, `y = (1 − FPOGY) × 768`.
6. Prints range checks and saves `{movie_type}_{participant_id}_filtered_eye_data.csv` with columns `timestamp`, `x`, `y`.

The following cell plots the x and y gaze coordinates over time as a visual quality check.

**Input expected by Step 2.** After Step 1, the two parts of each movie must be combined and labelled so that Step 2 can read them from `eye_tracking_filtered_qc/<participant>/<movie>/`:

- `<movie>_part_01_<participant>_filtered_eye_data.csv` and `<movie>_part_02_<participant>_filtered_eye_data.csv` (single parts), and/or
- `<movie>_<participant>_merged_eye_data.csv` (both parts)

Each file must contain `x`, `y`, `timestamp`, `movie_part` (`01`/`02`), and `part_timestamp` (time within the part; the merged `timestamp` continues across parts, i.e., it includes the duration of part 1 for part 2). The merging/quality-control code that produces these files is **not yet included** in this repository (see [Not yet included](#not-yet-included)).

### Step 2: `match_movie_segments_to_eye_tracking.ipynb`

**Settings cell.** The first code cell defines the paths (`ANNOTATION_DIR`, `FILTERED_ROOT`, `OUTPUT_DIR`), the window half-width (`TIME_WINDOW = 1.5` s), and the exclusions:

- `EXCLUDED_PARTS`: participant × movie × part combinations removed from all further analyses (for example, poor tracking quality);
- `EXCLUDED_SEGMENTS`: segments removed for every participant (`sprout` seg_26 and `prayer` seg_22);
- `BEHAVIORAL_TARGET_OVERRIDES`: a witness-movie crosswalk. In the movie-matching numbering, `seg_34` uses `frame_35_witness.jpg`, but the same trial is `frame_34_witness.jpg` in the behavioral task. The matched files therefore carry both `target_image` (used to find the movie timestamp) and `behavioral_target_image` (used when merging with behavior).

**Step 2a: matching.** `match_movie_segments_to_eye_tracking(movie, participant_id, output_file, movie_part=None)` reads the annotation file `annotations_<movie>_movie.csv` (columns `movie_part`, `target_image`, `match_timestamp_sec`) and labels every gaze sample within ±1.5 s of each target-frame timestamp with `segment_id` (`seg_1`, `seg_2`, …, numbered across the whole movie), `target_image`, and `behavioral_target_image`.

- **Set `movie` and `participant_id`** (and `movie_part`) in the single-participant cell and run it. `movie_part=None` processes the merged file; `"01"` or `"02"` processes one part, which is how participants with only one usable part are handled.
- The next cell loops over `sub_01` … `sub_32` for one `movie`; **change `movie` and run it once per movie**. Participants whose parts are both excluded are skipped, and missing inputs are listed at the end.
- If two windows overlap, the later segment overwrites the earlier label and a warning is printed (a sample can carry only one `segment_id`).
- Output: `<movie>_<participant>_merged_eye_data_with_movie_segments.csv` (or `<movie>_part_<NN>_<participant>_eye_data_with_movie_segments.csv`).

**Step 2b: pairwise correlations.** `pearson_corr_per_segment(movie_name, participants)` loads the matched files for one movie, removes the excluded parts and segments (the lists are repeated inside this cell and must match the settings cell), and for every participant pair and every shared segment:

1. truncates both gaze series to the shorter of the two lengths;
2. stacks each participant's x values followed by their y values into one vector; and
3. computes the Pearson correlation *r* (and p-value) between the two vectors.

**Call it once per movie** (`movie_name="guest"`, `"prayer"`, `"sprout"`, `"witness"`). It writes one CSV per participant pair (`<movie>_correlation_<sub_A>_<sub_B>.csv`). `plot_average_segment_correlations(movie_name)` then summarizes these files (Fisher-z-averaged *r*, median, interquartile range, per segment), saves `<movie>_average_correlation_by_segment.csv`, and plots it as a quality check.

**Step 2c: IGS and GS.** `compute_group_and_pilot_sync(movie, participants, reject_segments=[])` reads the pairwise files for one movie (call it once per movie) and, for each segment and participant, computes:

- **IGS** (`pilot_mean`): the mean *r* between that participant and every other participant;
- **GS** (`group_mean`): the mean *r* among all pairs that **exclude** that participant (leave-one-out).

Output: `participants_sync_metrics_<movie>_final_all_subjects.csv` with columns `segment_num`, `participant`, `pilot_mean` (IGS), and `group_mean` (GS). The files for the four movies are in [`results/IGS_GS_synchrony/`](results/). `reject_segments` takes segment *numbers* (for example `[22]`); segments already rejected in Step 2b do not need to be listed again.

### Step 3: `synchrony_visualization.ipynb`

The notebook starts from a **merged trial-level CSV** that combines IGS/GS with behavior. One row is one trial for one participant, with at least `participant_id`, `movie`, `segment_num`, `pilot_mean` (IGS), `group_mean` (GS), and the behavioral columns: `correct_response` and `mean_correct_2AFC_LOO` (2AFC accuracy and its leave-one-out group accuracy), and, for the main experiment, `correct_response_occlusion`, `response_occlusion`, and `mean_correct_YN_LOO` (YN accuracy, response, and leave-one-out accuracy). **Set the path to this file** in the first data-loading cell (and in the later cells that reload it). Output paths for the AME tables and figures must also be edited.

The R cells run through `rpy2` (`%load_ext rpy2.ipython`) and need a working R installation with `lme4` and `marginaleffects`.

1. **Decomposition.** IGS and GS are strongly collinear, so both are z-scored and transformed to `Z1 = (z(IGS) + z(GS)) / 2` (shared synchrony) and `Z2 = (z(IGS) − z(GS)) / 2` (the individual's deviation from the group).
2. **GLMM.** `glmer(accuracy ~ Z1 + Z2 + leave-one-out accuracy + (1 | participant_id), family = binomial, optimizer = "bobyqa")`.
3. **Average marginal effects** on the probability scale (`avg_slopes(..., type = "response")`), saved as CSV files and plotted as forest plots (change in predicted % correct, 95% CI).
4. **Main experiment only: YN signal-present vs. signal-absent trials.** The YN trials are split into signal-present (hits and misses) and signal-absent (correct rejections and false alarms) trials, a condition-specific leave-one-out accuracy covariate is computed, and the GLMM and AMEs are repeated per condition.

## Main experiment vs. control study

All of Step 1 and Step 2 is identical for the two studies. Only the participant list, the exclusion lists, and the file locations differ.

| | Main experiment (YN-2AFC) | Control study (2AFC only) |
| --- | --- | --- |
| Participants | 30 | 32 (`sub_01` … `sub_32`) |
| Step 1: gaze preprocessing | same | same |
| Step 2: segment matching, pairwise *r*, IGS/GS | same code, main-experiment participants and exclusions | same code, control-study participants and exclusions |
| Step 3: models | YN **and** 2AFC GLMMs; YN signal-present/absent split | **2AFC GLMM only** |
| Notebook cells to run in Step 3 | all | only the 2AFC cells (2AFC GLMM, saved AME table, and the 2AFC half of the forest plot); skip the YN and signal-present/absent cells |

**The notebooks as saved are configured as follows, and must be edited for the other study:**

- `match_movie_segments_to_eye_tracking.ipynb` uses `sub_01`–`sub_32`, the control-study exclusion list, and paths under `2AFC_STUDY/`. To process the main experiment, change `CURRENT_PARTICIPANTS`, `EXCLUDED_PARTS` (all three places they are defined), and the input/output paths. The exclusions differ between the two studies.
- `synchrony_visualization.ipynb` reads the `..._30_subjects_final.csv` file, i.e., the main-experiment data. To analyze the control study, point it to the control-study merged CSV and run only the 2AFC cells.
- The IGS/GS files in `results/IGS_GS_synchrony/` come from the configuration saved in the Step 2 notebook.

## Not yet included

These steps are used in the workflow but their code is not in this repository yet:

- merging the two movie parts, adding `movie_part` / `part_timestamp`, and gaze-quality control, which produces the `eye_tracking_filtered_qc` files read by Step 2;
- the movie annotation files (`annotations_<movie>_movie.csv`) with the target-frame timestamps;
- calculation of YN and 2AFC response times from the raw behavioral files;
- merging IGS/GS with trial-level behavior and computing the leave-one-out group accuracy, which produces the input CSV of Step 3 (the notebook includes the code that creates the signal-absent file, but not its signal-present counterpart); and
- the signal-detection criterion (*C*) and repeated-measures ANOVA analyses, if stored separately.

## Reuse

The notebooks reference the original local directory structure, so paths must be edited before running them. Package names are listed in [`environment/`](environment/README.md), but exact package versions were not recoverable for every dependency. Gaze, behavioral, and movie-stimulus data are described in [`data/`](data/README.md). A license and formal citation file should be added once the authors have selected reuse terms and finalized the manuscript citation.

## Documentation website

The GitHub Pages source is ready in [`docs/`](docs/index.md). To publish it, select **Deploy from a branch**, then choose the `main` branch and `/docs` folder in **Settings → Pages**. A private repository requires a GitHub plan that supports Pages for private repositories; otherwise the repository must first be made public.
