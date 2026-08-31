# Data — Runtime Artifacts

The 31 frozen files [CricSelect-XAI-Backend](https://github.com/MateenA100/CricSelect-XAI-Backend)'s `app.py` reads at startup to serve the dashboard — nothing more. **Total size: ~22 MB.** This is deliberately a small, focused subset, not the dissertation's full dataset (see [Not here: the full dataset](#not-here-the-full-dataset) below).

## Why this repo exists

The backend resolves this folder as a sibling of its own repo (`Path(__file__).resolve().parents[1] / "data"`) and fails fast at startup if any of these files are missing — see the backend repo's [Startup validation](https://github.com/MateenA100/CricSelect-XAI-Backend#startup-validation). Cloning this repo as a folder named exactly `data`, alongside a clone of the backend repo, is what makes `python app.py` actually work.

```bash
mkdir cricselect && cd cricselect
git clone https://github.com/MateenA100/CricSelect-XAI-Backend.git
git clone https://github.com/MateenA100/CricSelect-XAI.git Notebooks
git clone https://github.com/MateenA100/CricSelect-XAICricSelect-XAI-Data.git data

cd CricSelect-XAI-Backend
pip install -r requirements.txt
python app.py
```

## What's here

Every file below is documented — with its exact size and what it's for — in the backend repo's [Data dependencies](https://github.com/MateenA100/CricSelect-XAI-Backend#data-dependencies) table. Summary by category:

| Category | Files |
|---|---|
| Core player data | `player_master_final.csv`, `player_clusters2.csv` |
| K-Means (general + role-specific) | `kmeans2_representative_players.json`, `kmeans_{bat,bowl,ar}_labels.csv`, `kmeans_{bat,bowl,ar}_cluster_names.json` |
| KNN recommender | `knn_recommender.pkl` (17.3 MB — the largest file here), `knn_selected_design.json`, `knn_validation_metrics.json`, `knn_validation_coverage_report.json`, `knn_outcome_validation_final_2024_euclidean_supplementary.json`, `knn_stability_results.csv` |
| Frozen ILP teams + validation | `ilp_ft_selected_xi_2024.csv`, `ilp_ft_cross_league_xi_2024.csv`, `ilp_ft_team_summary_2024.csv`, `ilp_ft_2024_actual_vs_random.csv`, `ilp_ft_weight_sensitivity_summary.csv`, `ilp_ft_primary_forecast_pool_2024.csv`, `ilp_ft_frozen_selection_config.json`, `ilp_logistic_vs_ft_report.json` |
| Forecasting results | `final_2024_predictions.csv`, `final_2024_model_comparison.csv`, `final_2024_bootstrap_comparison.csv`, `final_model_selection_decision.json` |
| SHAP explanations | `shap_player_explanations.json`, `shap_validation_report.json` |
| Explanation wording | `llm_template_explanations_all711.json`, `llm_hybrid_30player_results.json` |

## Not here: the full dataset

This is the *deployment* slice only. The dissertation's complete pipeline output — raw five-league match data, every intermediate CSV from all 23 notebooks, EDA plots, model-development artifacts, the full K-Means/KNN/ILP/SHAP/LLM development trail — is **862 MB**, well past what a normal git repo (and GitHub's 100 MB per-file limit) can hold. It exists locally and is documented separately; it is not required to run the dashboard, only to audit development history or re-run the pipeline from raw data.

## Related repositories

| Repository | Role |
|---|---|
| [CricSelect-XAI-Backend](https://github.com/MateenA100/CricSelect-XAI-Backend) | The Flask API that loads these files |
| [CricSelect-XAI](https://github.com/MateenA100/CricSelect-XAI) | The 23-notebook pipeline that produced them |
| [CricSelect-XAI-Frontend](https://github.com/MateenA100/CricSelect-XAI-Frontend) | The dashboard UI |
