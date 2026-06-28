# Lesson 4.4 — Wall Conditions: No-Slip, Slip, Moving Walls

## Concept

Not every solid surface behaves the same way. A real viscous fluid drags against a wall and sticks to it (no-slip). An idealized inviscid fluid, or a wall far enough from what you care about, can be treated as frictionless (slip). And some walls aren't stationary at all — they rotate or translate, and the fluid touching them must move with them.

---

## The Wall Velocity BC Family

| BC | Physical meaning | Use Case |
|----|------------------|----------|
| `noSlip` | U = 0 exactly at the wall (viscous drag, fluid "sticks") | Real solid surfaces: pipe walls, car body, building |
| `slip` / `freeSlip` | Normal velocity = 0 (no penetration), tangential velocity free (no friction) | Inviscid approximation, symmetry-like far-field boundaries, idealized smooth walls |
| `partialSlip` | Blend between no-slip and slip via a `valueFraction` | Surfaces with partial roughness or approximate friction |
| `movingWallVelocity` | Wall velocity follows the **mesh motion** at that patch (used with dynamic mesh) | Pistons, oscillating surfaces, deforming geometry |
| `rotatingWallVelocity` | Computes `U = ω × r` directly, without moving the mesh | Rotating drums, mixers, turbomachinery casings (when you don't need full mesh rotation, e.g. via MRF) |

---

## No-Slip vs Slip — Why the Distinction Matters

`noSlip` enforces the full viscous boundary layer: velocity ramps from 0 at the wall up to the free-stream value over some thin region (Module 03's boundary layer concept). This costs you mesh resolution — you need fine cells near the wall to capture that ramp.

`slip` skips all of that. The fluid is allowed to glide along the surface with zero friction. No boundary layer forms. This is appropriate when:
- You're modeling an inviscid (Euler) flow deliberately
- The "wall" is actually a far-field domain edge, not a real surface (your windtunnel topWall/bottomWall — there's no real wall there, it's just where you chose to stop your computational domain)
- You want a cheap approximation and don't care about near-wall friction effects

**Mistake to avoid**: using `slip` on an actual solid object you care about (like a car body) will give you a *completely different* flow field — no boundary layer, no separation, wrong drag. Always use `noSlip` for real solid surfaces where viscous effects matter.

---

## Worked Example — Your Own Cases

```cpp
// windtunnel: car body is REAL, topWall/bottomWall are FAR-FIELD EDGES
carBody  { type noSlip; }     // viscous effects matter here — drag, boundary layer
topWall  { type freeSlip; }   // not a real wall — just where the domain ends
bottomWall { type freeSlip; }
```

You already got this right intuitively. This lesson is naming *why* it was right.

---

## Moving Walls — Two Approaches

### Approach 1: `movingWallVelocity` (mesh actually moves)

Used when the mesh itself deforms — e.g. a piston compressing a cylinder. The solver moves mesh points according to a prescribed motion, and `movingWallVelocity` makes the fluid at that boundary match the new wall position's velocity automatically.

```cpp
piston
{
    type    movingWallVelocity;
    value   uniform (0 0 0);
}
```

This requires a `dynamicMeshDict` describing how the mesh deforms — covered in Module 08 (Advanced Topics).

### Approach 2: `rotatingWallVelocity` (mesh stays still, wall velocity computed analytically)

Used for steady rotating machinery where you don't want to actually rotate the mesh (expensive, complex) — instead you tell OpenFOAM the wall is spinning, and it computes velocity from `U = ω × r` at every face on that patch.

```cpp
drum
{
    type            rotatingWallVelocity;
    origin          (0 0 0);
    axis            (0 0 1);
    omega           10;   // rad/s
}
```

This is often combined with the **MRF (Multiple Reference Frames)** technique for turbomachinery, covered later — for now, know that `rotatingWallVelocity` is the BC-level tool for "this surface spins."

---

## Wall Functions — A Preview

When you use a turbulence model (RANS), walls also need BCs for `k`, `epsilon`/`omega`, and `nut` — these use **wall functions** (`kqRWallFunction`, `epsilonWallFunction`, `nutkWallFunction`) which model the near-wall turbulent region analytically rather than resolving it with an extremely fine mesh. Full detail is in Lesson 4.6 (turbulent BCs) — for now, just know: wall functions are a *type of wall BC*, specific to turbulence fields, layered on top of the `noSlip`/`slip` choice you make for `U`.

---

## Exercise 4E — Wall Condition Selection

For each scenario, pick `noSlip`, `slip`/`freeSlip`, or a moving-wall BC, and justify:

1. The inside surface of a pipe carrying water (Re = 5000, turbulent).
2. The far-field top/bottom of a 2D external aerodynamics domain (like your windtunnel) where you've placed the boundary 10 body-lengths away from the object.
3. A washing machine drum spinning at 60 rpm, where you want a steady-state solution without deforming the mesh.
4. A symmetry plane down the centerline of a car (you're only meshing half the car to save cells).

(Hint for #4 — is this really a "wall" at all, conceptually? We'll cover the dedicated `symmetry` BC type properly in Lesson 4.5, but think about what slip vs symmetry have in common.)

---

## Teach-Back

> "Your friend builds a wind tunnel case and accidentally sets the car body to `slip` instead of `noSlip`. The simulation runs fine, no errors, no warnings. What's wrong with the result, and why didn't OpenFOAM catch it?"

---

## Key Takeaways

- `noSlip`: U = 0 at the wall — use for real solid surfaces where viscous drag matters.
- `slip`/`freeSlip`: no penetration, but no friction either — use for far-field domain edges or deliberate inviscid approximations.
- Using `slip` on a real object you care about silently gives you the wrong flow field — no error, just wrong physics.
- `movingWallVelocity` follows actual mesh motion; `rotatingWallVelocity` computes rotation analytically without moving the mesh.
- Wall functions (`kqRWallFunction`, etc.) are wall BCs specific to turbulence fields — covered fully in Lesson 4.6.
