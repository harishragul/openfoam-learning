# Lesson 4.3 — Pressure Boundary Conditions

## Concept

Pressure is not like velocity. Velocity is a real physical quantity that gets transported and diffused by the Navier-Stokes momentum equation. Pressure in an incompressible solver is different — it's whatever field makes the velocity field satisfy continuity (∇·U = 0). It is solved via a Poisson equation, and its boundary conditions have a different character from velocity's.

---

## The Golden Rule, Restated

At every patch: fix EITHER U or p, never both, never neither. This is because the pressure-velocity coupling (PISO/SIMPLE) needs exactly one constraint per patch to close the system.

```
U fixed (you know velocity)   →   p floats (zeroGradient)
p fixed (you know pressure)   →   U floats (inletOutlet / pressureInletOutletVelocity)
```

---

## Pressure BC Types

| BC Type | Meaning | Use Case |
|---------|---------|----------|
| `fixedValue` | Exact static pressure at the patch | Outlet to atmosphere (p=0 gauge), known back-pressure |
| `zeroGradient` | Pressure floats, no gradient enforced | Paired with fixed-velocity inlets and solid walls |
| `totalPressure` | Total (stagnation) pressure fixed; static pressure adjusts based on local velocity | Inlet where you know a fan/compressor's total pressure, not velocity directly |
| `freestreamPressure` | Switches between fixed and floating based on local flow direction | External aero far-field, paired with `freestreamVelocity` |
| `prghPressure` | Pressure with the hydrostatic component (ρgh) subtracted out | Buoyancy-driven flows — paired with `p_rgh` field in buoyant solvers |
| `fixedFluxPressure` | Adjusts so the pressure-driven flux at the wall exactly matches the (often zero) velocity flux | Walls in buoyant or compressible solvers, where gravity creates a flux mismatch that plain `zeroGradient` can't resolve |
| `waveTransmissive` | Allows pressure waves to exit without reflecting back | Compressible flow outlets (avoids spurious wave reflection) |

---

## totalPressure — Worked Example

Total pressure (Bernoulli) combines static and dynamic pressure:

```
p0 = p + 0.5 * |U|^2     (kinematic form, since OpenFOAM stores p/ρ)
```

If you know a fan delivers total pressure `p0 = 50 m²/s²` at an inlet, but you don't know the exact velocity (it depends on what's downstream), `totalPressure` lets the solver work out the static pressure and velocity together:

```cpp
inlet
{
    type    totalPressure;
    p0      uniform 50;
}
```

This is paired with `pressureInletVelocity` for U — velocity at the inlet is *derived*, not prescribed:

```cpp
inlet
{
    type    pressureInletVelocity;
    value   uniform (0 0 0);
}
```

---

## Why prghPressure Exists — The Hydrostatic Problem

In a tall domain with gravity, the static pressure varies hugely with height even with no flow at all (just ρgh from the weight of air/fluid above). If the solver tried to resolve total pressure directly, this huge background gradient would swamp the much smaller pressure variations actually caused by flow — leading to numerical inaccuracy.

`p_rgh` (and its BC `prghPressure`) is pressure with that hydrostatic part already subtracted:

```
p_rgh = p - ρ*g·h
```

The solver only has to resolve the *dynamic* part — the part that actually drives motion. This is why your heatedroom case correctly used `prghPressure` at the vent: buoyancy problems always need this treatment.

---

## fixedFluxPressure — Walls in Buoyant/Compressible Flows

For a simple incompressible case without gravity, `zeroGradient` at a wall is fine — there's no flux imbalance to correct. But once gravity (or compressibility) enters the picture, the pressure gradient at a wall needs to exactly balance the body force so that the velocity flux through the wall stays zero (no flow through solid boundaries). `fixedFluxPressure` computes whatever pressure gradient is needed to guarantee that — `zeroGradient` can't do this automatically.

Rule of thumb: incompressible, no gravity → `zeroGradient` at walls. Buoyant or compressible → `fixedFluxPressure` at walls.

---

## Your Cases, Revisited

| Case | Inlet p | Why |
|------|---------|-----|
| pipeflow | `zeroGradient` | U fixed at inlet, no gravity, simple incompressible |
| windtunnel | `freestreamPressure` | External aero far-field, paired with `freestreamVelocity` |
| heatedroom | `prghPressure` | Buoyancy-driven — needs hydrostatic component removed |

---

## Exercise 4C — Pressure BC Selection

For each scenario, pick the correct pressure BC and explain why:

1. A centrifugal fan inlet where the manufacturer specifies total pressure rise, not velocity.
2. A compressible nozzle outlet, where you want pressure waves to leave cleanly without reflecting back into the domain.
3. Revisit the `heatedroom` case: should the `walls` and `radiator` patches use `zeroGradient` or `fixedFluxPressure` for `p_rgh`? (Hint: does this case have gravity?)

---

## Teach-Back

> "Why can't you just always use `fixedValue 0` for pressure everywhere it's not the obvious outlet? What goes wrong?"

---

## Key Takeaways

- Pressure is solved, not transported — its BCs follow different logic from velocity's.
- Golden rule unchanged: fix U or p per patch, never both, never neither.
- `totalPressure` + `pressureInletVelocity` is the pair for known stagnation pressure inlets (fans, compressors).
- `prghPressure` removes the hydrostatic background so buoyancy solvers resolve only the dynamic pressure.
- `fixedFluxPressure` replaces `zeroGradient` at walls whenever gravity or compressibility creates a flux imbalance.
- `freestreamPressure` is the external-aero far-field partner to `freestreamVelocity`.
