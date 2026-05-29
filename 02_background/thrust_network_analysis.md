# Thrust Network Analysis (TNA)

## Overview

**Thrust Network Analysis (TNA)** is a computational method for finding compression-only equilibrium solutions for three-dimensional funicular structures — vaults, shells, and gridshells. It extends the classical concept of the thrust line for arches to vaulted surfaces.

TNA is the first computational stage of the CARBCOMN pipeline. It starts with a form diagram (Γ) which defines the planar connectivity and topology of the network, its support locations and a load distribution applied in the nodes of the network. TNA finds the three-dimensional thrust network geometry (G) through with compression-only forces that can be edited graphically by modifying the force diagram (Γ\*) enabling to reshape/design the structure by shaping its internal forces.<sup>\[1]</sup> Figure 1 shows TNA applied to a dome cap.

<figure><img src="../.gitbook/assets/Screenshot 2026-05-28 at 18.50.03.png" alt=""><figcaption></figcaption></figure>

## The form diagram and the force diagram

TNA is built on the concept of **graphic statics** extended to three dimensions. It operates on two dual planar graphs:

* The **form diagram** Γ — a planar network in the horizontal plane representing the projection of the vault's force network. It defines the topology and connectivity of the thrust network, i.e., the path that loads take to travel to the supports.
* The **force diagram** Γ\* — the dual graph of the form diagram, whose edge lengths are proportional to the horizontal components of the internal forces in the corresponding edges of Γ.

The duality between Γ and Γ\* is the geometric expression of equilibrium: for every internal node of the form diagram in equilibrium, a closed force polygon is formed in the force diagram by the corresponding edges. This is the graphical equivalent of ΣF = 0.

## Force densities

Rather than working directly with forces, TNA parameterises equilibrium in terms of **force densities**<sup>\[2]</sup> — the ratio of the axial force in an edge to its projected length:

$$q_{ij} = \frac{f_{ij}^{(h)}}{l_{ij}^{(h)}} = \frac{f_{ij}}{l_{ij}}$$

where $$f_{ij}^{(h)}$$ is the horizontal component of the internal force in edge $$(i, j)$$ and $$l_{ij}^{(h)}$$ is the length of that edge in the horizontal projection (the form diagram), and $$f_{ij}$$ is the actual force in the 3D edge of the thrust network and $$l_{ij}$$ is the 3D length of the edge in space in the thrust network.

Force densities are dimensionless and enable to linearlize the equations of equilibrium providing a numerically stable parameterisation of the equilibrium problem.

## Horizontal equilibrium

For a given form diagram topology and a set of force densities $$\mathbf{q}$$, the horizontal equilibrium at every free node $$i$$ is:

$$\sum_{j \in \mathcal{N}(i)} q_{ij} \, (x_i - x_j) = p_{x,i}$$ , in x direction

$$\sum_{j \in \mathcal{N}(i)} q_{ij} \, (y_i - y_j) = p_{y,i}$$, in your direction

where $$\mathcal{N}(i)$$ denotes the neighbours of node $$i$$ in the form diagram, $$(x_i, y_i)$$ are the horizontal coordinates of node $$i$$, and $$(p_{x,i}, p_{y,i})$$ are the horizontal components of the applied load at node $$i$$.

In matrix form, separating free nodes (subscript $$f$$) from fixed boundary nodes (subscript $$b$$) we can write:

$$\mathbf{C}_f^T \mathbf{Q} \mathbf{C}_f \, \mathbf{x}_f = \mathbf{p}_x - \mathbf{C}_f^T \mathbf{Q} \mathbf{C}_b \, \mathbf{x}_b$$&#x20;

$$\mathbf{C}_f^T \mathbf{Q} \mathbf{C}_f \, \mathbf{y}_f = \mathbf{p}_y - \mathbf{C}_f^T \mathbf{Q} \mathbf{C}_b \, \mathbf{y}_b$$

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

**Key insight:** For a fixed topology and boundary, every choice of force densities $$\mathbf{q} < 0$$ (all compressive) produces a valid funicular solution. TNA is therefore a **parameterisation of the space of compression-only equilibrium surfaces** for a given vault plan and load.

## The pattern and the floor form diagram

In the CARBCOMN pipeline, the form diagram is generated from a **`Pattern`** object. For a barrel vault floor, the pattern is created with `Pattern.from_barrelvault()`, which produces an orthogonal grid with edges in the horizontal and vertical directions, corresponding to the vault span and columns direction respectively.

The **`FloorFormDiagram`** class wraps the form diagram and manages the boundary conditions — the perimeter nodes are fixed at specified boundary heights corresponding to the support level of the vault.

The equilibirum is solved graphically with TNA such that the funicular geometry and force densities of the compression-only barrel-like vault is obtained.

## The RefMesh

The following the equilibrium computation, the output of TNA is a **`RefMesh`** — a mesh object that stores:

* The equilibrium node positions $$(x_i, y_i, z_i)$$
* The force density $$q_{ij}$$ and internal force on each edge $$f_{ij}$$
* The edge frames used by the geometry template to orient voussoir cuts

The `RefMesh` is the input to the geometry generation stage. It defines the **mid-surface** of the vault — the surface that the centroids of the voussoir blocks will follow — and the force distribution that determines how blocks should be oriented and sized.

## Summary

<table><thead><tr><th width="219.421875">Step</th><th width="256.95703125">Operation</th><th>Output</th></tr></thead><tbody><tr><td>Define pattern</td><td><code>Pattern.from_barrelvault()</code></td><td>Combinatorial layout of the vault grid</td></tr><tr><td>Compute TNA Equilibrium</td><td>Find Form and Force diagrams in Equilibrium and generate the 3D structure</td><td>3D geometry and lumped forces</td></tr><tr><td>Build RefMesh</td><td>Assemble mesh from equilibrium positions</td><td><code>RefMesh</code> object</td></tr></tbody></table>

> **See also:** [Step 1 — TNA in the Pipeline Reference](../03_pipeline/tna_step.md)

***

## References

1. Block, P. & Ochsendorf, J. (2007). Thrust network analysis: A new methodology for three-dimensional equilibrium. _Journal of the International Association for Shell and Spatial Structures_, 48(3), 167–173.
2. Schek, H.-J. (1974). The force density method for form finding and computation of general networks. _Computer Methods in Applied Mechanics and Engineering_, 3(1), 115–134.
3. Block, P. (2009). _Thrust Network Analysis: Exploring Three-Dimensional Equilibrium_. PhD thesis, Massachusetts Institute of Technology. https://dspace.mit.edu/handle/1721.1/49539
