# Splatterhouse

A real-time 3D Gaussian splat demo in one HTML file: a starling murmuration at dusk over a wind-blown meadow, with an optional peregrine falcon attack.

Every splat is generated in the browser and drawn the way trained splat scenes are. Each one is an anisotropic 3D Gaussian (position, rotation, scale, colour, opacity), projected to a 2D covariance in the vertex shader, depth-sorted on the CPU each frame and alpha-blended back to front. There is no mesh and no depth buffer.

## Run it

Open `index.html`, or serve the repo with GitHub Pages. It needs WebGL2 and has no dependencies or build step.

## Controls

- **Meadow**: stand in the grass and watch. The view tracks and frames the flock like binoculars. Drag to look, pinch to zoom, tap Meadow again to re-track.
- **Orbit**: circle the flock from below against the sky.
- **Ride**: fly behind one starling inside the flock.
- **Falcon** (`F`): a peregrine makes three stoops at the flock edge.
- **Slow-mo** (`S`): 0.2× time, so the 13 Hz wingbeat becomes visible.
- **Starlings**: 150 to 2,400 birds.

## The flight model

The flock follows StarDisplay (Hildenbrandt, Carere & Hemelrijk 2010, *Behavioral Ecology* 21:1349), which reproduced the shapes and dynamics of real roosting murmurations:

- **Topological neighbours.** Each bird aligns with and is attracted to its 7 nearest neighbours, whatever their distance. Ballerini et al. (2008) measured 6–7 in real flocks. There is a 90° blind cone behind each bird.
- **Separation.** A 0.2 m hard sphere, then a Gaussian falloff. The flock settles at a nearest-neighbour distance of about 0.8–1 m.
- **Cohesion weighted by centrality.** Birds on the flock edge are pulled inward more strongly than birds in the interior.
- **Aerodynamics.** Cruise speed is 10 m/s, with relaxation back toward it. Lift scales with v² along the bird's up vector, and gravity acts throughout.
- **Turning only by banking.** Social and roost forces never yaw a bird directly. Their sideways component sets a target roll angle, and the tilted lift vector does the turning. Banked birds lose height in turns, as real ones do.
- **Roost attraction.** Birds turn back toward a sleeping site once they stray more than 48 m from it, which keeps the flock wheeling overhead. The preferred altitude drifts slowly.

Two simplifications are not from the paper. Small, slow noise forces keep the alignment near the 0.96 measured in real flocks instead of locking at 1.0. A weak pull toward the main cluster stands in for StarDisplay's longer perception range, so fragments regroup after an attack.

**Escape waves.** When the falcon comes within 24 m, nearby birds perform a "zig": a fast roll to one side and back. Neighbours copy a zig they see after 0.1 s, with slight damping per hop. This follows Hemelrijk, van Zuidam & Hildenbrandt (2015), who showed that the dark bands seen in real murmurations are orientation waves, not density waves: rolled birds briefly show more wing area. In the sim the wave travels at about 14–16 m/s, close to the 13.4 m/s measured in real flocks. Birds within 10 m of the falcon also flee radially, opening a gap around it.

**Wings.** Each bird flaps at about 13.3 Hz, the wind-tunnel value for European starlings, with roughly 47° stroke amplitude. The wrist flexes and sweeps on the upstroke. Flapping alternates with short glides, and wings tuck back in dives.

## How the splats work

- **Data.** Each splat is four RGBA32F texels in one data texture. Static splats (grass, trees, clouds) are uploaded once. Bird splats are rebuilt and re-uploaded every frame into a dynamic region of the same texture.
- **Birds.** Every bird is posed on the CPU: a body frame from its velocity and bank angle, two-segment wings, and a tail. Near birds use 46 splats, with separate top and underside wing splats that fade by facing, so wings look glossy black from above and grey-brown from below. Distant birds collapse to 3 splats.
- **Distance.** Bird splats keep their physical coverage with a mip-style opacity correction, so far-off birds read as faint specks and density shows up as darkness.
- **Meadow.** The grass tiles a 36 m torus centred on the camera. It sways in a periodic wind field, thins out with distance (each blade widening to compensate), and fades into a ray-traced ground plane. A sunset backlight term makes the grass tips glow.
- **Sort.** Visible splats are culled against the frustum and ordered with a 16-bit counting sort.
