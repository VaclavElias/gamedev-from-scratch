# Lesson 1 — Bodies, forces, and integration

## The one idea

A rigid body's motion is governed by Newton's second law:

```
F = m * a          (force = mass × acceleration)
```

Rearranged for simulation, acceleration is what we solve for:

```
a = F / m = F * invMass
```

Everything in this lesson is about turning that into code that advances a body
through time, one small step `dt` at a time.

## Why inverse mass, not mass

We store `InvMass = 1 / mass`, not `mass`. Two reasons, both of which you'll
lean on for the rest of the engine:

1. **The math divides by mass constantly.** `a = F / m`, and later the impulse
   equations divide by mass over and over. Precomputing the reciprocal turns
   every division into a multiply. Minor for one body, meaningful across
   thousands per frame.
2. **`InvMass = 0` *is* a static (immovable) body.** Infinite mass means any
   finite force produces zero acceleration: `a = F * 0 = 0`. The force and
   impulse maths need no `if (isStatic)` branch — the ground, walls, and
   platforms fall out of the same equation. This trick is everywhere in Box2D
   and Bepu. It has one gap, gravity, which you meet in the `World` spec below.

## Integration: turning acceleration into motion

We know `a`. We want the body's new position and velocity after `dt` seconds.
Calculus says `v = ∫a dt` and `x = ∫v dt`, but we can't integrate symbolically
at runtime — we approximate with small steps. Two schemes, **identical cost**,
differ only in the order of two lines.

### Explicit (forward) Euler — the "obvious" one

```
x += v * dt        // use the OLD velocity to move
v += a * dt        // then update velocity
```

This is what everyone writes first. It's also subtly wrong in a way that
matters: it consistently **adds energy** to oscillating systems. A spring
simulated this way gains amplitude every cycle until it explodes. Included so
you can *see* it fail — do not ship it.

### Semi-implicit (symplectic) Euler — the one engines use

```
v += a * dt        // update velocity FIRST
x += v * dt        // then move using the NEW velocity
```

One line swapped. This scheme is **symplectic**: it approximately conserves
energy over long runs (it preserves the geometric structure of the underlying
Hamiltonian system). A spring stays bounded forever — bounded, not constant:
the energy wobbles by a few percent around the true value and never trends.
Box2D and Bepu both use this. The lesson lands when you print the energy from both and watch one drift
up while the other holds.

> Spoiler-free for now: *why* symplectic Euler conserves energy is the thing
> we'll discuss after you've measured it.

## Error: what "O(dt)" means and why we care

Both schemes are **first-order**: the error per step is proportional to `dt²`,
and the accumulated error over a fixed time span is proportional to `dt`. That
"O(dt) global error" is the property you'll verify in checkpoint 1:

- Halve `dt` → the projectile position error should drop by roughly **2×**.
  So from `1/60` to `1/240`, two halvings, it drops by 4× — still first-order.

If you ever move to a scheme where halving `dt` cuts error by **4×**, that's
second-order (O(dt²)) — e.g. Verlet or RK2. We stay first-order here because
that's what real-time engines use; stability matters more than order.

## What to build

A class library `LitePhysics` and a console sample. `float` everywhere,
`System.Numerics.Vector2` allowed, no physics packages.

### 1. `Body`
- `Vector2 Position`
- `Vector2 Velocity`
- `Vector2 Force`   — accumulator; things that push the body add here
- `float InvMass`   — `0` for static
- A way to construct dynamic (given mass) vs static bodies.
- `ApplyForce(Vector2)` that adds into the accumulator.

### 2. `World`
- Holds a list of bodies and a `Vector2 Gravity`.
- A selectable integrator (enum: `ExplicitEuler` / `SemiImplicitEuler`).
- `Step(float dt)` that, for each body:
  1. Computes `a = Force * InvMass + Gravity`
     (gravity is an acceleration, so it's added *after* the `* InvMass`, not
     as a force — think about why: gravitational force is `m*g`, so
     `m*g * invMass = g` regardless of mass. This is why a feather and a
     hammer fall at the same rate in a vacuum.)
  2. Integrates position & velocity per the chosen scheme.
  3. Clears `Force` back to zero (forces last one step only).
  - Static bodies (`InvMass == 0`) must never move. Applied forces cancel by
    themselves: `Force * 0 = 0`. Gravity does not, because step 1 adds it as an
    acceleration *after* the multiply, so a static body would fall at `g` like
    everything else. Run it once and watch the ground drop, then decide how a
    static body escapes gravity and write down why. (Box2D integrates only
    dynamic bodies; the alternative is to multiply gravity by a 0-or-1 factor.)
    A static body is also created with zero velocity and keeps it.

### 3. Two experiments in the console app

**Projectile check.** Launch from origin with `v0 = (10, 10)`, gravity
`(0, −9.81)`, `dt = 1/60`, for 2 simulated seconds. Each step, compare
simulated position to the analytic solution:

```
x(t) = x0 + v0x * t
y(t) = y0 + v0y * t − ½ * g * t²
```

Print the final error. Then rerun at `dt = 1/120` and `dt = 1/240`. Count
steps (`steps = 2 / dt`) and compute `t` as `step * dt`, so that float error in
an accumulated `t` does not leak into the comparison.
**Answer this:** by what factor did the error shrink at each halving, and does
that match the O(dt) prediction? Which component carries the error, and why is
the other one exact? Then derive the size of the error at time `t` — it is a
one-line sum — and use it as the bound in the test.

**Spring check.** One body, mass 1, no gravity, force `F = −k * Position` with
`k = 50`, initial position `x = (1, 0)`, initial velocity zero. This is a
harmonic oscillator; total mechanical energy

```
E = ½ * m * |v|²  +  ½ * k * |x|²
```

is *analytically constant*. Run 60 simulated seconds at `dt = 1/60`, print `E`
once per simulated second, once per integrator, and keep the minimum and
maximum over every step (a once-a-second sample hides the wobble).
**Answer this:** which integrator drifts, and in which direction? How wide is
the other one's band, and what happens to it when you halve `dt`?

## Checkpoints (definition of done)

1. Projectile matches analytic solution within a stated bound; you can explain
   the 2× error reduction when `dt` halves.
2. Spring energy stays bounded under semi-implicit Euler and grows under
   explicit Euler — shown with printed numbers.
3. A static body doesn't move under any applied force, with gravity on.

Each checkpoint is also an xunit test in `tests/LitePhysics.Tests`: the console
sample shows the numbers, the test keeps them true in later lessons.

Say when it runs and I'll review. We'll dig into the *why* of the spring result
before Lesson 2.
