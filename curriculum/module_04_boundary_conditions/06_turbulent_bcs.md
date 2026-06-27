# Lesson 4.6 — Turbulent Boundary Conditions (k, epsilon, omega)

## Concept

When using a turbulence model (RANS), you have additional fields to prescribe BCs for:

- `k` — turbulent kinetic energy (m²/s²)
- `epsilon` (ε) — turbulent dissipation rate (m²/s³)  [k-epsilon model]
- `omega` (ω) — specific dissipation rate (1/s)  [k-omega SST model]
- `nut` — turbulent (eddy) viscosity (m²/s)  [computed, not prescribed]

Getting these wrong leads to incorrect turbulence development and wrong results — even if everything else is correct.

---

## Estimating Inlet Turbulence

At an inlet, you typically don't know k and ε exactly. Use these engineering estimates:

### Turbulence Intensity (I)
```
I = u'/U_mean     (u' = RMS turbulent velocity fluctuation)
```
- Pipe/duct inlet (well-developed): I ≈ 0.05 (5%)
- Wind tunnel (low-turbulence): I ≈ 0.001 (0.1%)
- Fan outlet: I ≈ 0.10–0.20 (10-20%)

### Turbulent Length Scale (l)
For pipe/duct flow:
```
l = 0.07 * D_hydraulic
```

### Computing k and epsilon from I and l

```
k     = 1.5 * (U * I)²
epsilon = C_mu^0.75 * k^1.5 / l     (C_mu = 0.09)
omega = k^0.5 / (C_mu^0.25 * l)
```

**Example**: U = 10 m/s, I = 5%, D = 0.1 m
```
l       = 0.07 * 0.1 = 0.007 m
k       = 1.5 * (10 * 0.05)² = 0.375 m²/s²
epsilon = 0.09^0.75 * 0.375^1.5 / 0.007 = 14.9 m²/s³
omega   = 0.375^0.5 / (0.09^0.25 * 0.007) = 213 1/s
```

---

## Standard Turbulent Inlet BCs

`0/k`:
```cpp
boundaryField
{
    inlet
    {
        type    fixedValue;
        value   uniform 0.375;    // computed k
    }
    outlet
    {
        type    inletOutlet;
        inletValue  uniform 0;
        value   uniform 0;
    }
    walls
    {
        type    kqRWallFunction;  // wall function for k
        value   uniform 0;
    }
}
```

`0/epsilon`:
```cpp
boundaryField
{
    inlet
    {
        type    fixedValue;
        value   uniform 14.9;     // computed epsilon
    }
    outlet
    {
        type    inletOutlet;
        inletValue  uniform 0;
        value   uniform 0;
    }
    walls
    {
        type    epsilonWallFunction;  // wall function for epsilon
        value   uniform 0;
    }
}
```

`0/nut` (turbulent viscosity — computed by the model, not prescribed at walls):
```cpp
boundaryField
{
    inlet
    {
        type    calculated;
        value   uniform 0;
    }
    outlet
    {
        type    calculated;
        value   uniform 0;
    }
    walls
    {
        type    nutkWallFunction;   // wall function for nut
        value   uniform 0;
    }
}
```

---

## Wall Functions

Wall functions model the near-wall region analytically, allowing coarser meshes. They require **y+ ≈ 30–300** (log-law region).

For **resolved walls** (fine mesh, y+ ≈ 1): use `kLowReWallFunction`, `omegaWallFunction` with `blended true`.

| Field | Coarse mesh (y+ 30-300) | Fine mesh (y+ ~1) |
|-------|------------------------|-------------------|
| k     | `kqRWallFunction` | `kLowReWallFunction` |
| epsilon | `epsilonWallFunction` | `epsilonWallFunction` (lowRe) |
| omega | `omegaWallFunction` | `omegaWallFunction` |
| nut   | `nutkWallFunction` | `nutLowReWallFunction` |

---

## Exercise 4C

Set up turbulent BCs for a pipe flow:
- Diameter: 0.05 m, bulk velocity: 5 m/s, ν = 1.5e-5 m²/s (air)
- Turbulence intensity at inlet: 5%
- Use k-omega SST model (k and omega)
- Compute the inlet values for k and omega manually, then write the 0/k and 0/omega files

---

## Key Takeaways

- Turbulent simulations need BCs for k + epsilon (or k + omega), plus nut.
- Estimate inlet k and epsilon from turbulence intensity I and length scale l.
- Wall functions (`kqRWallFunction`, `epsilonWallFunction`) model near-wall physics for y+ 30–300.
- Low-Re wall treatment resolves the viscous sublayer (y+ ~1) without wall functions.
- The `nut` field is typically `calculated` at inlets and uses a wall function at walls.
