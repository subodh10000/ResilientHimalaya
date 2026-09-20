# RootLedger — frontend handoff (what the backend can demo)
For the judged UI. This is what the **backend already produces** and what you can put on screen. No new science is required.
- API: `http://127.0.0.1:8000`
- Offline judging: `VITE_USE_CACHE=true` → read `/demo_cache/...` only (do **not** call `/optimize`)
- City query: `?city=koshi` | `bangalore` | `kathmandu`
- Header alternative: `X-RootLedger-City`
**Koshi is the judged story.** Bengaluru and Kathmandu are screening packs. They must not look like a second Koshi proof.
---
## Judge path (already in the UI)
Walk this in order:
1. **Noise** — how NOAA records were cleaned
2. **Tail** — 7.75-year headline + HMA negative control + ERA5 check
3. **Plant** — $2M nature-based portfolio, click a site
4. **Proof** — CSI 0.053, miss vs JRC, 2017 transfer, before/after
5. **Ask** — grounded Q&A
6. **Export** — Preventive Measures Plan (markdown + PDF)
If you add anything, add it around this path. Do not invent CSI, lives saved, or blame percentages.
---
## 1. Demo these (wired today)
### Rainfall tail — `GET /signal?city=koshi`
Show:
- **498 stations / 12,066 station-years** (High Mountain Asia scale)
- Headline: the early-period **100-year** daily rain depth now has a fitted recurrence of **7.75 years** (Nepal-adjacent, 1979–1999 vs 2000–2024)
- HMA-pooled GEV is the **negative control** — it did **not** identify a shift. Do **not** put 7.75 on all 498 stations
- Return-level curve with bootstrap **95% CI**
- Trend: **+5.229 mm/decade** (p = 0.00035)
- Landslide: held-out **rainfall classifier AUC 0.934**, n = 138. **Not** a Caine I–D threshold
- Lake growth rows when present (`lake_growth`)
Useful extra inside the same payload (not required on the first pass): `pot_gpd` (tail cross-check, not the headline), `provenance.headline_note`.
### Noise / cleaning — `GET /noise`
Station-year counts, missingness, 9999 rejection, QC flags.
Note: the Noise tab currently always loads the Koshi/HMA noise file, even if another city is selected. If you keep it that way, label it **HMA / Koshi noise**, not “this city’s stations.”
### Independent check — `GET /replication`
- ERA5-Land vs ISD: **partial** (about **16.72-year** vs **7.75-year**)
- Scatter, Pearson r, bias, station count
Show this on the Tail tab for Koshi. City packs reuse the same HMA replication file — do not present it as Bengaluru/Kathmandu-specific.
### Map cells — `GET /hazard`
GeoJSON cells with:
- `eal_people` — annual expected people-risk (choropleth)
- `population`, `landslide_prob`
- `equity_weight` / `low_income_score` when present (up to **1.5×** weight on flagged low-income cells)
Hover is enough. Do not call `eal_people` “lives.”
### Sites + portfolio — `GET /candidates` + `GET /plan`
- Pale dots = candidate intervention sites
- Coloured dots = selected preventive measures
- Types: wetland restore, floodplain restore, riverbank bio, vetiver, bamboo, afforestation
- Click a site: **what / why here / cost / modeled benefit / assumption / verification**
- KPIs: spend vs **$2M** budget, annual people-risk avoided, tCO₂ / 10 yr, income / yr
- Objective: `expected` vs `cvar` (worst 10% tail)
- Efficiency frontier (`plan.frontier`)
- Greedy vs knapsack upper bound (`plan.optimality.gap_pct`) — optional line of copy
**Budget slider / CVaR dropdown:** only live if `POST /optimize` succeeds. If the API is down or `VITE_USE_CACHE=true`, **disable the controls** and show the frozen cached plan. Do not pretend a slider move recomputed anything.
### Proof — `GET /backtest` + flood layers
Koshi frozen numbers:
| Claim | Value | Label |
|---|---|---|
| 27 Sep 2024 CSI | **0.053** | In-sample calibration, not independent validation |
| POD / FAR | 0.212 / 0.935 | Real and low / high |
| vs JRC seasonal water | model 0.053 vs JRC **0.067** | **We do not beat climatology. Report the miss.** |
| vs elevation baseline | 0.001 | We do beat this |
| 2017 transfer CSI | **0.088** | Frozen 2024 model, different valley, not same-valley validation |
| 2024 west/east holdout | **0.056** | Same storm, spatial split only |
Layers:
- `GET /flood_observed` — blue fill (2024)
- `GET /flood_modeled` — gold outline (2024)
- `GET /flood_observed_2017` / `/flood_modeled_2017` — Koshi only
- Skill vs scale: `backtest.skill_vs_scale` (and `validation.skill_vs_scale` for 2017)
Counterfactual (simulation):
- 77,057 → 54,174 people-exposure units (**29.7%** lower)
- **Not** unique people and **not** observed lives saved
- Map toggle: `GET /risk_before` vs `GET /risk_with_plan`
If CSI is `null`, show “unavailable.” **Never substitute 0.**
### Ask — `POST /ask`
```json
{ "question": "Why is the 100-year storm now a 7.75-year storm?", "city": "koshi" }
```
Answers are grounded in artifacts (Claude if a key is set, otherwise cache heuristics). Prefer questions about the rainfall tail, selected measure, CSI/proof, or counterfactual.
### Export — `GET /preventive-measures-plan` (+ `.pdf`)
Title: **Preventive Measures Plan**. Legacy aliases `/conceptnote` still work.
The Export tab can keep using the local cache PDF so hand-off works with no API.
---
## 2. Backend-ready, not on the UI yet
Use these if you want extra judge surface. Do not overclaim.
| Feature | Endpoint | What to show | Forbidden copy |
|---|---|---|---|
| Implementation roles | `GET /attribution` | Government / community / household **levers** (spend, modeled people-risk, source) | **No blame %.** `government_pct` etc. are stripped to `null` |
| Delivery scorecard | `GET /scorecard` | Ministry-style markdown | Same: no causal % |
| Citizen brief | `GET /citizenbrief` | Plain-language markdown | Screening-grade |
| POT/GPD | `/signal` → `pot_gpd` | Second tail model | Not the 7.75 headline |
| Equity | hazard `equity_weight` | “Flagged low-income cells carry up to 1.5× weight” | Not a poverty census |
| SMS | `POST /sms` | Booth gimmick only if Twilio is configured | Skip by default |
`web/src/api.ts` already has `markdownDoc("scorecard" | "citizenbrief")`. App does not render them yet.
---
## 3. City switcher
`GET /cities` lists packs. Built-in + `demo_cache/cities.json`.
| | Koshi / Madhesh | Bengaluru / Kathmandu |
|---|---|---|
| Rainfall | NOAA ISD GEV + **7.75** headline | ERA5-Land screening GEV at centroid |
| Flood CSI | 0.053 / 0.088 / holdout | **`null` — do not invent** |
| 2017 overlays | yes | no |
| ERA5 scatter | yes (HMA file) | not city-specific |
| Plan + map + candidates | yes | yes, screening NbS |
| Observed-flood counterfactual | 29.7% labeled simulation | often 0 / empty — no SAR scene |
**Spoken line for city switch:** “Same engine on public DEM / OSM / ERA5-Land. We do not invent CSI.”
Kathmandu rainfall trend can be **negative**. Do not hard-code a `+` in front of the slope.
---
## 4. Live vs frozen
| Mode | How | UI |
|---|---|---|
| Frozen (judging) | `VITE_USE_CACHE=true` or API down | Lock budget + CVaR. Show cached `$2M expected` plan |
| Live | API up, `POST /optimize` | Slider/objective may rerun the portfolio |
`POST /optimize` body:
```json
{ "budget": 2000000, "mode": "expected", "draws": 120, "city": "koshi" }
```
`mode` is `"expected"` or `"cvar"`.
If the response contains `error` / `data_status: "cached fallback"` (API still returns HTTP 200), treat it as **frozen**. Do not mark the optimizer live.
---
## 5. Copy rules (swear jar)
Use these strings. Judges will check.
- **“Annual expected people-risk avoided”** — not unique lives saved
- CSI **0.053** is **in-sample calibration** on the 2024 scene
- **2017 CSI 0.088** is a **frozen transfer**, not same-valley validation
- ERA5 **partially replicates**
- Carbon and income are **literature screening factors**, not a field trial
- Counterfactual is a **simulation**, not an observed outcome
- `households_benefiting` is **null** — do not invent a household count
- **No causal responsibility percentages**
- **No “validated portfolio” / “hydrodynamic twin” / “Whitebox HAND”** for the frozen CSI (it is a GLO-30 **local-min HAND proxy**)
---
## 6. Endpoint cheat sheet
```
GET  /health
GET  /cities
GET  /signal?city=
GET  /noise?city=
GET  /hazard?city=
GET  /candidates?city=
GET  /plan?city=
GET  /backtest?city=
GET  /replication?city=
GET  /flood_observed?city=
GET  /flood_modeled?city=
GET  /flood_observed_2017?city=          # Koshi
GET  /flood_modeled_2017?city=           # Koshi
GET  /risk_before?city=
GET  /risk_with_plan?city=
GET  /attribution?city=
GET  /preventive-measures-plan?city=
GET  /preventive-measures-plan.pdf?city=
GET  /scorecard
GET  /citizenbrief
POST /optimize   { budget, mode, draws, city }
```
Offline equivalents live under `/demo_cache/` (Koshi) and `/demo_cache/cities/<id>/` (other packs). GeoJSON names match the GET paths (`hazard.geojson`, `flood_observed.geojson`, …). Replication and noise for non-Koshi packs currently fall back to the Koshi/HMA files.
---
## 7. Existing UI map (so you don’t duplicate)
Already consumed in `web/src/App.tsx` / `HazardMap.tsx` / `api.ts`:
- signal, hazard, candidates, plan, backtest
- flood 2024 + 2017, risk before/with plan, replication
- concept note / preventive measures plan
- optimize + ask
- city list
Not rendered yet: attribution, scorecard, citizen brief, `pot_gpd`, SMS.
---
Questions on payload shape: `contracts/schemas.md`. Spoken demo lines: `DEMO_SCRIPT.md`.
