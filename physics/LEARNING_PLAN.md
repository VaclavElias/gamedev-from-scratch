# 2D Physics Engine — Learning Plan

Goal: learn 2D game physics deeply by **building a lite engine from scratch**
("LitePhysics"), using **Box2D.NET as the reference text** and **BepuPhysics as
the advanced architecture chapter**, so that every lesson can later become a
Stride Community Toolkit example.

## How we work

- **You write the code.** Each lesson I give you: the theory (with the
  derivations, not just formulas), a small spec of what to implement, and
  measurable checkpoints. You implement it in this repo and push.
- **I review and explain.** After you push, I review the code, run it, and we
  debug the physics together. The bugs are the curriculum — sinking boxes and
  jittering stacks teach more than any article.
- **Every lesson ends with a demo** — first as console output/tests, later
  rendered — that can be turned into a community example.
- **Verification is analytic where possible.** Free fall has a closed-form
  solution; a spring has one too. We test the simulation against math, not
  against "looks right".

Repo layout:

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
  3. A static body never moves no matter what forces you apply.

### Lesson 2 — The fixed timestep and the game loop
Physics wants a fixed `dt`; rendering delivers a variable one.

- Why variable `dt` breaks determinism and stability; the **accumulator
  pattern** ("Fix Your Timestep"); render interpolation.
- Determinism: same inputs → bit-identical results, and why engines care
  (replays, networking, debugging).
- **Checkpoint**: run the same simulation with frame times jittered randomly;
  positions after 10 simulated seconds are bit-identical.

## Phase B — Collision detection

### Lesson 3 — Shapes and the broad phase
- Circles and AABBs; why "does anything touch anything" is O(n²) and how a
  **spatial hash** fixes it (we pick spatial hash over BVH — much easier to
  write, great for learning; Box2D's dynamic BVH is the "further reading").
- **Checkpoint**: 5,000 bodies, broad phase returns the same pairs as the
  brute-force check, measurably faster.

### Lesson 4 — Narrow phase and contact manifolds
The geometric core of the engine.

- Circle–circle (trivial), circle–box (closest point), box–box via the
  **Separating Axis Theorem**.
- What a **contact** really is: point(s), normal, penetration depth — and why
  a box resting on the ground needs **two** contact points (a manifold), not
  one, or it tips over.
- **Checkpoint**: a table-driven test suite of shape pairs with known
  penetrations/normals; all pass.

## Phase C — Collision response (the famous part)

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
- **Checkpoint**: a box on a slope stays put below the friction angle and
  slides above it — matching `tan(θ) = μ` from school physics.

### Lesson 7 — Sequential impulses, stacking, and stability
Where naive engines die and real ones are born. This is Erin Catto's
signature contribution.

- Solving contacts one at a time, iterating (**Gauss–Seidel / sequential
  impulses**); why one pass isn't enough.
- Penetration recovery: **Baumgarte bias vs position correction**, allowed
  **slop**; accumulated impulses and clamping; **warm starting**.
- **Checkpoint**: a 10-box stack stands still without jitter or sinking.
  (This is the "hello world" of solver quality — expect to fail it first,
  that's the lesson.)

## Phase D — Rotation

### Lesson 8 — Rotational dynamics
Everything from Lessons 1–7 again, but with torque.

- Orientation, angular velocity, torque, **moment of inertia** (the rotational
  mass); the 2D "cross product" scalar trick.
- Impulses applied at a contact point now cause spin: the full 2D contact
  impulse with rotational terms.
- **Checkpoint**: a box dropped on its corner tips over and settles flat; a
  ball with friction rolls (spin matches surface speed).

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

## Reference shelf (in reading order)

- Randy Gaul, *How to Create a Custom 2D Physics Engine* (4-part tutorial) —
  companion for Phases A–C.
- Erin Catto, *Sequential Impulses* GDC slides (2006+) — companion for
  Lesson 7; his other GDC talks for Lessons 8–9.
- Ian Millington, *Game Physics Engine Development* — the book version of
  Phases A–D.
- Glenn Fiedler, *Fix Your Timestep* — Lesson 2.
- Box2D.NET source — keep open from Lesson 4 onward as the "answer key".
- BepuPhysics v2 source + its documentation on solver design — Lesson 10.

## Ground rules

- **float, not double** (matches game engines; we'll *see* precision issues
  and learn to live with them, e.g. slop values).
- **No physics NuGet packages** in LitePhysics — `System.Numerics.Vector2`
  is allowed and encouraged.
- **Every formula gets derived once** in the lesson notes before it's allowed
  into the code.
- Expect Lessons 1–6 to move fast with your C# background; Lesson 7 is where
  we slow down on purpose.
