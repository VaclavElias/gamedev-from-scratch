# Now

The one page to open first: where I am, what is next, and what each finished lesson taught me.
Everything deeper is a link. Updated 2026-11-15.

> **Reprioritised:** (nothing yet - write a line here to reorder, and the queue follows it)

⬜ not started · 🟡 in progress · ⏸ waiting on a decision · ✅ done

## Current lesson

🟡 **Physics 1 - Bodies, forces, and integration** → [notes](physics/docs/lessons/lesson-01-integration.md)

- ✅ Lesson notes written (2026-08-08), corrected 2026-11-15.
- ⬜ `Body`, `World` and the `Integrator` enum in `physics/src/LitePhysics`.
- ⬜ The projectile and spring experiments in `physics/samples/Lesson01.Integration`.
- ⬜ One xunit test per checkpoint in `physics/tests/LitePhysics.Tests` (the project has no test yet).
- ⬜ Review, then the *why* of the spring result.

## Waiting on me

- Write the Lesson 1 code (the list above).
- Answer the engine plan's open questions, Q2 to Q6 → [engine/LEARNING_PLAN.md](engine/LEARNING_PLAN.md#open-questions)

## Queue

In the order I would take them; a phase of one subject, then a phase of the other.

| | Lesson | Checkpoint in one line |
|---|---|---|
| 🟡 | Physics 1 - Integration | Projectile matches the closed form; spring energy bounded or growing by integrator |
| ⬜ | Physics 2 - Fixed timestep | Jittered frame times, bit-identical positions after 600 steps |
| ⬜ | Engine 1 - The loop | Fixed-rate simulation inside a variable-rate loop; find the same loop in `GameBase.cs` |
| ⬜ | Engine 2 - Determinism and state | Update order changes the result in place, and not with double-buffered state |
| ⬜ | Physics 3 - Shapes and broad phase | Spatial hash returns the brute-force pairs for 5,000 bodies, faster |
| ⬜ | Physics 4 - Narrow phase | Table of shape pairs with known normals and depths |
| ⬜ | Engine 3 - Vectors and matrices | Hand-written matrix maths agrees with `System.Numerics` and Stride |
| ⬜ | Engine 4 - Transforms | A child's world matrix matches `TransformComponent.WorldMatrix` |

The rest of each plan: [physics](physics/LEARNING_PLAN.md) · [engine](engine/LEARNING_PLAN.md).

## What I have learned

One line per finished lesson, in my own words, written when the lesson closes. If I cannot write the
line, the lesson is not finished.

| Date | Lesson | The one thing |
|---|---|---|
| | | |

## Done

- 2026-08-08: both learning plans written; the repository restructured into `physics/` and `engine/`.
- 2026-11-15: both plans and Lesson 1 reviewed; AI assistant instructions and this page added.
- 2026-11-15: plans changed after the review: a static half-plane shape (Physics 3, 4, 6), the stack
  checkpoint repeated with rotation (Physics 8), `box2d-lite` as the solver answer key (Physics 7),
  Physics 2 before Engine 1, and a double-buffering checkpoint for Engine 2.
