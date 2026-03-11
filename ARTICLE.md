# Encoding Perception: An Interactive Op Art Experience Inspired by Grignani and Escher

There is a moment, adjusting the sliders on this project, when the pattern crosses a threshold. At low settings, you see what the code is actually drawing: isometric cubes, a tumbling blocks tessellation, gently warped by perspective and ripple. Push the twist higher, crank the wave amplitude, increase the grid density -- and the cubes vanish. What replaces them is something the underlying geometry never described: dense, swirling moiré fields, rivers of black and white flowing into vortex spirals, interference patterns that vibrate and shimmer like marbled paper caught in turbulence.

Nothing has changed except the parameters. The shader is still drawing the same tessellation through the same distortion pipeline. But the eye can no longer resolve the structure -- it sees only the sensation. This is the territory that Franco Grignani and M.C. Escher spent their careers exploring, from opposite directions.

---

## Two Traditions

### Escher: Structure as Illusion

M.C. Escher's tessellations are exercises in the geometry of the plane. Regular divisions, interlocking forms, and the exploitation of visual ambiguity -- the same arrangement of rhombuses can be read as convex cubes or concave cavities depending on how the eye assigns light and shadow. His tumbling blocks, staircases, and impossible waterfalls work because the human visual system compulsively constructs three dimensions from two-dimensional cues, even when the construction is logically impossible.

The tumbling blocks pattern at the foundation of this project comes from that tradition. Three rhombus orientations, assigned by the expression `(ci - cj) mod 3`, tile the plane in a way that the brain reads as isometric cubes -- a top face lit white, two side faces in shadow. It is a minimal Escher: the simplest possible tessellation that triggers involuntary depth perception.

### Grignani: Distortion as Sensation

Franco Grignani (1908--1999) arrived at optical art from a different angle -- literally. An architect by training, he spent four decades in Milan creating what he called "traumas to space": systematic distortions of geometric grids that produce sensations of depth, movement, and physical discomfort in the viewer.

His best-known creation is the Woolmark logo, but his real legacy is the body of 14,000+ experimental works -- black-and-white compositions where grids buckle, converge, and oscillate through carefully calculated spatial variations. He was an Op Art pioneer who preceded the movement by a decade, yet was absent from MoMA's 1965 "The Responsive Eye" exhibition. While Vasarely published manifestos and Riley gained gallery fame, Grignani worked in seclusion, spreading his influence through commercial design for Pirelli, Fiat, and Alfieri & Lacroix.

What makes Grignani uniquely relevant to creative coding is his methodology. His **hyperbolic** works were created by dividing 50x50 cm surfaces into grids of 22x22 squares, each unit just 3mm. He applied named distortion systems -- Pavia, Milano, Roma, Chicago -- each a different rule set for warping the grid. Modules were placed at calculated diagonal offsets. Spacing followed integer progressions (14, 13, 12, 11...). He maintained notebooks filled with mathematical formulas governing the transformations.

The critical point: Grignani achieved all of this using **only straight lines**. As he explained: "The twists are not produced by curved signs but by the gradual meeting of structures placed diagonally with grid coordinates." The curves in his work are illusions -- emergent properties of carefully positioned straight marks.

---

## Where This Project Diverges

This experience does not recreate Grignani's straight-line constraint. It does something he deliberately chose not to do: it applies actual mathematical curves to the tessellation.

The distortion pipeline uses sinusoidal waves, power-law scaling, and rotational functions -- true curves that Grignani's methodology excluded. And unlike Grignani's static compositions, every parameter updates in real time, letting you explore a continuous space of distortions rather than committing to a single configuration.

But the most significant difference is what happens at the extremes. Grignani's works, however disorienting, always retain the legibility of their underlying grid -- you can trace the structure even as it warps. In this project, pushing the parameters hard enough causes the tessellation to disappear entirely. The isometric cubes dissolve into dense moiré interference patterns: flowing, turbulent fields of black and white that bear no visible relationship to the rhombuses that generate them. The geometry becomes invisible; only the optical sensation remains.

This emergent behavior -- where a simple tessellation, passed through enough mathematical distortion, produces something that looks like fluid dynamics or marbled paper -- was not designed. It is a natural consequence of the interaction between regular pattern and nonlinear transformation. Grignani, who spent his career investigating exactly these kinds of perceptual thresholds, would have recognized the phenomenon immediately. He called it "trauma": the point where visual order tips into optical chaos, and the viewer's eye can no longer resolve the structure, only feel the disturbance.

---

## Building the Experience

### The Pattern

The foundation is a tumbling blocks tessellation generated through triangular-grid mathematics. Cartesian coordinates are skewed into triangular-grid axes:

```
j = y * 2 / sqrt(3)
i = x - y / sqrt(3)
```

Each parallelogram cell splits into two triangles. The expression `(ci - cj) mod 3` assigns each triangle to one of three rhombus directions -- the three visible faces of an isometric cube. Top faces render white; side faces render black. At low grid density, you see distinct cubes. At high density, the cubes shrink until they become texture -- raw material for the distortion to sculpt.

### The Distortion Pipeline

Before the pattern is sampled, the shader warps the coordinate space through five stages:

**1. Fold Axis Rotation** -- The coordinate system rotates to align with a diagonal fold axis, creating angular tension against the tessellation's inherent geometry. This echoes Grignani's technique of placing structures at angles to orthogonal grids, though here the rotation is continuous rather than discrete.

**2. Perspective Foreshortening** -- A power-law function (`sign(s) * pow(|s|, bulgePower)`) compresses coordinates near the origin and stretches them at the periphery. This creates forced perspective -- the pattern appears to converge toward a vanishing point. Grignani achieved a similar effect through progressively tightened grid spacing; the shader does it with a single mathematical operation per pixel.

**3. Dual-Axis Tilt** -- A second power-law compression along the perpendicular axis adds depth, creating the impression of a surface tilting away from the viewer.

**4. Sinusoidal Folds** -- This is where the project most clearly departs from Grignani's straight-line discipline. Two layers of sinusoidal displacement create valley-and-ridge folds:

```glsl
float ripple = waveAmp * sin(s * waveFreq);
t += ripple;
s += waveAmp * 0.4 * sin(t * waveFreq * 1.3);
```

The primary ripple displaces perpendicular to the fold axis. A cross-ripple at 1.3x frequency prevents the waves from locking into simple repetition. At low amplitude, this produces gentle fabric-like undulation. At high amplitude and frequency, the two wave layers interfere with each other and with the tessellation to produce the turbulent, marbled textures visible in the screenshots -- dense flowing forms that look like nothing the underlying geometry would predict.

**5. Twist** -- Rotational distortion proportional to distance from the convergence point:

```glsl
float twistAngle = twist * sqrt(s*s + t*t);
```

The grid spirals inward toward the pinch point. This is the single most dramatic parameter: a strong twist transforms the entire composition into a vortex, pulling the tessellation into sweeping curves that create powerful moiré interference where adjacent regions of the pattern compress against each other.

### The Emergent Moiré

The moiré-like effects visible at high parameter settings are not coded explicitly -- they emerge from the interaction between the regular tessellation and the nonlinear distortion. When the twist compresses adjacent regions of the pattern into slightly different orientations, the repeating black-and-white elements interfere with each other at a scale larger than the individual cubes. The result is flowing, large-scale patterns that the eye reads as movement, depth, and turbulence -- even though every pixel is computed independently with no awareness of its neighbors.

This is the same phenomenon that Grignani exploited in his moiré photography experiments of the 1950s, where he overlaid regular structures to produce interference patterns. The difference is that here, the overlay happens mathematically: the distortion pipeline creates regions where the tessellation effectively overlaps itself at different scales and angles.

### Interactivity

Ten parameters are exposed as real-time sliders. Values pass to the shader as uniforms, meaning the GPU recalculates every pixel every frame without recompilation.

The real-time control lets you feel the transition from structure to sensation -- you can watch the cubes form, warp, and dissolve as you drag a slider. This continuous exploration is perhaps the most significant departure from both artists' working methods. Grignani's exploration was sequential -- each variation required a new drawing, days of manual work. Escher's tessellations were painstakingly constructed through geometric reasoning. The shader collapses both timelines, letting you sweep through the full spectrum from ordered tessellation to optical chaos in seconds.

---

## The Technology

The stack is deliberately minimal:

- **Three.js** and **React Three Fiber** provide the WebGL canvas
- **A single GLSL fragment shader** handles all pattern generation and distortion
- **React 19** manages the control panel state
- **An orthographic camera** ensures that all perceived depth comes from the shader, not the 3D scene

Everything runs on a flat plane. No mesh deformation, no ray marching, no textures -- just a mathematical function from screen position to black or white. The three-dimensionality, the flowing rivers, the turbulent vortices -- all of it is illusory, computed from coordinates alone.

---

## The Lineage

Escher and Grignani never collaborated, but they were working adjacent problems. Escher asked: what impossible structures can emerge from the rules of tessellation? Grignani asked: what perceptual disturbances can emerge from the systematic warping of a grid? Both worked in black and white. Both relied on the viewer's visual system to construct the illusion.

This project sits between them -- taking Escher's tessellation as raw material, subjecting it to distortions inspired by Grignani's perceptual vocabulary, and pushing both further than either artist's manual tools could reach. At low settings, you see Escher's cubes gently warped by Grignani's perspective. At high settings, both references dissolve into something neither artist made but both would have recognized: pure optical turbulence, the point where geometry becomes sensation and structure becomes feeling.

Grignani believed that "to affirm its useful role in visual communication, graphic art must rely on a large number of experiments to achieve perfect freedom." Escher believed that "we adore chaos because we love to produce order."

The slider panel invites you to experiment with both impulses at once.
