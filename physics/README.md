# Physics — LitePhysics

A 2D physics engine built from scratch, using **Box2D as the reference text** (`box2d-lite` for the
solver, Box2D.NET for collision detection) and **BepuPhysics as the advanced architecture chapter**.

📋 **[LEARNING_PLAN.md](LEARNING_PLAN.md)** — the curriculum: phases, lessons, and checkpoints.

## Status

| Phase | Lessons | Status |
|---|---|---|
| A — Motion | 1. Bodies, forces, integration · 2. Fixed timestep | 🟡 Lesson 1 |
| B — Collision detection | 3. Shapes and broad phase · 4. Narrow phase and manifolds | ⚪ |
| C — Collision response | 5. Impulses · 6. Friction · 7. Sequential impulses and stacking | ⚪ |
| D — Rotation | 8. Rotational dynamics | ⚪ |
| E — Constraints | 9. Joints and the constraint view | ⚪ |
| F — Architecture | 10. Data-oriented design | ⚪ |
| G — Stride payoff | 11. Rendering and community examples | ⚪ |

## Layout

```
LEARNING_PLAN.md          the curriculum
Physics.slnx              solution for this subject only
docs/lessons/             theory notes, written before the code
src/LitePhysics/          the engine, growing lesson by lesson
samples/LessonXX.Name/    one runnable demo per lesson
tests/LitePhysics.Tests/  the "physics lab" - where behaviour is verified
```

## Running

```bash
dotnet build physics/Physics.slnx
dotnet run --project physics/samples/Lesson01.Integration
dotnet test physics/tests/LitePhysics.Tests
```

## The rule that makes this work

Every checkpoint is verified against something that cannot be argued with: a closed-form solution, a
conservation law, or a reference implementation. Free fall has an exact answer. So does a spring. A
ball dropped from height `h` with restitution `e` returns to `e²·h`.

"Looks right" is not a checkpoint.
