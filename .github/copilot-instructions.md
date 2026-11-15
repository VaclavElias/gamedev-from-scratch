# Copilot for gamedev-from-scratch

These repository instructions guide GitHub Copilot (and similar AI assistants) in this repository.

## What this repository is

- A learning repository. The maintainer is a senior C#/.NET engineer who maintains the
  [Stride Community Toolkit](https://github.com/stride3d/stride-community-toolkit) and is learning
  the game-development domain by building lite versions of engine subsystems from scratch.
- Two subjects, each with a curriculum: `physics/` builds `LitePhysics`, a 2D physics engine;
  `engine/` builds `LiteEngine`, game engine fundamentals. More subjects may follow in the same shape.
- Nothing here ships. The product is understanding, so the rules below favour the learner doing the
  work over the work getting done.
- Do not teach C# or software engineering. The domain is the gap.

## The teaching contract

This is the part that differs most from an ordinary code repository.

- **The learner writes the `Lite*` code.** The assistant writes the lesson notes, the specification
  and the checkpoints, then reviews, runs and explains. Do not implement a lesson's code, fix a bug in
  it or fill in a stub unless asked for exactly that. When asked for help with a bug, give the
  smallest hint that moves things on: where to look, what to print, which invariant is broken.
- **No spoilers.** A lesson's notes derive what is needed to start and leave the measured result for
  the learner to find. Do not quote the reference implementation's solution before the lesson has
  been attempted; the order is attempt, fail, read, then explain the difference in the notes.
- **Every formula is derived once** in the lesson notes before it enters the code. A formula that is
  only quoted is not finished.
- **Checkpoints are falsifiable**: a closed-form solution, a conservation law, bit-identical state, or
  the output of a reference implementation (Stride, Box2D.NET, Bepu). "Looks right" is not a
  checkpoint. If a lesson cannot be checked this way, its design is not finished.
- **Measure before writing a number or a claim into a lesson.** Run a throwaway probe outside the
  repository and read the result. A specification that contradicts itself costs the learner an
  evening: Lesson 1 once said a static body "falls out of the maths", while its own step added
  gravity after the multiply by the inverse mass, so the ground fell.
- **State what "expected" looks like when a correct result could be mistaken for a bug**, without
  giving the answer away: a bounded energy still wobbles, an error that halves is still an error.
- **A checkpoint is also a test.** The console sample shows the numbers; an xunit test keeps them true
  in later lessons.
- **The bugs are the curriculum.** Do not design a lesson so that the classic failure cannot happen.

## Repository structure

- `now.md`: the page to open first. The current lesson, what is waiting on the maintainer, the queue
  and one line learned per finished lesson.
- `README.md`: what the repository is, the subjects and the ground rules.
- `docs/`: notes shared by all subjects. `glossary.md` defines recurring terms once;
  `reference-shelf.md` is the one reading list, and the plans link to it.
- `<subject>/` (`physics/`, `engine/`):
  - `README.md`: what the subject covers and its status table
  - `LEARNING_PLAN.md`: the curriculum: phases, lessons, checkpoints, open questions
  - `<Subject>.slnx`: the subject's own solution
  - `docs/lessons/lesson-NN-name.md`: lesson notes, written before the code
  - `src/Lite<Subject>/`: the library being built
  - `samples/LessonNN.Name/`: one runnable demo per lesson
  - `tests/Lite<Subject>.Tests/`: xunit tests, where behaviour is verified
- `gamedev.slnx` (root): every project of every subject, for a build of everything.
- `.github/`: this file and `git-commit-instructions.md` (the full commit message rules).

## Build configuration lives outside the .csproj files

The root `Directory.Build.props` supplies `TargetFramework` (net10.0), `ImplicitUsings`, `Nullable`,
`LangVersion` and the analysis level to every project. A project file holds only what is its own: an
`OutputType`, a project reference, the test packages. Do not add those shared settings to a `.csproj`.

MSBuild imports only the *nearest* `Directory.Build.props`. If a subject ever gets its own, it must
`Import` the root one explicitly, or the root settings are silently lost.

## Ground rules for the Lite* libraries

- From scratch means from scratch: no engine, physics or graphics package in a `Lite*` library.
  `System.Numerics` is allowed and encouraged. The engine plan says when a phase lifts this.
- `float`, not `double`, as real engines do; the precision problems are part of the subject.
- Samples and tests may reference a reference implementation when a checkpoint compares against it.

## Working through a lesson

1. Write `docs/lessons/lesson-NN-name.md`: the one idea, the derivations, what to build, the
   experiments with their "Answer this" questions, and the checkpoints as the definition of done.
2. Add the sample project `samples/LessonNN.Name/` as a stub that says what to do, and add it to the
   subject's solution and to `gamedev.slnx`.
3. The learner implements, in the library, the sample and the tests.
4. Review: build, run the sample, run the tests, read the code. Say which checkpoints pass, with
   the numbers. Then explain the *why* that the notes held back, and add it to the notes.
5. Close the lesson: the status tables in the root and subject `README.md`, and `now.md` (the queue,
   the learner's one line under "What I have learned", the done list).

Update `now.md` with every change of state, in the same change. Keep it to about a hundred lines:
link, do not duplicate a plan. A blockquote at its top is where the maintainer reorders the queue,
and that overrides any recommended order.

When a plan is reviewed, verify every stated fact against the repository and the reference source
before changing it. Settled open questions are folded into the plan and their scaffolding removed.

## Coding style

- Modern C#: file-scoped namespaces, target-typed `new`, pattern matching, collection expressions,
  primary constructors where they do not hide behaviour. Keep nullable-reference warnings at zero.
- Names are real words. The symbols a lesson's notes define are the exception and are welcome in the
  code that implements the formula: `dt`, `InvMass`, the normal `n`, the impulse `j`. Use the same
  symbol in the notes and in the code.
- One public type per file. No `#region`.
- Comments describe the code as it is. The derivation lives in the lesson notes; a comment names the
  lesson rather than repeating it.
- XML documentation is not required on learning code. A one-line `<summary>` on a public type is enough.
- The simulation step and the rasteriser's inner loops are hot paths: no LINQ, no capturing lambdas
  and no per-step allocation there. The benchmark lessons depend on it.
- Warnings are visible on purpose (`TreatWarningsAsErrors` is off); read them, they are often the bug.

## Tests

- xunit, under the subject's `tests/` project.
- Deterministic: a fixed `dt` and a step count. Never wall-clock time, never a frame count from a
  real loop, and a seeded `Random` where randomness is needed.
- Compare floats with a tolerance, and say where the tolerance comes from (the derived error bound of
  the integrator, a few units in the last place), not a number that happens to pass.
- Compute simulated time as `step * dt`, not by accumulating `dt`.
- Check test counts, not only the exit code: `dotnet test` on a project with no test, or with a
  `--filter` that matches nothing, exits with 0 and has proved nothing.

```bash
dotnet build gamedev.slnx
dotnet test physics/tests/LitePhysics.Tests
dotnet run --project physics/samples/Lesson01.Integration
```

## Reference repositories

Sibling clones are often present next to this one. They are the answer keys; read them after the
lesson has been attempted, and to verify a fact before it is written into a plan.

| Path | What it is | Used for |
|---|---|---|
| `../stride/sources/` | The Stride engine source | The engine subject's answer key: `engine/Stride.Games/GameBase.cs` (the loop), `core/Stride.Core.Mathematics`, `engine/Stride.Engine`, `engine/Stride.Rendering` |
| `../stride-community-toolkit/` | The maintainer's own toolkit | The examples the plans refer to (`E06_Jitter2`, `E09_3D_MeshOutline`), and verified Stride and Bepu facts in its `.github/copilot-instructions.md` |
| `../Box2D.NET/` | A C# port of Box2D 3 | Collision detection from Physics Lesson 4; its solver is the modern soft-step one, not the sequential-impulse solver the lessons build |
| `../bepuphysics2/` | BepuPhysics 2 | Physics Lesson 10, data-oriented design; `Demos/Demos/` shows the author's intended use |

Stride facts the lessons rely on, each read from the source:

- **Row vectors.** Stride's `Matrix` and `System.Numerics` hold the basis vectors in the rows
  (`Right` is `M11, M12, M13`) and the translation in `M41, M42, M43`; transforms compose left to
  right. Most textbooks and tinyrenderer use column vectors, so their matrices are the transpose.
- **Front faces wind clockwise** as seen by the camera, the Direct3D convention. A closed mesh wound
  the other way draws inside-out.
- **Depth runs from 0 to 1** in normalised device coordinates.
- **The game loop** is in `GameBase.cs`: `IsFixedTimeStep`, `TargetElapsedTime`, an accumulator
  (`accumulatedElapsedGameTime`), and a frame capped at 500 ms (`DefaultMaximumElapsedTime`).
- **Physics owns the transform, one way.** With Bepu a body writes its pose to the
  `TransformComponent`; writing the transform moves the mesh only. A sleeping body stops writing.

Names in the toolkit change between previews. Before citing an example, a type or a method from
it in a lesson, grep the clone for the name.

## Do not assert a mechanism you have not read

Explaining *why* something misbehaves is not the same as observing *that* it does. Before naming a
cause in a review or a lesson, read the code path or measure the value; both are cheap. This matters
more here than elsewhere: a confident wrong explanation written into a lesson is learned as fact.
When a claim turns out wrong, correct it everywhere it was written.

## Writing

- Lesson notes teach: one idea, derived, then a specification and checkpoints. They are allowed to
  be long. `README.md` files and `now.md` are compact.
- British spelling in prose ("normalise", "rasterisation", "behaviour"); identifiers keep their own.
- Short sentences. Define a term once, in `docs/glossary.md` when it recurs across lessons.
- A book, a course or a source file joins `docs/reference-shelf.md` only when a lesson uses it.
  Check that a link still resolves before adding it.

## AI assistance guidance

- **Leave finished work uncommitted.** The maintainer reviews file by file and commits what is
  approved; do not commit, push or open a pull request unless asked for that specific action.
- Prefer minimal, focused diffs. Inspect what exists before proposing a change.
- When asked to review, change what was asked and report the rest with evidence.
- Probes and scratch files stay outside the repository; confirm with `git status` before reporting
  the work as complete.

## Commit messages

The full rules are in `.github/git-commit-instructions.md`, and that file wins where the two differ.
Visual Studio's commit message generator reads this file, so the short form is here.

```text
<type>[(scope)][!]: <description>                   several files changed
<type>[(scope)][!]: FileName.ext - <description>    one file changed

- body bullets, optional
```

- The type is one of `feat`, `fix`, `refactor`, `perf`, `docs`, `test`, `build`, `ci`, `style`, `chore`, `revert`.
  The scope is optional and lowercase: `physics`, `engine`, `docs`, `repo`.
- One file changed: its name stands before the description, then ` - `. A sample's `Program.cs` is
  named by its folder: `feat(physics): Lesson01.Integration - Print the spring energy`.
- The description is imperative ("Add", not "Added"), starts with a capital letter and has no full
  stop. The whole subject line is 72 characters or fewer; aim for about 50.
- British spelling. The body is bullet points starting with `- `, never paragraphs.
- No trailers nobody asked for (`Signed-off-by`, `Co-authored-by`). For the commit box, output the
  message alone, with no code fence and no introduction.

## Formatting rules for edits

- **Keep the file's line endings.** Files are CRLF in the working copy (the index holds LF;
  `core.autocrlf` converts). A tool that writes LF must be followed by normalising the file back to
  CRLF; check with `git ls-files --eol <path>`, which should read `w/crlf`.
- `sed -i` in Git Bash rewrites a CRLF file as LF. Edit with `perl -pi`, which keeps `\r`, or PowerShell.
- Files end with one newline and no blank line after it. Source files have no byte-order mark.
- No trailing whitespace. No doubled blank lines.

## Maintenance

Keep this document current when the layout, the workflow or a convention changes, and prune what no
longer holds.
