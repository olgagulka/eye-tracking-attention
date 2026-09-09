# Computational environment

The manuscript reports Python 3.12.7 and R 4.5.2 for the GLMM analysis. The synchrony notebook metadata reports Python 3.12.7; the preprocessing notebook metadata reports Python 3.9.6.

[`requirements.txt`](requirements.txt) and [`r-packages.txt`](r-packages.txt) list packages imported by the supplied notebooks. They are intentionally unpinned because exact package versions were not present in the supplied files. These lists document dependencies but do not yet constitute an exact lock file.

The R cells are executed from Jupyter through `rpy2` and require a working local R installation. `pymediainfo` also requires the MediaInfo system library.
