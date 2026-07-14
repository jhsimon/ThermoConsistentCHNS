# Diffuse-Interface Two-Phase Flow with a Porous Obstacle

A **FreeFEM** simulation of a rising two-phase bubble interacting with a diffuse porous obstacle. The model couples a thermodynamically consistent Cahn–Hilliard/Navier–Stokes equations, Brinkman penalization inside the obstacle, adaptive mesh refinement, and discrete energy diagnostics.

## Model overview

The program evolves:

- `phi`: binary-fluid phase field,
- `mu`: chemical potential,
- `(ux, uy)`: fluid velocity,
- `p`: pressure,
- `Psi`: diffuse obstacle phase field.

Material density and viscosity are regularized functions of `phi`. The obstacle is represented by a phase field and enters the equations through:

- `alphau`: Brinkman penalization of velocity in the obstacle,
- `alphaphi`: interpolation coefficient in the phase-field terms.

At each time step, the code:

1. solves the nonlinear Cahn–Hilliard system with Newton iteration;
2. updates density, its derivative, and viscosity;
3. solves the Navier–Stokes system;
4. evaluates mass and energy diagnostics;
5. exports VTK/VTU data;
6. adapts the mesh around the fluid and obstacle interfaces.

## Requirements

- [FreeFEM](https://freefem.org/)
- FreeFEM's `iovtk` plugin
- UMFPACK support in the FreeFEM installation
- ParaView or another VTK-compatible application for visualization
- A Unix-like shell, because the program uses `rm` and `mkdir`

The source also includes `getARGV.idp`, which is normally distributed with FreeFEM and is used to read command-line parameters.

## Running the simulation

The uploaded source is named `main.txt`. FreeFEM programs conventionally use the `.edp` extension, so rename or copy it first:

```bash
cp main.txt main.edp
FreeFem++ main.edp
```

Run with a custom obstacle-interface thickness:

```bash
FreeFem++ main.edp --delta 0.02
```

The default is:

```text
delta = 0.01
```

If `delta > 1`, the program interprets it as a scaled integer and multiplies it by `1e-4`. For example:

```bash
FreeFem++ main.edp --delta 100
```

uses `delta = 0.01` after scaling.

## Main parameters

| Parameter | Default | Meaning |
|---|---:|---|
| `T` | `30` | Terminal simulation time |
| `dt` | `0.01` | Time-step size |
| `epsilon` | `0.01` | Binary-fluid interface thickness |
| `delta` | `0.01` | Obstacle-interface thickness |
| `m` | `2e-5` | Phase-field mobility |
| `sigma` | `24.5 * 3/(2*sqrt(2))` | Interfacial-energy coefficient |
| `g` | `0.98` | Gravitational acceleration |
| `alp` | `10000` | Brinkman/interpolation constant |
| `rh1`, `rh2` | `1`, `1000` | Lower and upper density parameters |
| `et1`, `et2` | `0.1`, `10` | Lower and upper viscosity parameters |
| `tol` | `1e-6` | Newton stopping tolerance and pressure regularization |
| `maxiter` | `500` | Maximum Newton iterations per time step |
| `nn` | `101` | Base horizontal mesh resolution |

Parameters used in a run are written to `parameter_values.txt` in the output directory.

## Geometry and initial conditions

The computational domain is the rectangle:

```text
[0, 1] x [0, 2.5]
```

The active configuration initializes:

- a diffuse obstacle centered at `(0, 1.35)` with radius parameter `0.125`;
- a circular bubble centered at `(0, 1.875)` with radius `0.25`;
- zero initial velocity;
- zero initial chemical potential.

Because the obstacle uses an `L1` distance,

```text
abs(x - trcenterx) + abs(y - trcentery) - trrad
```

its level sets are diamond-shaped rather than circular. Alternative obstacle/bubble configurations for intersecting and pinning tests are retained as commented blocks in the source.

## Spatial and temporal discretization

The code uses the following finite-element spaces:

- velocity: `P1b` (mini-element velocity),
- pressure: `P1`,
- phase field and chemical potential: `P1`,
- auxiliary vector fields: combinations of `P1b` and `P1`.

The nonlinear Cahn–Hilliard problem is solved by Newton iteration with UMFPACK. The Navier–Stokes problem is then solved using the updated phase-dependent material coefficients. No-slip velocity conditions are imposed on all four exterior boundaries.

Adaptive refinement follows the combined indicator `phi * Psi`, with finer resolution near the binary-fluid and obstacle interfaces.

## Output

For a given `delta`, results are stored in a directory named approximately as follows:

```text
Data_delta<delta*1e4>x1e-4/
```

Example for `delta = 0.01`:

```text
Data_delta100x1e-4/
```

The program deletes an existing directory with the same name before starting a run.

```text
Data_delta.../
├── parameter_values.txt
├── energy_values.txt
├── mass_evals.txt
├── phase/
│   ├── phi0.vtu
│   └── phi*.vtu
├── chem/
│   ├── mu0.vtu
│   └── mu*.vtu
├── state/
│   ├── state0.vtu
│   └── state*.vtu
└── obstacle/
    └── Psi*.vtk
```

### Diagnostic files

`energy_values.txt` records time-dependent quantities including:

- total energy,
- instantaneous dissipation,
- hydrophilic/increment energy,
- gravitational contribution (labeled `Kinetic Energy` in the current output header),
- the accumulated left-hand side of the discrete energy relation.

`mass_evals.txt` records:

- relative mass,
- obstacle-interface convective contribution,
- chemical-potential contribution.

## Visualization in ParaView

1. Open ParaView.
2. Load a time series such as `phase/phi*.vtu`.
3. Click **Apply**.
4. Use **Coloring → phi** to display the phase field.
5. Load `state/state*.vtu` to inspect velocity and pressure.
6. Use **Glyph**, **Stream Tracer**, or **Warp By Vector** for velocity visualization.

The obstacle snapshots in `obstacle/Psi*.vtk` can be loaded separately and displayed using a contour near `Psi = 0`.


## Reproducibility notes

For a reproducible run, record:

- the FreeFEM version;
- the command-line value of `delta`;
- any changes to geometry or initial conditions;
- mesh-adaptation settings;
- compiler/plugin information;
- the generated `parameter_values.txt` file.

Output files can become large because the default setup performs `3000` time steps and writes phase, chemical-potential, and state files at every step.


## Accompanying Work

This repository contains the accompanying code for forthcoming joint work by **John Sebastian Simon**, **Christian Kahle**, and **Michael Hinze**. The associated manuscript is currently in preparation. Citation information will be added once the work becomes publicly available.



## Numerical Examples

The following figure shows the evolution of the phase field with varying values of delta as the bubble interacts with the diffuse porous obstacle.



<p align="center">
  <igure/example1.png
</p>


<p align="center">
  <em>Figure 1: Evolution of the droplet and its interaction with the diffuse porous obstacle.</em>
</p>

The corresponding simulation data can be visualized using ParaView from the VTU files stored in the `phase/`, `chem/`, and `state/` output directories.




## Author

John Sebastian Simon
