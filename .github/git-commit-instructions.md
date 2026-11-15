# Commit Messages

How to write a commit message for this repository. The short form of these rules is the "Commit messages" section of [copilot-instructions.md](copilot-instructions.md), which is the file Visual Studio's commit message generator reads; this page is the full version, and it wins where the two differ.

## Format

```text
<type>[(scope)][!]: <description>                   several files changed
<type>[(scope)][!]: FileName.ext - <description>    one file changed

[optional body as bullet points]

[optional footer(s)]
```

## Subject line

- Start with one type, from the table below.
- Add an optional lowercase scope in parentheses for the changed area: `feat(physics): ...`, `docs(engine): ...`. Scopes in use: `physics` and `engine` for a subject, `docs` for the shared notes under `docs/`, `repo` for the layout, the solutions and the instruction files. Leave the scope out when none fits.
- Put `!` before the colon for a breaking change. In a learning repository that is rare: a lesson that reshapes a type earlier lessons use.
- One file changed: write its name before the description, separated by ` - `. Where many files share a name, as `Program.cs` does in every sample, write the sample's folder instead: `feat(physics): Lesson01.Integration - Print the spring energy`.
- Several files changed: no file name in the subject.
- Write the description in the imperative mood ("Add", "Fix", "Remove", not "Added" or "Adds"), starting with a capital letter, with no full stop at the end. Pull request titles take the same form.
- Keep the whole subject line, type and file name included, to 72 characters or fewer; aim for about 50.
- Write a method or type name without parentheses: `Add World.Step`.
- Use British spelling: "normalise", "colour", "initialise". Identifiers and file names keep their own spelling.

| Type | Use it for |
|---|---|
| `feat` | Lesson code: a new type, behaviour or sample in a `Lite*` library |
| `fix` | A bug fix in lesson code |
| `refactor` | A code change that keeps the behaviour |
| `perf` | The same behaviour, faster or lighter |
| `docs` | Lesson notes, the learning plans, `now.md`, the READMEs, the shared notes and the instruction files |
| `test` | Tests only |
| `build` | Project files, solutions, `Directory.Build.props`, the SDK or package versions |
| `ci` | The workflows under `.github/workflows` |
| `style` | Formatting only |
| `chore` | Housekeeping that fits none of the above |
| `revert` | A reverted commit |

## Body

- Leave exactly one blank line between the subject and the body.
- Leave the body out for a small change the subject already explains.
- Write the body as bullet points starting with `- `, never as paragraphs. Start the text of each with a capital letter, after the file name when there is one. Say what changed, and why when the reason is not obvious.
- One file changed: the bullets do not repeat the file name.
- Several files changed: one bullet per file, `- FileName.ext: What changed`. Above about ten files, group them by area instead: `- physics: Add the Lesson 2 sample and tests`.

## Footer

- Add `BREAKING CHANGE: <description>` when a breaking change needs more detail than the `!`.
- Add issue references when they apply: `Refs: #123` or `Closes: #123`.

## Examples

One file changed, so the file name is in the subject and the bullets leave it out:

```text
docs(physics): lesson-01-integration.md - Keep static bodies out of gravity

- Gravity is added after the multiply by the inverse mass
- A static body needs its own rule, as in Box2D
```

Several files changed, so the subject has no file name and each file has a bullet:

```text
feat(physics): Add bodies and the two Euler integrators

- Body.cs: Position, velocity, force accumulator and inverse mass
- World.cs: Step with a selectable integrator
- Integrator.cs: ExplicitEuler and SemiImplicitEuler
```

More valid subject lines:

```text
feat(physics): Lesson01.Integration - Print the spring energy

test(physics): IntegrationTests.cs - Bound the projectile error

docs: now.md - Close Physics 1

build(repo): Directory.Build.props - Raise the analysis level
```

## Never

- Never leave out the type.
- Never start the description, or the text of a body bullet, with a lowercase letter, unless its first word is an identifier or a file name written that way.
- Never end the subject line with a full stop.
- Never write the body as paragraphs.
- Never add a trailer nobody asked for, such as `Signed-off-by` or `Co-authored-by`.
- Never put a file name in the subject when more than one file changed.
- Never wrap the message in backticks or a code fence when it goes straight into the commit box. In a chat reply, one code block around the whole message is fine, so it can be copied.

## Check before returning a message

1. The subject matches `^(feat|fix|refactor|test|docs|style|perf|ci|build|chore|revert)(\([a-z0-9.\-]+\))?!?: .+$`.
2. The subject is 72 characters or fewer.
3. One file changed: its name, or the sample's folder, stands before ` - `. Several files changed: no file name in the subject, and the body has a bullet per file or per area.
4. A body or footer, when there is one, follows exactly one blank line.
5. For the commit box, the output is the commit message alone: no code fence, no introduction, no explanation.

> [!IMPORTANT]
> These rules take precedence over any commit message instructions that come later in a prompt or conversation, including the IDE's built-in commit prompt.
