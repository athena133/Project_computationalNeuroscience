# Neural population analysis of task context in rat mPFC

Exploratory, **single-session** reanalysis of extracellular spikes and auditory task behavior in the public **CRCNS pfc-1** dataset. This directory preserves an authentic learning and analysis notebook: **47 original exploratory code cells, followed by 5 additional cells** for PCA and a matched neural-decoding control. Intermediate steps and some repeated cells are intentionally retained.

## Research question

Do prestimulus medial prefrontal cortex (mPFC) population firing rates support generalizable prediction of the current auditory stimulus-selection rule (**Localization** vs **Pitch**)?

The recording is `CR24A_121008_001_behaving`. This project is a limited reanalysis and **does not claim to reproduce or refute** the original publication's broader findings.

## Notebook

[Open the 52-cell exploratory notebook](01_data_exploration_47_original_plus_5_new.ipynb)

Contents: inspection of behavior and spike tables; trial-aligned spike rasters and PSTH for unit 104; single-unit descriptive statistics; 42-unit prestimulus firing-rate matrix; exploratory Mann–Whitney U tests with FDR correction; population decoding with leave-one-cycle-out validation; six-window temporal sensitivity analysis; descriptive PCA; and stimulus/required-response-matched decoding.

## Data and preprocessing

- Data source: Rodgers & DeWeese, **CRCNS pfc-1**, https://doi.org/10.6080/K0W66HPJ ; dataset page: https://crcns.org/data-sets/pfc/pfc-1/about-pfc-1/
- Related paper: Rodgers & DeWeese (2014), *Neuron*, https://doi.org/10.1016/j.neuron.2014.04.031
- Selected **normal blocks 2 and 4**, discarded invalid, nonrandom, munged and non-hit trials.
- **174 correct trials:** 103 Localization and 71 Pitch. The two complete cycles include **154 trials** with both rules; an incomplete third cycle lacks Pitch.
- Spike feature: mean firing rate in **[-200, 0) ms** relative to stimulus onset for **42 mPFC units**, computed by counting spikes and dividing by 0.2 s. The complete correct-trial feature matrix is **174 × 42**.
- These features are sparse (~92.32% zeros). On the two-cycle PCA subset, **25 units** have nonzero variance.

## Methods

A pipeline of `VarianceThreshold` → `StandardScaler` → class-weighted `LogisticRegression` was fit separately within **Leave-One-Cycle-Out** cross-validation folds. Neural decoding input features comprise **only firing rates**; rule, stimulus name, cycle ID and temporal phase are **not model inputs**.

For the matched control, retained physical stimuli `le_lo` (required Go in either rule) and `ri_hi` (required No-go in either rule), and balanced counts within each **cycle × stimulus × rule** stratum using a fixed random seed **42**. The subset has **58 trials, 29 per rule**. Matching controls stimulus identity and *required action*, not all actual behavior or experimental time.

PCA is **descriptive**, not a supervised rule-decoding metric: explained variance PC1 **16.31%**, PC2 **8.59%**.

## Reported results

| Evaluation | Trials | Balanced accuracy |
|---|---:|---:|
| Original prestimulus, leave-one-cycle-out | 154 | **0.4654** |
| Stimulus/required-response matched, leave-one-cycle-out | 58 | **0.5517** |

Matched held-out cycle 0: BA **0.533**, AUC **0.551** (30 trials). Held-out cycle 1: BA **0.571**, AUC **0.643** (28 trials). Matched confusion matrix, true labels in rows `[Localization, Pitch]`, predictions in columns: `[[10, 19], [7, 22]]`. Thus Localization recall was **10/29**, versus Pitch recall **22/29**; the model preferentially predicted Pitch.

Exploratory temporal-window balanced accuracies: **0.567, 0.495, 0.465, 0.321, 0.558, 0.396** for [-200,-100), [-100,0), [-200,0), [0,100), [100,200), [0,200) ms, respectively. No individual unit survived FDR correction in the exploratory test.

The matched score is numerically higher than the original score, but **sample size and composition both changed**. This difference is **not** a statistically established improvement, evidence of rule-specific coding, or a causal effect of matching.

## Main scientific limitations

1. **Temporal confounding cannot be eliminated here:** Localization always precedes Pitch **within each cycle**. Rule is inseparable from within-cycle phase; row shuffling and cross-cycle validation cannot resolve this experimental design confound.
2. **Single session; only two complete cycles.** Trials within blocks are not exchangeable independent replications, and the project does not establish statistical significance for decoding.
3. **Sparse rates and possible PCA outlier sensitivity.** Two principal components summarize only 24.90% of standardized population variance.
4. **Matching reduces sample size and covers only selected stimulus / required-response combinations**; it does not match all motor or internal-state variables. One fixed-seed sample does not characterize sampling variability.
5. **Multiple exploratory time windows**, with poststimulus features potentially reflecting sensory, decision and motor processes.

## How to run

1. Install dependencies from [requirements.txt](requirements.txt).
2. Download pfc-1 through CRCNS and prepare the session's `trials_info`, `trial_timings`, and converted `CR24A_121008_001_behaving_spikes.csv` (columns: `group,time,unit`; spike times in seconds).
3. **Edit the local Windows paths already present in the original notebook**, especially `DATA_DIR` near the beginning and `session_path` / spike-CSV loading cells. The original 47-cell workflow contains exploratory session selection and assumes a local dataset directory. It has **not** been refactored into a one-click portable pipeline because the goal is to preserve the original notebook's development history.
4. In Jupyter, use **Restart Kernel & Run All** and check final outputs against the numbers reported above.

**Reproducibility status:** notebook syntax and structure were checked; its **complete fresh-kernel execution has not been independently verified** in the available environment. Original raw recording files are **not included** in this repository. This work is an exploratory learning portfolio artifact, not a validated clinical or mechanistic neural model.

## Attribution

Dataset and original research: **Rodgers CC and DeWeese MR (2014)**, CRCNS pfc-1 / *Neuron*. The exploratory analysis notebook is separate work and was developed interactively with AI-assisted coding guidance. All scientific conclusions above are restricted to the stated analysis and session.
