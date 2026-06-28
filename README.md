# CFD-Portfolio
While writing this, I am learning OpenFOAM; this portfolio is my tiny attempt to document everything that I learn and the projects that I do along the way. This means it is going to be an evolving repository. In every project I add, I will be prone to keep the following things.

- The working theory in that specific project.
- My personal thoughts
- CFD setup (along with the codes)
- And of course, some cool contours.

---
### Technical Skills to be Demonstrated here:
1. OpenFOAM
2. Finite Volume Method (FVM)
3. Shock Capturing
4. Dynamic Meshing
5. Compressible Flow Simulation
6. Python Automation and Post-Processing.

# Rayleigh-Taylor Instability — OpenFOAM v2512

2D numerical simulation of the Rayleigh-Taylor Instability (RTI) using the Volume of Fluid (VOF) method in ESI OpenFOAM v2512. Case setup follows the benchmark by Popinet (2003).

---

## Physics

The Rayleigh-Taylor Instability occurs at the interface between two fluids of different densities when the denser fluid is placed above the lighter one in a gravitational field. The configuration is gravitationally unstable: any perturbation at the interface grows exponentially in the linear regime, then nonlinearly develops into the characteristic spike-and-bubble topology, with secondary Kelvin-Helmholtz roll-ups forming on the mushroom stems.

The key dimensionless parameter is the **Atwood number**:

$$At = \frac{\rho_h - \rho_l}{\rho_h + \rho_l} = \frac{1.225 - 0.1694}{1.225 + 0.1694} \approx 0.757$$

The linear growth rate is:

$$\sigma = \sqrt{g \cdot At \cdot k - \frac{\sigma_s}{\rho_h + \rho_l} k^3}$$

where $k = 2\pi/\lambda$ is the wavenumber of the initial perturbation.

---

## Numerical Setup

| Parameter | Value |
|---|---|
| Solver | `interFoam` (VOF, incompressible two-phase) |
| OpenFOAM version | ESI v2512 |
| Mesh | 128 × 512 structured quad cells (`blockMesh`) |
| Domain | 1 m × 4 m (2D, single cell in z) |
| Regime | Laminar |
| Pressure-velocity coupling | PISO (PIMPLE, nOuterCorrectors = 1) |
| Interface scheme | MULES-corrected `vanLeer` |
| Time discretisation | Euler (1st-order implicit) |
| Courant limit | Co_max = 0.3, AlphaCo_max = 0.3 |
| Simulated time | 10 s |
| Write interval | 0.02 s (250 frames) |

### Fluid Properties

| Property | Heavy fluid (α = 1) | Light fluid (α = 0) |
|---|---|---|
| Density ρ | 1.225 kg/m³ | 0.1694 kg/m³ |
| Kinematic viscosity ν | 2.555 × 10⁻³ m²/s | 1.847 × 10⁻² m²/s |
| Dynamic viscosity μ | 3.13 × 10⁻³ Pa·s | 3.13 × 10⁻³ Pa·s |

Surface tension σ = 0.01 N/m | Gravity g = (0, −9.81, 0) m/s²

### Initial Condition

The interface is initialized with a single cosine mode:

$$y_{int}(x) = 2.0 + 0.05 \cos(2\pi x)$$

where A = 0.05 m is the perturbation amplitude and λ = 1 m is the wavelength (matching the domain width). The heavy fluid occupies the region y > y_int; the light fluid occupies y < y_int. Initialization is done via `codeStream` directly in `0/alpha.water` — no STL or `setFields` required.

### Boundary Conditions

| Patch | alpha.water | U | p_rgh |
|---|---|---|---|
| top | `inletOutlet` | `pressureInletOutletVelocity` | `totalPressure` |
| bottom | `zeroGradient` | `slip` | `fixedFluxPressure` |
| left | `zeroGradient` | `slip` | `fixedFluxPressure` |
| right | `zeroGradient` | `slip` | `fixedFluxPressure` |
| frontAndBack | `empty` | `empty` | `empty` |

---

## Repository Structure

```
RTI/
├── 0/
│   ├── alpha.water       # Phase fraction — cosine interface via codeStream
│   ├── p_rgh             # Modified pressure (p - ρgh)
│   └── U                 # Velocity
├── constant/
│   ├── g                 # Gravity vector
│   ├── transportProperties
│   └── momentumTransport # Laminar (ESI v2512)
└── system/
    ├── blockMeshDict     # Single hex block, 128×512×1
    ├── controlDict
    ├── fvSchemes
    └── fvSolution
```

---

## How to Run

```bash
# Source OpenFOAM environment
source /usr/lib/openfoam/openfoam2512/etc/bashrc

# Generate mesh
blockMesh

# Verify mesh quality
checkMesh

# Run solver (codeStream compiles alpha initializer on first step)
interFoam 2>&1 | tee log.interFoam

# Post-process in ParaView
paraview foam.foam
```

To clean and rerun from scratch:

```bash
foamListTimes -rm
blockMesh
interFoam 2>&1 | tee log.interFoam
```

---

## Results

Visualize `alpha.water` in ParaView. The four stages visible in the simulation:

1. **Linear regime** (t ≈ 0–0.5 s) — exponential growth of the cosine perturbation
2. **Nonlinear elongation** (t ≈ 0.5–1.5 s) — heavy fluid fingers extend downward, light fluid bubbles rise
3. **Mushroom cap formation** (t ≈ 1.5–2.5 s) — characteristic mushroom topology on the spike tips
4. **Secondary KH roll-ups** (t ≈ 2.5–5 s) — Kelvin-Helmholtz instability develops on the stems

---

## References

- Popinet, S. (2003). Gerris: a tree-based adaptive solver for the incompressible Euler equations in complex geometries. *Journal of Computational Physics*, 190(2), 572–600.
- Tryggvason, G. (1988). Numerical simulations of the Rayleigh-Taylor instability. *Journal of Computational Physics*, 75(2), 235–282.
- OpenFOAM ESI Documentation: https://www.openfoam.com/documentation

---

## Author

**Prithul** — Mechanical Engineering, AUST | AASML Co-founder  
CFD Portfolio: [github.com/yourhandle](https://github.com/yourhandle)
