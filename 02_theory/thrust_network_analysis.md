# Thrust Network Analysis (TNA)

## Overview

**Thrust Network Analysis (TNA)** is a computational method for finding compression-only equilibrium solutions for two-dimensional funicular structures — vaults, shells, and gridshells. It was developed at the Block Research Group and extends the classical concept of the thrust line for arches to the fully three-dimensional case of vaulted surfaces.

<!-- [PLACEHOLDER: Add primary TNA reference — Block & Ochsendorf 2007 / Block 2009 PhD] -->

TNA is the first computational stage of the CARBCOMN pipeline. Given a **floor plan** (the projection of the vault onto the horizontal plane) and a **load distribution**, TNA finds the three-dimensional surface geometry through which forces can flow in compression under that load.

## The form diagram and the force diagram

TNA is built on the concept of **graphic statics** extended to three dimensions. It operates on two dual planar graphs:

- The **form diagram** Γ — a planar network in the horizontal plane representing the projection of the vault's force-carrying ribs. Its topology (which nodes are connected) defines the connectivity of the thrust network.
- The **force diagram** Γ\* — the dual graph of the form diagram, whose edge lengths are proportional to the horizontal components of the internal forces in the corresponding edges of Γ.

The duality between Γ and Γ\* is the geometric expression of equilibrium: for every internal node of the form diagram, the force polygon formed by the corresponding edges of the force diagram must close. This is the graphical equivalent of ΣF = 0.

<!-- [IMAGE PLACEHOLDER: Form diagram and force diagram side by side for a simple barrel vault] -->

## Force densities

Rather than working directly with forces, TNA parameterises equilibrium in terms of **force densities** — the ratio of the axial force in an edge to its projected length:

$$q_{ij} = \frac{f_{ij}}{l_{ij}^{(h)}}$$

where $$ f_{ij} $$ is the horizontal component of the internal force in edge $$ (i, j) $$ and $$ l_{ij}^{(h)} $$ is the length of that edge in the horizontal projection (the form diagram). Force densities are dimensionless in this sense and provide a numerically stable parameterisation of the equilibrium problem.

## Horizontal equilibrium

For a given form diagram topology and a set of force densities $$ \mathbf{q} $$, the horizontal equilibrium at every free node $$ i $$ is:

$$\sum_{j \in \mathcal{N}(i)} q_{ij} \, (x_i - x_j) = p_{x,i}$$
$$\sum_{j \in \mathcal{N}(i)} q_{ij} \, (y_i - y_j) = p_{y,i}$$

where $$ \mathcal{N}(i) $$ denotes the neighbours of node $$ i $$ in the form diagram, $$ (x_i, y_i) $$ are the horizontal coordinates of node $$ i $$, and $$ (p_{x,i}, p_{y,i}) $$ are the horizontal components of the applied load at node $$ i $$.

In matrix form, separating free nodes (subscript $$ f $$) from fixed boundary nodes (subscript $$ b $$):

$$\mathbf{C}_f^T \mathbf{Q} \mathbf{C}_f \, \mathbf{x}_f = \mathbf{p}_x - \mathbf{C}_f^T \mathbf{Q} \mathbf{C}_b \, \mathbf{x}_b$$
$$\mathbf{C}_f^T \mathbf{Q} \mathbf{C}_f \, \mathbf{y}_f = \mathbf{p}_y - \mathbf{C}_f^T \mathbf{Q} \mathbf{C}_b \, \mathbf{y}_b$$

where:
- $$ \mathbf{C} $$ is the signed edge–node connectivity matrix of the form diagram
- $$ \mathbf{Q} = \text{diag}(\mathbf{q}) $$ is the diagonal matrix of force densities
- $$ \mathbf{x}_f, \mathbf{y}_f $$ are the free node horizontal coordinates (unknowns)
- $$ \mathbf{x}_b, \mathbf{y}_b $$ are the fixed boundary coordinates

For a given set of force densities $$ \mathbf{q} $$, horizontal equilibrium uniquely determines the horizontal positions of the free nodes. In the CARBCOMN pipeline the form diagram is **fixed** (its topology is set by the `Pattern` class) and only vertical equilibrium is solved. The horizontal positions of the network nodes are read from the pattern geometry.

## Vertical equilibrium

Once horizontal equilibrium is satisfied, vertical equilibrium uniquely determines the **heights** $$ z_i $$ of the free nodes — i.e., the three-dimensional shape of the thrust network:

$$\mathbf{C}_f^T \mathbf{Q} \mathbf{C}_f \, \mathbf{z}_f = \mathbf{p}_z - \mathbf{C}_f^T \mathbf{Q} \mathbf{C}_b \, \mathbf{z}_b$$

where $$ \mathbf{p}_z $$ contains the vertical load components (typically self-weight, distributed to nodes). This is the same linear system as for horizontal equilibrium but with the vertical coordinate as the unknown.

The solution $$ \mathbf{z}_f $$ gives the funicular surface: the unique three-dimensional geometry consistent with the prescribed boundary heights $$ \mathbf{z}_b $$, the force density distribution $$ \mathbf{q} $$, and the load $$ \mathbf{p}_z $$.

**Key insight:** For a fixed topology and boundary, every choice of force densities $$ \mathbf{q} > 0 $$ (all compressive) produces a valid funicular solution. TNA is therefore a **parameterisation of the space of compression-only equilibrium surfaces** for a given vault plan and load.

## The pattern and the floor form diagram

In the CARBCOMN pipeline, the form diagram is generated from a **`Pattern`** object that encodes the combinatorial layout of the vault's ribs and rings. For a barrel vault floor, the pattern is created with `Pattern.from_barrelvault()`, which produces a rectangular grid of radial and circumferential edges corresponding to the vault columns and span direction respectively.

The force density on rib edges (spanning direction) and ring edges (perpendicular direction) can be set independently, allowing the designer to control the anisotropy of the thrust network and thereby the curvature distribution of the resulting surface.

The **`FloorFormDiagram`** class wraps the form diagram and manages the boundary conditions — the perimeter nodes are fixed at specified boundary heights corresponding to the support level of the vault.

## The RefMesh

The output of TNA is a **`RefMesh`** — a mesh object that stores:
- The equilibrium node positions $$ (x_i, y_i, z_i) $$
- The force density on each edge $$ q_{ij} $$
- The internal horizontal force on each edge $$ f_{ij} $$
- The edge frames used by the geometry template to orient voussoir cuts

The `RefMesh` is the input to the geometry generation stage. It defines the **mid-surface** of the vault — the surface that the centroids of the voussoir blocks will follow — and the force distribution that determines how blocks should be oriented and sized.

<!-- [IMAGE PLACEHOLDER: RefMesh rendered over the floor plan — show horizontal form diagram and 3D thrust surface] -->

## Summary

| Step | Operation | Output |
|------|-----------|--------|
| Define pattern | `Pattern.from_barrelvault()` | Combinatorial layout of the vault grid |
| Set force densities | Assign $$ q $$ values to rib and ring edges | Parameterised equilibrium |
| Solve vertical equilibrium | $$ \mathbf{C}_f^T \mathbf{Q} \mathbf{C}_f \, \mathbf{z}_f = \mathbf{p}_z $$ | Node heights $$ \mathbf{z}_f $$ |
| Build RefMesh | Assemble mesh from equilibrium positions | `RefMesh` object |

> **See also:** [Step 1 — TNA in the Pipeline Reference](../03_pipeline/tna_step.md)

> **References:** <!-- [PLACEHOLDER: Block & Ochsendorf 2007; Block 2009; Van Mele & Block 2014] -->

---

## References

<!-- Add references below using the format:
[N] Author, A., Author, B. (Year). *Title of paper or book*. Journal / Publisher, Volume(Issue), Pages. DOI/URL
Example:
[1] Block, P., Ochsendorf, J. (2007). *Thrust network analysis: A new methodology for three-dimensional equilibrium*. Journal of the IASS, 48(3), 167–173.
-->
