# Lesson 4.5 — Cyclic/Periodic and Symmetry Patches

## Concept

Some domains repeat themselves — either as a mirror image (symmetry) or as a repeating pattern that wraps around (periodicity). Exploiting this lets you simulate a fraction of the real geometry and get the full answer, at a fraction of the computational cost.

---

## 1. Symmetry — `symmetry` / `symmetryPlane`

Covered conceptually last lesson: a plane where the domain is the mirror image of itself. No real solid surface — just a computational shortcut.

```cpp
midplane
{
    type    symmetryPlane;   // or symmetry, for non-planar symmetry surfaces
}
```

`symmetryPlane` requires the patch to be geometrically flat (which blockMesh/snappyHexMesh patches usually are). `symmetry` is the more general version for curved symmetry surfaces (e.g. an axisymmetric body's wedge boundary, when not using `wedge` type directly).

**Key requirement**: the mesh on either side of the symmetry plane must actually be mirror images of each other for the result to be physically valid. If your real geometry has any asymmetric feature (an exhaust pipe on one side only, a steering wheel offset, etc.), `symmetry` is invalid — you must mesh the full domain.

---

## 2. Periodicity — `cyclic`

A periodic (cyclic) boundary says: "whatever leaves through this face re-enters through the matching face on the other side, as if the domain repeated infinitely in that direction."

```
Domain with cyclic boundary in x:

  →→→ [CELL A] →→→ [CELL B] →→→ [CELL C] →→→
  ↑                                          ↓
  └──────────────── wraps around ────────────┘
```

Classic use case: **turbulent channel flow** or **fully-developed pipe flow** where you don't want entrance/exit effects at all — you want the flow to behave as if the pipe were infinitely long. Flow exiting the right face is fed directly back in on the left face.

```cpp
inlet
{
    type        cyclic;
    neighbourPatch  outlet;
}
outlet
{
    type        cyclic;
    neighbourPatch  inlet;
}
```

Both faces must have **identical geometry** (same point count, same shape) — OpenFOAM matches them face-by-face.

### Driving Flow Through a Cyclic Domain

If both ends are cyclic, there's no pressure difference to push the flow — pressure is also periodic! You need an external driving force. The usual tool: a **momentum source term** (`meanVelocityForce` function object) that adds exactly enough body force each timestep to maintain a target mean velocity, mimicking the effect of a pressure gradient without actually having an inlet/outlet.

---

## 3. `cyclicAMI` — Periodic Boundaries With Mismatched Meshes

Regular `cyclic` requires identical face geometry on both sides. `cyclicAMI` (Arbitrary Mesh Interface) relaxes this — useful when the two periodic faces have different mesh resolution or even different shapes that still represent the same physical repeat pattern (common in turbomachinery blade-to-blade periodicity, where blade passages might be meshed independently).

---

## 4. `wedge` — Axisymmetric Geometry

For a problem that's symmetric around a central axis (a nozzle, a rocket, an axisymmetric pipe with no swirl), you don't need a full 3D mesh at all — model a **thin wedge slice** (a few degrees) and tell OpenFOAM the two flat faces of the wedge represent a full rotation:

```cpp
wedgeFront
{
    type    wedge;
}
wedgeBack
{
    type    wedge;
}
```

This is the most extreme cost saving of the family — a full 3D axisymmetric flow becomes effectively a 2D problem.

---

## Decision Table — Which One?

| Geometry feature | BC type | Saves what |
|---|---|---|
| Mirror-symmetric across a flat plane | `symmetryPlane` | Half the domain |
| Mirror-symmetric across a curved surface | `symmetry` | Half the domain |
| Infinitely repeating pattern (channel, periodic array) | `cyclic` | The entire repeat length — only mesh one period |
| Repeating pattern with mismatched mesh on each side | `cyclicAMI` | Same as `cyclic`, with mesh flexibility |
| Fully axisymmetric (no swirl) | `wedge` | Reduces 3D to effectively 2D |

---

## Exercise 4F

For each scenario, choose the right BC type and justify:

1. Flow through a long straight pipe, far from any entrance or exit effects — you want to study fully-developed turbulence statistics.
2. A rocket nozzle with no swirl, where you want a cheap 2D-equivalent simulation.
3. An airfoil in a 2D wind tunnel — you've already exploited the fact that nothing changes along the wing's span by making the domain 1 cell thick (this used `empty`, not symmetry — explain why `empty` was the right choice here, not `symmetryPlane`).

---

## Teach-Back

> "What would go wrong physically if you used `cyclic` on a pipe inlet/outlet, but your case had a heater partway down the pipe that's NOT symmetric or periodic at all?"

---

## Key Takeaways

- `symmetryPlane`/`symmetry`: mirror image across a plane — only valid if the real geometry actually IS symmetric there.
- `cyclic`: flow wraps around — used for infinitely-repeating domains (channel/pipe turbulence studies), needs a momentum source to drive flow since there's no real pressure drop.
- `cyclicAMI`: like `cyclic` but tolerates mismatched mesh resolution between the two periodic faces.
- `wedge`: collapses a fully axisymmetric 3D problem into an effectively 2D one.
- `empty` (from Module 03) is different from all of these — it just means "this direction has no physics at all," not "this direction repeats or mirrors."
