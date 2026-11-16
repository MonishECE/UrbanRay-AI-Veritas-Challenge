# Independent UrbanRay Dashboard Audit — 2026-11-15

This note documents checks completed on the uploaded Dashboard Bugfix Complete ZIP. It does not certify field accuracy.

## Tests actually performed
- The uploaded ZIP passed CRC integrity checking, with 98 non-directory entries.
- The original consolidated package contained 63 files and passed 29 pytest tests. This version adds 35 files, retains all previous files and passes 38 pytest tests.
- A local clean extraction of the uploaded project passed `python -m pytest -q`: 38 passed.
- A local two-hour, synthetic-weather end-to-end smoke test of `run_physics_plus.py --radiation-mode compare` produced 12 method/scenario/hour results; physics, AI and no-reflection thermal histories were separate. Synthetic weather is NOT observed 2025 weather.
- Regenerating the HTML using the supplied builder and archived face CSV produced identical embedded geometry, prediction data and JavaScript; a generation-timing metadata field differed. The supplied HTML maps 2,082 triangular faces and 88 documented NYC buildings over 24 hours and two hypothetical albedo scenarios.
- Headless Chromium executed the HTML in its 2D fallback mode at 1280px desktop and 390px mobile widths without page errors. Interaction checks covered method switches, error/reflection/cooling modes, timeline, search and ranked-building selection. Mobile had no horizontal overflow.
- GPU/WebGL 3D display could NOT be reliably verified in this environment; a separate GPU-mode test did not complete. The presence of a 3D renderer and mesh arrays is verified, but actual 3D behavior on all user devices is not certified.

## Original-file issue and limited fix
The uploaded ZIP's CHECKSUMS_SHA256.json was stale for `build_ai_dashboard.py` and `outputs/ai_upgrade/UrbanRay_AI_Explorer_OFFLINE.html` (92/94 passed). This audited copy only refreshes the file hash manifest and adds this audit note; it does not change project code, trained models, data, or dashboard HTML. The refreshed manifest now covers every package file except the manifest itself.

## Scientific interpretation
- The newer reflection-residual AI has a saved simulator-held-out MAE of 3.635 W/m2 versus 7.898 W/m2 for the no-reflection simulator baseline, and 4.144 W/m2 for the direct AI model. These are *simulator-target* errors, not observed roof temperatures.
- The historical Landsat comparison remains the same exploratory ten-date comparison previously available; no additional field validation was added by this dashboard update.
- The dashboard is a self-contained, static 24-hour June 23, 2025 explorer. Changing controls selects stored predictions; it does not recompute new solar or thermal scenarios live.
- The new AI runtime was *slower* than the existing sparse-physics shortwave operator in the small synthetic smoke benchmark. No speedup is established.
- Model material assumptions, Landsat spatial resolution and the absence of measured roof retrofit outcomes preclude a claim of real-roof cooling accuracy.
