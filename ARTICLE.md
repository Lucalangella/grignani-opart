# Encoding Perception: Recreating Franco Grignani's Op Art with GPU Shaders

Franco Grignani spent four decades proving that straight lines could lie. Working at a desk in Milan with nothing but a ruler, a sharp pencil, and black tempera, he created images that twist, ripple, and recede into illusory depths -- optical distortions so convincing that viewers instinctively lean sideways to catch a different angle of a perfectly flat surface.

This project is an attempt to translate that physical process into code: a real-time, interactive WebGL experience that renders Grignani-style compositions entirely on the GPU, letting anyone explore the optical phenomena he mapped across 14,000 hand-drawn experiments.

---

## Who Was Franco Grignani

Born in 1908 in Pieve Porto Morone, a small town south of Milan, Franco Grignani trained as an architect in Turin before dedicating his life to the boundaries between design, perception, and visual research. He passed through almost every major Italian art movement of the twentieth century -- second-wave Futurism, Constructivism, geometric abstraction -- before arriving at what he would spend decades refining: a systematic, almost scientific investigation of how the human eye can be tricked, guided, and unsettled by geometry alone.

His best-known creation is the Woolmark logo, a set of interlacing black bands that suggest a skein of wool. It was voted "Best Logo of All Time" by Creative Review in 2011 and has appeared on over five billion products worldwide. But the logo is almost incidental to his larger body of work -- thousands of black-and-white compositions that use grids, moiré patterns, and carefully calculated distortions to produce the sensation of depth, movement, and tension on a flat surface.

He called this sensation *trauma* -- not in the clinical sense, but as a deliberate disruption of visual expectation. "If you intervene by shifting your attention to a different, eccentric point," he wrote in 1971, "it automatically creates a state of mind of tension that multiplies interest and communication, becoming stronger as it manages to evoke a kind of physical discomfort."

He was an Op Art pioneer who arrived at optical research a decade before the movement had a name, yet was notably absent from MoMA's 1965 "The Responsive Eye" exhibition that brought Op Art to mainstream attention. While Vasarely published manifestos and Riley gained gallery fame, Grignani worked in seclusion, his influence spreading not through the art world but through commercial design -- Pirelli, Fiat, Dompé, Alfieri & Lacroix.

He never considered himself an artist. "I'm scared of being called an artist," he once said. "I simply indicated, or tried to indicate, a new graphic language."

---

## The Technique: Illusion Through Structure

What makes Grignani's work so relevant to creative coding is that his process was essentially algorithmic. Despite working by hand, his methodology reads like a shader program.

His **hyperbolic** works -- large-format pieces on Schoeller cardboard -- were created by dividing a 50x50 cm surface into a grid of 22x22 squares, each unit measuring just 3mm. He would then apply named distortion systems -- Pavia, Milano, Roma, Chicago -- each a different set of rules for how the grid should warp. Modules of geometric marks (typically 5x3 units) were placed at calculated diagonal offsets, creating undulations through precise positioning rather than curved drawing.

As he explained: "The twists are not produced by curved signs but by the gradual meeting of structures placed diagonally with grid coordinates."

His spacing followed integer progressions -- 14, 13, 12, 11... -- adaptable to different canvas sizes, functioning as what a programmer would recognize as parametric functions. He maintained detailed notebooks filled with mathematical formulas that governed these transformations. Each distortion grid was, in essence, a hand-computed transformation matrix.

This is the conceptual bridge to creative coding. Grignani was performing, by hand and pencil, the same operations that a fragment shader executes millions of times per second: coordinate transformation, power-law scaling, sinusoidal displacement, rotational mapping.

---

## Building the Experience

### The Pattern: Tumbling Blocks

The foundation of the piece is a **tumbling blocks** tessellation -- an arrangement of rhombuses that the eye reads as isometric cubes, half lit and half in shadow. Grignani used similar isometric grids as the raw material for his distortions.

In the shader, this pattern is generated through triangular-grid mathematics. Cartesian UV coordinates are skewed into triangular-grid axes using the transformation:

```
j = y * 2 / sqrt(3)
i = x - y / sqrt(3)
```

Each parallelogram cell splits into two triangles. The expression `(ci - cj) mod 3` assigns each triangle to one of three rhombus directions -- the three faces of an isometric cube. The "top" face renders white; the two "side" faces render black. The result is a seamless, infinitely tiling pattern of cubes emerging from pure arithmetic.

### The Distortion Pipeline

The real work happens before the pattern is sampled. The shader warps the coordinate space through a five-stage distortion pipeline, each stage modeled on a different aspect of Grignani's visual vocabulary:

**1. Fold Axis Rotation**

The coordinate system rotates to align with a diagonal "fold axis" -- the line along which the primary distortion occurs. This mirrors Grignani's technique of placing geometric structures at angles to orthogonal grids, creating tension between the inherent structure and the imposed angle.

**2. Perspective Foreshortening**

A power-law function -- `sign(s) * pow(|s|, bulgePower)` -- compresses coordinates near the origin and stretches them at the periphery (or vice versa, depending on the exponent). This creates the same forced-perspective effect that Grignani achieved by progressively tightening his grid spacing toward a vanishing point. It is the mathematical equivalent of watching railroad tracks converge.

**3. Dual-Axis Tilt**

A second power-law compression along the perpendicular axis adds depth to the perspective. Where the first axis creates convergence, the second creates the impression of a surface tilting away from the viewer -- a plane receding into imagined space.

**4. Sinusoidal Folds**

Two layers of sinusoidal displacement create valley-and-ridge folds across the grid:

```glsl
float ripple = waveAmp * sin(s * waveFreq);
t += ripple;
s += waveAmp * 0.4 * sin(t * waveFreq * 1.3);
```

The primary ripple displaces perpendicular to the fold axis. A secondary cross-ripple at 1.3x the base frequency adds visual complexity without appearing random -- the irrational ratio prevents the two wave patterns from locking into a simple repetition. This dual-wave approach produces the kind of flowing, organic distortion visible in Grignani's pieces, where rigid geometric grids appear to ripple like fabric.

**5. Twist**

A rotational distortion proportional to distance from the convergence point:

```glsl
float twistAngle = twist * sqrt(s*s + t*t);
```

This creates a vortex effect -- the grid appears to spiral inward toward the pinch point. The square-root relationship means the twist accelerates smoothly rather than linearly, producing a natural-looking spiral.

### Interactivity

All ten parameters -- grid scale, distortion strength, convergence point, wave amplitude and frequency, bulge power, twist, fold angle, and perspective tilt -- are exposed as real-time sliders. The values pass to the shader as uniforms, meaning the GPU recalculates every pixel every frame without any shader recompilation.

This interactivity is where the project diverges most from Grignani's process. His exploration of parameter space was inherently sequential -- each variation required a new drawing, days of precise manual work. The shader collapses that timeline, letting you sweep through distortion configurations in real time and develop an intuitive feel for how the parameters interact. You can find compositions that Grignani might never have reached because the search space is simply too vast to explore by hand.

---

## The Technology

The rendering stack is deliberately minimal:

- **Three.js** and **React Three Fiber** provide the WebGL canvas and a React-friendly way to manage the shader material
- **A single GLSL fragment shader** handles all pattern generation and distortion -- no vertex manipulation, no textures, no post-processing
- **React 19** manages the UI state for the control panel
- **An orthographic camera** ensures that all perceived depth and distortion comes from the shader mathematics, not from the 3D scene itself

The choice to put everything in the fragment shader is deliberate. Every pixel independently computes its own distorted coordinate and samples the tumbling blocks pattern at that location. There is no mesh deformation, no ray marching -- just a flat plane and a mathematical function from screen position to black or white. This mirrors the conceptual purity of Grignani's work: the three-dimensionality is entirely illusory, arising from pattern and distortion alone.

---

## What Grignani Would (Maybe) Think

Grignani spent his career constructing what he called "a very independent visual vocabulary which aimed at becoming a language able to connect scientific rigor with artistic production." He worked at a time when the tools available -- ruler, pencil, compass, photographic emulsion -- imposed a natural speed limit on experimentation. Each variation was an investment of hours.

The GPU removes that constraint. What took days by hand now recalculates in microseconds. But the underlying questions remain exactly the same: How does a grid distortion create the sensation of depth? At what point does a subtle ripple become visually disorienting? Where is the threshold between order and optical chaos?

Grignani himself was not precious about tools. His architectural training had, as he said, transformed him "from poet to constructor," and he embraced whatever instruments served the investigation -- from photographic lenses and broken mirrors to industrial textured glass. He used printing presses, darkrooms, and graphic reproduction processes as creative tools, not just production infrastructure.

It seems reasonable to imagine that he would have recognized the shader as another instrument -- one that happens to execute his distortion grids at the speed of light.

> "To affirm its useful role in visual communication, graphic art must rely on a large number of experiments to achieve perfect freedom."
> -- Franco Grignani

This project is a small contribution to that number.
