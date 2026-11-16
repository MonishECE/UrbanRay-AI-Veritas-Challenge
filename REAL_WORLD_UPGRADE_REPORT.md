# UrbanRay — Practical Decision Advantage + Observational Integrity Upgrade

**Research status (15 Nov 2026):** A functional offline dashboard and decision tests, grounded in our 88-building NYC simulation. **NOT proven real-world temperature prediction. NOT an AI speedup.** All new decision accuracy figures compare against the same approximate physics simulator used for labels. We built this project during the November 14–15, 2026 event window.

## 1. What is now implemented

- **AI-estimated extra reflected solar heat:** a new map mode shows `AI residual above the no-reflection floor` next to the existing full-physics reflection mode, both in W/m². On the 3D map the important wall surfaces can be viewed; in 2D only rooftops are drawn. This makes the AI contribution explicit rather than showing nearly identical absolute-solar maps.
- **A new decision benchmark on previously locked holdout data:** `reflection_hotspot_audit.py` selects high-reflection wall patches and compares its selected *simulator-reference exposure* to a simple no-reflection/direct-exposure rule. It does not train or tune anything on test labels. Results and face-level exposure tables go to `outputs/real_world_upgrade/`.
- **A rooftop planning metric:** `build_ai_dashboard.py` now calculates the absorbed **solar heat avoided** under dark-vs-reflective assumptions, in **kWh heat per modeled day** from each patch's W/m², actual modeled patch area, and 24 hourly simulation samples. The dashboard can switch between total daily solar heat and daylight mean W/m² ranking. This is not a measured cooling or electricity-bill estimate.
- **A falsification check:** `decision_benchmark.py` checks whether AI changes top-K roof-selection choices versus the simpler method. The results are recorded even when AI makes **no difference**.
- **Observational honesty:** a new evidence panel plots the **older physics-only** Landsat-vs-emission-proxy comparison, explicitly displaying **zero direct AI-vs-observation validation**. `observational_evidence.py` reads the original 10-scene archive without re-labeling results.
- **Real-data evaluation path:** `validate_ai_landsat.py`, with a method-aware extension to `validate_landsat_pixels.py`, compares all three models to the **same** quality-screened scene-level Landsat observations after an authenticated, time-matched `--radiation-mode compare` thermal run. It rejects synthetic or unverified weather, missing warmups, and incomplete comparisons. **It has not been used to produce new AI-vs-real-observation results in this package.**

## 2. Demonstrated decision result — holdout simulator reference

One scenario (`dark_roof`), wall patches only, 24 hourly samples on the locked June 23, 2025 test date and withheld spatial blocks. The task is **find the K wall patches with greatest modeled extra reflected-sun exposure**. All values are sums of the reference exposure (Wh/m²/day per patch), not measured urban surface temperature.

| K wall patches | AI-recommended captures | Direct-exposure rule captures | AI overlap with oracle patches |
|---:|---:|---:|---:|
| 10 | 68.94% | 3.27% | 2 / 10 |
| 20 | 81.14% | 11.46% | 7 / 20 |
| 50 | 83.52% | 23.36% | 23 / 50 |

The population is **314 held-out wall patches**, not 314 independent buildings or experiments. Their locations and hours are correlated. The cheap rule ranks using direct-plus-sky exposure, and does *not* estimate reflections; it is a useful reflection-blind comparator but **not** an equal-cost trained ML competitor. AI did **not** show a measured runtime benefit over the full reference solver; the previous small synthetic run found it slower.

### Negative result: roof project prioritization

**21 buildings with held-out roof portions**, not necessarily their complete roofs, appear in the strict test split. When selecting the top 3, 5, or 10 held-out roof portions by modeled *total solar heat avoided*, AI, direct AI, and the no-reflection approximation all selected oracle-equivalent groups under the simulator. **No incremental AI selection advantage was found for those K values.** See `outputs/real_world_upgrade/locked_decision_topk_benchmark_SIMULATED.csv`.

This matters: the AI adds most value for **reflection-sensitive surface mapping**, not for simply choosing the biggest high-sun roofs under two uniform, hypothetical roof albedos.

## 3. Actual observed satellite data, not AI validation

The old physics study's independently sourced Landsat Collection 2 surface-temperature scenes are displayed on the dashboard:

- Five 2025 development scenes; five 2026 subsequent evaluation scenes.
- The 2026 **raw** scene-mean disagreement, *physics emission proxy vs Landsat LST*, is **3.526 °C MAE** (five scenes).
- A previous separate development-set bias correction was documented but is *not applied* to the new raw scatter plot.
- The 30 m Landsat ST grid cells reuse coarser thermal measurements and may be correlated. Mixed land cover, atmospheric/emissivity processing and unknown roof materials prevent interpreting this error as rooftop thermometer accuracy.
- **The new AI model has never been compared directly to these observations in this release.** The already archived comparison covers a physics-only thermal pathway. No cool-roof intervention effectiveness has been observed.

Sources: [USGS Landsat Collection 2 Surface Temperature](https://www.usgs.gov/landsat-missions/landsat-collection-2-surface-temperature), [USGS Collection 2 Known Issues](https://www.usgs.gov/landsat-missions/landsat-collection-2-known-issues), [Open-Meteo Historical Weather API](https://open-meteo.com/en/docs/historical-weather-api), and the package's original `outputs/multidate_validation/audit/AUDIT_REPORT.md`.

## 4. How to conduct the *first actual AI thermal observational comparison*

Requires internet access, CPU processing, and authentic historical reanalysis inputs. These steps **have not been run successfully as an observational experiment in the provided archive**. The following selects a real 2026 Landsat overpass that appears in the bundled Earth Engine export (June 5, 2026); use unmodified CSV and preserve raw provenance files. Commands run from the project root:

```bash
python -m pip install -r requirements.txt
python fetch_weather_range.py --start 2026-06-03 --end 2026-06-05 --out data/nyc_2026_06_03_05_REANALYSIS.csv
python run_physics_plus.py \
    --weather-csv data/nyc_2026_06_03_05_REANALYSIS.csv \
    --radiation-mode compare --rays 8 --shadow-samples 4 --bounces 3 \
    --warmup-hours 48 --hours 24 \
    --output outputs/real_world_upgrade/scene_2026_06_05
python validate_ai_landsat.py \
    --faces outputs/real_world_upgrade/scene_2026_06_05/physics_training_faces.csv \
    --pixels urbanray_qualified_landsat_pixels_2025_2026.csv \
    --out outputs/real_world_upgrade/land_sat_comparison_2026_06_05
```

Expected output: `PAIRED_VALIDATION_STATUS.json` and `paired_Landsat_scene_predictions_EXPLORATORY.csv`, *only if the weather is authentic and any scenes survive the conservative geometry, cloud, uncertainty and time filters*. If no Landsat scenes match, the program writes a truthful **NO_COMMON_SATELLITE_MATCHES** status; do not manufacture results.

For meaningful comparison across more independent satellite dates, repeat the same protocol on multiple acquisition days; retain the 2026 scenes as untouched evaluation cases and do not use their residuals to tune model parameters. For actual **roof-temperature** validation, add properly located rooftop infrared or contact-sensor readings and independently observed roof materials/geometry. This is a future study, not completed work.

## 5. Commands to reproduce this upgrade **offline**

```bash
python -m pytest -q tests
python decision_benchmark.py
python reflection_hotspot_audit.py
python build_ai_dashboard.py
```

Open: `outputs/ai_upgrade/UrbanRay_AI_Explorer_OFFLINE.html`. Everything displayed in it runs without internet. New test count: **46 passed** in the verified local environment. One automated browser interaction check using Chromium and the generated HTML content produced no page errors, and the practical and observational plots both rendered.

## 6. Critical limitations

1. The model is not established to transfer to other neighborhoods, seasons, materials, future climates, or instrument data. One spatial-block test date cannot establish field robustness.
2. **AI was previously slower than this efficient sparse shortwave solver** in the measured synthetic-weather smoke run. No acceleration claim should be made.
3. AI performance still measures simulator agreement. The top-K hotspot advantage is a **simulator-relative selection result**.
4. The rooftop `kWh` measure is **absorbed solar thermal energy**, not electricity, air-conditioning energy, carbon emissions, health impact, or retrofit return on investment. All roofs use uniform assumed albedo alternatives. The existing analysis cannot predict optimal spending.
5. Archived raw weather for earlier 10-date physics results is still absent. Fresh true-weather inputs are required to re-create those historical predictions.
6. The historical 131 MB face CSV exceeds GitHub's single-file default push limit. Host via Git LFS or an appropriate data archive, or provide a lightweight code-only release plus checksum and source acquisition instructions.
7. We developed UrbanRay during the November 14–15, 2026 event window. We document the underlying data sources and technical dependencies.

## 7. Data sources and software libraries

- NYC Open Data public building geometry; Open-Meteo archived gridded weather; USGS Landsat Collection 2 Level-2 ST, with source credits listed in this repository.
- Scikit-learn, NumPy, pandas, SciPy, Plotly, Shapely, PyProj, Numba and related dependencies; see `requirements.txt` and existing credits.
