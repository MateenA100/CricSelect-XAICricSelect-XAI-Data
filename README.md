# Data — Complete Pipeline Artifact Trail

Every file this dissertation's analytical pipeline ever wrote, from raw ball-by-ball match data through to the frozen artifacts the dashboard serves: **862 MB across 454 top-level files and 8 subdirectories** — the complete, unfiltered output of all 23 notebooks in [CricSelect-XAI](https://github.com/MateenA100/CricSelect-XAI), produced across the five leagues (IPL, PSL, BBL, CPL, T20 Blast) through the 2024 season.

If you only want to **run the dashboard**, you don't need most of this — jump straight to [If you just want to run the dashboard](#if-you-just-want-to-run-the-dashboard). Everything else here exists for auditability: every intermediate step, every model comparison, every validation report that backs a number in the dissertation is a file you can open and check.

## Contents

- [If you just want to run the dashboard](#if-you-just-want-to-run-the-dashboard)
- [Why this folder isn't in a normal git remote yet](#why-this-folder-isnt-in-a-normal-git-remote-yet)
- [Raw source match data](#raw-source-match-data)
- [Everything else, by pipeline phase](#everything-else-by-pipeline-phase)
- [Three subdirectories worth knowing about](#three-subdirectories-worth-knowing-about)
- [The three largest files](#the-three-largest-files)
- [How to use this folder](#how-to-use-this-folder)

## If you just want to run the dashboard

[CricSelect-XAI-Backend](https://github.com/MateenA100/CricSelect-XAI-Backend)'s `app.py` loads exactly **30 files** at startup — nothing else in this folder is read by the deployed system. That subset is only **21 MB**, fully documented file-by-file (with size and purpose) in the backend repo's [Data dependencies](https://github.com/MateenA100/CricSelect-XAI-Backend#data-dependencies) table. If your goal is "clone the repos and see the dashboard work," those 30 files — placed in a folder named `data` as a sibling of the backend repo — are all you need. Everything below this point is the other 841 MB: development history, model comparisons, and validation evidence that never gets read at runtime, but is exactly what backs up every number the dissertation reports.

## Why this folder isn't in a normal git remote yet

Two files here exceed GitHub's 100 MB hard per-file push limit (`master_step6_5_clean_5leagues.csv` at 180 MB and `master_step6_5_clean.csv` at 171 MB — see [The three largest files](#the-three-largest-files)), and the folder as a whole is far past what's sensible to clone just to read code. That's *why* the runtime-critical subset above was split out separately — it's the one piece of this folder that both matters for reproducing the live system and is small enough to actually live in git.

## Raw source match data

The pipeline's true starting point — Cricsheet-format data for all five leagues, before Notebook 03 ever touches it:

| Folder | League | Format | Size |
|---|---|---|---|
| `IPL/` | Indian Premier League | Two pre-combined CSVs (`deliveries.csv`, `matches.csv`) | 28 MB |
| `PSL/` | Pakistan Super League | One pre-combined CSV (`Psl_Complete_Dataset(2016-2024).csv`) | 16 MB |
| `bbl_male_json/` | Big Bash League | Per-match Cricsheet JSON | 57 MB |
| `cpl_male_json/` | Caribbean Premier League | Per-match Cricsheet JSON | 36 MB |
| `ntb_male_json/` | T20 Blast (Cricsheet competition code `ntb`, historically the *NatWest T20 Blast*) | Per-match Cricsheet JSON | 132 MB |

`Notebooks/03_preprocessing.ipynb` is the only notebook that reads these directly — it's the schema-normalisation step that turns five differently-shaped sources into one common format for everything downstream.

## Everything else, by pipeline phase

Grouped by the notebook phase that produced it (see [CricSelect-XAI](https://github.com/MateenA100/CricSelect-XAI)'s own README for what each notebook does). Counts are top-level files only.

| Phase | Notebook(s) | ~Files | What's in it |
|---|---|---|---|
| Data preparation | 03 – 06.6 | ~10 | The standardised master datasets (`master_step6_5_clean*.csv`), data-quality/missing-value reports, the player-master evolution (`player_master_step9/12/13/final.csv`), and the CV fold/split registries |
| Exploratory analysis | 04 | 15 | `eda_*.png` — batting/bowling/credit distributions, correlations, boxplots, league and role breakdowns |
| General K-Means (superseded, K=6) | 07 | 14 | `kmeans_*` (model, cluster names/justifications, PCA scatter, silhouette plot, role-specific-vs-main comparison) and `player_clusters.csv` |
| General K-Means (final, K=4) | 08 | 23 | `kmeans2_*` — everything from the k-selection sweep and stability/ARI analysis to the deployed `player_clusters2.csv` and `kmeans2_representative_players.json` |
| Role-specific K-Means | 09 | 72 (24 each) | `kmeans_bat_*`, `kmeans_bowl_*`, `kmeans_ar_*` — one parallel set of model/scaler/diagnostics/validation files per role population |
| Iteration 1 Random Forest | 10 – 11 | ~50 | Cross-sectional train/test splits (`X_train.csv`, `y_test_tier.csv`, …), `rf_*` and `rf_enhanced_*` (pipeline, tuning, learning curves, ablations), plus `rf_tier_*` (28 files — calibration, confusion matrices, feature importance, overfitting diagnostics) |
| Player-season panel + RF | 12 – 13 | 35 | `season_rf_*` — the leakage-safe panel, rolling-origin CV results, calibration, feature ablation, and the frozen pre-2024 config |
| KNN recommender | 14 | 26 | `knn_*` — case studies, hubness/metric comparisons, outcome validation, stability results, plus the fitted `knn_recommender.pkl`, `knn_role_scalers.pkl` |
| Initial ILP (RF-scored) | 15 | 25 | `ilp_*` (excluding `ilp_ft_*`) — the first working ILP: player pool, solver report, sensitivity results, historical backtest |
| Model comparison | 16 | ~85 | `xgb_*` (15), `catboost_*` (20), `mlp_*` (16), `ft_transformer_*` (16, dev-stage), plus `baseline_results.json`, `gmm_comparison.json`, `kruskal_wallis_results.json`, `final_model_comparison.csv` |
| Final 2024 holdout | 17 | 15 | `final_2024_*` (9 files — predictions, per-model confusion matrices, bootstrap CIs) plus `final_deployment_manifest.json`, `final_model_selection_decision.json` |
| ILP forecast prep | 18 | 4 | `ilp_ft_primary_forecast_*`, `ilp_ft_scored_player_pool_2024.csv` |
| Final ILP (FT-Transformer-scored) | 19 | 4 | `ilp_ft_selected_xi_2024.csv`, `ilp_ft_cross_league_xi_2024.csv`, `ilp_ft_team_summary_2024.csv`, `ilp_ft_frozen_selection_config.json` |
| ILP sensitivity + random baselines | 20 | ~8 | `ilp_ft_weight_sensitivity*.csv`, `ilp_ft_random_valid_team_distribution.csv`, `ilp_ft_sensitivity_report.json` |
| Logistic-vs-FT ILP comparison | 21 | 5 | `ilp_logistic_comparator_*`, `ilp_logistic_vs_ft_*` |
| SHAP explanations | 22 | 7 | `shap_global_importance.csv`, `shap_local_values.csv.gz`, `shap_player_explanations.json`, `shap_validation_report.json` (plus `shap_plots/` — see below) |
| Llama explanation layer | 23 | 19 | `llm_baseline_vs_hybrid_comparison.json`, the blinded human-evaluation trail (`llm_human_evaluation_*`), `llm_hybrid_*`, `llm_template_explanations_all711.json`, `llm_validator_unit_tests.json` |

## Three subdirectories worth knowing about

- **`league_comparison/`** (71 MB) — the cross-league experiment evidence behind Chapter 4.10. `two_league/` (44 files) holds the original two-league-era master dataset, kept as the historical baseline the five-league expansion is compared against; `shared_holdout/` (14 files) holds the shared IPL/PSL holdout configuration and registries that Notebook 05 establishes.
- **`llm_human_reviews/`** (54 KB) — the raw reviewer `.xlsx` workbooks for Notebook 23's blinded A/B/C human evaluation. Filenames ending `_complete`/`_completed` are what `Notebooks/llama_human_evaluation_analysis.py` looks for; the method key that unblinds `A`/`B`/`C` back to real method names (`llm_human_evaluation_unblinding_key.json`, `llm_hybrid_human_evaluation_key.json`) lives alongside them but is only opened by that script *after* scoring.
- **`shap_plots/`** (1.4 MB) — per-feature SHAP dependence plots (`shap_dependence_career_balls_bowled.png`, etc.) generated in Notebook 22, supporting the global feature-importance discussion in Chapter 4.9.

## The three largest files

All three are successive versions of the combined five-league master dataset from Notebooks 06–06.6, and are the reason this folder can't be pushed to GitHub as-is:

| File | Size | Notebook |
|---|---|---|
| `master_step6_5_clean_5leagues.csv` | 180 MB | 06.6 |
| `master_step6_5_clean.csv` | 171 MB | 06 |
| `master_step6_5_clean_standardized.csv` | 72 MB | 06.5 |

Both of the first two individually exceed GitHub's 100 MB hard push limit.

## How to use this folder

| Your goal | What you need |
|---|---|
| **See the dashboard working** | Just the 30-file, 21 MB subset — see [If you just want to run the dashboard](#if-you-just-want-to-run-the-dashboard) |
| **Audit a specific claimed result** (e.g. the 0.250 silhouette score, the 23/30 hybrid pass rate, the 98.6th–99.9th percentile ILP result) | Find the phase in the table above, open the relevant `*_summary`/`*_report`/`*_validation` file directly — every reported number in the dissertation traces back to a file here |
| **Re-run the pipeline from raw data** | Everything — start from [Raw source match data](#raw-source-match-data) and run [CricSelect-XAI](https://github.com/MateenA100/CricSelect-XAI)'s notebooks 03→23 in order, per that repo's README |
