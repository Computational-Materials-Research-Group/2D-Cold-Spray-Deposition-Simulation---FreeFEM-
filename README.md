# 2D Cold Spray Deposition Simulation - FreeFEM++

<p align="center">
  <img src="https://img.shields.io/badge/FreeFEM++-Simulation-blue?style=for-the-badge&logo=gnu&logoColor=white"/>
  <img src="https://img.shields.io/badge/Cold%20Spray-Impact%20Dynamics-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Johnson--Cook-Plasticity-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/ParaView-VTK%20Export-green?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/DOI-10.5281%2Fzenodo.19709385-blue?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/License-CC%20BY--NC%204.0-lightgrey?style=for-the-badge"/>
</p>

<p align="center">
  A 2D finite element simulation of <b>cold spray particle deposition</b> using FreeFEM++.
  Models a copper particle impacting an aluminium substrate at 500 m/s with
  Johnson-Cook elasto-viscoplastic material behaviour and adiabatic heating.
</p>

<img width="1008" height="772" alt="COL 0009" src="https://github.com/user-attachments/assets/2a357b91-f82d-400f-8676-7c492c51ae49" />

---

## Citation

If you use this code in your research, please cite:

```bibtex
@software{mishra_2026_coldspray,
  author    = {Mishra, A.},
  title     = {2D Cold Spray Deposition Simulation - FreeFEM++},
  year      = {2026},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.19709385},
  url       = {https://doi.org/10.5281/zenodo.19709385}
}
```

Plain text citation:

> Mishra, A. (2026). *2D Cold Spray Deposition Simulation - FreeFEM++*. Zenodo. https://doi.org/10.5281/zenodo.19709385

---

## Physics

Cold spray is a solid-state deposition process where metallic particles are accelerated to supersonic velocities and impact a substrate below their melting point. Bonding occurs through severe plastic deformation at the interface, driven by adiabatic shear instability when the impact velocity exceeds a material-dependent critical value.

This simulation models the following coupled phenomena:

- Large deformation elasto-plasticity with inertia (dynamic impact)
- Johnson-Cook strain hardening, strain rate hardening, and thermal softening
- Adiabatic temperature rise from plastic dissipation (Taylor-Quinney coefficient)
- Spatially varying material properties (copper particle + aluminium substrate in one mesh)

---

## Geometry

```
          [Copper particle - circle, D=25 um]
          cx = Lsub/2,  cy = Hsub + Rp

  |-------------------------------------------|
  |                                           |  Hsub = 5D
  |         [Aluminium substrate]             |
  |                                           |  Lsub = 10D
  |___________________________________________|
```

The particle base sits directly on the substrate top surface with shared mesh nodes at the interface. No gap is used so contact is enforced through mesh continuity rather than a contact algorithm.

---

## Material Parameters

### Copper Particle

| Parameter | Symbol | Value | Unit |
|-----------|--------|-------|------|
| Density | rhoP | 8960 | kg/m3 |
| Young modulus | EP | 124 | GPa |
| Poisson ratio | nuP | 0.34 | - |
| JC yield stress | AP | 90 | MPa |
| JC hardening | BP | 292 | MPa |
| JC hardening exp | nJCP | 0.31 | - |
| JC strain rate | CP | 0.025 | - |
| Melting temp | TmP | 1356 | K |
| Specific heat | CpP | 385 | J/kg/K |
| Taylor-Quinney | chiP | 0.9 | - |

### Aluminium Substrate

| Parameter | Symbol | Value | Unit |
|-----------|--------|-------|------|
| Density | rhoS | 2700 | kg/m3 |
| Young modulus | ES | 70 | GPa |
| Poisson ratio | nuS | 0.33 | - |
| JC yield stress | AS | 110 | MPa |
| JC hardening | BS | 150 | MPa |
| JC hardening exp | nJCS | 0.36 | - |
| JC strain rate | CS | 0.001 | - |
| Melting temp | TmS | 933 | K |
| Specific heat | CpS | 900 | J/kg/K |
| Taylor-Quinney | chiS | 0.9 | - |

---

## Constitutive Model

### Johnson-Cook Flow Stress

```
sigY = (A + B * epspeq^n) * (1 + C * ln(deps/deps0)) * (1 - Tstar^m)
```

Where:
- `epspeq` = equivalent plastic strain (accumulated)
- `deps` = current strain rate
- `Tstar = (T - T0) / (Tm - T0)` = homologous temperature
- When Tstar approaches 1, flow stress collapses to near zero (adiabatic shear instability = bonding)

### Adiabatic Heating

```
dT = chi * sigY * d_epspeq / (rho * Cp)
```

Where `chi = 0.9` is the Taylor-Quinney coefficient (fraction of plastic work converted to heat).

### Dynamic Equation of Motion

```
rho * d2u/dt2 - div(sigma) = f_body
```

Discretised using central difference (explicit) time integration with inertia on the left hand side to resist the impact body force.

---

## Numerical Method

| Aspect | Choice |
|--------|--------|
| Spatial discretisation | Finite Element Method (FEM) |
| Element type | P2 (quadratic) for displacement, P1 (linear) for scalar fields |
| Time integration | Explicit central difference |
| Mesh | Fixed (Eulerian) - no movemesh |
| Material assignment | P0 indicator function per element |
| Contact | Shared nodes at particle-substrate interface |
| Mesh refinement | adaptmesh once before loop, refined at contact zone |

The mesh is fixed throughout the simulation (Eulerian formulation). Deformation is visualised in ParaView using the Warp By Vector filter applied to the displacement field.

---

## Output Fields

Each `.vtu` file contains the following fields:

| Field | Description | Typical range |
|-------|-------------|--------------|
| `ux` | Horizontal displacement | nm scale |
| `uy` | Vertical displacement (penetration) | nm to um |
| `epspeq` | Equivalent plastic strain | 0 to 5+ at interface |
| `vonMises` | Von Mises equivalent stress | 0 to 500+ MPa |
| `Temp` | Temperature | 300 to 1356 K |
| `sigY` | Johnson-Cook flow stress | 90 to 400 MPa |

---

## Repository Structure

```
cold-spray-freefem/
|
|-- cold3.edp                  # Main FreeFEM++ simulation script
|-- coldspray_results/
|   |-- coldspray.pvd          # ParaView collection file
|   |-- result_004.vtu         # Time step 4
|   |-- result_008.vtu         # Time step 8
|   |-- ...                    # Every 4 steps up to result_200.vtu
|-- README.md
```

---

## How to Run

### Requirements

- FreeFEM++ v4.10 or later: https://freefem.org
- ParaView v5.x or later: https://www.paraview.org

### Step 1 - Run the simulation

```bash
FreeFem++ cold3.edp
```

The script will:
1. Build the combined particle-substrate mesh
2. Refine the mesh near the contact zone
3. Run 200 dynamic time steps at dt = 2e-9 s
4. Save VTU files every 4 steps to `coldspray_results/`
5. Write `coldspray.pvd` linking all files with time values

Console output per step:
```
Step 4/200  t=8e-09  uy_min=-3.2e-07  sigvm_max=1.4e+08  Tmax=312.4
Step 8/200  t=1.6e-08  uy_min=-6.1e-07  sigvm_max=2.1e+08  Tmax=318.7
```

### Step 2 - Open in ParaView

1. File > Open > navigate to `coldspray_results/`
2. Change Files of type to `All Files (*.*)`
3. Select `coldspray.pvd` > OK
4. Choose PVD Reader when prompted > OK
5. Click Apply
6. Set colour field to `epspeq` or `vonMises`
7. Click Rescale to Data Range Over All Timesteps

### Step 3 - Add deformation warp

```
Filters > Search > Warp By Vector
  Vectors:      ux uy
  Scale Factor: 500
  Apply
  Colour by:    epspeq
```

Press Play to watch the impact animation.

---

## What to Look for in Results

### Equivalent Plastic Strain (epspeq)

The most physically meaningful output. A mushroom-shaped high epspeq zone should form at the base of the particle and a matching crater in the substrate surface. Values above 1.5 at the interface indicate adiabatic shear instability and successful bonding.

### Temperature (Temp)

Peak temperature occurs at the particle-substrate interface. If it approaches the melting point of aluminium (933 K) the simulation has captured the thermal softening that enables bonding.

### Von Mises Stress

Shows the stress wave propagating through both particle and substrate after impact. The peak migrates from the contact zone outward as the elastic wave travels through the material.

---

## Common Errors and Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `syntax error: _` | Underscores not allowed in variable names | Rename `ev_xx` to `evxx` |
| `syntax error: load` | Reserved keyword | Rename variable to `trac` |
| `syntax error: ?` | Nested ternary not supported | Use `if / else if / else` |
| `func int` invalid | Only `func` (real) supported | Use `func isPart = ... ? 1. : 0.` |
| `ux = 0.` fails | Vector FE component cannot be assigned directly | Use `ux[] = 0.` |
| Em dash in string | Non-ASCII character causes lex error | Replace with plain hyphen |
| Blow-up (values 1e+270) | Explicit scheme unstable, dt too large | Switch to implicit Newmark-beta |
| movemesh: triangles reversed | Elements invert under large deformation | Use fixed Eulerian mesh instead |
| Bamg meshing error 1100 | adaptmesh called on distorted moved mesh | Only call adaptmesh on clean geometry |

---

## Extending the Model

| Extension | What to change |
|-----------|---------------|
| Different materials | Update rhoP, EP, AP, BP etc. with new Prony series |
| Higher impact velocity | Change `vimp`, reduce `dt` for stability |
| Multiple particles | Add more border circles, extend indicator func |
| 3D model | Replace mesh with mesh3, use 6-component strain tensor |
| Kelvin-Voigt substrate | Add viscous term to substrate stiffness |
| Thermal conduction | Add `int2d(Th)(k * (dx(T)*dx(v) + dy(T)*dy(v)))` solve |

---

## Author

**akshansh11**
GitHub: https://github.com/akshansh11

---

## License

<p>
<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">
<img alt="Creative Commons Licence" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png"/>
</a>
<br/>
This work is licensed under a
<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">Creative Commons Attribution-NonCommercial 4.0 International License</a>.
</p>

You are free to:

- **Share** - copy and redistribute the material in any medium or format
- **Adapt** - remix, transform, and build upon the material

Under the following terms:

- **Attribution** - You must give appropriate credit to akshansh11 and provide a link to this repository
- **NonCommercial** - You may not use the material for commercial purposes

Copyright 2025 akshansh11. All rights reserved for commercial use.
