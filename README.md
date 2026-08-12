[README.md](https://github.com/user-attachments/files/30968596/README.md)
# Code and data for "Persistent Directional Anisotropy in Dynamic Visual Acuity and Attenuated Increase in Saccadic Eye Movements with Stroboscopic Training"

This package contains the code and the processed data needed to reproduce every
data-derived figure panel and every statistical table in the manuscript. Participant
names are replaced by anonymised identifiers (`Subject A` … `Subject P`); the mapping to
the original identifiers is **not** included.

The package is self-contained: scripts resolve their own location, so no path
configuration is needed.

## Which script makes which manuscript item

| Manuscript item | Script | Input data |
|---|---|---|
| **Figure 4** C/D (DVA split violins) | `src/make_dva_violin.m` | `data/processed/dva_pertrial.csv`, `config/subject_group.csv` |
| Figure 4 significance markers | `stats/make_manuscript_stats.R` | same as above |
| **Figure 5** (saccade validation, main sequence) | `src/make_figure1_saccade_validation.m` | `data/processed/rotation_waveforms_hbw.mat` |
| **Figure 6** (saccadic / non-saccadic components) | `src/make_figure2_amplitude_decomposition.m` | `data/processed/pathlen_pertrial.csv` |
| **Table 1** (fixed effects on DVA) | `stats/make_manuscript_stats.R` | `data/processed/dva_pertrial.csv` |
| **Table 2** (saccadic contribution, %) | `stats/make_manuscript_stats.R` | `data/processed/pathlen_pertrial.csv` |
| **Table 3** (fixed effects on A_s and A_ns) | `stats/make_manuscript_stats.R` | `data/processed/pathlen_pertrial.csv` |
| **MAE table** (fixed effects on E_MAE) | `stats/make_manuscript_stats.R` | `data/processed/position_error_pertrial.csv` |

Figures 1–3 of the manuscript are schematic illustrations of the apparatus and the
protocol. They are not generated from data and are therefore not included here.
Panels A and B of Figure 4 are a scatter plot and a descriptive table assembled from
`dva_pertrial.csv` and the participant characteristics reported in the text.

`src/pathlen_pertrial.m` regenerates `pathlen_pertrial.csv` from the waveforms, so the
chain from `rotation_waveforms_hbw.mat` through Figure 6 and Tables 2–3 is reproducible
end to end.

## Data files

| File | Rows | Contents |
|---|---|---|
| `data/processed/dva_pertrial.csv` | 640 | DVA per trial (16 participants × 4 sessions × 2 directions × 5 trials) |
| `data/processed/pathlen_pertrial.csv` | 631 | Cumulative eye displacement `A` split into `saccadic` (A_s) and `non_saccadic` (A_ns) |
| `data/processed/position_error_pertrial.csv` | 631 | Mean absolute eye–target position error (`maperr`) and percentile summaries |
| `data/processed/rotation_waveforms_hbw.mat` | — | Per-rotation eye and target waveforms (500 Hz), ~23 MB |
| `config/subject_group.csv` | 16 | Participant → group assignment (`glass` = `T` for the strobe group) |

The raw recordings (DeepLabCut output and LabChart exports, tens of GB) are not
included. Everything here starts from the per-rotation waveforms.

## Group assignment

`config/subject_group.csv` is the single source of truth for the group assignment.
`stats/make_manuscript_stats.R` stops with an error if the `glass` column of any data
file disagrees with it, so the assignment cannot silently diverge between figures and
tables. All results in the manuscript were produced with the assignment in this file.

## How to run

MATLAB (tested on R2026a; requires the Statistics and Machine Learning Toolbox for
`fitlme` and the Signal Processing Toolbox for `butter`, `filtfilt`, `findpeaks`):

```matlab
cd <this folder>
run_all
```

Statistics (R 4.3.3 with `lme4`, `lmerTest`, `emmeans`):

```
Rscript stats/make_manuscript_stats.R
```

The R script writes five CSV files to `tables/`. Degrees of freedom use the
Satterthwaite approximation. Every model is

```
y ~ D * G * T + (1 | participant),  REML
```

with effect coding `D = ±0.5` (vertical / horizontal), `G = ±0.5` (strobe / control),
and `T` centred on the mean session. Each coefficient is therefore the marginal effect
averaged over the remaining factors, and the intercept is the grand mean. The Figure 4
markers are pairwise vertical-versus-horizontal contrasts of the conditional means
within each group × session cell, with the eight comparisons Holm-adjusted.

## Supporting functions

Not run directly: `src/threshold_events.m` (velocity-threshold event extraction),
`src/detect_saccades_ms.m`, `src/align_velocity.m`, `src/load_waveforms.m`,
`src/diff_filter.m`, `src/replicate_amp.m`.

## Third-party code

`src/lib/Violin` is the Violinplot-Matlab package, redistributed under its own licence
(see `src/lib/Violin/LICENSE`).
