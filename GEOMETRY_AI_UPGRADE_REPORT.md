# UrbanRay — Implemented Geometry-Aware AI Upgrade

**Edition:** 15 November 2026 | **Status:** Reproducible, simulator-based development and retrospective benchmark. **Not** independently validated urban-temperature prediction.

## Open this first

`outputs/geometry_ai_upgrade/dashboard/UrbanRay_AI_Explorer_OFFLINE.html`

The offline browser dashboard now contains the new geometry-aware model, two-model evaluation, controlled feature ablation, reflection-hotspot selection benchmarks, uncertainty/limitations, and the older **physics-only** Landsat observational diagnostic. It loads without external services. An adjacent preview PNG is provided.

### What's actually implemented

1. **Geometry-aware neural-free AI surrogate.** `src/geometry_features.py` builds a reusable sparse graph from the *known* triangle-center coordinates, orientation, area, and neighboring building IDs. The graph contributes five non-target, available-at-prediction features, including neighboring building density, neighbor rooftop height, geometric facing potential, and an approximate reflected-light proxy from contemporary direct/sky source flux and assumed albedo. It is a *cosine-distance heuristic*, **not** visibility/ray tracing or new ground truth. Training and inference use the **same implementation**, with tests that reject missing faces or changing geometry.
2. **Two independent training objectives.** `train_geometry_ai.py` trains and saves a nonnegative reflection-residual regression model chosen on validation MAE, plus a hotspot specialist trained with a log-reflection target and high-reflection weighting, chosen on validation top-20 reflected exposure. Candidate hyperparameters are chosen without consulting the old test labels; model output is constrained above the direct+sky floor and subject to the project's energy cap at integrated inference. Original model artifacts were kept unchanged.
3. **Decision benchmarking.** The specialist is ranked against the previous AI, updated regression, controlled no-geometry ablation and direct-exposure heuristic. The UI shows all relevant competitors and preserves the negative rooftop top-K selection result.
4. **Run-time and real-world honesty.** `run_physics_plus.py` uses the cached geometry graph for AI calls. An actual, two-hour **SYNTHETIC WEATHER SMOKE TEST** compares the integrated AI and original sparse physics solver under the same inputs. `observational_preflight.py` rejects unsupported observational claims when the necessary genuine-weather three-method face run is absent. A real-world comparator already in this repository, `validate_ai_landsat.py`, remains available but **was not executed with new genuine-weather outputs**.
5. **Testing and packaging.** Additional tests cover geometry-feature training/inference parity, model transforms, missing neighbor inputs, observed-data preflight and the reflection-baseline ranking behavior. No original datasets or models were overwritten.

## Measured results — simulator labels only

Historical benchmark: **20,976 correlated face-hours** from *2025-06-23*, one area of NYC, two roof-albedo scenarios, spatially excluded from model training. Development/validation dates are 2025-06-20 through 2025-06-22. This is an **old, previously inspected test**, NOT a fresh untouched lockbox, and it is not an independent satellite or sensor experiment. The labels are approximate multibounce-physics **absorbed shortwave radiation (W/m²)**.

| Prediction method | MAE W/m² (lower better) | MAE W/m² on highest 10% reflected-exposure rows | Simulated top-20 wall reflection exposure captured |
|---|---:|---:|---:|
| No-reflection physics | 7.898 | 36.370 | 11.46% (direct-exposure ranking rule) |
| Earlier released reflection-residual AI | 3.635 | 16.604 | 81.14% |
| **Controlled ablation:** same fitting rows/settings, without neighborhood features | 3.691 | 16.633 | 76.81% |
| **Geometry-aware regression** | **3.081** | **13.095** | 79.93% |
| **Dedicated hotspot AI** | **3.022** | **12.737** | **85.88%** |

**Interpretation:** The geometry-aware regression improves MAE by ~15.2% compared with the earlier residual AI comparator on this *reused simulator test*. A like-for-like ablation with the same fit rows and hyperparameters yields 3.691 vs 3.081 W/m², supporting the value of the added geometry features in this experiment. The hotspot specialist captures 85.88% vs the baseline residual model's 81.14% of the physics-reference top-20 wall exposure. The new regression model alone slightly **decreases** top-20 exposure captured compared with the previous model (79.93% vs 81.14%); we preserve this negative result. The specialist was selected **only on June 22 validation rankings**, not optimized against the June 23 metric.

Top-K target: the **sum of per-patch additional reflected solar irradiation, Wh/m²/day**, for 314 spatially withheld wall patches on one day, with equal patch slots. This is *not* a count of 314 independent physical experiments and not an area-weighted cooling benefit. Results can change with neighborhood, time, weather, and wall materials.

The upgrade does **not** improve selection of top 3/5/10 held-out roof portions in the existing homogeneous roof-albedo counterfactual. All 3 methods choose the same top portions; apparent roof savings are modeled **absorbed solar heat**, never electrical savings or measured cooling.

## Honest runtime result — no demonstrated speedup

The integrated `compare` run on the **bundled two-hour synthetic smoke weather**, using one view-factor ray per face, one shadow sample, and two bounces, measured:

| Method | Average per-call SW compute time |
|---|---:|
| Geometry-aware regression AI (features + prediction) | 18.30 ms |
| Existing sparse shortwave physics *using already-built operator* | 0.22 ms |
| No-reflection rule | 0.018 ms |

This limited, warm-cache, four-application microbenchmark shows the new model is **slower**, not faster, than the existing efficient sparse shortwave step. The expensive shared view-factor construction is excluded from per-call timings. The runtime comparison is *not* a production speed claim. Source: `outputs/geometry_ai_upgrade/SMOKE_ONLY_2H_COMPARISON/thermal_hourly_summary.csv`.

## Real-world validation: explicitly incomplete

The archive includes independently sourced, quality-screened Landsat pixel exports and a separate earlier **physics-only** surface-temperature-proxy diagnostic. This update has **zero new paired AI-vs-Landsat results** because it does not contain an authenticated, time-matched, warmed-up, genuine historical-weather `compare` face simulation. The preflight report is `outputs/geometry_ai_upgrade/observational_preflight.json`, with status `NOT_READY_NO_NEW_AI_OBSERVATIONAL_VALIDATION`.

When new genuine archived-weather comparisons are available, `validate_ai_landsat.py` can score all three methods on exactly matched scenes after quality, spatial, temporal and weather-provenance checks. This requires acquiring and verifying the weather; a user-provided CSV alone is not proof of authentic observations. Landsat LST is a mixed-pixel surface temperature at coarse thermal support, not thermometer-grade truth for an individual roof or a verified cool-roof intervention.

## Reproduce locally

From the project root:

```bash
python -m pip install -r requirements.txt
PYTHONPATH=src python -m pytest -q tests
python train_geometry_ai.py
python build_ai_dashboard.py \
  --model outputs/geometry_ai_upgrade/regression_surrogate.joblib \
  --heldout outputs/geometry_ai_upgrade/retrospective_heldout_predictions_SIMULATED.csv \
  --output outputs/geometry_ai_upgrade/dashboard
python observational_preflight.py
python verify_checksums.py
```

On Windows PowerShell, omit `PYTHONPATH=src` and run `python -m pytest -q tests`; the tests insert the needed paths themselves. `verify_checksums.py` verifies the delivered snapshot. **Retraining/regenerating the dashboard will legitimately change some files**, so the release checksum snapshot must then be regenerated intentionally; it is not a permanent hash for all future outputs.

For a synthetic sanity test of end-to-end physics/AI/temperature integration only:

```bash
python run_physics_plus.py \
  --weather-csv outputs/ai_upgrade/SMOKE_ONLY_daylight_2hour_weather.csv \
  --output outputs/geometry_ai_upgrade/SMOKE_ONLY_2H_COMPARISON \
  --radiation-mode compare \
  --ai-model outputs/geometry_ai_upgrade/regression_surrogate.joblib \
  --rays 1 --shadow-samples 1 --warmup-hours 0 --hours 2 --bounces 2 --no-face-csv
```

Never submit that synthetic smoke as independent observational evidence. For the real-weather observational test, follow `REAL_WORLD_UPGRADE_REPORT.md` and the weather/satellite provenance requirements; ensure the generated face CSV includes all three methods and at least 24h genuine warmup.

## Files added or changed

- **Added:** `src/geometry_features.py`, `train_geometry_ai.py`, `observational_preflight.py`, `tests/test_geometry_ai_upgrade.py`, `GEOMETRY_AI_UPGRADE_REPORT.md`, `outputs/geometry_ai_upgrade/**`.
- **Updated:** `src/ai_inference.py` (compatible original + new bundle schemas), `run_physics_plus.py` (reuse graph), `build_ai_dashboard.py` (new metrics/features), `reflection_hotspot_audit.py` (more method comparators), `dashboard_template.html` (new model table/chart), `README.md` (new entry point), and checksums.
- **Preserved:** the original 126 MB labeled physics face CSV; original surrogate models and audit; the existing independently sourced Landsat exports and older studies.

## Scientific and competition restrictions

The model can mimic its own training simulator with lower errors and make better *simulator-relative* reflection-hotspot choices. It has not established accurate heat stress, building temperature, exposure health risks, or measured intervention benefits. A neighborhood-plus-date set evaluated after previous result inspection is not a new independent confirmation. A geographically different region and seasons, verified roof properties and independent observations would be needed to support generalization claims.

We built and evaluated the UrbanRay geometry-aware models during November 14–15, 2026. The data sources and software dependencies are documented in this repository.
