# Step 5 — DEM Problem & Solvers

**Script prefix:** `X41`  
**Session inputs:** `session["blockmodel"]`  
**Session outputs:** `session["blockmodel_results"]`

## Purpose

The DEM problem step defines the **mechanical problem** — material, contact law, boundary conditions, and loads — and runs the DEM solver. After solving, contact forces are stored on the model's graph edges and nodal displacements on the nodes.

## Code walkthrough

```python
from compas_dem.models import BlockModel
from compas_dem.problem import Problem, Solver
from compas_model.materials import Concrete

model: BlockModel = session["blockmodel"]

# Assign material (provides density for self-weight)
conc = Concrete.from_strength_class("C20")
model.add_material(conc)
model.assign_material(conc, elements=list(model.elements()))

# Define the problem
problem = Problem(model)
problem.add_contact_model("MohrCoulomb", phi=40, c=0)  # friction angle [°], cohesion [kPa]
problem.add_supports_from_model()                      # fix blocks with is_support=True

# Solve
solver = Solver.LMGC90()
problem.solve(solver)

session["blockmodel_results"] = model
session.sync()
```

## The contact model

The `MohrCoulomb` contact model encodes the no-tension and friction constraints described in the [DEM Theory section](../02_theory/discrete_element_method.md):

| Parameter | Description | Typical value |
|-----------|-------------|---------------|
| `phi` | Friction angle [degrees] | 30–40° for dry masonry |
| `c` | Cohesion [kPa] | 0 for dry joints; > 0 for mortared joints |

Setting `c=0` and `phi=40` represents a **dry joint** with moderate friction — appropriate for the CARBCOMN system where blocks are held in place by geometry and post-tension rather than mortar.

## Support conditions

`problem.add_supports_from_model()` reads the `is_support` flag on each block element and fixes those blocks kinematically. Support blocks do not move and act as the fixed boundary conditions for the rest of the assembly.

## Adding load cases

Beyond self-weight, the problem supports additional load cases:

```python
# Prescribed support displacement (e.g. to find minimum/maximum thrust state)
problem.add_support_displacement(support_index=0, displacement=[0.01, 0, 0])

# Point load on a specific block
problem.add_point_load(block_index=70, force=[0, 0, -145000])  # [N]
```

Support displacement load cases are used in example `100` (Arch) to drive the structure toward collapse and find the minimum/maximum thrust states.

## Choosing a solver

Multiple solver scripts at the same `X41` prefix level demonstrate the same problem solved with different solvers:

```python
# LMGC90 (default, recommended)
solver = Solver.LMGC90()

# CRA — Contact and Rigid-body Analysis (BRG, precise but limited to small assemblies)
solver = Solver.CRA()

# RBE — Rigid Body Equilibrium (BRG, fast but less precise; good for initial iteration)
solver = Solver.RBE()
```

See [Solver Overview](../06_solvers/overview.md) for a full comparison.

## Result data

After solving, results are stored on the model graph:

- **Edge data** — contact force vectors $(F_n, F_{t1}, F_{t2})$ at each contact interface
- **Node data** — displacement of each block after solving (rigid-body displacement if blocks have moved)

```python
for u, v, contact in model.graph.edges(data=True):
    fn  = contact["Fn"]    # normal force [N]
    ft1 = contact["Ft1"]   # tangential force component 1 [N]
    ft2 = contact["Ft2"]   # tangential force component 2 [N]
```

> **See also:** [Step 6 — DEM Visualisation is handled in example pages], [Solver Overview](../06_solvers/overview.md), [DEM Theory](../02_theory/discrete_element_method.md)
