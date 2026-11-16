# UrbanRay release verification — November 15, 2026

## Checks performed in the packaged source workspace

- **64 Python tests passed** (`PYTHONPATH=src python -m pytest -q tests`) after the scientific-integrity and reviewer-audit changes.
- `python competition_evidence_audit.py` completed successfully, checking reported five-date arithmetic, model SHA-256 fingerprints against the engine report, 88 building screening rows, 20 specialist hotspot selections, and distinct 24-hour arrays for the regression AI, hotspot specialist, physics, and no-reflection methods in each of two roof scenarios.
- **Headless Chromium rendered the full self-contained dashboard HTML** using in-memory page content, with no JavaScript page errors. The reported five-date AI/physics/baseline metrics appeared as 6.23/6.17/6.32 °C, the unified decision section displayed, and the **Reset to demo scene** button correctly selected specialist reflection viewing. Browser navigation to local files/localhost was restricted by the test environment; this is not a direct verification of double-clicking the file from a Windows desktop.
- **Synthetic-only end-to-end smoke simulation completed** (`run_physics_plus.py --radiation-mode compare_decision --rays 1 --shadow-samples 1 --warmup-hours 0 --hours 2 --bounces 2 --no-face-csv`). Output: **12 summary rows** (3 methods x 2 scenarios x 2 hours). The input weather intentionally lacks archived API provenance and **must not be treated as observed data or an accuracy benchmark**.
- The final ZIP archive's CRC and SHA-256 inventory are checked separately.

## Changed since the unchanged fallback submission

1. Stricter paired Landsat consistency validation: reject nonfinite prediction/observation coordinates, negative uncertainty, inconsistent per-method Landsat uncertainty, and report unpaired-cell exclusion counts.
2. Five-date **reported development** AI/physics/no-reflection comparison shown in the main dashboard, correctly distinguished from the separate physics-only 2026 satellite plot.
3. Accessible guided demo-reset control, concise reviewer-facing README, two-minute demo script, and updated one-page brief (MD and PDF).
4. Reproducible competition evidence auditor and five additional unit tests.
5. Clarified that full simulator or five-date raw-observation reproduction requires original archived inputs omitted from this curated ZIP.

## Unresolved scientific work (not hidden or claimed complete)

- Original 131 MB face-hour training archive and raw historical weather omitted from this curated package. Rebuilding all historical models or maps requires those source assets.
- The five-date 2025 paired Landsat results are transcribed from prior JSON summaries; their raw per-date validator records are not included, and independent replay is outstanding.
- No independent wall-reflection sensor measurements, field cooling tests, fully calibrated uncertainties, single-roof causal estimates, operational speedup, or cross-city transfer demonstration.
- The previously inspected test day is not an untouched new lockbox.
