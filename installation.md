# Installation

## Prerequisites

- **Anaconda or Miniconda** — the environment is managed with Conda. [Download here](https://docs.conda.io/en/latest/miniconda.html).
- **Git** — to clone the repository.
- Access to the private `CARBCOMN.core` GitHub repository (contact the Block Research Group at ETH Zurich).

## 1. Clone the repository

```bash
git clone https://github.com/blockresearchgroup/carbcomn.git
cd carbcomn.core
```

## 2. Create and activate the Conda environment

The repository ships with an `environment.yml` that pins all required dependencies:

```bash
conda env create -f environment.yml
conda activate carbcomn-core
```

This installs Python 3.10 and the full dependency stack including:

| Package | Role |
|---------|------|
| `compas >= 2.14` | Core geometry and data structures |
| `compas_viewer` | 3D interactive viewer |
| `compas_tna` | Thrust Network Analysis |
| `compas_dem >= 0.5.0` | Discrete Element Method models and solvers |
| `compas_lmgc90 >= 0.1.8` | LMGC90 solver bindings |
| `compas_cra` | CRA solver |
| `compas_rbe` | RBE solver |
| `compas_cgal == 0.9.4` | Computational geometry (contact detection) |
| `compas_model >= 0.9.3` | Block model data structures |
| `compas_occ >= 1.4` | OpenCASCADE geometry kernel |
| `numpy < 2.0` | Numerical arrays |

## 3. Verify the installation

Run the placeholder test to confirm the environment is working:

```bash
pytest tests/test_placeholder.py
```

## 4. Run your first example

```bash
cd examples/workflows/testing_dem/000_threeblocks
python 000_init.py
python 020_geometry.py
python 040_dem_model.py
python 041_dem_problem.py
python 044_SW_Viz.py
```

Each script opens a 3D viewer. Close the viewer window to proceed to the next script.

## Updating the environment

If the `environment.yml` or `requirements.txt` change (which happens frequently during active development), update your environment with:

```bash
conda activate carbcomn-core
conda env update -f environment.yml --prune
pip install -r requirements.txt -e .
```

## Troubleshooting

**`compas_lmgc90` installation fails:** LMGC90 bindings require a C++ build environment. On macOS, install Xcode Command Line Tools with `xcode-select --install`. On Linux, install `build-essential`.

**Viewer does not open:** Ensure you are running scripts from a desktop environment (not a headless server). The COMPAS Viewer requires an OpenGL-capable display.

**`KeyError` when reading from session:** The session data for the step you are trying to run has not been written yet — make sure all upstream scripts have been run in order.

> ⚠️ The repository is under active development. Dependency versions may change. If you encounter incompatibilities, check the latest `environment.yml` in the repository.
