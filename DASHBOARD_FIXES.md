# UrbanRay offline dashboard fixes — 2026-11-15

## Scope
Only the dashboard user interface and generator template were changed. The archived 2-scenario × 24-hour × 3-method predictions, building-level roof series, model scores, and feature-importance records were preserved verbatim. Training data and trained model artifacts were not changed.

## Confirmed problem
The original full 3D mesh used a single fixed color range from 0–700 W/m², while all three methods have very similar large direct-sun absorption components. The maps therefore looked essentially identical even when the predictions were numerically different. The AI was trained to approximate full physics, so similar predictions are scientifically expected. The original 700 W/m² cap also clipped some source predictions (maximum in the archive: 770.64 W/m²).

## Fixes
- All absolute radiation maps use one fixed 0–800 W/m² scale (covers all values), for valid comparisons across methods, scenarios and hours.
- Added signed `selected method - full physics`, extra absorption from reflections (`physics - no-reflection`), and scenario difference (`dark - reflective`) map views, with visible numerical limits and clipping disclosures. The comparison color range uses a fixed archive-wide 98th percentile for each view, not per-frame adaptive re-normalization.
- Top-level display includes means of each method and relevant differences at the selected hour. Values are descriptive averages of all 2,082 mesh faces, not held-out errors.
- Building rankings are recalculated for the selected model. Physics and AI use original archived rankings; no-reflection uses daylight mean of the archived area-weighted building series. Daylight and ranking calculations checked against the original AI/physics ranking values (maximum discrepancy 0.003 W/m², from rounding).
- Added a three-method selected-building roof curve and a separate time-series of differences from full physics.
- Uses a real roof-triangle 2D canvas view instead of tiny scattered dots; click building markers and hover roof patches for exact values. 3D mesh retained as an optional mode and still uses `mesh3d` cell-based color intensities. 2D is the safer default.
- Fixed misleading labels, disabled controls that have no meaning for a view, and added building ID search and clearer scientific limitations.
- Improved responsive layout and rendering scheduler; retained the original dark UrbanRay aesthetic (not the Apple redesign).

## Testing
The standalone repaired explorer was run in headless Chromium in forced 2D mode at desktop and 390px mobile widths. Confirmed no JavaScript page errors, no mobile horizontal overflow, that all four comparison modes respond correctly, that physics minus physics is zero, that method selection loads distinct archived arrays and reranks buildings, and that building selection, time control and plots respond. All archived dataset fields were deep-compared against the user-uploaded HTML and are identical. 3D Plotly traces were checked with a mocked renderer (correct 2,082 cell intensities and triangle indexing); GPU-rendered WebGL could not be visually verified in this environment.

## Regeneration
`python build_ai_dashboard.py --output outputs/ai_upgrade`

This now reads `dashboard_template.html` from the project root. It otherwise preserves the original model/data pipeline. Rebuilding requires the dependencies and model/archived simulation inputs documented elsewhere in this project.

## Scientific limitations
Full physics is a simulator reference, not measured truth. AI MAE is measured on spatially held-out simulator labels, not rooftop temperature observations. The dashboard's per-hour differences are descriptive and should not be treated as independent held-out validation or field cooling effectiveness. No acceleration improvement is claimed.
