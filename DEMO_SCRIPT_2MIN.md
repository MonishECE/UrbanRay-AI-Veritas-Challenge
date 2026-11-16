# UrbanRay revised 2-minute research demonstration — transparent decision benchmark

**0:00–0:20 | The decision.** Urban surfaces receive direct sunlight plus reflections from nearby buildings. UrbanRay asks where reflected-solar exposure is most concentrated, with modeling caveats.

**0:20–0:45 | What's actually AI?** In the **City radiation explorer**, switch to **AI-estimated reflected solar heat**. Compare to the **full-physics reflected solar heat** mode, or switch to **Selected method − full physics**. The AI predicts an extra reflected component; it does not learn real temperature sensors.

**0:45–1:10 | Does the model help a decision?** Scroll to the new **Does AI improve a real decision task?** panel. Locked withheld wall patches: top-20 AI captures **81.1%** of the simulator's top-20 reflected exposure; a direct-exposure rule captures **11.5%**. State clearly this is **against simulation, not measurement**. Note rooftop top-K selection showed no AI benefit.

**1:10–1:30 | Rooftop planning metric.** In the ranking dropdown, switch between **thermal kWh per modeled day** and **W/m²**. A larger roof can have more total simulated solar heat avoided even if its per-area flux benefit is lower. **Heat kWh is not electrical savings, validated cooling, or ROI.**

**1:30–1:50 | Real observations.** Show the physics-only Landsat evidence plot: five 2026 scenes, raw scene-mean proxy disagreement 3.53 °C. Say plainly that the current AI is **not validated against satellite temperatures**; the new paired evaluation command is ready for verified weather.

**1:50–2:00 | Limit and next experiment.** Geometry and albedo are simplified, the AI is not faster than reference physics in the measured small smoke test, and on-site temperatures remain unverified. Next run matched real-weather/AI thermal simulations and compare with independent satellite scenes and roof sensors.

**Competition rules:** Do not submit prior UrbanRay code as a new Veritas AI hackathon project without an eligibility confirmation. The event rules prohibit extensions of pre-existing projects.

---

## Archived earlier AI explorer script (retained below for reference)

# UrbanRay — Optional two-minute demo script

**Use only in an appropriate demonstration or if the project is genuinely eligible for the relevant event. Do not claim pre-event work was created during the hackathon.**

**0:00–0:20 — Why it matters**

"Cities want to understand which rooftops absorb the most sunlight and how reflective materials might change that. But direct-sun calculations can overlook light bouncing between neighboring buildings. UrbanRay uses real NYC footprint geometry and a small machine-learning correction to explore that gap."

**0:20–0:45 — Make the AI clear**

Open the browser dashboard and select **No-reflection baseline**. Then switch to **AI surrogate** and **Full physics reference** at the same hour. "The physics baseline directly estimates sunlight and diffuse sky absorption. Our AI learns the extra absorption associated with reflections. We trained it on outputs of a simplified physical simulator, not on rooftop sensor readings."

**0:45–1:10 — Quantitative evidence**

"In a test held out by date and spatial block, the baseline had 7.90 watts per square meter mean absolute error relative to the simulator. Direct AI had 4.14, while our reflection-correction AI achieved 3.63: about a 54 percent error reduction against the simpler baseline. We chose this approach based on validation, not test results."

**1:10–1:40 — A working decision tool**

Switch roof treatment from **dark** to **reflective**, drag the hourly slider, and click one building in the rooftop ranking. Point to the building's absorption trend and roof solar-reduction estimate. "Here we screen 88 documented buildings. The ranking is based on modeled daylight solar absorption changes, not proven electricity savings or an actual retrofit recommendation."

**1:40–2:00 — Scientific honesty**

"The main limitation is that our AI has learned from simulations, not measured roof temperatures. We also found no runtime advantage over the sparse reference physics at this scale. Our next steps are independent cross-season testing, actual-weather end-to-end comparisons, and rooftop observations."

## Demo preparation

- Open `outputs/ai_upgrade/UrbanRay_AI_Explorer_OFFLINE.html` locally in a current browser. It is self-contained and works without internet; 3D needs WebGL but an interactive 2D fallback is available.
- Record at your own speed; keep video **under two minutes** if using Veritas's optional format.
- Do **not** claim the physics-only modeled temperature decrease was AI-predicted or field verified.
- Explain each model, data split, architecture, and limitation without reading from a script if asked live.
