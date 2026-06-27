# Lesson 4.2 — Velocity Inlet/Outlet Conditions

## Concept

Lesson 4.1 introduced `fixedValue`, `noSlip`, `inletOutlet`, and `zeroGradient` at a basic level. This lesson goes deeper into the **velocity-specific** BC types — when each one applies, and how to drive an inlet from something other than a single flat number.

---

## The Velocity BC Family

| BC Type | What it does | When to use |
|---------|--------------|-------------|
| `fixedValue` | Exact U at every point on the patch, fixed for all time | Known, constant inlet velocity |
| `noSlip` | U = 0 (shorthand for `fixedValue uniform (0 0 0)`) | Solid stationary walls |
| `zeroGradient` | Copies whatever U is just inside the domain | Rarely correct on its own for U — usually a sign you should use `inletOutlet` instead |
| `inletOutlet` | `zeroGradient` when flow leaves, `fixedValue` (= `inletValue`) when flow reverses in | Outlets that might see backflow |
| `pressureInletOutletVelocity` | Velocity computed from the local pressure gradient — direction not prescribed | Openings/vents where flow direction is unknown (driven by pressure, not by you) |
| `pressureInletVelocity` | Velocity magnitude derived from a fixed total pressure at the inlet | Inlet where you know pressure, not velocity |
| `freestreamVelocity` | Acts as `fixedValue` for inflow, `zeroGradient`-like for outflow, switching automatically | External aerodynamics far-field |
| `flowRateInletVelocity` | You specify a volumetric or mass flow rate; OpenFOAM computes the velocity profile that matches it | Known flow rate but unknown/irrelevant velocity profile shape |
| `movingWallVelocity` | Wall velocity follows mesh motion (for moving/rotating geometry) | Rotating machinery, oscillating walls |

The common thread: **every one of these is still either Dirichlet, Neumann, or a smart switch between the two** — the categories from Lesson 4.1 never go away, OpenFOAM just gives you more sophisticated versions.

---

## Beyond a Flat Number — Non-Uniform Inlet Profiles

So far every example has used `uniform (U 0 0)` — the same velocity at every point on the patch. Real inlets are rarely uniform. A few ways to prescribe a *shaped* profile:

### 1. `codedFixedValue` — write the profile as inline C++

```cpp
inlet
{
    type            codedFixedValue;
    value           uniform (0 0 0);

    name            parabolicInlet;

    code
    #{
        const fvPatch& boundaryPatch = patch();
        vectorField& field = *this;

        const vectorField& Cf = boundaryPatch.Cf();   // face centres
        scalar R = 0.05;                                // pipe radius (m)
        scalar Umax = 2.0;                               // centerline velocity

        forAll(Cf, i)
        {
            scalar r = mag(Cf[i].y() - R);   // distance from centerline
            field[i] = vector(Umax * (1.0 - sqr(r/R)), 0, 0);
        }
    #};
}
```

This computes `U(r) = Umax * (1 - (r/R)²)` — the classic parabolic profile for fully-developed laminar pipe flow. OpenFOAM compiles this code at runtime (you'll see a compilation step in the log the first time it runs).

### 2. `mapped` / `timeVaryingMappedFixedValue` — read a profile from a file

Useful when you have experimental or precomputed data (e.g., from a RANS precursor simulation) and want to impose it exactly rather than deriving a formula.

### 3. `flowRateInletVelocity` — let OpenFOAM derive the profile from a flow rate

```cpp
inlet
{
    type            flowRateInletVelocity;
    volumetricFlowRate   0.002;   // m^3/s
}
```

OpenFOAM computes whatever uniform velocity satisfies that flow rate over the patch area — simpler than `codedFixedValue` when you don't care about profile shape, only total throughput.

---

## Worked Example: Why a Flat Profile Is Sometimes Wrong

Consider pipe flow. If you set `fixedValue uniform (2 0 0)` at the inlet, you are telling OpenFOAM that *every point* across the pipe cross-section moves at exactly 2 m/s — including right next to the wall. But the wall has `noSlip` — U = 0 there. These two conditions don't contradict each other (they're on different patches), but physically, a flat inlet profile takes some distance downstream to "relax" into the real parabolic shape — this is called the **entrance length**.

```
Flat inlet profile:        →→→→→→→→→
                            →→→→→→→→→     (unrealistic at the wall edge)
                            →→→→→→→→→

After entrance length:        →→→
                             →→→→→→→
                            →→→→→→→→→    (parabolic, physically correct)
```

If your domain is short, you may never reach fully-developed flow, and a flat inlet profile gives you wrong answers in the whole domain. Using `codedFixedValue` to impose the parabolic shape directly at the inlet skips the entrance length problem entirely — useful when you're modeling a short pipe segment and need realistic flow from the very first cell.

---

## Exercise 4B — Parabolic Velocity Inlet

For the `pipeflow` case (diameter 0.05 m, you set bulk velocity 2 m/s earlier):

1. Compute the centerline velocity `Umax` for a parabolic profile with the same bulk (mean) velocity of 2 m/s.
   (Hint: for laminar pipe flow, `U_mean = Umax / 2`, so `Umax = 2 * U_mean`.)
2. Write the `codedFixedValue` block for `0/U` at the inlet using that `Umax`.
3. Conceptual: would a turbulent pipe flow's velocity profile be flatter or more peaked than the laminar parabola? Why?

---

## Teach-Back

> "Your colleague sets `fixedValue uniform (5 0 0)` at a short pipe's inlet and gets weird results near the inlet that smooth out further downstream. What's actually happening, and what would you change?"

---

## Key Takeaways

- All velocity BCs are still Dirichlet, Neumann, or a hybrid — just specialized for different physical situations.
- `inletOutlet` handles backflow at outlets; `pressureInletOutletVelocity` handles unknown-direction openings (vents).
- A uniform/flat inlet profile is a simplification — real profiles develop a shape (e.g., parabolic for laminar) over the entrance length.
- `codedFixedValue` lets you impose any mathematical profile directly via inline C++, compiled at runtime.
- `flowRateInletVelocity` is the right tool when you know throughput but not profile shape.
- Use a shaped inlet profile when your domain is too short to reach fully-developed flow naturally.
