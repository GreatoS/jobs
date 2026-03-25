# Test on a small batch first:
  uv run python score_robotics.py --start 0 --end 5

  This produces scores_robotics.json — each occupation gets a 0-10 robotics score + 
  rationale.

  Step 5: Update build_site_data.py to include robotics scores

  In build_site_data.py, add loading the new scores and merging them:

  # Load robotics scores
  with open("scores_robotics.json") as f:
      robotics_list = json.load(f)
  robotics = {s["slug"]: s for s in robotics_list}

  Then in the merge loop, add two fields to the dict:

  robotics_score = robotics.get(slug, {})
  data.append({
      ...
      "exposure": score.get("exposure"),
      "exposure_rationale": score.get("rationale"),
      "robotics": robotics_score.get("exposure"),          # NEW
      "robotics_rationale": robotics_score.get("rationale"), # NEW
      ...
  })

  Step 6: Add the color layer in docs/index.html

  Three changes in the HTML/JS:

  a) Add a button in the layer selector:
  <button data-mode="robotics">Robotics Exposure</button>

  b) Add a color function (next to exposureColor):
  function roboticsColor(v, a) {
    if (v == null) return `rgba(200,200,200,${a})`;
    const t = v / 10;
    return `rgba(${Math.round(30 + 225*t)},${Math.round(180 -
  140*t)},${Math.round(30)},${a})`;
  }

  c) Add cases in the existing colorFor, metricLabel, and tooltip logic where you   
  see the pattern for "exposure":
  if (colorMode === "robotics") return roboticsColor(d.robotics, alpha);

  Step 7: Rebuild and serve

  uv run python build_site_data.py   # rebuild docs/data.json
  cd docs && python -m http.server 8000

  Open http://localhost:8000 — you'll have a new "Robotics Exposure" toggle
  alongside the existing layers.

  ---
  Total cost/time: The LLM scoring is the only step that costs money (~$0.50 for 342
   occupations with Gemini Flash). The whole pipeline takes about 10-15 minutes     
  end-to-end once you have the pages scraped.