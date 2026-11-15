# Engine — LiteEngine

Game engine fundamentals built from scratch, using **Stride's source as the reference text** and the
**Stride Community Toolkit as the payoff**.

📋 **[LEARNING_PLAN.md](LEARNING_PLAN.md)** — the curriculum: phases, lessons, checkpoints, and the
open questions still to settle before Lesson 1.

## Status

| Phase | Lessons | Status |
|---|---|---|
| A — The frame | 1. The loop · 2. Determinism and state | ⚪ |
| B — Space | 3. Vectors and matrices · 4. Transforms and the scene graph | ⚪ |
| C — Making pixels | 5. Triangle to screen · 6. Depth and interpolation · 7. Light | ⚪ |
| D — The GPU | 8. What the hardware changes · 9. Shaders | ⚪ |
| E — The scene | 10. Entities, components, systems | ⚪ |
| F — Physics meets engine | 11. Integrating a simulation | ⚪ |
| G — The rest | 12. Content and assets · 13. Stride end to end | ⚪ |
| H — Payoff | Toolkit examples and engine contributions | ⚪ |

Not started — [the open questions](LEARNING_PLAN.md#open-questions) come first, particularly how deep
to go in Phase C.

## Layout

```
LEARNING_PLAN.md          the curriculum
Engine.slnx               solution for this subject only (once Lesson 1 starts)
docs/lessons/             theory notes, written before the code
src/LiteEngine/           the engine, growing lesson by lesson
samples/LessonXX.Name/    one runnable demo per lesson
tests/LiteEngine.Tests/   maths, transforms, determinism
```

## Checkpoints

Checkpoints are verified *against a real engine*, Stride, rather than against intuition: a transform
hierarchy that matches `TransformComponent.WorldMatrix` to float tolerance, a projected point that
matches Stride's own, a frame that renders the same scene.

## Where it meets the physics subject

Phase F hosts [`LitePhysics`](../physics/README.md) inside `LiteEngine` — fixed-step simulation
inside a variable-rate renderer, with the transform-ownership question answered properly. That is
where the two subjects stop being separate.
