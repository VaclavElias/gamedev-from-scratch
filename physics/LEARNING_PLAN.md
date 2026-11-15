# 2D Physics Engine — Learning Plan

Goal: learn 2D game physics deeply by **building a lite engine from scratch**
("LitePhysics"), using **Box2D as the reference text** (`box2d-lite` for the
solver, Box2D.NET for collision detection) and **BepuPhysics as the advanced
architecture chapter**, so that every lesson can later become a
Stride Community Toolkit example.

## How we work

- **You write the code.** Each lesson I give you: the theory (with the
  derivations, not just formulas), a small spec of what to implement, and
  measurable checkpoints. You implement it in this repo and say when it runs.
- **I review and explain.** Then I review the code, run it, and we
  debug the physics together.
- **Every lesson ends with a demo** — first as console output/tests, later
  rendered — that can be turned into a community example.
- **Verification is analytic where possible.** Free fall has a closed-form
  solution; a spring has one too. We test the simulation against math, not
  against "looks right".

Layout, relative to `physics/`:

```
src/LitePhysics/            engine library (grows lesson by lesson)
samples/LessonXX.Name/      one console/demo app per lesson
tests/LitePhysics.Tests/    xunit "physics lab" tests
docs/lessons/               my theory notes per lesson
```

---

## Phase A — Motion (no collisions yet)

### Lesson 1 — Bodies, forces, and integration
The heart of every engine is three lines of code; the lesson is *which* three.

- **State**: position, velocity, force accumulator; why engines store
  **inverse mass** (cheap math + `invMass = 0` means "static body" with no
  special cases).
- **Newton's second law** as code: `a = F * invMass`.
- **Explicit Euler vs semi-implicit (symplectic) Euler** — same cost, one line
  swapped, completely different long-term behavior. You'll implement both and
  *measure* the energy drift on a spring: explicit Euler explodes, symplectic
  stays bounded forever. This is why Box2D and Bepu both use symplectic Euler.
- **Checkpoints**:
  1. Projectile under gravity matches the closed-form solution
     `y(t) = y0 + v0*t − ½g*t²` within a known error bound (and you can state
     *why* the error is O(dt)).
  2. Spring-mass oscillator: energy bounded with symplectic Euler, grows with
     explicit Euler (print the energy every simulated second).
  3. A static body never moves no matter what forces you apply — and you can say which term of
     the step would have moved it (gravity is an acceleration, so `invMass = 0` does not cancel it).

### Lesson 2 — The fixed timestep and the game loop
Physics wants a fixed `dt`; rendering delivers a variable one.

- Why variable `dt` breaks determinism and stability; the **accumulator
  pattern** ("Fix Your Timestep"); render interpolation.
- Determinism: same inputs → bit-identical results, and why engines care
  (replays, networking, debugging).
- **Checkpoint**: run the same simulation with frame times jittered randomly;
  positions after the same number of steps (600 at 60 Hz, 10 simulated
  seconds) are bit-identical. Compare by step count, not by wall-clock time.

## Phase B — Collision detection

### Lesson 3 — Shapes and the broad phase
- Circles and AABBs; why "does anything touch anything" is O(n²) and how a
  **spatial hash** fixes it (we pick spatial hash over BVH — much easier to
  write, great for learning; Box2D's dynamic BVH is the "further reading").
- A third shape, static only: the **half-plane** (a point and a normal). It is
  the floor, the walls and, tilted, the slope of Lesson 6 — a tilted box would
  need rotation, which waits for Lesson 8. It is unbounded, so it stays out of
  the spatial hash and is tested against every body.
- **Checkpoint**: 5,000 bodies, broad phase returns the same pairs as the
  brute-force check, measurably faster.

### Lesson 4 — Narrow phase and contact manifolds
The geometric core of the engine.

- Circle–circle (trivial), circle–box (closest point), box–box via the
  **Separating Axis Theorem**; circle and box against the half-plane (a
  signed distance, the easiest pair of all).
- What a **contact** really is: point(s), normal, penetration depth. Build
  the manifold with room for **two** points now; why a resting box needs both
  is Lesson 8's question, because without rotation a box cannot rock.
- **Checkpoint**: a table-driven test suite of shape pairs with known
  penetrations/normals; all pass.

## Phase C — Collision response

### Lesson 5 — Impulse resolution
The single most important derivation in game physics.

- Why forces can't stop an already-overlapping pair, and why engines use
  **impulses** (instantaneous velocity changes) instead.
- Deriving the normal impulse from relative velocity + restitution:
  `j = −(1+e)·(v_rel·n) / (invMassA + invMassB)`. You will derive this, not
  copy it.
- **Checkpoint**: two balls collide head-on and bounce with correct momentum
  and restitution (checked against conservation laws); a ball dropped on the
  floor bounces to `e²·h`.

### Lesson 6 — Friction
- Coulomb friction as a **tangent impulse clamped by the normal impulse**
  (the friction cone).
- **Checkpoint**: a box on a slope (a tilted half-plane) stays put below the
  friction angle and slides above it — matching `tan(θ) = μ` from school
  physics.

### Lesson 7 — Sequential impulses, stacking, and stability
Where naive engines die and real ones are born. This is Erin Catto's
signature contribution.

- Solving contacts one at a time, iterating (**Gauss–Seidel / sequential
  impulses**); why one pass isn't enough.
- Penetration recovery: **Baumgarte bias vs position correction**, allowed
  **slop**; accumulated impulses and clamping; **warm starting**.
- **Checkpoint**: a 10-box stack stands still without jitter or sinking.
  Without rotation this is the easy half: the boxes cannot rock, so the stack
  is close to a one-dimensional problem. Lesson 8 runs it again.
- **Answer key**: `box2d-lite`, which is this lesson in about a thousand
  lines. Box2D.NET ports Box2D 3, whose solver has since moved to soft
  constraints and sub-stepping; read it afterwards, as "what changed since".

## Phase D — Rotation

### Lesson 8 — Rotational dynamics
Everything from Lessons 1–7 again, but with torque.

- Orientation, angular velocity, torque, **moment of inertia** (the rotational
  mass); the 2D "cross product" scalar trick.
- Impulses applied at a contact point now cause spin: the full 2D contact
  impulse with rotational terms.
- Oriented boxes: SAT again, with the box's own axes, and a **two-point
  manifold** from clipping. Now a resting box needs both points: give it one
  and watch it rock.
- **Checkpoints**:
  1. A box dropped on its corner tips over and settles flat; a ball with
     friction rolls (spin matches surface speed).
  2. The 10-box stack of Lesson 7 again, with rotation on: it stands without
     jitter, sinking or leaning. Then drop the manifold to one point and
     record what happens.

## Phase E — Constraints and joints

### Lesson 9 — The constraint view of the world
The unifying idea: contacts, joints, and motors are all the *same thing*.

- Position constraints `C(x) = 0`, velocity constraints `J·v = 0`, the
  **Jacobian**; sequential impulses reinterpreted as a constraint solver.
- Implement a **distance joint** and a **revolute joint**; build a pendulum
  and a chain.
- **Checkpoint**: pendulum period matches `2π√(L/g)` for small angles; a
  10-link chain doesn't stretch apart.

## Phase F — Architecture & performance (the Bepu chapter)

### Lesson 10 — Data-oriented design
Now that the physics is understood, study how Bepu makes it fast.

- Array-of-structs vs **struct-of-arrays**; why the solver is memory-bound;
  constraint **batching/islands**; sleeping.
- Port one concept into LitePhysics (e.g. an SoA contact solver) and
  **benchmark before/after**, and against Box2D.NET on the same scene.
- **Checkpoint**: a benchmark table you could publish, with an explanation of
  *why* the numbers moved.

## Phase G — Stride payoff

### Lesson 11 — Rendering and community examples
- Host LitePhysics in a Stride scene (you're the Toolkit expert — I follow
  your lead here); debug-draw contacts, normals, and impulses.
- The unique piece nobody else can write: **"same scene, three engines"** —
  LitePhysics vs Box2D.NET vs Bepu-restricted-to-2D, comparing behavior,
  determinism, and performance.

---

## Reference shelf

The reading list, in the order it becomes useful, is the Physics section of the shared
[reference shelf](../docs/reference-shelf.md).

## Ground rules

- **float, not double** (matches game engines; we'll *see* precision issues
  and learn to live with them, e.g. slop values).
- **No physics NuGet packages** in LitePhysics — `System.Numerics.Vector2`
  is allowed and encouraged.
- **Every formula gets derived once** in the lesson notes before it's allowed
  into the code.
