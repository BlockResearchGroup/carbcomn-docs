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
**Status:** Under active development

LMGC90 is an open-source contact mechanics framework developed at Université de Montpellier / CNRS, designed for problems where many bodies interact through contact and friction. It can handle granular media, masonry structures, jointed rock, and similar assemblies. Its main feature is the underlying Non-Smooth Contact Dynamics (NSCD) method (Moreau, 1988; Jean, 1999), in which Signorini's non-penetration condition is imposed as an exact mathematical constraint. The practical consequence is that LMGC90 takes large time steps and stays stable through violent impacts and persistent dense contact.

LMGC90 is broader than just NSCD: its general framework exposes all combinations of contact law (smooth or non-smooth) and integration scheme (explicit or implicit). For non-smooth contact, the integration scheme is selected smoothly through a θ-integrator —$\theta = 0$ is fully explicit, $\theta = 1$ is fully implicit. This is backed by an extensive catalog of about 25 interaction laws covering both smooth and non-smooth models. The framework supports rigid and FEM-deformable bodies in the same simulation, scales to tens of thousands of bodies, and is the most capable open-source contact solver in the CARBCOMN pipeline for masonry and dense block-assembly problems.

**Capabilities**:

- Handles large assemblies
- Supports rigid and deformable blocks (deformable not yet exposed in the CARBCOMN pipeline)
- Handles dynamic and quasi-static loading (prescribed forces, displacements, kinematic loading)

**Non-smooth simulations** are governed by three choices:

- Contact law — selects the physics (Signorini-Coulomb, CZM, etc.)
- θ value — sets the integration scheme between explicit and implicit
- Non-Linear Gauss-Seidel (NLGS) parameters — number of iterations and convergence tolerance for the contact solver

Result quality depends on the interaction law, the number of NLGS iterations (convergence of the contact problem at each step), and the time step.

**Use for:** Non-smooth joint simulations, large floor models, contact force investigation, support settlement studies, force-driven simulations.

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
**Type**: Distinct Element Method (DEM, explicit penalty-based)
**License**: Commercial (Itasca Consulting Group)
**Status**: Available via `Solver.3DEC()` with a valid license

3DEC (3 Dimensional Distinct Element Code) is Itasca's commercial implementation of the Distinct Element Method, originally developed by Peter Cundall in the 1970s. It is designed for problems where a system of blocky bodies interact through joints and contact surfaces. It is widely used in jointed rock mechanics, mining, tunneling, slope stability, rock caverns, and large masonry analysis. Its main feature is the underlying Distinct Element Method (Cundall, 1971; Cundall & Hart, 1992), in which contact between blocks is modeled through penalty springs at the joint interfaces — each contact is represented by a normal stiffness k_n​, a shear stiffness ksk_s
ks​, and a constitutive law governing slip and separation. The practical consequence is that 3DEC resolves the full force-displacement history at every joint and integrates the equations of motion explicitly in time using a velocity-Verlet-style scheme.

3DEC's strength lies in its optimised algorithms, extensive catalog of joint constitutive models, such as Coulomb slip, continuously-yielding, Barton-Bandis, and other rock-specific laws, combined with fully deformable block support, where each block can be internally zoned with a FEM mesh and assigned a bulk constitutive model (elastic, Mohr-Coulomb, strain-softening, etc.).

The framework supports rigid and deformable blocks, handles seismic and blast loading, and is backed by decades of validated case studies. It is the industry-standard tool for problems where joint behavior is the dominant physics and where commercial-grade documentation, GUI workflows, and technical support are required.

**Capabilities**:

- Handles large blocky assemblies
- Supports rigid and fully deformable blocks 
- Handles dynamic and quasi-static loading(prescribed forces, displacements, seismic input)
- Rich library of joint constitutive models for rock and structural applications
- Mature pre- and post-processing GUI with commercial support

**Distinct Element simulations** are governed by three main choices:

- Joint constitutive model — selects the physics at the interface (Coulomb, Barton-Bandis, continuously-yielding, etc.)
- Joint stiffnesses ($k_n$​, $k_s$) — set the penalty stiffness controlling penetration and shear response.
- Time step — bounded by the stability condition for the explicit integrator

Result quality depends on the joint constitutive model (which physics is captured at the interface), the choice of joint stiffnesses (which trade off interpenetration against numerical stability), and the time step (small enough for stability and accuracy, large enough for tractable run times).

**Use for**: Smooth joint simulations, large masonry analyses, projects where commercial software with audit trails and technical support is expected (regulatory contexts, heritage assessments, structural certification), and dynamic loading scenarios for masonry (seismic, blast, impact).


> **See also:** [Solver Comparison](solver_comparison.md)
