## 15 Nov 2026 — Decision benefit and observational-integrity upgrade

- Added `reflection_hotspot_audit.py`, `decision_benchmark.py`, `observational_evidence.py` and `validate_ai_landsat.py`; refined method matching in `validate_landsat_pixels.py`.
- Rebuilt the offline HTML to show **AI reflection**, heat-vs-flux rooftop rankings, held-out decision quality and the properly labeled **physics-only** Landsat external cross-check.
- Added `outputs/real_world_upgrade/` evidence artifacts and preview; new `REAL_WORLD_UPGRADE_REPORT.md` and `ONE_PAGE_REAL_WORLD_BRIEF.md`; expanded tests to 46 passing.
- No AI runtime speedup or observed rooftop cooling effect established. Historical code and data remain clearly labeled pre-existing for Veritas rules.

---

# AI Upgrade Change Log — 2026-11-15

- **Added:** `src/ai_inference.py`: consistent feature encoding, residual/direct mode support, nighttime zeroing, nonnegative and aggregate-energy guards, audited timings and model bundle validation.
- **Changed:** `run_physics_plus.py`: `--radiation-mode physics|ai|compare` and `--ai-model`, independent per-method thermal histories and matched shortwave/thermal outputs. Default physics path remains supported.
- **Added:** `train_reflection_residual.py`: validation-selected direct-vs-residual model comparison, locked test scoring, serialized model with scikit-learn version stamp, and selection audit.
- **Added:** `build_ai_dashboard.py`: archived 2025-06-23 data reconstructions, source geometry checks, strict held-out diagnostic reporting, model-correction permutation importance, 88-building rankings, and self-contained, responsive 3D/2D HTML explorer.
- **Added:** New model/score/audit datasets in `outputs/ai_upgrade/` and a synthetic-only, two-hour software integration smoke run.
- **Added:** `tests/test_ai_upgrade.py` (5 tests). Full project: 34 tests passing at the time of packaging.
- **Changed:** `requirements.txt` pins scikit-learn 1.8.0 to match locally generated persisted models.
- **Added:** `AI_UPGRADE_REPORT.md`, `ONE_PAGE_AI_PROJECT_BRIEF.md`, `DEMO_SCRIPT_2MIN.md`, offline dashboard preview, and guidance in `README.md` / `SCIENTIFIC_STATUS.md`.

All original archived 10-date exploratory Landsat results are retained unchanged. Historical simulator labels have **not** been relabeled as observations.
- **Added:** `verify_checksums.py` and machine-readable `outputs/ai_upgrade/verification.json`. One-page standalone PDF brief is rendered and checked.
