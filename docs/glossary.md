# Glossary

Terms that recur across subjects, defined once. The depth lives in the lesson notes.

## Time and the loop

| Term | Meaning |
|---|---|
| **Frame** | One pass of input → update → render. |
| **Tick / step** | One advance of the simulation. Not the same as a frame: a frame may run zero, one, or several ticks. |
| **Fixed timestep** | Advancing the simulation by a constant `dt` regardless of frame duration. Required for stability and determinism. |
| **Accumulator** | The pattern that reconciles a fixed simulation rate with a variable frame rate: bank elapsed time, spend it in fixed-size steps. |
| **Interpolation (render)** | Drawing between the last two simulation states so motion looks smooth when the render rate exceeds the tick rate. |
| **Determinism** | Same inputs produce bit-identical results. Needed for replays, networking, and sane debugging. |

## Space

| Term | Meaning |
|---|---|
| **Local space** | Coordinates relative to an object's own origin. |
| **World space** | Coordinates relative to the scene origin. |
| **View space** | World rotated and translated so the camera sits at the origin looking down an axis. |
| **Clip space** | After the projection matrix, before the perspective divide. Homogeneous - the `w` still matters. |
| **NDC** | Normalised device coordinates: clip space divided by `w`. x,y in [-1,1]; z in [0,1] in D3D convention. |
| **Screen space** | NDC mapped to pixels via the viewport. |
| **Basis vectors** | Where the object's x, y, z axes point in the parent space. A transform matrix holds them: as its columns in the column-vector convention, as its rows in the row-vector convention of Stride and `System.Numerics`. |

## Rendering

| Term | Meaning |
|---|---|
| **Rasterisation** | Deciding which pixels a triangle covers. |
| **Winding order** | Vertex order (clockwise vs counter-clockwise) which defines a triangle's facing, and therefore what gets culled. |
| **Backface culling** | Discarding triangles facing away from the camera — roughly half the work, for free. |
| **Z-buffer / depth buffer** | Per-pixel depth, so nearer fragments win regardless of draw order. |
| **Perspective-correct interpolation** | Interpolating `attribute/w` and `1/w`, then dividing per pixel, so textures do not warp under perspective. |
| **Draw call** | One instruction to the GPU to render a batch. Historically the thing you counted. |
| **Vertex / pixel (fragment) shader** | The programmable stages: per-vertex transform, per-pixel colour. |
| **Normal** | Direction a surface faces. Hard-edged meshes duplicate vertices so one position can carry several normals. |

## Physics

| Term | Meaning |
|---|---|
| **Inverse mass** | `1/m`, stored instead of mass: cheaper maths, and `0` cleanly means "immovable". |
| **Symplectic (semi-implicit) Euler** | Integrate velocity first, then position with the *new* velocity. Same cost as explicit Euler, bounded energy instead of explosion. |
| **Broad phase / narrow phase** | Cheaply reject pairs that cannot touch; then compute exact contacts for the survivors. |
| **Contact manifold** | The set of contact points for a pair — a resting box needs two, or it rocks. |
| **Impulse** | An instantaneous change in momentum. Collisions are resolved with impulses, not forces. |
| **Restitution** | Bounciness, `e`. A ball dropped from `h` returns to `e²·h`. |
| **Sequential impulses** | Solving contacts one at a time and iterating (Gauss–Seidel). Erin Catto's signature technique. |
| **Warm starting** | Reusing last frame's impulses as this frame's starting guess. Cheap, and the difference between a stable stack and a jittering one. |
| **Slop** | A small allowed penetration, so bodies stop rather than fighting float noise forever. |
| **Sleeping** | Deactivating settled bodies. Also why a settled body stops writing its transform. |

## Engine architecture

| Term | Meaning |
|---|---|
| **Entity / Component** | Composition over inheritance: an entity is an id plus attached data. |
| **Processor / System** | The code that runs over components of a given type each frame. |
| **Scene graph** | The parent-child transform hierarchy. |
| **Asset pipeline** | Build-time conversion from authoring formats to runtime formats. |
| **Kinematic / dynamic / static** | Who owns a body's motion: you, the simulation, or nobody. |
