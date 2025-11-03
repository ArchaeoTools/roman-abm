# Conceptual Model – Roman Settlement Growth ABM

**Goal:** Explore how a Roman settlement’s built-up area grows over time as a result of **population change** driven by **immigration** and **natural increase**, with optional crowding-driven **emigration**.

## 1. Research question
To what extent can observed increases in settlement size be explained by population growth due to immigration and natural increase, assuming a constant average area per inhabitant?

## 2. Archaeological context (evidence → phenomenon → mechanism)
- **Evidence (examples):** expansion of built-up area (m²/ha), housing phases, densification indicators, street extensions.
- **Phenomenon:** settlement size (built area) increases/decreases over time.
- **Mechanism:** net population change (immigration + natural increase − emigration) drives built area, scaled by area-per-inhabitant.

## 3. Scope and assumptions
- Aggregate (single-settlement) model.
- Time advances in discrete **ticks** (e.g., 1 tick ≈ 1 year or 5 years).
- **Area per inhabitant** is constant within a run (can vary across runs).
- Immigration/emigration are simplified rates; crowding can increase emigration.
- No explicit economy, construction constraints or policy (it may be be added later).

## 4. Entities and state variables
- **Model (global state)**
  - `population` (float ≥ 0) – current inhabitants.
  - `settlementSize` (float ≥ 0, m² or ha) – built-up area.
  - `immigration` (float ≥ 0) – people entering this tick.
  - `emigration` (float ≥ 0) – people leaving this tick.
  - `tick` (int) – time step counter.

## 5. Parameters (inputs, fixed per run)
- `initial-population` (≥ 0)
- `natural-growth-rate` (e.g., 0.0–0.03 per tick)
- `baseline-immigration-rate` (people per tick or proportion of population)
- `baseline-emigration-rate` (people per tick or proportion)
- `area-per-inhabitant` (m²/person or ha/person)
- Optional ideas:
  - `crowding-threshold` (people or density)
  - `crowding-emigration-slope` (extra emigration per person over threshold)
- Run control:
  - `max-ticks` (e.g., 200)
  - `seed` (random seed, if randomness is added later)

> **Units note:** Probably there will be an option to either pick people/tick for flows OR proportions, model needs to stay consistent.

## 6. Process overview and scheduling (per tick)
1. **Compute flows**
   - `immigration ← baseline-immigration-rate`
   - `emigration ← baseline-emigration-rate`
   - If crowding is enabled:
     - `if population > crowding-threshold: emigration += crowding-emigration-slope * (population - crowding-threshold)`
   - **Clamp** `emigration` so it never exceeds `population`.

2. **Update population**
   - `population ← population + (natural-growth-rate * population) + immigration - emigration`
   - **Clamp** to `population ≥ 0`.

3. **Update settlement size**
   - `settlementSize ← area-per-inhabitant * population`

4. **Record outputs**
   - According to time series (CSV/plots): `tick, population, settlementSize, immigration, emigration`.

5. **Advance time**
   - `tick ← tick + 1`, stop at `max-ticks`.


## 7. Initialization
- `population ← initial-population`
- `settlementSize ← area-per-inhabitant * population`
- `immigration ← 0`, `emigration ← 0`
- `tick ← 0`

## 8. Outputs (for analysis/validation)
- Time series: `tick, population, settlementSize, immigration, emigration`
- Derived indicators (optional for now):
  - `density = population / settlementSize` (if consistent units)
  - growth rates (Δpopulation, ΔsettlementSize)

## 9. Experiments (baseline BehaviorSpace)
- **Baseline sweep:**
  - `baseline-immigration-rate ∈ {0, 1, 2, 3, 5}`
  - `natural-growth-rate ∈ {0.00, 0.01, 0.02}`
  - Fixed: `baseline-emigration-rate = 0` (will be probablyfirst), `area-per-inhabitant = 30 m²`
  - `max-ticks = 200`, repetitions = 5
- **Crowding feedback test:**
  - Enable crowding, vary `crowding-threshold` and `crowding-emigration-slope`.
- Save CSV to `results/`.

## 10. Validation and sanity checks
- **Sanity:** with `immigration=0`, `natural-growth-rate=0`, `emigration=0` → population constant.
- **Monotonicity:** with positive immigration and zero emigration → population and settlementSize increase.
- **Non-negativity:** population and settlementSize never negative.
- **Plausibility:** area and population in realistic Roman ranges.

