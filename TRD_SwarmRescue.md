# TRD: Autonomous Drone Swarm — Search & Rescue Simulation
**Project:** Battlefield / Disaster Zone Search & Rescue  
**Stack:** Python, Mesa (ABM), PyGame  
**Builder:** Codex (task-by-task execution)  
**Owner:** Muhammad Majid Ali — COMSATS Wah Campus, ABM Course Project

---

## Problem Statement

A crashed aircraft is somewhere in a 5km² mountainous zone. 20 autonomous drones are deployed. There is no GPS, no central server, and no global map. Each drone can only communicate with neighbors within a fixed radio range. The swarm must locate the target as fast as possible — and self-reorganize if drones fail mid-mission.

---

## Simulation Goals

1. Model realistic swarm intelligence with stigmergy (pheromone trails)
2. Simulate drone failures and swarm recovery
3. Visualize the live search in real time using PyGame
4. Measure and display performance metrics (time to find target, drones lost, coverage %)
5. Produce exportable data logs for analysis

---

## Tech Stack

| Component | Tool |
|---|---|
| ABM Engine | Mesa (Python) |
| Real-time Visualization | PyGame |
| Data Export | CSV via Python |
| Language | Python 3.11+ |

---

## Project Structure (tell Codex to create this)

```
swarm-rescue/
├── agents/
│   └── drone.py
├── environment/
│   └── terrain.py
├── simulation/
│   └── model.py
├── visualization/
│   └── renderer.py
├── data/
│   └── logs/
├── config.py
├── main.py
└── requirements.txt
```

---

## Milestone 1 — Project Skeleton & Config

**What to build:**
- Create the full folder structure above
- Create `requirements.txt` with Mesa and PyGame listed
- Create `config.py` with all simulation parameters as constants:
  - Grid size: 100x100
  - Number of drones: 20
  - Drone radio range: 8 cells
  - Pheromone evaporation rate: 0.05 per tick
  - Drone failure probability per tick: 0.001
  - Max simulation ticks: 2000
  - Target count: 1
  - Visualization cell size: 8px

**Acceptance criteria:**  
Running `python main.py` starts without errors (even if screen is blank).

---

## Milestone 2 — Terrain & Environment

**What to build:**  
In `environment/terrain.py`:
- A 100x100 grid world
- Each cell has these properties:
  - `terrain_type`: one of `open`, `obstacle`, `target`
  - `pheromone_level`: float, starts at 0.0
  - `visited`: boolean
- Procedurally generate the map:
  - 15% of cells are obstacles (randomly placed clusters, not scattered pixels)
  - 1 target cell placed randomly, not inside an obstacle
  - All remaining cells are open
- Pheromone evaporation: every tick, all cell pheromone levels decay by the evaporation rate from config

**Acceptance criteria:**  
Print the grid to console showing `O` for open, `X` for obstacle, `T` for target. Obstacles form clusters, not random noise.

---

## Milestone 3 — Drone Agent

**What to build:**  
In `agents/drone.py`, create a `DroneAgent` class using Mesa's `Agent` base:

Each drone has these properties:
- `status`: one of `active`, `failed`, `returned`
- `position`: current (x, y) on grid
- `visited_cells`: set of cells this drone has scanned
- `found_target`: boolean
- `neighbor_range`: from config

Each drone does this every tick (in order):
1. Check if current cell is the target — if yes, set `found_target = True`, broadcast to all neighbors
2. Deposit pheromone on current cell
3. Look at all open neighbor cells within 1 step
4. Move to the neighbor with the **lowest pheromone level** (unexplored preference) — if tie, pick randomly
5. Roll failure check — if fails, set status to `failed` and stop moving
6. If a neighbor drone has `found_target = True`, this drone also sets `found_target = True`

**Acceptance criteria:**  
Run 100 ticks with print output showing drone positions changing each tick.

---

## Milestone 4 — Mesa Model (Simulation Core)

**What to build:**  
In `simulation/model.py`, create a `SwarmModel` class using Mesa's `Model` base:

- Initialize the terrain grid
- Place 20 drones at random open cells near the grid center (spawn zone: 10x10 center area)
- Each tick:
  1. Decay all pheromones on the terrain
  2. Step all active drones
  3. Check if any drone found the target
  4. Record tick data: tick number, drones active, drones failed, % cells visited, target found (yes/no)
- Expose a `run_step()` method that advances one tick
- Expose a `get_stats()` method returning current metrics dict
- Stop simulation automatically when target is found or max ticks reached

**Acceptance criteria:**  
Model runs 2000 ticks headlessly and prints final stats: ticks taken, drones lost, coverage %.

---

## Milestone 5 — PyGame Real-Time Visualization

**What to build:**  
In `visualization/renderer.py`, create a `Renderer` class:

- Open a PyGame window sized: grid_size × cell_size (100×8 = 800px)
- Each tick, draw the full grid:
  - White cell = open, unvisited
  - Light blue cell = visited by a drone
  - Dark gray cell = obstacle
  - Red cell = target (blinking after found)
  - Yellow dot on cell = active drone
  - Dark red dot = failed drone (stays visible where it died)
  - Green circle around a drone = it found the target
- Top bar HUD showing:
  - Tick count
  - Drones active / total
  - Coverage % (cells visited / total open cells)
  - Status: SEARCHING or TARGET FOUND
- Simulation runs at 10 ticks per second (controllable via config)

**Acceptance criteria:**  
Window opens, drones visibly spread across map, pheromone trails cause drones to avoid already-visited areas, failed drones stop moving.

---

## Milestone 6 — Data Logging & Export

**What to build:**  
In `main.py` and `data/logs/`:

- Every tick, append a row to a CSV file: `data/logs/run_{timestamp}.csv`
- CSV columns: `tick, drones_active, drones_failed, cells_visited, coverage_pct, target_found`
- After simulation ends, print a final summary to console:
  - Total ticks to find target (or "NOT FOUND" if max ticks hit)
  - Drones lost
  - Final coverage %
  - CSV file path saved

**Acceptance criteria:**  
A valid CSV exists after every run. Opening it in Excel shows clean data with no missing rows.

---

## Milestone 7 — Failure Scenario & Recovery Behavior

**What to build:**  
Add to `agents/drone.py` and `simulation/model.py`:

- At tick 500, forcefully fail 5 random active drones (simulate a jamming event)
- Remaining drones detect the reduction in neighbor count and increase their exploration radius by 2 cells temporarily (for 100 ticks)
- Add a `JAMMING EVENT` marker row in the CSV log at tick 500
- Display a red flash on the PyGame HUD at tick 500 with text: `⚠ JAMMING EVENT — 5 DRONES LOST`

**Acceptance criteria:**  
At tick 500 the HUD flashes, 5 drones visibly stop, and surviving drones visibly spread wider.

---

## Milestone 8 — Final Polish & Defence-Ready Output

**What to build:**

- Add a `PAUSED` mode: press `SPACE` to pause/resume simulation
- Add a `SPEED` toggle: press `+` / `-` to increase or decrease ticks per second
- After simulation ends, freeze the final frame and display a results overlay:
  - "TARGET FOUND in X ticks" or "SIMULATION ENDED — TARGET NOT FOUND"
  - Drones active: X / 20
  - Coverage: X%
  - Press `R` to restart, `Q` to quit
- Add a `README.md` to the project with:
  - What the project is (2 sentences)
  - How to install dependencies
  - How to run
  - What each key does

**Acceptance criteria:**  
A person with no context can clone the repo, follow the README, and run the simulation in under 5 minutes.

---

## Definition of Done (Full Project)

- [ ] All 8 milestones pass their acceptance criteria
- [ ] Simulation runs without crashing for 5 consecutive full runs
- [ ] CSV logs are clean and importable
- [ ] PyGame window is smooth at 10 ticks/sec on a normal laptop
- [ ] Code is split across files as specified — no single-file spaghetti
- [ ] No hardcoded values — everything uses `config.py`

---

## What This Demonstrates (For Defence)

- Agent-Based Modeling with emergent collective behavior
- Stigmergy (indirect communication via environment)
- Fault tolerance and swarm self-organization
- Real-time simulation with live metrics
- Data collection and analysis pipeline
