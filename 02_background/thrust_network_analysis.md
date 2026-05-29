# Thrust Network Analysis (TNA)

## Overview

**Thrust Network Analysis (TNA)** is a computational method for finding compression-only equilibrium solutions for three-dimensional funicular structures — vaults, shells, and gridshells. It extends the classical concept of the thrust line for arches to vaulted surfaces.

TNA is the first computational stage of the CARBCOMN pipeline. It starts with a form diagram (Γ), support locations and a load distribution applied in the nodes of the network. TNA finds the three-dimensional thrust network geometry (G) through which forces can flow in compression under that load and enable to reshape/design the structure by modifying directly the force and magnitude of the forces.[^1]

<figure><img src="../.gitbook/assets/Screenshot 2026-05-28 at 18.50.03.png" alt=""><figcaption></figcaption></figure>

## The form diagram and the force diagram

TNA is built on the concept of **graphic statics** extended to three dimensions. It operates on two dual planar graphs:

* The **form diagram** Γ — a planar network in the horizontal plane representing the projection of the vault's force network. It defines the topology and connectivity of the thrust network, i.e., the path that loads take to travel to the supports.
* The **force diagram** Γ\* — the dual graph of the form diagram, whose edge lengths are proportional to the horizontal components of the internal forces in the corresponding edges of Γ.

The duality between Γ and Γ\* is the geometric expression of equilibrium: for every internal node of the form diagram in equilibrium, a closed force polygon is formed in the force diagram by the corresponding edges. This is the graphical equivalent of ΣF = 0.

## Force densities

Rather than working directly with forces, TNA parameterises equilibrium in terms of **force densities**[^2] — the ratio of the axial force in an edge to its projected length:

$$q_{ij}  =  \frac{f_{ij}^{(h)}}{l_{ij}^{(h)}} = \frac{f_{ij}}{l_{ij}}$$

where $$f_{ij}^{(h)}$$ is the horizontal component of the internal force in edge $$(i, j)$$ and $$l_{ij}^{(h)}$$ is the length of that edge in the horizontal projection (the form diagram), and  $$f_{ij}$$ is the actual force in the 3D edge of the thrust network and $$l_{ij}$$ is the 3D length of the edge in space in the thrust network.

Force densities are dimensionless in this sense and provide a numerically stable parameterisation of the equilibrium problem.

## Horizontal equilibrium

For a given form diagram topology and a set of force densities $$\mathbf{q}$$, the horizontal equilibrium at every free node $$i$$ is:

$$\sum_{j \in \mathcal{N}(i)} q_{ij} \, (x_i - x_j) = p_{x,i}$$ $$\sum_{j \in \mathcal{N}(i)} q_{ij} \, (y_i - y_j) = p_{y,i}$$

where $$\mathcal{N}(i)$$ denotes the neighbours of node $$i$$ in the form diagram, $$(x_i, y_i)$$ are the horizontal coordinates of node $$i$$, and $$(p_{x,i}, p_{y,i})$$ are the horizontal components of the applied load at node $$i$$.

In matrix form, separating free nodes (subscript $$f$$) from fixed boundary nodes (subscript $$b$$):

$$\mathbf{C}_f^T \mathbf{Q} \mathbf{C}_f \, \mathbf{x}_f = \mathbf{p}_x - \mathbf{C}_f^T \mathbf{Q} \mathbf{C}_b \, \mathbf{x}_b$$ $$\mathbf{C}_f^T \mathbf{Q} \mathbf{C}_f \, \mathbf{y}_f = \mathbf{p}_y - \mathbf{C}_f^T \mathbf{Q} \mathbf{C}_b \, \mathbf{y}_b$$

where:

* $$\mathbf{C}$$ is the signed edge–node connectivity matrix of the form diagram
* $$\mathbf{Q} = \text{diag}(\mathbf{q})$$ is the diagonal matrix of force densities
* $$\mathbf{x}_f, \mathbf{y}_f$$ are the free node horizontal coordinates (unknowns)
* $$\mathbf{x}_b, \mathbf{y}_b$$ are the fixed boundary coordinates

For a given set of force densities $$\mathbf{q}$$, horizontal equilibrium uniquely determines the horizontal positions of the free nodes. In the CARBCOMN pipeline the form diagram is **fixed** (its topology is set by the `Pattern` class) and only vertical equilibrium is solved. The horizontal positions of the network nodes are read from the pattern geometry.

## Vertical equilibrium

Once horizontal equilibrium is satisfied, vertical equilibrium uniquely determines the **heights** $$z_i$$ of the free nodes — i.e., the three-dimensional shape of the thrust network:

$$\mathbf{C}_f^T \mathbf{Q} \mathbf{C}_f \, \mathbf{z}_f = \mathbf{p}_z - \mathbf{C}_f^T \mathbf{Q} \mathbf{C}_b \, \mathbf{z}_b$$

where $$\mathbf{p}_z$$ contains the vertical load components (typically self-weight, distributed to nodes). This is the same linear system as for horizontal equilibrium but with the vertical coordinate as the unknown.

The solution $$\mathbf{z}_f$$ gives the funicular surface: the unique three-dimensional geometry consistent with the prescribed boundary heights $$\mathbf{z}_b$$, the force density distribution $$\mathbf{q}$$, and the load $$\mathbf{p}_z$$.

**Key insight:** For a fixed topology and boundary, every choice of force densities $$\mathbf{q} > 0$$ (all compressive) produces a valid funicular solution. TNA is therefore a **parameterisation of the space of compression-only equilibrium surfaces** for a given vault plan and load.

## The pattern and the floor form diagram

In the CARBCOMN pipeline, the form diagram is generated from a **`Pattern`** object that encodes the combinatorial layout of the vault's ribs and rings. For a barrel vault floor, the pattern is created with `Pattern.from_barrelvault()`, which produces a rectangular grid of radial and circumferential edges corresponding to the vault columns and span direction respectively.

The force density on rib edges (spanning direction) and ring edges (perpendicular direction) can be set independently, allowing the designer to control the anisotropy of the thrust network and thereby the curvature distribution of the resulting surface.

The **`FloorFormDiagram`** class wraps the form diagram and manages the boundary conditions — the perimeter nodes are fixed at specified boundary heights corresponding to the support level of the vault.

## The RefMesh

The output of TNA is a **`RefMesh`** — a mesh object that stores:

* The equilibrium node positions $$(x_i, y_i, z_i)$$
* The force density on each edge $$q_{ij}$$
* The internal horizontal force on each edge $$f_{ij}$$
* The edge frames used by the geometry template to orient voussoir cuts

The `RefMesh` is the input to the geometry generation stage. It defines the **mid-surface** of the vault — the surface that the centroids of the voussoir blocks will follow — and the force distribution that determines how blocks should be oriented and sized.

## Summary

| Step                       | Operation                                                                 | Output                                 |
| -------------------------- | ------------------------------------------------------------------------- | -------------------------------------- |
| Define pattern             | `Pattern.from_barrelvault()`                                              | Combinatorial layout of the vault grid |
| Set force densities        | Assign $$q$$ values to rib and ring edges                                 | Parameterised equilibrium              |
| Solve vertical equilibrium | $$\mathbf{C}_f^T \mathbf{Q} \mathbf{C}_f \, \mathbf{z}_f = \mathbf{p}_z$$ | Node heights $$\mathbf{z}_f$$          |
| Build RefMesh              | Assemble mesh from equilibrium positions                                  | `RefMesh` object                       |

***

## References

[^1]: Block, P. & Ochsendorf, J. (2007). Thrust network analysis: A new methodology for three-dimensional equilibrium. *Journal of the International Association for Shell and Spatial Structures*, 48(3), 167–173.

[^2]: Schek, H.-J. (1974). The force density method for form finding and computation of general networks. *Computer Methods in Applied Mechanics and Engineering*, 3(1), 115–134.

[^3]: Block, P. (2009). *Thrust Network Analysis: Exploring Three-Dimensional Equilibrium*. PhD thesis, Massachusetts Institute of Technology. https://dspace.mit.edu/handle/1721.1/49539

> **See also:** [Step 1 — TNA in the Pipeline Reference](../03_pipeline/tna_step.md)

