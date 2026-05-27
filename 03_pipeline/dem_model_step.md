# Step 4 — DEM Model Assembly

**Script prefix:** `X40`  
**Session inputs:** `session["block_elements"]` (typed workflows) or `session["block_meshes"]` (standard-block workflows)  
**Session outputs:** `session["blockmodel"]`

## Purpose

The DEM model step assembles all block elements into a `BlockModel` and **computes the contact interfaces** between adjacent blocks. The result is a complete discrete-element model — a contact graph where nodes represent blocks and edges represent detected contact interfaces — ready to be handed to a solver.

## Building the BlockModel

```python
from compas_dem.models import BlockModel
from carbcomn.model.elements.structural import StructuralElement

model = BlockModel()

for i in range(len(blocks)):
    for j in range(len(blocks[i])):
        block = blocks[i][j]
        model.add_element(block)

model.compute_contacts()

session["blockmodel"] = model
session.sync()
```

For simpler workflows (examples `200`, `300`) that do not use the RefBlock step, block meshes are passed directly:

```python
model = BlockModel.from_meshes(block_meshes_flat)
model.compute_contacts()
```

## Contact detection

`model.compute_contacts()` performs a geometric search over all block pairs to find adjacent blocks and compute their shared interface geometry. For each detected contact it:

1. Identifies the **type** of contact: face-to-face (a polygon) or edge-to-edge (a degenerate line contact)
2. Computes the **common interface polygon** — the intersection of the two adjacent block faces
3. Discretises the interface into **contact points** where forces are applied and resolved

The contact graph is stored on `model.graph`: each node corresponds to a block element and each edge corresponds to a detected contact interface.

## Inspecting the contact graph

After running `compute_contacts()`, the model topology can be visualised to verify that all expected contacts were detected:

```python
# Nodes: block elements
for node in model.graph.nodes():
    element = model.graph.node_element(node)
    print(node, element.name, element.is_support)

# Edges: contact interfaces
for contact in model.contacts():
    print(contact.mesh)   # the interface polygon mesh
```

In the viewer, support blocks are shown in red and non-support blocks in light grey. Contact polygons are shown in cyan. The contact graph edges are shown as magenta lines connecting block centroids.

<!-- [IMAGE PLACEHOLDER: BlockModel viewer showing blocks, contact polygons (cyan), and graph edges (magenta)] -->

## Degenerate contacts

Contacts which lose one or more contact points are considered degenerate contacts, a face contact can degenerate into an edge contact, or an edge contact can become a point contact. These cases are flagged as **degenerate contacts**. They appear in the `DEMViewer` under the `Degenerate_Contacts` group and are useful for diagnosing post-peak behavior, or hinging locations more clearly.
In the current implementation, only dynamic solvers can have degenerate contacts.

> **See also:** [Step 5 — DEM Problem & Solvers](dem_problem_step.md), [DEM Theory](../02_theory/discrete_element_method.md)
