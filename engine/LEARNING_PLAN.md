# Game Engine Fundamentals — Learning Plan

Goal: understand how a game engine actually works by **building a lite engine from scratch**
("LiteEngine"), using **Stride's source as the reference text** and the **Stride Community Toolkit as
the payoff**, so that every lesson deepens the engine knowledge behind the toolkit you maintain.

Companion to [LEARNING_PLAN.md](LEARNING_PLAN.md) (2D physics). That plan goes deep on one subsystem;
this one goes broad across the engine and then meets it — Phase F is where they join.

## Who this is written for

A senior C#/.NET engineer who maintains a game engine's toolkit but has not built games. So:

- **No C# teaching, no software-engineering teaching.** You have that.
- **The domain is the gap**: what a frame *is*, what the GPU actually does with a matrix, why engines
  are shaped the way they are.
- **Your Stride API knowledge is an asset, not a shortcut.** Repeatedly the lesson will be "you already
  call `game.Create3DPrimitive(...)` — here is what it does, and now you have written it yourself".

## How we work

Same contract as the physics plan:

- **You write the code.** Each lesson: the theory (derived, not quoted), a small spec, and measurable
  checkpoints. You implement and push.
- **I review and explain.** The bugs are the curriculum. A cube that renders inside-out teaches
  winding order better than any diagram.
- **Every lesson ends with something you can see or measure**, and most can become a toolkit example.
- **Verification is against Stride wherever possible.** This is the trick that makes the plan work:
  render the same scene in LiteEngine and in Stride, and compare. A checkpoint that says
  "matches Stride's output" is falsifiable in a way that "looks right" never is.

Repo layout (mirroring the physics repo):

```
src/LiteEngine/            engine library (grows lesson by lesson)
samples/LessonXX.Name/     one console/window app per lesson
tests/LiteEngine.Tests/    xunit tests - math, transforms, determinism
docs/lessons/              your theory notes per lesson
```

---

## Phase A — The frame

### Lesson 1 — The loop
Everything in an engine hangs off one `while (true)`.

- Input → update → render, and why that order is not arbitrary.
- **Variable vs fixed timestep**, the accumulator pattern, and render interpolation. You have already
  built this once in `Example19_Jitter2Physics`; here you derive *why* the alternatives fail.
- Where the time actually goes: vsync, present, frame pacing, and why "60 FPS" is a latency
  statement as much as a throughput one.
- **Checkpoints**:
  1. A loop that simulates at a fixed rate while rendering at a jittered variable rate; simulated
     time tracks wall-clock time to within one step.
  2. Deliberately break it — step by `Elapsed` — and *measure* the speed change under a frame-rate
     change, so the failure is a number, not a feeling.
  3. Read `Stride.Games/GameBase.cs` and find its loop. Explain `IsFixedTimeStep`,
     `WindowMinimumUpdateRate`, and where your accumulator lives in Stride's design.

### Lesson 2 — Determinism and the shape of state
- Why engines care about determinism (replays, networking, debugging, screenshot tests) and why
  float math makes it hard.
- Mutable world state vs the frame that reads it; double buffering; why "read last frame, write this
  frame" removes whole classes of order-dependence bugs.
- **Checkpoint**: same inputs, jittered frame times, bit-identical state after 10 simulated seconds.

## Phase B — Space

### Lesson 3 — Vectors and matrices, geometrically
The goal is intuition, not algebra you can already do.

- Dot product as projection, cross product as area/normal — with the sign conventions that will
  later flip your triangles inside out.
- A matrix as a **change of coordinate frame**: the columns *are* the basis vectors. Once you see
  that, "model → world → view → clip" stops being a magic incantation.
- Row-vector vs column-vector conventions, and why Stride's `Matrix` multiplies the way it does.
- **Checkpoint**: implement `Matrix4x4` operations by hand; property-test against
  `System.Numerics` and against `Stride.Core.Mathematics.Matrix` — including a test that catches a
  transposed convention.

### Lesson 4 — Transforms and the scene graph
- Position/rotation/scale as a matrix; **why quaternions** (gimbal lock, interpolation) and what a
  quaternion actually is geometrically.
- Parent-child hierarchies: local vs world, and the cost of recomputing world matrices.
- **Checkpoint**: a transform hierarchy where a child's world matrix matches Stride's
  `TransformComponent.WorldMatrix` for the same hierarchy, to float tolerance. Then rotate a parent
  and explain, before running it, where the child lands.

## Phase C — Making pixels (a software rasterizer)

No GPU, no graphics API — pure C# writing to a bitmap. This phase is where the pipeline stops being
folklore. It is the single highest-value phase in the plan for someone who has only used engines
from above.

### Lesson 5 — From triangle to screen
- The transform chain, one stage at a time: model → world → view → **clip** → perspective divide →
  **NDC** → viewport. You have already read this code in
  `EntityDebugSceneRenderer` — now you write it.
- Why the divide by `w` *is* perspective; why clip space exists at all rather than dividing early.
- Rasterizing a triangle: edge functions, barycentric coordinates, top-left fill rules.
- **Checkpoint**: a spinning wireframe cube in a PNG sequence, with a projected point verified by
  hand-computed arithmetic.

### Lesson 6 — Depth, winding, and interpolation
- The z-buffer, and why painter's algorithm fails.
- Backface culling and winding order — deliberately reverse it and see the classic inside-out cube.
- Perspective-**correct** interpolation: interpolate `1/w`, not the attribute, and see the wrong
  version's warped texture.
- **Checkpoint**: two intersecting cubes render correctly from any angle; a checkerboard-textured
  floor shows correct perspective (and you can produce the wrong one on demand).

### Lesson 7 — Light
- Surface normals; Lambert diffuse derived from the dot product; why normals need the
  **inverse-transpose** under non-uniform scale (and what breaks if you skip it).
- Flat vs smooth shading — the same hard-vs-smooth normal distinction that makes
  `Example13_MeshOutline` show gaps on a cube.
- **Checkpoint**: a lit sphere; a lit cube with hard edges; explain from your own code why the mesh
  outline technique behaves differently on each.

## Phase D — Handing it to the GPU

### Lesson 8 — What the hardware changes
- Why the GPU's API looks the way it does: buffers, immutable pipeline state, draw calls, and the
  cost of talking to another processor.
- Vertex/index buffers, input layouts, the programmable stages — mapped onto what you wrote by hand
  in Phase C.
- **Checkpoint**: reproduce your Phase C cube using Stride's low-level graphics API directly
  (`GraphicsDevice`, buffers, `MutablePipelineState`) with no scene, no entities. You have seen this
  shape already in `MeshOutlineRenderFeature`.

### Lesson 9 — Shaders
- Vertex and pixel shaders as the programmable slots in the pipeline you built.
- **SDSL** specifically: mixins, composition, and what Stride's shader system buys over raw HLSL.
- Case study: read your own `MeshOutlineShader.sdsl`, explain `inflateAndProject` line by line, then
  write a second effect (e.g. a dissolve or a fresnel rim) from scratch.
- **Checkpoint**: an SDSL shader you wrote, driven by a parameter from C#, working in a Stride scene.

## Phase E — The scene

### Lesson 10 — Entities, components, and systems
- Why engines abandoned deep inheritance for composition; what "data-oriented" actually means and
  when it matters.
- Stride's model specifically: `Entity`, `EntityComponent`, `EntityProcessor`, and how a processor
  finds its components. Compare with Bepu's data layout (the physics plan's Lesson 10) and with a
  pure ECS.
- Update order, and why "who runs before whom" causes more bugs than any algorithm. Concrete case:
  `ISimulationUpdate.SimulationUpdate` can run before `Start`.
- **Checkpoint**: add a component + processor to LiteEngine; then write the equivalent Stride
  processor and articulate what Stride does that yours does not.

## Phase F — Where physics meets the engine

### Lesson 11 — Integrating a simulation
This is where this plan and the physics plan join.

- Who owns the transform, and why the answer is "physics, one-way". You met this the hard way; now
  derive why every engine converges on it.
- Fixed-step simulation inside a variable-rate renderer: interpolation, and the sleeping-body
  optimisation that makes transforms stop updating.
- Kinematic vs dynamic vs static as an *ownership* question rather than a physics one.
- **Checkpoint**: host LitePhysics inside LiteEngine with interpolated rendering; then do the same
  in Stride via the toolkit and compare the integration seams.

## Phase G — The rest of the engine

### Lesson 12 — Content and the asset pipeline
You already fight this daily; this lesson makes it make sense.

- Why engines *compile* assets: runtime formats are not authoring formats. GPU texture compression,
  mesh layouts, shader permutations.
- Build-time vs load-time vs run-time costs; content hashing and incremental builds.
- **Checkpoint**: trace one asset through Stride's compiler end to end and explain each step. You
  have already read `Stride.AssetCompiler.targets` — connect the MSBuild layer to what the compiler
  actually produces.

### Lesson 13 — Stride, end to end
The synthesis lesson: read the engine with everything above in hand.

- `GraphicsCompositor` — how Stride assembles a frame, and where a `RootRenderFeature` sits.
- The script system: `StartupScript` / `SyncScript` / `AsyncScript`, and their ordering guarantees.
- Services and the game's object graph.
- **Checkpoint**: draw the frame — a diagram of one Stride frame from `Run()` to present, with every
  stage named. If you can draw it, you understand the engine.

## Phase H — Payoff

- Turn the strongest lessons into toolkit examples (several map almost directly).
- Write the page that does not exist: *"What Stride is doing when you call `SetupBase3DScene()`"*.
- With Phase C + Lesson 13 done, engine-side contributions stop being intimidating — you already
  landed one this year in `Stride.AssetCompiler.targets`.

---

## Reference shelf (in reading order)

- **Dmitry Sokolov, *tinyrenderer*** — the software-rasterizer course. Phase C's spine; do it in C#.
- **Jason Gregory, *Game Engine Architecture*** — the canonical survey. Read Phase A/B chapters early,
  the rest as reference.
- **Scratchapixel** — rasterization and perspective, derived properly. Companion to Lessons 5–6.
- **Akenine-Möller et al., *Real-Time Rendering*** — the rendering reference. Dip in, don't read cover
  to cover.
- **LearnOpenGL** — concepts transfer cleanly even though the API differs; good for Lesson 8.
- **Glenn Fiedler, *Fix Your Timestep*** — Lesson 1 (already on the physics shelf).
- **Stride source** — the answer key from Lesson 1 onward. Especially `Stride.Games/GameBase.cs`,
  `Stride.Engine`, `Stride.Rendering`, `Stride.Graphics.Regression`.
- **Stride Community Toolkit source** — your own code, re-read as a student rather than a maintainer.

## Ground rules

- **From scratch means from scratch** in LiteEngine: `System.Numerics` and a bitmap writer are
  allowed; graphics/engine packages are not, until Phase D says so.
- **Every formula gets derived once** in the lesson notes before it enters the code.
- **Compare against Stride at every opportunity** — it is right there, and "why does mine differ?" is
  the most productive question in the plan.
- **float, not double**, for the same reasons as the physics plan.
- Expect Phase C to be the hardest and the most rewarding. Expect Phase A to go fast — you have
  already done most of Lesson 1 in anger.

---

## Open questions

Answer inline; I will fold these into the plan.

> **Q1** — Where should LiteEngine live? A new repo alongside `physics`, a second solution inside the
> `physics` repo, or somewhere else? (This file is currently parked in `physics/docs/`; easy to move.)
>
> **Your answer:**

> **Q2** — How deep into Phase C? A full software rasterizer is the highest-value part of this plan
> and also the largest — realistically several evenings per lesson. The alternative is a reduced
> version (triangle + depth + one lit cube, skipping textures/perspective-correct interpolation).
> Full, reduced, or decide after Lesson 5?
>
> **Your answer:**

> **Q3** — 3D throughout, or mirror the physics plan's 2D focus for the early phases? 2D is less
> math up front but skips the parts (perspective, depth, quaternions) that are most of the value.
> My inclination is 3D from Lesson 3.
>
> **Your answer:**

> **Q4** — Is engine-side contribution to Stride an explicit goal, or is deep understanding for
> toolkit work enough? It changes how much time Phase G deserves.
>
> **Your answer:**

> **Q5** — Cadence: one lesson per week, per fortnight, or entirely opportunistic? This only affects
> how much I bundle into each lesson.
>
> **Your answer:**

> **Q6** — Should lessons be written to become toolkit examples from the start (metadata block,
> naming convention, docs), or kept as scratch work and adapted later? The former adds overhead per
> lesson but means the payoff is continuous rather than deferred to Phase H.
>
> **Your answer:**
