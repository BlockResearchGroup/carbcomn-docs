# Discrete Element Modelling (DEM)

## Overview

This stage covers the structural analysis of block assemblies. Such systems are handled by the family of **block-based methods (BBM)** — computational approaches for analysing assemblies of rigid bodies that interact only through their contacts. In masonry mechanics, BBMs descend from Heyman's idealisation of the stone skeleton[^heyman1966], in which masonry is treated as an assembly of rigid blocks obeying three assumptions: infinite compressive strength, no tensile capacity at joints, and no sliding failure (later relaxed to a Coulomb friction limit).

Within this family, methods divide broadly into two branches:

- **Static methods** solve for equilibrium or collapse states directly, typically as a mathematical optimisation problem. Representative methods include Thrust Network Analysis (TNA)[^block2007], Coupled Rigid Block Analysis (CRA)[^kao2022][^iannuzzo2021], the Piecewise Rigid Displacement (PRD) method[^iannuzzo2020], and Rigid Block Equilibrium[^kao2021].
- **Dynamic methods** integrate the equations of motion of the blocks forward in time, resolving contact forces at each step. The two main representatives are Contact Dynamics (CD)[^moreau1988][^jean1999][^dubois2018] and the Distinct/Discrete Element Method[^cundall1971][^cundall1979].

Within the CARBCOMN pipeline, DEM is the third and final computational stage. It takes the block geometry produced by TNA and geometry generation as input and answers the central structural question: **given the geometry and the loads, do equilibrium contact forces exist that satisfy the no-tension, friction-limited conditions?**

## Geometry 
**The rigid block assumption**: Geometries are currently modelled as rigid blocks. This means:

- No deformation occurs within a block.
- All structural behaviour is localised at the contacts between blocks.
- The kinematics of the system are described entirely by the six rigid-body degrees of freedom of each block (three translations, three rotations).

The rigid block assumption is appropriate for early-stage assessment of the CARBCOMN system: under working loads, block deformation is negligible compared to joint deformation, and the principal concern at this stage of the design process is global stability. Block-level stresses can be checked later using a finite element continuum submodel.


### Block equilibrium

Independently of the solver, every BBM enforces the same physical principle: each block satisfies the Newton–Euler equations under the combined action of external loads (gravity, applied forces) and the contact forces transmitted by its neighbours. For a block $$ i $$ of mass $$ m_i $$ and inertia tensor $$ \mathbf{I}_i $$:

$$
m_i \ddot{\mathbf{x}}_i
= \mathbf{F}_i^{\text{ext}} + \sum_{c \in \mathcal{C}_i} \mathbf{F}_i^{c}
$$

$$
\mathbf{I}_i \dot{\boldsymbol{\omega}}_i
+ \boldsymbol{\omega}_i \times \mathbf{I}_i \boldsymbol{\omega}_i
= \mathbf{M}_i^{\text{ext}}
+ \sum_{c \in \mathcal{C}_i} \mathbf{M}_i^{c}
$$

where $$ \mathcal{C}_i $$ is the set of contacts shared with neighbouring blocks, $$ \mathbf{F}_i^{c} $$ and $$ \mathbf{M}_i^{c} $$ are the force and moment transmitted across contact $$ c $$, and $$ \mathbf{x}_i $$, $$ \boldsymbol{\omega}_i $$ are the position of the centroid and the angular velocity of the block.

The contact forces $$ \mathbf{F}_i^c $$ come in action–reaction pairs, where contact $$ c $$ is shared between exactly two blocks, so Newton's third law requires that if block $$ i $$ receives $$ \mathbf{F}_i^c $$, its neighbour $$ j $$ receives $$ \mathbf{F}_j^c = -\mathbf{F}_i^c $$ and the corresponding moment about its own centroid. Beyond this pairing, each contact force is physically constrained: it must be compressive (no tension across the joint), and its tangential component cannot exceed the friction limit. These constraints are formalised in the next section and, together with the Newton–Euler equations above, fully determine the problem.

Equilibrium of the assembly thus reduces to finding contact forces $$ \{\mathbf{F}_i^c\} $$ and block configurations $$ \{\mathbf{x}_i, \boldsymbol{\theta}_i\} $$ that simultaneously satisfy:

1. **Newton–Euler equations** on every block (force and moment balance);
2. **Newton's third law** at every contact (equal and opposite forces);
3. **Contact admissibility** at every contact (no-tension, friction cone).

## Contact conditions

At the heart of every BBM lies a contact detection algorithm that identifies the shared faces between adjacent blocks – the same faces where the structural response is resolved. Whether stated explicitly or enforced implicitly through the solver, two contact conditions must hold at each interface:

### 1. No-tension (Signorini) condition

Masonry joints cannot transmit tension. The normal contact force $$ F_n $$ at any interface must be non-negative (compressive convention):

$$F_n \geq 0$$

If the equilibrium solution would require $$ F_n < 0 $$ at some interface, that contact opens and the blocks separate. A structure for which no equilibrium solution with $$ F_n \geq 0 $$ everywhere exists is **not in equilibrium** and would collapse.

### 2. Mohr–Coulomb friction condition

The tangential (shear) force at a contact is bounded by Coulomb friction. The resultant tangential force $$ F_t = \sqrt{F_{t1}^2 + F_{t2}^2} $$ must satisfy

$$F_t \leq c\,A + \mu F_n,$$

where $$ c $$ is the cohesion, $$ A $$ the contact area, and $$ \mu = \tan\phi $$ the friction coefficient (with $$ \phi $$ the friction angle). Typical dry-masonry values are $$ \phi = 28 $$–$$ 35° $$; for mortarless joints, $$ c = 0 $$.

Together, the no-tension and friction conditions define the **friction cone** at each contact: admissible contact forces lie within

$$\mathcal{K} = \{ (F_n, F_{t1}, F_{t2}) \mid F_n \geq 0,\;
F_{t1}^2 + F_{t2}^2 \leq \mu^2 F_n^2 \}$$

### Contact parameters

The exact parameter set depends on the method — in particular on the shape of the friction cone and on whether the friction law is associative or non-associative. The Mohr–Coulomb parameters that define the cone, at minimum the friction angle $$ \phi $$, are required by all of the methods listed above.

DEM additionally requires **joint stiffnesses**, because it is a *soft-contact* formulation[^cundall1971]: a small interpenetration is permitted at each contact, and the contact forces are derived from the penetration depth through a penalty law. The two stiffnesses are:

- $$ K_n $$ — normal stiffness, relating normal force to normal interpenetration;
- $$ K_t $$ — tangential stiffness, relating shear force to relative tangential displacement until the friction limit is reached.

Static methods and Contact Dynamics, by contrast, treat the contact as rigid (no interpenetration) and do not require these stiffness parameters.

## Result quantities

A block-based analysis answers the central stability question by returning two coupled fields, defined on different parts of the model:

**Block displacements and rotations.** For each block $$ i $$, the analysis returns a rigid-body displacement $$ \mathbf{u}_i $$ and rotation $$ \boldsymbol{\theta}_i $$ from the reference configuration — six values per block in 3D. For a structure that is stable under the applied loads these are small (ideally zero, up to numerical tolerance); they grow without bound and identify a collapse mechanism when the structure is not. The pattern of non-zero block displacements is what reveals *how* a structure fails: which blocks form the moving part of the mechanism, where hinges open, and which contacts slide.


**Contact forces at interfaces.** For each contact $$ c $$ between two blocks,
the analysis returns the normal force $$ F_n^c $$, the two tangential
components $$ F_{t1}^c, F_{t2}^c $$, and the resultant moment, all expressed in
the local frame of the contact. Aggregated across all contacts, these form
the **internal force network** of the structure: the discrete analogue of
a stress field in continuum mechanics, and the direct generalisation of
Heyman's thrust line to three-dimensional assemblies. Three pieces of
information are read off it routinely:

- *Force magnitudes* indicate which contacts are most heavily loaded and
  where local crushing might occur, feeding into the optional block-level
  FE submodel mentioned earlier.
- *Force eccentricity* — where the resultant normal force acts within the
  contact polygon — indicates how close a joint is to opening. A resultant
  near the edge of the contact face means the joint is on the verge of
  forming a hinge.
- *Active vs inactive contacts.* Contacts where $$ F_n^c = 0 $$ have opened;
  contacts where $$ |F_t^c| = \mu F_n^c $$ are sliding. The set of inactive
  contacts is the crack pattern of the assembly.


These are visualised in the `DEMViewer` as force vectors scaled to their magnitude, and can be printed to the terminal for numerical inspection.

> **See also:** [Step 4 — DEM Model Assembly](../03_pipeline/dem_model_step.md), [Step 5 — DEM Problem & Solvers](../03_pipeline/dem_problem_step.md), [Solver Overview](../06_solvers/overview.md)

[^heyman1966]: Heyman, J. (1966). The stone skeleton. *International Journal of Mechanical Sciences*, 8(4), 249–279.
[^block2007]: Block, P. & Ochsendorf, J. (2007). Thrust network analysis: A new methodology for three-dimensional equilibrium. *Journal of the International Association for Shell and Spatial Structures*, 48(3), 167–173.
[^iannuzzo2020]: Iannuzzo, A., Van Mele, T. & Block, P. (2020). Piecewise rigid displacement (PRD) method: A limit analysis-based approach to detect mechanisms and static admissible stress fields. *Mechanics Research Communications*, 107, 103557.
[^iannuzzo2021]: Iannuzzo, A., Dell'Endice, A., Maia Avelino, R., Kao, G.T.-C., Van Mele, T. & Block, P. (2021). COMPAS Masonry: A computational framework for practical assessment of unreinforced masonry structures. *SAHC Symposium*, Barcelona.
[^kao2021]: Kao, G.T.-C., Iannuzzo, A., Coros, S., Van Mele, T. & Block, P. (2021). Understanding the rigid-block equilibrium method by way of mathematical programming. *Proceedings of the Institution of Civil Engineers – Engineering and Computational Mechanics*.
[^kao2022]: Kao, G.T.-C., Iannuzzo, A., Thomaszewski, B., Coros, S., Van Mele, T. & Block, P. (2022). Coupled Rigid-Block Analysis: Stability-Aware Design of Complex Discrete-Element Assemblies. *Computer-Aided Design*, 146, 103216.
[^moreau1988]: Moreau, J.J. (1988). Unilateral contact and dry friction in finite freedom dynamics. In *Non-Smooth Mechanics and Applications*, CISM Courses and Lectures 302. Springer, Vienna, pp. 1–82.
[^jean1999]: Jean, M. (1999). The non-smooth contact dynamics method. *Computer Methods in Applied Mechanics and Engineering*, 177(3–4), 235–257.
[^dubois2018]: Dubois, F., Acary, V. & Jean, M. (2018). The Contact Dynamics method: A nonsmooth story. *Comptes Rendus Mécanique*, 346, 247–262.
[^cundall1971]: Cundall, P.A. (1971). A computer model for simulating progressive, large-scale movements in blocky rock systems. *Proceedings of the Symposium of the International Society of Rock Mechanics*, Nancy, Vol. 1, Paper II-8.
[^cundall1979]: Cundall, P.A. & Strack, O.D.L. (1979). A discrete numerical model for granular assemblies. *Géotechnique*, 29(1), 47–65.
[^lemos2007]: Lemos, J.V. (2007). Discrete element modeling of masonry structures. *International Journal of Architectural Heritage*, 1(2), 190–213.
