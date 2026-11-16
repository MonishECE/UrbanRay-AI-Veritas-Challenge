# Two-minute demonstration — UrbanRay unified AI system

**0:00–0:20 Problem.** "Solar-reflective roofs may reduce absorbed sunlight on the roof, but an urban environment also redirects light between nearby facades. A naive sunlight-only ranking misses that context."

**0:20–0:40 Architecture.** "We use one shared geometry-and-weather input pipeline. Our original geometry-aware AI estimates absorbed solar shortwave radiation, and a separately trained specialist predicts the strongest additional reflection hotspots. One decision engine brings those two outputs together without replacing either model."

**0:40–1:10 Live app.** Open `outputs/unified_decision_engine/dashboard/UrbanRay_AI_Explorer_OFFLINE.html`. Select a building. Show the roof modeled heat saved and the neighboring-wall scenario context; explicitly explain this is an *all-roofs-changed scenario* and **not** the individual roof's causal effect. Choose **Hotspot specialist · modeled reflections** in the map selector.

**1:10–1:30 Results.** "Against an approximate radiation simulator on a previously inspected held-out date, the regression specialist gets 3.08 W/m² MAE, while the hotspot specialist captures 85.9% of the physics oracle's top-20 modeled wall reflection exposure. We preserve the negative rooftop selection result and do not present this as fresh real-world validation."

**1:30–2:00 Limitations / next experiment.** "This is a model-based *screening* tool, not a validated cooling forecast. Next we need independently observed facade/roof data, truly new locations and dates, and a single-roof intervention experiment to estimate causal impacts."

**Competition disclosure:** Confirm eligibility first: this project is an extension of earlier UrbanRay work, which may not comply with the event's code-creation rules.
