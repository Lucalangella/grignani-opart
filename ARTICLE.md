# Encoding Perception: An Interactive Op Art Experience Inspired by Grignani and Escher

There is a moment, adjusting the sliders on this project, when the flat tessellation stops being a pattern and starts being a place. Cubes that existed only as arithmetic -- rhombuses assigned to faces by a modulo operation -- suddenly appear to recede into depth, ripple like fabric, spiral into a vortex. Nothing has changed except the coordinates. The geometry is the same. But the eye insists otherwise.

This is the territory that Franco Grignani and M.C. Escher spent their careers exploring, from opposite directions. Escher built impossible architectures from the symmetries of tessellation. Grignani warped grids into perceptual disturbances using only straight lines. This project borrows from both traditions and departs from each -- applying real mathematical curves to an Escher-inspired pattern, computed on the GPU in real time.

---

## Two Traditions

### Escher: Structure as Illusion

M.C. Escher's tessellations are exercises in the geometry of the plane. Regular divisions, interlocking forms, and the exploitation of visual ambiguity -- the same arrangement of rhombuses can be read as convex cubes or concave cavities depending on how the eye assigns light and shadow. His tumbling blocks, staircases, and impossible waterfalls work because the human visual system compulsively constructs three dimensions from two-dimensional cues, even when the construction is logically impossible.

The tumbling blocks pattern at the foundation of this project comes directly from that tradition. Three rhombus orientations, assigned by the expression `(ci - cj) mod 3`, tile the plane in a way that the brain reads as isometric cubes -- a top face lit white, two side faces in shadow. It is a minimal Escher: the simplest possible tessellation that triggers involuntary depth perception.

### Grignani: Distortion as Sensation

Franco Grignani (1908--1999) arrived at optical art from a different angle -- literally. An architect by training, he spent four decades in Milan creating what he called "traumas to space": systematic distortions of geometric grids that produce sensations of depth, movement, and physical discomfort in the viewer.

His best-known creation is the Woolmark logo, but his real legacy is the body of 14,000+ experimental works -- black-and-white compositions where grids buckle, converge, and oscillate through carefully calculated spatial variations. He was an Op Art pioneer who preceded the movement by a decade, yet was absent from MoMA's 1965 "The Responsive Eye" exhibition. While Vasarely published manifestos and Riley gained gallery fame, Grignani worked in seclusion, spreading his influence through commercial design for Pirelli, Fiat, and Alfieri & Lacroix.

What makes Grignani uniquely relevant to creative coding is his methodology. His **hyperbolic** works were created by dividing 50x50 cm surfaces into grids of 22x22 squares, each unit just 3mm. He applied named distortion systems -- Pavia, Milano, Roma, Chicago -- each a different rule set for warping the grid. Modules were placed at calculated diagonal offsets. Spacing followed integer progressions (14, 13, 12, 11...). He maintained notebooks filled with mathematical formulas governing the transformations.

The critical point: Grignani achieved all of this using **only straight lines**. As he explained: "The twists are not produced by curved signs but by the gradual meeting of structures placed diagonally with grid coordinates." The curves are illusions -- emergent properties of carefully positioned straight marks.

---

## Where This Project Diverges

This experience does not recreate Grignani's straight-line constraint. It does something he deliberately chose not to do: it applies actual mathematical curves to the tessellation.

The distortion pipeline uses sinusoidal waves, power-law scaling, and rotational functions -- true curves that Grignani's methodology excluded. The result is a different kind of visual experience: where Grignani's illusions emerge from the accumulated precision of straight marks, this project's distortions are computed directly, each pixel independently transformed by continuous mathematical functions.

This is not a limitation of the approach -- it's a conscious choice to explore what happens when you combine Escher's tessellation vocabulary with distortion techniques that go beyond what either artist allowed. The GPU makes this possible: computing per-pixel coordinate transformations that would be impossible to draw by hand.

---

## Building the Experience

### The Pattern

The foundation is a tumbling blocks tessellation generated through triangular-grid mathematics. Cartesian coordinates are skewed into triangular-grid axes:

```
j = y * 2 / sqrt(3)
i = x - y / sqrt(3)
```

Each parallelogram cell splits into two triangles. The expression `(ci - cj) mod 3` assigns each triangle to one of three rhombus directions -- the three visible faces of an isometric cube. Top faces render white; side faces render black. The result is a seamless, infinitely tiling pattern of cubes that exist nowhere except in the viewer's perception.

### The Distortion Pipeline

Before the pattern is sampled, the shader warps the coordinate space through five stages:

**1. Fold Axis Rotation** -- The coordinate system rotates to align with a diagonal fold axis, creating angular tension against the tessellation's inherent geometry. This echoes Grignani's technique of placing structures at angles to orthogonal grids, though here the rotation is continuous rather than discrete.

**2. Perspective Foreshortening** -- A power-law function (`sign(s) * pow(|s|, bulgePower)`) compresses coordinates near the origin and stretches them at the periphery. This creates forced perspective -- the grid appears to converge toward a vanishing point. Grignani achieved a similar effect through progressively tightened grid spacing; the shader does it with a single mathematical operation per pixel.

**3. Dual-Axis Tilt** -- A second power-law compression along the perpendicular axis adds depth, creating the impression of a surface tilting away from the viewer.

**4. Sinusoidal Folds** -- This is where the project most clearly departs from Grignani's straight-line discipline. Two layers of sinusoidal displacement create valley-and-ridge folds:

```glsl
float ripple = waveAmp * sin(s * waveFreq);
t += ripple;
s += waveAmp * 0.4 * sin(t * waveFreq * 1.3);
```

The primary ripple displaces perpendicular to the fold axis. A cross-ripple at 1.3x frequency prevents the waves from locking into simple repetition. The result is flowing, fabric-like distortion -- the Escher cubes appear to undulate across a surface that doesn't exist.

**5. Twist** -- Rotational distortion proportional to distance from the convergence point:

```glsl
float twistAngle = twist * sqrt(s*s + t*t);
```

The grid spirals inward toward the pinch point. The square-root relationship produces a smooth acceleration rather than a linear one, creating a natural-looking vortex.

### Interactivity

Ten parameters are exposed as real-time sliders: grid scale, distortion strength, convergence point, wave amplitude and frequency, bulge power, twist, fold angle, and perspective tilt. Values pass to the shader as uniforms, meaning the GPU recalculates every pixel every frame without recompilation.

This real-time control is perhaps the most significant departure from both artists' working methods. Grignani's exploration was sequential -- each variation required a new drawing, days of manual work. Escher's tessellations were painstakingly constructed through geometric reasoning. The shader collapses both timelines, letting you sweep through configurations in seconds and develop intuition for how the parameters interact. You can reach compositions that neither artist would have found, because the search space is too vast to explore by hand.

---

## The Technology

The stack is deliberately minimal:

- **Three.js** and **React Three Fiber** provide the WebGL canvas
- **A single GLSL fragment shader** handles all pattern generation and distortion
- **React 19** manages the control panel state
- **An orthographic camera** ensures that all perceived depth comes from the shader, not the 3D scene

Everything runs on a flat plane. No mesh deformation, no ray marching, no textures -- just a mathematical function from screen position to black or white. The three-dimensionality is entirely illusory, which is the whole point.

---

## The Lineage

Escher and Grignani never collaborated, but they were working adjacent problems. Escher asked: what impossible structures can emerge from the rules of tessellation? Grignani asked: what perceptual disturbances can emerge from the systematic warping of a grid? Both worked in black and white. Both relied on the viewer's visual system to construct the illusion.

This project sits between them -- taking Escher's tessellation as raw material, subjecting it to distortions inspired by Grignani's perceptual vocabulary, and executing the whole thing with tools neither had access to. The GPU computes in microseconds what would take days by hand. But the fundamental questions haven't changed: How does a flat pattern become a space? Where is the boundary between order and optical chaos? At what point does geometry stop being a structure and start being an experience?

Grignani believed that "to affirm its useful role in visual communication, graphic art must rely on a large number of experiments to achieve perfect freedom." Escher believed that "we adore chaos because we love to produce order."

The slider panel invites you to experiment with both impulses at once.
