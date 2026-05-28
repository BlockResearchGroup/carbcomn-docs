# 300 — Floor, Running Stagger, Standard Blocks

**Session name:** `FloorStagStandardTest`\
**Folder:** `examples/workflows/testing_dem/300_floor_stag_standard/`

## Goal

Same floor geometry as `200`, but with **running-bond stagger** between vault columns and the vault embedded in a surrounding structural frame of columns and tie-beams. This is the first workflow to use the full `FloorModel` assembly.

![Floor Stagger 3D View](<../.gitbook/assets/Floor_3D (1).png>)

## Concepts introduced

* **Running bond stagger** (`stagger_type="running"`, `stagger_amount=0.2`) — each vault column offset by 20% of the voussoir length relative to its neighbour, improving structural interlocking
* **`StructuralGrid`** — parametric layout of columns and beams from floor plan dimensions
* **`Column` and `TieBeam` elements** — structural frame members with physical geometry and contact surfaces
* **`FloorModel`** — container for the combined vault + frame model with a unified contact graph
* **Grid model step (`350`)** — building and inspecting the structural frame independently before vault integration
* **Floor assembly step (`351`)** — merging vault blocks into the frame, resolving geometric intersections, recomputing global contacts

## Workflow steps

| Script               | Stage               | Description                                                              |
| -------------------- | ------------------- | ------------------------------------------------------------------------ |
| `300_init.py`        | X00 Init            | Parameters — same floor, `stagger_type="running"`, frame dimensions      |
| `310_tna.py`         | X10 TNA             | Same as `210`, `RefMesh` for 6 × 7.5 m barrel vault                      |
| `320_geometry.py`    | X20 Geometry        | `FlatBarrelTemplate` with `stagger_type="running"`, `stagger_amount=0.2` |
| `340_dem_model.py`   | X40 DEM Model       | Vault-only `BlockModel`                                                  |
| `341_dem_problem.py` | X41 DEM Problem     | Solve vault in isolation                                                 |
| `342_dem_results.py` | X42 Results         | Print reaction forces                                                    |
| `342_dem_viz.py`     | X42 Visualisation   | View vault DEM results                                                   |
| `350_model_grid.py`  | X50 Structural Grid | `StructuralGrid`, `Column`, `TieBeam`, frame contacts                    |
| `351_model_floor.py` | X51 Floor Assembly  | Merge vault into `FloorModel`, `TrimModifier`, global contacts           |

## What to observe

Comparing this workflow against `200` shows the effect of stagger on the inter-column contact pattern. With running bond, contacts between adjacent columns are staggered and provide more effective interlocking under differential loading.

After running `351_model_floor.py`, inspect the full `FloorModel` contact graph. The tie-beam–voussoir contacts at the support interfaces should be clearly visible. The `TrimModifier` ensures these contacts are clean polygons without spurious overlaps.
