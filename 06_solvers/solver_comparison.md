# Solver Comparison

This table summarises the practical trade-offs between the available solvers for typical CARBCOMN analysis tasks.

| Property | LMGC90 | CRA | RBE | 3DEC |
|----------|:------:|:---:|:---:|:----:|
| License | Open-source | Open-source | Open-source | Commercial |
| Formulation | NSCD (time-stepping) | Convex optimisation | Direct equilibrium | Distinct Element |
| Max assembly size | Large (1000s) | Small (~50–100) | Medium (~200) | Very large |
| Precision | High | Exact | Approximate | Very high |
| Dynamic / kinematic loading | ✓ | — | — | ✓ |
| Deformable blocks | (planned) | — | — | ✓ |
| Convergence at near-collapse | ✓ | ✓ | partial | ✓ |
| Install complexity | Medium | Low | Low | High (license) |

## Recommendation for CARBCOMN workflows

- **Default:** Use **LMGC90** for all standard analyses. It handles the full floor models (300–500 blocks) robustly and supports the support-displacement load cases needed for min/max thrust analysis.
- **Validation:** Run **CRA** on the simpler examples (`000_threeblocks`, `100_arch`) to cross-validate LMGC90 results.
- **Early iteration:** Use **RBE** when iterating quickly over parameter variations where precise force magnitudes are less important than the qualitative force flow pattern.
- **Final verification:** Use **3DEC** for high-stakes analyses where commercial-grade precision is required, if a license is available.

## Comparing solvers on the same model

The arch example (`100`) provides the cleanest comparison: scripts `141_dem_problem_LMGC90.py` and `142_dem_problem_CRA.py` solve the identical problem with LMGC90 and CRA respectively. Comparing their contact force outputs is a good way to build confidence in the LMGC90 results.

```python
# In 141_dem_problem_LMGC90.py
problem.solve(Solver.LMGC90())

# In 142_dem_problem_CRA.py
problem.solve(Solver.CRA())
```

Both store results in the same `session["blockmodel_results"]` format. Running both scripts and comparing the printed contact forces quantifies the solver-to-solver discrepancy for a given model.
