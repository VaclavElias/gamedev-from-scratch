# gamedev-from-scratch

Learning game development by **building lite versions of the real thing**, one subsystem at a time.

Each subject follows the same method: build it from scratch in C#, use a production implementation as
the reference text, and verify every lesson against something measurable — a closed-form solution, a
conservation law, or the output of a real engine. Not "looks right".

## Subjects

| Subject | Building | Reference text | Status |
|---|---|---|---|
| [Physics](physics/LEARNING_PLAN.md) | `LitePhysics` — a 2D physics engine | Box2D.NET, then BepuPhysics | 🟡 Lesson 1 |
| [Engine](engine/LEARNING_PLAN.md) | `LiteEngine` — game engine fundamentals | Stride | ⚪ Planned |

Status: ⚪ planned · 🟡 in progress · 🟢 complete

## Why this exists

I maintain the [Stride Community Toolkit](https://github.com/stride3d/stride-community-toolkit) but
came to it as a .NET developer rather than a game developer. The C# was never the gap; the domain
was — what a frame actually is, what the GPU does with a matrix, why engines are shaped the way they
are.

Reading about it did not stick. Building it does.

## How a subject is laid out

```
<subject>/
├── README.md              what this subject covers
├── LEARNING_PLAN.md       the curriculum: phases, lessons, checkpoints
├── <Subject>.slnx         one solution per subject, so each stays fast to open
├── docs/lessons/          theory notes, written before the code
├── src/                   the library being built
├── samples/               one runnable demo per lesson
└── tests/                 xunit tests - the "lab" where behaviour is verified
```

Subjects are independent, but they are allowed to meet: the engine plan hosts `LitePhysics` inside
`LiteEngine` once both are far enough along.

## Ground rules

- **From scratch means from scratch.** No engine or physics packages in the `Lite*` libraries.
  `System.Numerics` is allowed and encouraged.
- **Every formula gets derived once** in the lesson notes before it is allowed into the code.
- **Checkpoints are falsifiable.** If a lesson cannot be verified against maths, a conservation law,
  or a reference implementation, the lesson is not finished being designed.
- **float, not double** — matching real engines, precision problems included.
- The bugs are the curriculum. Sinking boxes and inside-out cubes teach more than any article.

## Shared notes

- [Glossary](docs/glossary.md) — terms that recur across subjects
- [Reference shelf](docs/reference-shelf.md) — books, courses and source to read, per subject
