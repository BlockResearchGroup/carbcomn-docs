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

LMGC90 is an open-source contact mechanics framework developed at Université de Montpellier / CNRS, designed for problems where many bodies interact through contact and friction. It can handle granular media, masonry structures, jointed rock, and similar assemblies. Its main feature is the underlying Non-Smooth Contact Dynamics (NSCD) method<sup>[1]</sup><sup>[2]</sup><sup>[3]</sup>, in which Signorini's non-penetration condition is imposed as an exact mathematical constraint. The practical consequence is that LMGC90 takes large time steps and stays stable through violent impacts and persistent dense contact.

LMGC90 is broader than just NSCD: its general framework exposes all combinations of contact law (smooth or non-smooth) and integration scheme (explicit or implicit). For non-smooth contact, the integration scheme is selected smoothly through a θ-integrator — $$ \theta = 0 $$ is fully explicit, $$ \theta = 1 $$ is fully implicit. This is backed by an extensive catalog of about 25 interaction laws covering both smooth and non-smooth models. The framework supports rigid and FEM-deformable bodies in the same simulation, scales to tens of thousands of bodies, and is the most capable open-source contact solver in the CARBCOMN pipeline for masonry and dense block-assembly problems.

**Capabilities**:

- Handles large assemblies
- Supports rigid and deformable blocks (deformable not yet exposed in the CARBCOMN pipeline)
- Handles dynamic and quasi-static loading (prescribed forces, displacements, kinematic loading)

**Non-smooth simulations** are governed by three choices:

- Contact law — selects the physics (Signorini-Coulomb, CZM, etc.)
- θ value — sets the integration scheme between explicit and implicit
- Non-Linear Gauss-Seidel (NLGS) parameters — number of iterations and convergence tolerance for the contact solver

Result quality depends on the interaction law, the number of NLGS iterations (convergence of the contact problem at each step), and the time step.

**Use for:** Non-smooth contact simulations, large floor models, contact force investigation, support settlement studies, force-driven simulations. 

---

### 3DEC
**Type**: Distinct Element Method (DEM, explicit penalty-based)
**License**: Commercial (Itasca Consulting Group)
**Status**: Available via `Solver.3DEC()` with a valid license

3DEC (3 Dimensional Distinct Element Code) is Itasca's commercial implementation of the Distinct Element Method, originally developed by Peter Cundall. It is designed for problems where a system of blocky bodies interact through joints and contact surfaces. It is widely used in jointed rock mechanics, mining, tunneling, slope stability, rock caverns, and large masonry analysis. Its main feature is the underlying Distinct Element Method<sup>[4]</sup><sup>[5]</sup>, in which contact between blocks is modeled through penalty springs at the joint interfaces — each contact is represented by a normal stiffness $$ k_n $$, a shear stiffness $$ k_s $$, and a constitutive law governing slip and separation. 3DEC uses a velocity-Verlet **explicit integration** scheme.

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
- Joint stiffnesses ($$ k_n $$, $$ k_s $$) — set the penalty stiffness controlling penetration and shear response.
- Time step — bounded by the stability condition for the explicit integrator

Result quality depends on the joint constitutive model (which physics is captured at the interface), the choice of joint stiffnesses (which trade off interpenetration against numerical stability), and the time step (small enough for stability and accuracy, large enough for tractable run times).

**Use for**: Smooth contact simulations, dynamic loading of masonry (seismic, blast, impact), equilibrium tests and projects where commercial software with audit trails and technical support is expecte.

> **See also:** [Solver Comparison](solver_comparison.md)

---

### CRA — Contact and Rigid-body Analysis

**Type:** Limit analysis with coupled kinematics (Nonlinear programming)
**License:** Open-source (BRG)  
**Status:** Stable

CRA is a static stability analyzer developed by the Block Research Group at ETH Zürich, designed for rigorous limit analysis of rigid-block assemblies under static loading. It extends the Livesley-Whiting rigid-block equilibrium formulation with two additional nonlinear constraints — the Signorini complementarity condition (contact transmits compression only when surfaces are actually touching) and the Mohr-Coulomb friction law in its kinematically coupled form (friction forces aligned opposite to virtual sliding directions). CRA either returns a force distribution that is equilibrated and kinematically admissible, or proves the assembly cannot stand under the given loads.

Beyond stability assessment, CRA is also a design tool. Its penalty formulation extends the analysis to infeasible configurations by allowing tensile contact forces in regions that cannot be supported by compression and friction alone. The location and magnitude of these tensile forces tell the designer where the structure fails and how much additional support is needed to stabilize it — whether through added blocks, modified interface geometry, or local reinforcement. This makes CRA suitable for iterative stability-aware design workflows, where each modification is evaluated immediately and the designer is guided toward a structurally sound configuration.

CRA is implemented in Python using Pyomo and the IPOPT nonlinear solver, integrated into the COMPAS framework<sup>[6]</sup>.

**Capabilities:**

- Rigorous mathematical certificate of (in)feasibility — a true collapse criterion
- Correctly handles sharp wedges, concave interfaces, and curved 3D geometries
- Penalty formulation localizes unstable regions in infeasible assemblies
- Supports rigid blocks with frictional contact and no-tension joints

CRA analysis are mainly governed by the friction coefficient ($$ \mu $$) and the optimisation parameters.

Result quality depends on the geometric accuracy of the block assembly (contact normals, contact points, interface discretization), the friction coefficient, and the convergence behavior of the nonlinear solver. Computational cost scales rapidly with assembly size: small assemblies solve in seconds; a 399-block model like the Armadillo Vault takes around 40 minutes<sup>[7]</sup>.

**Use for**: Rigorous stability proofs of small-to-medium masonry assemblies, limit analysis of arches, vaults, and domes, and validation of dynamic solvers on benchmark cases.

---

### RBE — Rigid Body Equilibrium

**Type:** Equilibrium solver (Quadratic optimization)
**License:** Open-source (BRG)  
**Status:** Stable

RBE is a fast equilibrium analyzer developed by the Block Research Group, building on the limit analysis formulations of Livesley<sup>[8]</sup><sup>[9]</sup> and Whiting et al.<sup>[10]</sup><sup>[11]</sup> It frames the equilibrium of a rigid-block assembly as a quadratic or linear program with one mechanical parameter — the friction coefficient — and a penalty formulation that allows tensile forces on unstable interfaces, localizing where a structure fails.

RBE is purely force-based, which makes it very fast and well suited to interactive analysis, but it can return physically wrong solutions. For classical masonry structures (Arches, Domes) RBE is well validated.

**Capabilities:**

Very fast equilibrium analysis (interactive / real-time)
Penalty formulation localizes unstable regions
Only one mechanical parameter (friction coefficient)
Well-validated for classical masonry geometries. However, for complex 3D assemblies, stability verdicts from RBE must be verified before drawing structural conclusions.

**Known limitations:**
- Can incorrectly classify unstable assemblies as stable — RBE's pure force-based formulation admits equilibrated solutions that are not kinematically realizable, leading to false-positive stability verdicts on sharp wedges, non-planar interfaces, and certain 3D configurations<sup>[7]</sup>
- Static only



**Use for:** Interactive early-stage design exploration, classical arch / vault / dome analysis with planar interfaces, rapid parameter sweeps.

---

<sup>[1]</sup>: Moreau, J.J. (1988). Unilateral contact and dry friction in finite freedom dynamics. In *Non-Smooth Mechanics and Applications*, CISM Courses and Lectures 302. Springer, Vienna, pp. 1–82.
<sup>[2]</sup>: Jean, M. (1999). The non-smooth contact dynamics method. *Computer Methods in Applied Mechanics and Engineering*, 177(3–4), 235–257.
<sup>[3]</sup>: Dubois, F., Acary, V. & Jean, M. (2018). The Contact Dynamics method: A nonsmooth story. *Comptes Rendus Mécanique*, 346, 247–262.
<sup>[4]</sup>: Cundall, P.A. (1971). A computer model for simulating progressive, large-scale movements in blocky rock systems. *Proceedings of the Symposium of the International Society of Rock Mechanics*, Nancy, Vol. 1, Paper II-8.
<sup>[5]</sup>: Cundall, P.A. & Hart, R.D. (1992). Numerical modelling of discontinua. *Engineering Computations*, 9(2), 101–113.
<sup>[6]</sup>: Iannuzzo, A., Dell'Endice, A., Maia Avelino, R., Kao, G.T.-C., Van Mele, T. & Block, P. (2021). COMPAS Masonry: A computational framework for practical assessment of unreinforced masonry structures. *SAHC Symposium*, Barcelona.
<sup>[12]</sup>: Kao, G.T.-C., Iannuzzo, A., Coros, S., Van Mele, T. & Block, P. (2021). Understanding the rigid-block equilibrium method by way of mathematical programming. Proceedings of the Institution of Civil Engineers – Engineering and Computational Mechanics, 174(4), 178–192.
<sup>[7]</sup>: Kao, G.T.-C., Iannuzzo, A., Thomaszewski, B., Coros, S., Van Mele, T. & Block, P. (2022). Coupled Rigid-Block Analysis: Stability-Aware Design of Complex Discrete-Element Assemblies. *Computer-Aided Design*, 146, 103216.
<sup>[8]</sup>: Livesley, R.K. (1978). Limit analysis of structures formed from rigid blocks. *International Journal for Numerical Methods in Engineering*, 12(12), 1853–1871.
<sup>[9]</sup>: Livesley, R.K. (1992). A computational model for the limit analysis of three-dimensional masonry structures. *Meccanica*, 27(3), 161–172.
<sup>[10]</sup>: Whiting, E., Ochsendorf, J. & Durand, F. (2009). Procedural modeling of structurally-sound masonry buildings. *ACM Transactions on Graphics (SIGGRAPH Asia)*, 28(5), Article 112.
<sup>[11]</sup>: Whiting, E., Shin, H., Wang, R., Ochsendorf, J. & Durand, F. (2012). Structural optimization of 3D masonry buildings. *ACM Transactions on Graphics (SIGGRAPH Asia)*, 31(6), Article 159.
