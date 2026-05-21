# API Overview

The `carbcomn` package is organised into four sub-packages. Each section in this chapter documents the public classes and functions in one sub-package.

| Sub-package | Contents |
|-------------|----------|
| [`carbcomn.datastructures`](datastructures.md) | `RefMesh`, `RefBlock`, `FloorFormDiagram`, `Pattern` |
| [`carbcomn.model`](model_elements.md) | All structural element types: blocks, beams, columns; `FloorModel`, `StructuralGrid`, `TrimModifier` |
| [`carbcomn.templates`](templates.md) | Parametric geometry templates: `ArchTemplate`, `FlatBarrelTemplate`, `BarrelTemplate` |
| [`carbcomn.session`](session.md) | `WorkflowSession` |

## Import conventions

All public classes are importable directly from the sub-package:

```python
from carbcomn.datastructures import RefMesh, RefBlock, FloorFormDiagram, Pattern
from carbcomn.model import BlockModel, Column, TieBeam, FloorModel, StructuralGrid, TrimModifier
from carbcomn.model.elements.blocks.standard_block import StandardBlock
from carbcomn.model.elements.blocks.ridge_voussoir import RidgeVoussoir
from carbcomn.model.elements.blocks.carbcomn_voussoir import CarbcomnVoussoir
from carbcomn.templates.barrel import FlatBarrelTemplate
from carbcomn.templates.arch import ArchTemplate
from carbcomn.session import WorkflowSession
```

> **Note:** The API is under active development. Class signatures and method names may change between versions.
