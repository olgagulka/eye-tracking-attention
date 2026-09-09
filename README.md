# Eye-tracking synchrony and episodic memory

This repository accompanies a manuscript investigating whether inter-individual eye-gaze synchrony during movie viewing predicts subsequent recognition-memory performance. The project contains two experiments: Study 1 used sequential Yes/No (YN) and two-alternative forced-choice (2AFC) recognition tasks, whereas Study 2 used the 2AFC task.

## Repository contents

| Location | Description | Status |
| --- | --- | --- |
| [`notebooks/preprocessing/preprocessing_Eyetracking.ipynb`](notebooks/preprocessing/preprocessing_Eyetracking.ipynb) | Eye-tracking alignment, cleaning, coordinate conversion, interpolation, and merging of movie parts | Included verbatim |
| [`notebooks/analysis/synchrony_visualization.ipynb`](notebooks/analysis/synchrony_visualization.ipynb) | Segment-level correlations, synchrony measures, GLMMs, marginal effects, and visualizations | Included verbatim |
| [`docs/`](docs/index.md) | Study design, processing workflow, statistical analysis, and reproducibility notes | Included |
| [`environment/`](environment/README.md) | Python and R dependency information recoverable from the notebooks and manuscript | Included |
| [`data/`](data/README.md) | Data-availability and expected-input documentation | Data not included |
| [`results/`](results/README.md) | Expected generated outputs | Outputs not included |

## Analysis workflow

1. Record gaze position during movie encoding and behavioral responses during the subsequent recognition task.
2. Calculate YN and 2AFC response times from the raw behavioral files.
3. Align each gaze recording to movie onset, trim it to the movie duration, mark invalid normalized coordinates, interpolate missing samples, convert coordinates to pixels, and merge the two movie parts.
4. Match the 3-second windows surrounding target-frame onsets to each participant's merged gaze data.
5. Within each movie and segment, calculate Pearson correlations for every participant pair using the gaze-coordinate time series.
6. Derive individual-to-group synchrony (IGS) and leave-one-out group synchrony (GS), then merge them with trial-level behavioral outcomes.
7. Fit binomial generalized linear mixed-effects models (GLMMs) predicting trial accuracy, with participant identity as a random intercept and leave-one-out trial accuracy as a covariate.

The detailed [preprocessing](docs/eye-tracking-preprocessing.md) and [synchrony/GLMM](docs/synchrony-and-glmm.md) pages distinguish steps contained in the uploaded notebooks from scripts that are still to be deposited.

## Code provenance

The two notebooks were copied into this repository without changing their code, markdown, stored outputs, metadata, or hard-coded paths. Their source-file checksums are recorded in the [reproducibility notes](docs/reproducibility.md). The notebooks are preserved as the analysis record and contain exploratory as well as final sections; consult the documentation before running individual cells.

## Reuse

The notebooks currently reference the original local directory structure and require project data that are not deposited here. Package names are listed in [`environment/`](environment/README.md), but exact package versions were not recoverable for every dependency. A license and formal citation file should be added once the authors have selected reuse terms and finalized the manuscript citation.

## Documentation website

The GitHub Pages source is in [`docs/`](docs/index.md). If Pages is not already configured, select **Deploy from a branch**, choose the `main` branch and `/docs` folder in the repository's **Settings → Pages**.
