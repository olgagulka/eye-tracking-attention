# Data

No study data are currently included in this repository.

The notebooks expect the following classes of input:

- Gazepoint all-gaze exports containing `FPOGX`, `FPOGY`, and `TIMETICK(f=10000000)`;
- movie files used to obtain exact presentation durations;
- separately recorded movie-onset timestamps;
- merged participant/movie gaze files labeled by `movie_part` and `segment_id`;
- trial-level behavioral files containing YN and/or 2AFC correctness and response times; and
- merged synchrony/behavior files containing participant, movie, segment, IGS, GS, and leave-one-out accuracy fields.

Raw or identifiable participant data should not be committed unless sharing is permitted by the consent procedure, ethics approval, and institutional policy. The repository `.gitignore` excludes additions to this directory by default; revise that rule deliberately if an approved de-identified dataset is released.
