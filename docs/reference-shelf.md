# Reference shelf

What to read, per subject, in roughly the order it becomes useful. Deliberately short — a shelf, not
a library. Anything that has not earned its place through an actual lesson does not belong here.

## Shared

- **Glenn Fiedler, [*Fix Your Timestep*](https://gafferongames.com/post/fix_your_timestep/)** — the
  accumulator pattern. Relevant to both subjects; read it once, early.
- **Jason Gregory, *Game Engine Architecture*** — the canonical survey of how engines are put
  together. Best used as a reference to dip into per lesson, not read front to back.

## Physics

In reading order for `LEARNING_PLAN.md`:

- **Randy Gaul, [*How to Create a Custom 2D Physics Engine*](https://gamedevelopment.tutsplus.com/series/how-to-create-a-custom-physics-engine--gamedev-12715)**
  — four-part tutorial; the companion for Phases A–C.
- **Erin Catto, [GDC slides](https://box2d.org/publications/)** — *Sequential Impulses* for Lesson 7,
  the rest for Lessons 8–9. Dense, and worth the effort; this is the source material for how modern
  2D solvers work.
- **Ian Millington, *Game Physics Engine Development*** — the book-length version of Phases A–D.
- **[Box2D.NET](https://github.com/ikpil/Box2D.NET) source** — the answer key from Lesson 4 onward.
  Read it *after* attempting each lesson, never before.
- **[BepuPhysics v2](https://github.com/bepu/bepuphysics2) source + docs** — Lesson 10, the
  data-oriented architecture chapter.

## Engine

In reading order for `engine/LEARNING_PLAN.md`:

- **Dmitry Sokolov, [*tinyrenderer*](https://github.com/ssloy/tinyrenderer)** — a software rasteriser
  in ~500 lines, taught properly. The spine of Phase C; do it in C# rather than C++.
- **[Scratchapixel](https://www.scratchapixel.com/)** — rasterisation and perspective derived from
  first principles. Companion to Lessons 5–6 when tinyrenderer moves too fast.
- **Akenine-Möller et al., *Real-Time Rendering*** — the rendering reference. Dip in; do not attempt
  cover to cover.
- **[LearnOpenGL](https://learnopengl.com/)** — the concepts transfer cleanly even though the API
  differs. Useful for Lesson 8, when the hand-written pipeline meets real hardware.
- **[Stride](https://github.com/stride3d/stride) source** — the answer key from Lesson 1 onward.
  Highest-value files: `sources/engine/Stride.Games/GameBase.cs` (the loop),
  `Stride.Engine` (entities and processors), `Stride.Rendering` (the compositor and render features),
  `Stride.Graphics.Regression` (how the engine tests itself by rendering).
- **[Stride Community Toolkit](https://github.com/stride3d/stride-community-toolkit) source** — my own
  code, re-read as a student rather than as a maintainer. Several lessons explain things I already
  shipped without fully understanding.

## A note on using source as a textbook

Both subjects lean on "read the real implementation" — but only *after* attempting the lesson. Read
first and you inherit someone else's solution without the understanding that produced it; the bug you
did not hit is the lesson you did not learn.

The productive order is: attempt, fail, read, then explain the difference in the lesson notes.
