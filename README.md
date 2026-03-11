# Interactive Op Art

An interactive WebGL experience inspired by Franco Grignani's optical distortions and M.C. Escher's impossible tessellations, built with GPU shaders and real-time parameter control.

**[Live Demo](https://lucalangella.github.io/interactive-opart/)**

## About

This project sits at the intersection of two visual traditions:

**M.C. Escher's** tessellations -- interlocking geometric forms that trick the eye into reading flat patterns as three-dimensional structures. The tumbling blocks pattern at the heart of this piece is a direct descendant of Escher's isometric illusions: rhombuses arranged so the brain involuntarily assembles them into cubes.

**Franco Grignani's** optical distortions -- systematic warping of grids and geometric structures to create sensations of depth, movement, and visual tension. Grignani (1908--1999) was an Italian graphic designer and Op Art pioneer who created over 14,000 experimental works exploring visual perception. He is best known for designing the Woolmark logo -- voted "Best Logo of All Time" by Creative Review.

Where Grignani famously achieved his distortions using only straight lines -- "the twists are not produced by curved signs but by the gradual meeting of structures placed diagonally with grid coordinates" -- this project takes a different path. It applies actual mathematical curves (sinusoidal waves, power-law functions, rotational mappings) to an Escher-inspired tessellation, using the GPU to compute distortions that would be impossible to draw by hand. The result is a hybrid: Escher's geometry filtered through Grignani's perceptual vocabulary, executed at computational speed.

## How It Works

The experience is built on a single GLSL fragment shader that renders an isometric **tumbling blocks** pattern and passes it through a multi-stage distortion pipeline:

1. **Fold Axis Rotation** -- Aligns the coordinate system along a diagonal axis, creating tension between the tessellation's inherent geometry and the imposed angle
2. **Perspective Foreshortening** -- Power-law scaling (`sign(x) * pow(|x|, n)`) compresses space toward a vanishing point, simulating depth on a flat plane
3. **Dual-axis Tilt** -- A second compression axis adds the impression of a surface receding into space
4. **Sinusoidal Folds** -- Valley-and-ridge ripples displace the grid with a cross-ripple at 1.3x frequency, producing flowing, fabric-like distortion
5. **Twist** -- Rotational distortion proportional to distance from the convergence point, producing a vortex effect

All distortions update per-frame through shader uniforms -- no recompilation needed -- giving smooth, immediate feedback as you explore the parameter space.

## Controls

Ten sliders let you shape the composition in real time:

| Parameter | Effect |
|-----------|--------|
| Grid Scale | Pattern density (2--30) |
| Distort Strength | Overall distortion intensity |
| Pinch X / Y | Convergence point position |
| Wave Amplitude | Fold depth |
| Wave Frequency | Fold density |
| Bulge Power | Perspective compression |
| Twist | Rotational vortex |
| Fold Angle | Diagonal axis direction |
| Persp Tilt | Secondary compression axis |

## Tech Stack

- **Three.js** + **React Three Fiber** -- WebGL rendering with React's component model
- **Custom GLSL shaders** -- All pattern generation and distortion computed per-pixel on the GPU
- **React 19** -- State management for interactive controls
- **Vite** -- Build tooling and dev server

## Run Locally

```bash
npm install
npm run dev
```

## Inspiration

Escher drew his tessellations by hand, exploiting the symmetries of the Euclidean plane to create impossible geometries that feel inevitable. Grignani divided 50x50 cm surfaces into 22x22 grids, applying named distortion systems (Pavia, Milano, Roma, Chicago) cell by cell -- hand-computed transformation matrices that warped straight lines into the illusion of curved space, without ever drawing a curve.

This project borrows from both but belongs fully to neither. It takes Escher's tessellation as raw material and subjects it to distortions that go beyond what Grignani allowed himself -- actual mathematical curves, computed per-pixel on the GPU 60 times per second. The result is a space where you can explore optical phenomena at a speed and scale that neither artist's manual tools could reach.
