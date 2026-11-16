# Splatterhouse

A real-time 3D Gaussian splat demo in one HTML file: a man with a sword and a pistol walks or runs through a wind-blown meadow.

Nothing is scanned or loaded. Every splat is generated in the browser at startup, then drawn the way trained splat scenes are: each one is an anisotropic 3D Gaussian (position, rotation, scale, colour, opacity), projected to a 2D covariance in the vertex shader, depth-sorted on the CPU each frame and alpha-blended back to front. There is no depth buffer and no mesh.

## Run it

Open `index.html`, or serve the repo with GitHub Pages. It needs WebGL2 and has no dependencies or build step.

## Controls

- Drag to orbit, pinch or scroll to zoom
- **Walk / Run** switches pace (`R` or `Shift`)
- **Slash** swings the sword (`Space` or `E`)
- **Fire** shoots the pistol (`F`)
- **Splat size** shrinks the Gaussians so you can see the point cloud underneath

## How it works

- **Data**: every splat is four RGBA32F texels in one data texture. The only per-instance attribute is a sorted index.
- **Character**: about 13k surface-aligned splats sampled on tapered capsules, each bound to one of 19 bones. Bone matrices are uniforms, so the splats rotate and re-light with the skeleton.
- **Animation**: a procedural gait cycle blended between walk and run, with the lower foot planted on the ground and a flight phase when running. Slash and fire are keyframed overlays on the arms.
- **Field**: grass tufts tile a 36 m torus that scrolls under the runner, so the meadow never ends. Tufts sway on a periodic wind field, bend away from his feet, thin out with distance (and widen to compensate), and fade into a ray-traced ground plane at 17 m.
- **Sort**: visible splats are culled against the frustum and ordered with a 16-bit counting sort. Ground discs are forced behind everything near, which removes the usual big-splat popping.
