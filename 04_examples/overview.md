# Examples Overview

The `examples/workflows/testing_dem/` folder contains six workflows of increasing complexity. Each one is self-contained and builds on the concepts introduced by the previous ones. Working through them in order is the recommended path for understanding the full pipeline.

## Workflow summary

| # | Folder | Name | TNA | RefBlock | Frame | New concepts introduced |
|---|--------|------|:---:|:--------:|:-----:|------------------------|
| 000 | `000_threeblocks/` | Three Blocks | — | — | — | `WorkflowSession`, `BlockModel.from_boxes`, contact detection, solver comparison |
| 100 | `100_arch/` | Arch | — | — | — | `ArchTemplate`, `MohrCoulomb`, support displacement, min/max thrust |
| 200 | `200_floor_nostag_standard/` | Floor — no stagger | ✓ | — | — | TNA, `FloorFormDiagram`, `Pattern`, `RefMesh`, `FlatBarrelTemplate` |
| 300 | `300_floor_stag_standard/` | Floor — running stagger | ✓ | — | ✓ | Running bond, `StructuralGrid`, `Column`, `TieBeam`, `FloorModel` |
| 400 | `400_floor_stag_ridge/` | Floor — ridge voussoirs | ✓ | ✓ | ✓ | `RefBlock`, `RidgeVoussoir`, mixed-type `BlockModel`, `TrimModifier` |
| 500 | `500_floor_stag_carbcomn/` | Floor — CARBCOMN voussoirs | ✓ | ✓ | ✓ | `CarbcomnVoussoir`, cable channel geometry, full CARBCOMN pipeline |

## How to run an example

Each example is a folder of numbered Python scripts. Run them in order from a terminal with the `carbcomn-core` conda environment active:

```bash
conda activate carbcomn-core
cd examples/workflows/testing_dem/300_floor_stag_standard

python 300_init.py       # create session and write params
python 310_tna.py        # TNA form-finding
python 320_geometry.py   # block mesh generation
python 340_dem_model.py  # assemble BlockModel and detect contacts
python 341_dem_problem.py # solve DEM problem
python 342_dem_viz.py    # inspect results in viewer
python 350_model_grid.py # structural frame
python 351_model_floor.py # full floor assembly
```

Each script opens a viewer at the end showing the output of that step. Close the viewer to continue to the next script. The viewer step can be skipped by commenting out the `viewer.show()` call if running in batch.

## Session data location

Each example creates a `<name>.session/` directory alongside its scripts. This directory holds all intermediate data as JSON files and persists between runs. Deleting the `.session/` directory forces a clean run from scratch.
