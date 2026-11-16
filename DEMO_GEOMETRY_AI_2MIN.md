# Two-minute demo — UrbanRay Geometry-Aware AI

**0:00–0:20 — Problem.** Urban reflections make it hard to identify highly illuminated walls using only direct sunlight. We want to rank where extra reflected sunlight is likely to land, based on known NYC geometry.

**0:20–0:45 — AI.** Our first model predicts the *additional absorbed shortwave radiation* beyond direct-plus-sky. We created five surrounding-building features and a specialist model for reflection-hotspot rankings. Show the two model scores in the new comparison table.

**0:45–1:20 — Evidence.** On a previously reviewed simulation holdout, the old AI had 3.63 W/m² MAE, the geometry-aware regression 3.08, and the specialist 3.02. The specialist captured 85.9% of top-20 simulated wall reflection exposure versus 81.1% from the old AI and 11.5% for the direct-exposure selection rule. Explain these are *simulator-relative*, not verified real-world measurements.

**1:20–1:40 — Demo.** Change the radiation map mode to "AI-estimated reflected solar heat" and switch between dark and reflective roof scenarios, then point out the specialist and other baselines in the lower chart. The maps show the regression model; the specialist appears in the ranked benchmark, not as an independently measured forecast.

**1:40–2:00 — Limits and next step.** No paired AI vs actual Landsat observational validation yet, and sparse physics is presently faster per hour. Next: held-out neighborhood/season tests, real-weather comparisons, and physical/observational constraints.

**Competition note:** This builds on earlier UrbanRay code. Verify the hackathon's originality policy with organizers; do not claim new-event eligibility if pre-existing code extensions are prohibited.
