# Explanation

## Contribution grid to 3D space

The GitHub contribution calendar is a 2D grid: weeks along the X axis, days of the week (Sunday through Saturday) along the Y axis. The visualization maps this to 3D coordinates:

- **X axis** = week index. The grid is centered at the origin by subtracting half the total week count. Each cell is spaced `0.38` units apart.
- **Z axis** = day of week (0-6), offset by -3 to center vertically.
- **Y axis** = fixed at 0 for all nodes. The grid is flat.

This produces a rectangular plane of nodes floating in space. The contribution count determines each node's size and color tier, not its Y position.

The camera orbits this plane. Because the grid is thin on the Y axis, the top-down view shows the traditional GitHub graph layout, while the perspective and front views reveal the depth created by the glow layers and particle systems.

## Layered glow effect

Days with contributions are not single spheres. Each node is a group of three concentric spheres:

1. **Core** -- the smallest sphere, using the tier's `base` color at 0.9 opacity with normal blending. This is the bright center of the orb.
2. **Inner glow** -- the same geometry scaled to 1.8x, using the `glow` color at 0.08 opacity with additive blending. This creates a soft halo around the core.
3. **Outer halo** -- scaled to 3.2x, 0.03 opacity, also additive. This produces the wide, faint bloom that makes nodes appear to emit light.

Additive blending is the key technique. It adds the sphere's color values to whatever is already in the frame buffer, so overlapping halos intensify rather than occlude. The low opacity values prevent blowout while still producing visible glow where multiple nodes are close together.

Zero-contribution days get a single tiny sphere (radius 0.02) at 0.15 opacity with normal blending. They appear as dim dots.

### Pulse animation

Each active node pulses by oscillating its group scale:

```
scale = 1 + sin(time * pulseSpeed + idHash) * 0.18
```

`pulseSpeed` varies per node (1.2 to 2.0) and `idHash` offsets the phase. This makes each node pulse at a slightly different rate and phase, avoiding a synchronized "breathing" effect.

On hover, the pulse is suppressed and the group scales to 1.8x. The inner and outer glow opacities increase to 0.25, making the hovered node flare.

## Particle systems

Two instanced mesh systems add depth to the scene.

### Core particles

1500 small green spheres distributed in a squashed spherical volume (radius 12, Z compressed by 0.3x to match the grid's aspect ratio). Each particle drifts along a sinusoidal path:

```
x = base.x + sin(t * speed + offset) * drift
y = base.y + cos(t * speed * 1.3 + offset) * drift * 0.4
z = base.z + sin(t * speed * 0.8 + offset * 1.7) * drift
```

The multipliers on the speed and offset terms ensure the three axes are not synchronized, producing organic-looking motion rather than elliptical orbits. Additive blending makes these particles contribute to the green glow of the contribution area.

### Ambient particles

800 white spheres in a 40x15x20 box around the scene. These move more slowly and use normal blending at very low opacity (0.04). They create a starfield effect visible when orbiting the scene, adding depth cues outside the contribution grid.

Both systems use `InstancedMesh` for performance. A single geometry and material are shared across all instances. Per-frame, a dummy `Object3D` is repositioned and its matrix is written to the instanced buffer.

## Zoom-to-recent focus shifting

The orbit target is not fixed. It shifts along the X axis based on how close the camera is:

```
zoomT = 1 - clamp((distance - minDistance) / (maxDistance - minDistance), 0, 1)
goalX = centerX + zoomT * (recentX - centerX)
```

`centerX` is 0 (the grid's center). `recentX` is the X position of the last week in the dataset. When the camera is zoomed out (`zoomT` near 0), the target stays at the center of the grid. As the user zooms in (`zoomT` approaches 1), the target slides toward the most recent weeks.

The shift is smoothed with `target.x += (goalX - target.x) * 0.04` per frame, so it feels like a gentle drift rather than a snap. The effect is that zooming in naturally draws attention to recent activity.

## Neural web strands

40 cubic Bezier curves connect random points within the bounds of active contributions. They serve no data purpose -- they exist as a visual texture layer that fills the space between nodes.

Each strand has four control points randomly placed within the contribution area (with a 0.3x shrink factor to keep them near the center). The Y values range from -0.3 to 0.3, giving the strands slight vertical displacement above and below the grid plane.

The strands pulse in opacity over time:

```
opacity = baseOpacity + (sin(t * speed + phase) * 0.5 + 0.5) * (peakOpacity - baseOpacity)
```

Base opacity ranges from 0.01 to 0.035. Peak is 0.05. The speeds vary per strand (0.2 to 0.8). This creates a subtle plasma-like pulsing across the grid -- some strands brightening as others dim.

The low opacity values and additive blending mean the strands are barely visible head-on. They become most apparent from oblique angles where many strands overlap.

## Scene drift

The entire grid group rotates slowly:

```
rotation.y = sin(t * 0.05) * 0.04
rotation.x = cos(t * 0.03) * 0.015
```

This is a subtle oscillation (about 2.3 degrees Y, 0.9 degrees X) that prevents the scene from feeling static when the user is not interacting. Combined with `autoRotate` on the orbit controls (0.15 degrees/frame), the scene has constant low-level motion.

## Deterministic randomness

All random values come from a seeded Lehmer PRNG (Park-Miller variant). The generator:

```javascript
s = (s * 16807) % 2147483647
return (s - 1) / 2147483646
```

Four separate seeds are used for different systems (nodes, strands, core particles, ambient particles). This means every visitor sees the same particle positions, strand paths, and pulse offsets. The visualization is deterministic given the same `contributions.json` input.
