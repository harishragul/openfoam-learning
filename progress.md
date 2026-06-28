# OpenFOAM Learning Progress

## Learner Profile

- **Name**: Harish
- **Started**: 2026-05-12
- **OpenFOAM Version**: ESI v2512 (openfoam.com)
- **Background**: Familiar with CFD concepts at a high level; strong intuition on physical meaning

## Current Status

**Current Module**: Module 02 — First Simulations
**Current Lesson**: Module 02 complete — next is Module 03: Mesh Generation
**Overall Progress**: 2 / 8 modules complete (Modules 01 and 02 done)

---

## Module Progress

### Module 01: Foundations

- [x] Lesson 1.1 — What is CFD and why OpenFOAM?
- [x] Lesson 1.2 — Installing and verifying OpenFOAM
- [x] Lesson 1.3 — CFD governing equations (intuitive intro)
- [x] Lesson 1.4 — OpenFOAM case structure and file layout
- [x] Exercise 1A — Explore a tutorial case directory
- [x] Exercise 1B — Answer: what does each subdirectory contain?

### Module 02: First Simulations

- [x] Lesson 2.1 — The lid-driven cavity case (icoFoam)
- [x] Lesson 2.2 — Running a simulation step by step
- [x] Lesson 2.3 — Reading solver output and log files
- [x] Lesson 2.4 — Visualizing results in ParaView
- [x] Exercise 2A — Run cavity tutorial and capture screenshots
- [x] Exercise 2B — Change Reynolds number and observe flow change
- [x] Exercise 2C — Read log file, identify Co > 1 issue, fix deltaT, verify residuals
- [x] Exercise 2D — Plot Over Line, Probe Location, locked animation

### Module 03: Mesh Generation

- [x] Lesson 3.1 — Mesh concepts: cells, faces, nodes, zones
- [x] Lesson 3.2 — blockMesh: structured grids from scratch
- [x] Lesson 3.3 — Mesh grading and refinement
- [x] Lesson 3.4 — snappyHexMesh: meshing around complex geometry
- [x] Lesson 3.5 — checkMesh and mesh quality metrics
- [x] Exercise 3A — Inspect cavity mesh with checkMesh (400 cells, perfect quality)
- [x] Exercise 3B — Build 2D channel mesh with blockMesh (8000 cells, aspect ratio ~10)
- [x] Exercise 3C — Mesh independence study; grading sensitivity; y+ workflow
- [x] Exercise 3D — snappyHexMesh sphere in channel (28,292 cells, non-ortho max 36°, all 3 phases clean)

### Module 04: Boundary Conditions

- [x] Lesson 4.1 — BC fundamentals: Dirichlet, Neumann, Robin
- [x] Lesson 4.2 — Velocity inlet/outlet conditions (codedFixedValue, flowRateInletVelocity, pressureInletOutletVelocity)
- [x] Lesson 4.3 — Pressure boundary conditions (totalPressure, prghPressure, fixedFluxPressure)
- [x] Lesson 4.4 — Wall conditions: no-slip, slip, moving walls (movingWallVelocity vs rotatingWallVelocity)
- [x] Lesson 4.5 — Cyclic/periodic and symmetry patches (cyclic, cyclicAMI, wedge)
- [x] Lesson 4.6 — Turbulent BCs (k, epsilon, omega) — renumbered from misfiled 4.2
- [x] Exercise 4A — BCs for pipeflow (forced convection), windtunnel (external aero + car body), heatedroom (natural convection: vent + radiator, T-driven buoyancy)
- [x] Exercise 4B — Parabolic velocity inlet via codedFixedValue (caught Umax=2x vs 1.5x Umean error: circular pipe vs 2D channel)
- [x] Exercise 4C — Pressure BC selection: centrifugal-fan (totalPressure), compressible-nozzle (waveTransmissive), heatedroom walls (fixedFluxPressure)
- [x] Exercise 4D — Turbulent k/omega BCs for pipe flow (k=0.09375, omega=159.719, hand-verified correct)
- [ ] Exercise 4E — Wall condition selection: pipe wall, far-field domain edge, rotating drum, symmetry plane

### Module 05: Solvers

- [ ] Lesson 5.1 — Solver families: what problem do they solve?
- [ ] Lesson 5.2 — icoFoam and simpleFoam (incompressible)
- [ ] Lesson 5.3 — pimpleFoam and pisoFoam (transient)
- [ ] Lesson 5.4 — rhoPimpleFoam (compressible)
- [ ] Lesson 5.5 — interFoam (multiphase VOF)
- [ ] Lesson 5.6 — fvSolution and fvSchemes dictionaries
- [ ] Exercise 5A — Tune under-relaxation and achieve convergence
- [ ] Exercise 5B — Compare SIMPLE vs PIMPLE on a channel flow

### Module 06: Turbulence Modeling

- [ ] Lesson 6.1 — Why turbulence models? Reynolds decomposition
- [ ] Lesson 6.2 — RANS models: k-epsilon, k-omega SST
- [ ] Lesson 6.3 — Wall functions and y+ requirements
- [ ] Lesson 6.4 — LES: Smagorinsky, dynamic Smagorinsky
- [ ] Lesson 6.5 — Choosing the right model for your problem
- [ ] Exercise 6A — Set up k-omega SST for a backward-facing step
- [ ] Exercise 6B — Compare k-eps vs k-omega SST results

### Module 07: Post-Processing

- [ ] Lesson 7.1 — ParaView workflow for OpenFOAM results
- [ ] Lesson 7.2 — functionObjects: probes, forces, averaging
- [ ] Lesson 7.3 — Sample and surface extraction
- [ ] Lesson 7.4 — foamCalc and postProcess utilities
- [ ] Exercise 7A — Plot residuals and monitor convergence
- [ ] Exercise 7B — Extract a velocity profile at a cross-section

### Module 08: Advanced Topics

- [ ] Lesson 8.1 — Writing a custom solver from scratch
- [ ] Lesson 8.2 — Custom boundary conditions with codedFixedValue
- [ ] Lesson 8.3 — Parallel decomposition and HPC runs
- [ ] Lesson 8.4 — Adjoint optimization and mesh morphing
- [ ] Lesson 8.5 — OpenFOAM with Python (PyFoam, Ofpp)
- [ ] Exercise 8A — Modify simpleFoam to add a source term
- [ ] Exercise 8B — Run a 16-core parallel simulation and reconstruct

---

## Completed Sessions Log

| Date       | Module    | Lessons Covered                       | Notes                                                                              |
|------------|-----------|---------------------------------------|------------------------------------------------------------------------------------|
| 2026-05-31 | Module 01 | All lessons + exercises               | Completed full foundations module                                                  |
| 2026-05-31 | Module 02 | All lessons 2.1-2.4, exercises 2A-2D | Ran cavity Re=10 and Re=100, caught Co>1 bug, read logs, visualized in ParaView   |
| 2026-06-05 | Module 03 | Lessons 3.1-3.5, exercises 3A-3D     | blockMesh channel, mesh independence, grading ratio correction, snappyHexMesh sphere |

---

## Notes & Questions to Revisit

- Residual vs correction distinction — closed via 2x=10 example (2026-05-31)
- Co > 1 bug caught in Re=100 run — fixed with deltaT=0.001 (2026-05-31)
- Steady-state observation: Re=100 cavity pressure barely changes over time → candidate for simpleFoam
