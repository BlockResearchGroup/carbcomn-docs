# Solver Overview

The DEM problem in `carbcomn.core` is solver-agnostic: the same `Problem` object can be handed to any of the available backends. The choice of solver affects **accuracy, speed, and the size of model it can handle**, but not the problem definition or the format of the results.

## Available solvers

Four solvers are available. They are accessed through `compas_dem.problem.Solver`:

```python
from compas_dem.problem import Solver

lmgc90 = Solver.LMGC90()
cra    = Solver.CRA()
rbe    = Solver.RBE()
# 3dec = Solver.3DEC()   # requires commercial license
```

---

### LMGC90

**Type:** Non-smooth Contact Dynamics (NSCD)  
**License:** Open-source  
**Status:** Default solver; under active development in `compas_dem`

LMGC90 is a molecular-dynamics-inspired solver that handles contact and friction through an implicit time-stepping scheme. It is the most capable open-source DEM solver available in the pipeline:

- Handles large assemblies (hundreds to thousands of blocks)
- Supports rigid **and** deformable blocks (deformable not yet exposed in the CARBCOMN pipeline)
- Handles dynamic load cases (prescribed displacements, kinematic loading)
- Produces robust results even at near-collapse configurations

**Use for:** All standard analyses, large floor models, min/max thrust investigations, support displacement.

---

### CRA — Contact and Rigid-body Analysis

**Type:** Mathematical programming (linear programming / conic programming)  
**License:** Open-source (BRG)  
**Status:** Stable

CRA formulates the DEM equilibrium problem as a convex optimisation program and solves it exactly. It is precise and converges reliably for problems where a feasible equilibrium exists:

- Exact solution within numerical tolerance
- Limited to **small assemblies** (up to ~50–100 blocks) due to memory and solve time scaling
- No dynamic capabilities

**Use for:** Validation, small test cases (three-block, arch), solver comparison against LMGC90.

> **See also:** <!-- [PLACEHOLDER: BRG CRA publication reference] -->

---

### RBE — Rigid Body Equilibrium

**Type:** Direct equilibrium (force-based)  
**License:** Open-source (BRG)  
**Status:** Stable

RBE is a fast, approximate solver designed for rapid design iteration. It finds an equilibrium solution without the overhead of a full optimisation or dynamic simulation:

- Very fast — suitable for real-time design feedback
- Less precise than CRA, may produce qualitatively correct but quantitatively approximate force distributions
- Limited to moderately sized assemblies

**Use for:** Early design exploration, parameter sweeps where exact values are not critical.

---

### 3DEC

**Type:** Distinct Element Method (commercial)  
**License:** Commercial (Itasca)  
**Status:** Available via `Solver.3DEC()` with a valid license

3DEC is the industry-standard commercial DEM software. It is highly optimised, supports deformable blocks, joint constitutive models beyond Mohr-Coulomb, and has been validated against physical experiments extensively:

- Fastest and most robust for large, complex assemblies
- Requires an expensive commercial license
- Full feature parity with LMGC90 plus deformable block support

**Use for:** Final structural verification analyses where commercial-grade accuracy is required.

> **See also:** [Solver Comparison](solver_comparison.md)
