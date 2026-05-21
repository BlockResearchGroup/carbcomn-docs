# Discrete Element Method (DEM)

## Overview

The **Discrete Element Method (DEM)** is a family of computational methods for analysing systems of rigid or deformable bodies that interact through contact. In the context of masonry structures, DEM treats each voussoir block as a separate rigid body and models the joints between blocks as contact interfaces subject to physical constraints: contacts can push but not pull (no tension), and tangential forces at contacts are limited by friction (Coulomb's law).

DEM is the third and final computational stage of the CARBCOMN pipeline. It takes the block geometry produced by TNA and geometry generation as input and answers the central structural question: **given the geometry and the loads, do equilibrium contact forces exist that satisfy the no-tension, friction-limited conditions?**

<!-- [PLACEHOLDER: Add primary DEM/masonry references — Cundall 1971, Lemos 2007, etc.] -->

## Rigid block assumption

In the current implementation, all blocks are treated as **rigid bodies**. This means:

- No deformation occurs within a block
- All structural behaviour is localised at the contacts between blocks
- The kinematics of the system are described entirely by the six rigid-body degrees of freedom of each block (three translations, three rotations)

The rigid block assumption is appropriate for the CARBCOMN system because the blocks are made from a high-strength composite material with stiffness orders of magnitude greater than the contact stiffness at joints. Under working loads, block deformation is negligible compared to joint deformation.

## Contact conditions

At every contact interface between two adjacent blocks, the contact forces must satisfy two conditions:

### 1. No-tension (Signorini) condition

Masonry joints cannot transmit tension. The normal contact force $F_n$ at any interface must be non-negative (compressive convention):

$$F_n \geq 0$$

If the equilibrium solution would require $F_n < 0$ (tension) at some interface, that contact opens and the blocks separate. A structure for which no equilibrium solution with $F_n \geq 0$ everywhere exists is **not in equilibrium** and would collapse.

### 2. Mohr-Coulomb friction condition

The tangential (shear) force at a contact is limited by Coulomb friction. The resultant tangential force $F_t = \sqrt{F_{t1}^2 + F_{t2}^2}$ must satisfy:

$$F_t \leq \mu \, F_n$$

where $\mu = \tan\phi$ is the coefficient of friction and $\phi$ is the friction angle. For dry masonry joints, typical values are $\phi = 30°$–$37°$ ($\mu \approx 0.58$–$0.75$).

Together, these two conditions define the **friction cone** at each contact: admissible contact forces lie within the cone

$$\mathcal{K} = \{ (F_n, F_{t1}, F_{t2}) \mid F_n \geq 0, \; F_{t1}^2 + F_{t2}^2 \leq \mu^2 F_n^2 \}$$

<!-- [IMAGE PLACEHOLDER: Friction cone diagram showing admissible contact force region] -->

## Equilibrium problem formulation

For a system of $N$ rigid blocks, the DEM equilibrium problem is:

**Given:** block geometries, contact interface geometries, applied loads $\mathbf{f}^{ext}$, friction angle $\phi$

**Find:** contact force vectors $\mathbf{f}_c^{(k)}$ at each contact $k$ such that:

1. **Static equilibrium** holds for every block $i$:
$$\sum_{k \in \mathcal{C}(i)} \mathbf{f}_c^{(k)} + \mathbf{f}_i^{ext} = \mathbf{0}$$
$$\sum_{k \in \mathcal{C}(i)} \mathbf{r}_c^{(k)} \times \mathbf{f}_c^{(k)} + \mathbf{m}_i^{ext} = \mathbf{0}$$
where $\mathcal{C}(i)$ is the set of contacts adjacent to block $i$ and $\mathbf{r}_c^{(k)}$ is the moment arm from the block centroid to the contact resultant point.

2. **Contact constraints** are satisfied at every contact:
$$F_n^{(k)} \geq 0, \quad F_t^{(k)} \leq \mu \, F_n^{(k)} \quad \forall k$$

This is a **constrained linear system** (linear equilibrium equations, linear inequality contact constraints). Different solvers approach it differently — see [Solver Overview](../06_solvers/overview.md).

## Contact detection and discretisation

Before the equilibrium problem can be solved, the contacts between blocks must be identified and their geometry discretised. In the CARBCOMN pipeline this is handled by `BlockModel.compute_contacts()`, which:

1. **Detects** pairs of adjacent blocks by testing for geometric proximity and shared face geometry
2. **Computes** the common interface polygon (the intersection of adjacent block faces)
3. **Discretises** each interface into a set of **contact points** — the locations at which contact forces are applied and resolved

The quality of the contact discretisation directly affects the accuracy of the force distribution. Face-to-face contacts produce polygon interfaces that are discretised into multiple contact points; edge-to-edge contacts produce degenerate (zero-area) interfaces that are handled separately.

<!-- [IMAGE PLACEHOLDER: Contact interface polygons between adjacent voussoir blocks — top view and perspective] -->

## Self-weight loading

The primary load case in all CARBCOMN examples is **self-weight** under gravity. For a block with density $\rho$ and volume $V$, the gravitational force is:

$$\mathbf{f}^{grav} = -\rho \, V \, g \, \hat{\mathbf{z}}$$

applied at the block centroid. The density is assigned via a material model (`Stone.from_predefined_material("LimeStone")` assigns standard limestone density). Self-weight is computed automatically from the block geometry and material.

## Load cases beyond self-weight

In addition to self-weight, the CARBCOMN pipeline supports the following load cases (demonstrated in the arch example, workflow `100`):

- **Support displacements** — prescribed translation of one or more support blocks, used to push a structure toward its minimum or maximum thrust state
- **Applied forces** — external point loads or distributed loads applied to specific blocks (planned for future implementation)

## Result quantities

After solving, the following quantities are available at each contact:

| Quantity | Symbol | Description |
|----------|--------|-------------|
| Normal force | $F_n$ | Compressive force perpendicular to the contact plane |
| Tangential forces | $F_{t1}, F_{t2}$ | Shear force components in the contact plane |
| Resultant force | $\mathbf{F}_c$ | Full 3D contact force vector |
| Contact location | $\mathbf{r}_c$ | Position of the force resultant within the interface |

These are visualised in the `DEMViewer` as force vectors scaled to their magnitude, and can be printed to the terminal for numerical inspection.

> **See also:** [Step 4 — DEM Model Assembly](../03_pipeline/dem_model_step.md), [Step 5 — DEM Problem & Solvers](../03_pipeline/dem_problem_step.md), [Solver Overview](../06_solvers/overview.md)

> **References:** <!-- [PLACEHOLDER: Cundall 1971; Lemos 2007; Gilbert & Melbourne 1994; Whiting et al. 2009] -->
