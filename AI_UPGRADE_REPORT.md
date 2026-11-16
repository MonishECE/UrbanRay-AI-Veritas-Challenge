**NEW ADDENDUM:** The real-world decision and observational-integrity extension is documented in `REAL_WORLD_UPGRADE_REPORT.md`. Updated dashboard: modeled rooftop solar **heat** (not electricity), held-out reflection-hotspot selection (top-20 81.14% AI vs 11.46% direct-only rule, simulator-referenced), older physics-only Landsat evidence, and a future genuine AI thermal/Landsat comparison workflow. These are not claims of observed AI cooling or faster model inference.

---

# UrbanRay AI Upgrade — Technical Report

**Status:** Working research prototype. All AI performance results below are measured against an **approximate physics simulator**, not instruments on real roofs. 

**Development timeline:** We built UrbanRay during November 14–15, 2026. This report documents our implementation, data sources, model evaluation and limitations.

## 1. What is new

1. **Reflection-residual machine learning.** `train_reflection_residual.py` compares learning total multibounce absorption directly against learning the **extra radiation absorbed after reflections**. Six candidate hyperparameter/model configurations are ranked on a validation partition; the selected model is then fitted once on training+validation, without tuning to the locked test set. The preferred model is `outputs/ai_upgrade/reflection_residual/selected_surrogate.joblib`.
2. **Functional integration.** `src/ai_inference.py` builds the same features used in training and predicts absorption in W/m². `run_physics_plus.py` now accepts `--radiation-mode physics|ai|compare`. The comparison runs full reference radiation, AI radiation, and simple no-reflection radiation through **independent thermal state histories**, including separate warmups, without treating simulator labels as observations.
3. **AI-based screening recommendations.** Rankings compare AI-estimated dark vs reflective roof absorption using assumed albedos 0.18 and 0.58 on the same June 23, 2025 modeled weather. Ranked buildings are source NYC geometry IDs, not invented sites; the metric is **mean daylight reduction in modeled roof shortwave absorption (W/m²)**.
4. **Interactive dashboard.** `outputs/ai_upgrade/UrbanRay_AI_Explorer_OFFLINE.html` is a self-contained HTML application with a rotatable 3D mesh where WebGL is available and an interactive 2D roof map otherwise. Users can change modeled hour, scenario, prediction method, and selected building; the dashboard includes hourly curves, ranked roofs, and feature dependence.
5. **Interpretability, error and auditing.** Per-surface-category MAE/RMSE/p90 error are saved, along with permutation importance on sampled strictly held-out model records. All dashboard data is labeled as modeled/simulated. All inputs and results are reproducible from included CSVs without internet, and source-to-output hashes are recorded.

## 2. New locked test results

**Target:** multi-bounce shortwave absorption simulated by the reference physical model; units W/m².

| Method | Held-out MAE (W/m²) | Held-out RMSE (W/m²) | Held-out R² |
|---|---:|---:|---:|
| Mean-only | 74.424 | 129.063 | -0.0076 |
| No-reflection physical approximation | 7.898 | 14.351 | 0.9875 |
| Direct AI prediction | 4.144 | 8.248 | 0.9959 |
| **AI reflection-residual correction (chosen by validation)** | **3.635** | **7.591** | **0.9965** |

- The chosen model reduces MAE by **53.98%** relative to the no-reflection approximation and by **12.29%** compared with the direct AI candidate, all against simulator labels. No real-world cooling accuracy follows from these statistics.
- Test data: **20,976 face-time-scenario rows** from June 23, 2025 and withheld spatial blocks. Those are correlated patches/timepoints, **not 20,976 independent buildings or measurements**. Training and selection came from earlier dates and non-test blocks, as described in the saved split audit.
- Held-out category MAE (W/m²): roof **2.044**, wall **4.020**, ground **3.212**. 90th percentile absolute error overall **9.916 W/m²**; this is an observed residual statistic, **not a calibrated uncertainty interval**.
- Test split and hyperparameter records: `outputs/ai_upgrade/reflection_residual/{split_audit.json,candidate_selection_audit.json,candidate_heldout_predictions.csv}`. Per-category diagnostics: `outputs/ai_upgrade/ai_heldout_diagnostics.csv`.

## 3. Cooling-screening results and interpretation

The dashboard uses **88 real NYC building footprints** and simulated patch outputs for a complete, one-day case from June 23, 2025. The AI-inferred dark-vs-reflective solar-absorption savings ranking has **Spearman ρ = 0.998657** against the full simulation's building solar-absorption ranking. This agreement is expected because the AI learns the simulator; it does **not** validate observed changes after actual roof upgrades. Because most roofs have similar sun exposure and the same assumed materials, ranking differences can be small. This is **not a cost-effective retrofit ranking**, and no costs, existing roofing materials, vulnerabilities, or social-equity indicators are established.

The reference simulator includes an existing, **physics-only** hypothetical roof-temperature difference. Do not attribute this thermal result to independently observed roofs, or to the AI's end-to-end temperature prediction. The AI-assisted thermal pathway now exists in code but requires an authentic-weather rerun for quantitative end-to-end thermal evaluation.

## 4. Actual runtime finding: no speedup claim

A **synthetic-weather smoke test** was run for two daylight hours, 88 documented geometry buildings, one ray per face, two shortwave bounces and zero warmup. The AI pathway ran correctly and its thermal states remained independent of the physics and baseline paths.

- Small-test shortwave step wall-clock averages: full shortwave physics ≈ **0.255 ms**, AI feature construction + inference ≈ **13.222 ms**, no-reflection ≈ **0.016 ms**, per scenario-hour.
- **The AI was slower** than the existing efficient sparse shortwave operator in this test. These timings have substantial scope/temperature caveats: they exclude initial geometry, visibility-operator construction, and the coupled thermal solver, and the weather input was synthetic. They are not a representative production benchmark. No speedup or scalability claim is justified.
- See `outputs/ai_upgrade/SMOKE_ONLY_COMPARE/thermal_hourly_summary.csv` and its `audit.json`. This data is labeled **SMOKE_ONLY** and must not be described as observed environmental evidence.

## 5. Limits and risk controls

- Input building shapes and identifiers are real NYC municipal records, but geometry is simplified extruded roof/wall/ground patches. Roof material, albedo, emissivity, convective coefficients and other physical quantities are assumed.
- Physics is an imperfect surrogate for atmospheric/city-scale reality. Satellite thermal retrievals in the older archive assess an emission proxy, not true individual-roof temperature; the AI has **not** been trained on those satellite temperatures.
- Only two roof albedos were simulated in training. A continuous slider or extrapolated material optimizer has intentionally **not** been created.
- AI predictions are clipped nonnegative, forced to zero for zero input solar radiation, and aggregate-capped so absorbed W cannot exceed source W. This engineering safety guard **does not prove per-face energy conservation**.
- AI is not currently faster than the reference sparse physics. Its main contribution is learning an interpretable correction to the no-reflection baseline and supporting scenario screening.
- The model may not generalize to other neighborhoods, seasons, geometries, climates, unusual materials, weather regimes or roof-level temperatures. More truly independent cases are needed.
- `joblib` pickle-based model loading can execute malicious code. **Load only your own trusted trained model artifact**. This package retrains its selected model locally, using scikit-learn **1.8.0**; the version is checked when loading.
- User-provided original weather needed to reproduce the full historical thermal run is absent; reruns of the 10-date satellite validation cannot be claimed offline. The 2025-06-23 offline AI dashboard is reconstructible from the included face-level simulation CSV, which is available.

## 6. Commands

From the project root:

```bash
python -m pip install -r requirements.txt
PYTHONPATH=src python -m pytest -q tests
# Selected residual-vs-direct comparison; uses archived, simulated face labels
python train_reflection_residual.py --faces outputs/historical_ml/physics_training_faces.csv --out outputs/ai_upgrade/reflection_residual
# Rebuild the offline dashboard and machine-readable evidence files
python build_ai_dashboard.py
# This synthetic run validates SOFTWARE execution only
python run_physics_plus.py --weather-csv outputs/ai_upgrade/SMOKE_ONLY_daylight_2hour_weather.csv --output outputs/ai_upgrade/SMOKE_ONLY_COMPARE --rays 1 --shadow-samples 1 --warmup-hours 0 --hours 2 --bounces 2 --radiation-mode compare --no-face-csv
```

For a **real-weather comparison** first retrieve and verify genuine consecutive Open-Meteo historical weather (the original input is not bundled), then run:

```bash
python run_physics_plus.py --weather-csv path/to/verified_weather.csv --radiation-mode compare --rays 8 --warmup-hours 48 --hours 24 --bounces 3 --no-face-csv --output outputs/verified_comparison
```

Do **not** call a fresh run real-weather-verified until its provenance file hash passes the existing check. Computational cost depends on hardware and settings.

## 7. Data sources and software libraries

- NYC Open Data building footprints: <https://data.cityofnewyork.us/resource/5zhs-2jue.json>. Original geometry provenance and SHA-256 retained in package.
- Open-Meteo historical weather API: <https://open-meteo.com/en/docs/historical-weather-api>. The archived simulation relied on sourced gridded weather; raw original CSV is not included.
- Landsat Collection 2 surface temperature products / remote-sensing background: <https://www.usgs.gov/landsat-missions/landsat-collection-2-surface-temperature>; the older satellite validation remains separately qualified in `SCIENTIFIC_STATUS.md`.
- scikit-learn histogram-based gradient boosting, used to train models: <https://scikit-learn.org/stable/modules/ensemble.html#histogram-based-gradient-boosting>. Model persistence/security caveats: <https://scikit-learn.org/stable/model_persistence>.
- Plotly interactive/offline HTML: <https://plotly.com/python/interactive-html-export/>.
- Competition guidelines: <https://highschoolhackathon.org/briefs>.

**Reproducibility reminder:** retained scientific historical data are archived source outputs, not freshly measured city temperatures. An independent reader should reproduce the table from the held-out prediction CSV and inspect the split and selection audit before citing performance.
