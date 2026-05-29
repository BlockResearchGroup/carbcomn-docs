# Step 5 — DEM Problem & Solvers

**Script prefix:** `X41`\
**Session inputs:** `session["blockmodel"]`\
**Session outputs:** `session["blockmodel_results"]`

## Purpose

The DEM problem step defines the **mechanical problem** — material, contact law, boundary conditions, loads and solver parameters — and runs the DEM solver. After solving, contact forces are stored on the model's graph edges and nodal displacements on the nodes.

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

The `MohrCoulomb` contact model encodes the friction constraints described in the [DEM Theory section](../02_background/discrete_element_modelling.md):

| Parameter | Description               | Typical value                             |
| --------- | ------------------------- | ----------------------------------------- |
| `phi`     | Friction angle \[degrees] | 30–40° for dry masonry                    |
| `c`       | Cohesion \[kPa]           | 0 for dry joints; > 0 for mortared joints |

Setting `c=0` and `phi=35°` represents a **dry joint** with moderate friction — appropriate for the CARBCOMN system where blocks are held in place by geometry and post-tension rather than mortar.

## Support Conditions

`problem.add_supports_from_model()` reads the `is_support` flag on each block and fixes those blocks kinematically. Support blocks do not move and act as fixed boundary conditions for the rest of the assembly.

## Adding Load Cases

Beyond self-weight, the problem supports additional load cases:

```python
# Prescribed support displacement (e.g. to find minimum/maximum thrust state)
problem.add_support_displacement(block_index=0, displacement=[0.01, 0, 0])
# Prescribe Rotation on a specified block [rad]
problem.add_rotation(block_index=99, rotation = [0,0,0.5])
# Point load on a specific block [N]
problem.add_point_load(block_index=70, force=[0, 0, -145000])

# Point load with moment on a specific block [N] and [N·m]
problem.add_point_load(block_index=30, moment=[0, 0, -10000])
```

All vectors follow the global `[x, y, z]` convention.

Support displacement load cases are used in example `100` (Arch) to drive the structure toward its collapse limit and find the minimum and maximum thrust states.

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

## Post-Processing of Results

Post-processing happens inside COMPAS_DEM and is handled separately in each solver. Results are retrieved from the solver, unified into a common COMPAS data structure where they are stored on the model graph, and then visualised using the `DEMViewer`:

```python
from compas_dem.viewer import DEMViewer
viewer = DEMViewer(problem.model)
viewer.add_solution(scale=1.0)
viewer.show()
```

After solving, results are stored on the model graph:

* **Edge data** — contact force vectors $(F\_n, F\_{t1}, F\_{t2})$ at each contact interface
* **Node data** — displacement of each block after solving (rigid-body displacement if blocks have moved)

```python
for u, v, contact in model.graph.edges(data=True):
    fn  = contact["Fn"]    # normal force [N]
    ft1 = contact["Ft1"]   # tangential force component 1 [N]
    ft2 = contact["Ft2"]   # tangential force component 2 [N]
```

### Degenerate Contacts

Contacts that lose one or more contact points during analysis are considered degenerate. A face contact can degenerate into an edge contact, or an edge contact can become a point contact. These cases are flagged as **degenerate contacts** and appear in the `DEMViewer` under the `Degenerate_Contacts` group, which is useful for diagnosing post-peak behavior or hinge locations.

In the current implementation, only dynamic solvers can produce degenerate contacts.

### Accessing Data

Block displacement and contact data can be accessed directly from the graph attributes:

```python
graph = problem.model.graph
for node in graph.nodes():
    block_transformation = graph.node_attribute(node, "transformation")

for edge in graph.edges():
    gap = graph.edge_attribute(edge, "gap")
    magnitude = graph.edge_attribute(edge, "force_magnitude")
    print(f"Edge {edge} gap: {gap}, force magnitude: {magnitude}")
```

> **See also:** \[Step 6 — DEM Visualisation is handled in example pages], [Solver Overview](../06_solvers/overview.md), [DEM Theory](../02_background/discrete_element_modelling.md)
