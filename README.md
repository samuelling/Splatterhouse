# Splatterhouse

A real-time 3D Gaussian splat demo in one HTML file: starling flocks at dusk over a wind-blown meadow, wheeling above a barn, a silo and a stand of trees. The flocks split apart and rejoin, and up to four peregrine falcons can attack at once.

Every splat is generated in the browser and drawn the way trained splat scenes are. Each one is an anisotropic 3D Gaussian (position, rotation, scale, colour, opacity), projected to a 2D covariance in the vertex shader, depth-sorted on the CPU each frame and alpha-blended back to front. There is no mesh and no depth buffer.

## Run it

Open `index.html`, or serve the repo with GitHub Pages. It needs WebGL2 and has no dependencies or build step.

## Controls

| | |
|---|---|
| **Meadow** (`1`) | Stand in the grass. The view follows the largest flock and keeps the horizon in shot while the birds are low. Drag to look, scroll or pinch to zoom, tap Meadow again to re-track. |
| **Orbit** (`2`) | Circle all the flocks from below, against the sky. |
| **Ride** (`3`) | Fly behind one starling inside the flock. |
| **Fly** (`4`) | Free flight. `W` `A` `S` `D` move along the view, `Space` climbs, `C` descends, `Shift` goes faster, drag to look, scroll sets speed. Pressing any movement key in another view takes off from where you are. On touch: drag to look, pinch to move. |
| **Falcon** (`F`) | Launch a peregrine. Press again for more, up to four at once. |
| **Split** (`X`) | Make the largest flock tear in two now. |
| **Slow-mo** (`T`) | 0.2× time, so the 13 Hz wingbeat becomes visible. |
| **Starlings** | 150 to 2,400 birds. |

## The flight model

The flock follows StarDisplay (Hildenbrandt, Carere & Hemelrijk 2010, *Behavioral Ecology* 21:1349), which reproduced the shapes and dynamics of real roosting murmurations:

- **Topological neighbours.** Each bird aligns with and is attracted to its 7 nearest neighbours, whatever their distance. Ballerini et al. (2008) measured 6–7 in real flocks. There is a 90° blind cone behind each bird.
- **Separation.** A 0.2 m hard sphere, then a Gaussian falloff. The flock settles at a nearest-neighbour distance of about 1 m.
- **Cohesion weighted by centrality.** Birds on the flock edge are pulled inward more strongly than birds in the interior.
- **Aerodynamics.** Cruise speed is 10 m/s, with relaxation back toward it. Lift scales with v² along the bird's up vector, and gravity acts throughout.
- **Turning only by banking.** Social and roost forces never yaw a bird directly. Their sideways component sets a target roll angle, and the tilted lift vector does the turning. Banked birds lose height in turns, as real ones do.
- **Roost attraction.** Birds turn back toward the roost (the barn and trees) once they stray more than 48 m from it. The preferred altitude drifts slowly, and birds climb over the barn, silo and treetops.
- **Wings.** Each bird flaps at about 13.3 Hz, the wind-tunnel value for European starlings, with wrist flex on the upstroke, flap-glide bursts and tucked wings in dives.

## Splitting and rejoining

A flock here is a connected component of the neighbour graph: two groups are separate flocks once no bird in one has a bird of the other among its seven neighbours. Because each bird only attends to seven neighbours, nothing glues a large flock together globally, so it can tear.

**What tears it.**

- *Turning waves.* Every several seconds a bird on the edge of a flock banks into a turn, and the birds that can see it copy the turn about 70 ms later. Attanasi et al. (2014, *Nature Physics* 10:691) tracked real starling turns starting from a few birds and sweeping through the flock at 20–40 m/s. If the wave reaches every bird, the flock wheels as one. If it dies out part-way, the birds that turned fly off as a separate flock. Papadopoulou et al. (2022, *R. Soc. Open Sci.* 9:211898) found the same in pigeons and in their model: one rule produces both collective turns and splits, and sharp turns by birds on the periphery mostly produce splits. Here the wave carries a hop budget: unlimited for a collective turn, short for a split.
- *Falcons.* Birds within 11 m of a falcon flee it, so a stoop through a flock punches a hole and often cuts it in two. Storms et al. (2019) catalogued this in real murmurations, where a flash expansion is often followed by a split.

**What brings it back.**

- *Attraction to flocks in view.* Every bird is drawn toward other flocks within about 100–140 m, with small groups drawn to large ones far more than the reverse. This is the long-range half of the projection model of Pearce et al. (2014, *PNAS* 111:10422), in which birds respond to the silhouette of the whole flock as well as to their neighbours. It is approximated here by attraction to flock centroids, not by an actual rendering of each bird's view.
- *Joining.* A group that comes close to another flock starts matching that flock's heading, and two groups that have just met steer onto their common course. Without this, groups mostly pass through each other and separate again. This rule is my own addition and is not taken from a paper.
- *The roost.* All flocks keep turning back over the same 48 m circle, so their paths cross every few seconds.

A split typically lasts 10–20 seconds, though a small group can stay out for a minute or more. In a 150 s unattended test run with 1,000 birds, the sky held one flock for about half the time, two for most of the rest, and three for a few seconds.

**Escape waves.** When a falcon comes within 24 m, nearby birds perform a "zig": a fast roll to one side and back. Neighbours copy a zig they see after 0.1 s. This follows Hemelrijk, van Zuidam & Hildenbrandt (2015), who showed that the dark bands seen in real murmurations are orientation waves, not density waves. In the sim the wave travels at about 14–16 m/s, close to the 13.4 m/s measured in real flocks.

Other simplifications that are not from the papers: small, slow noise forces keep alignment near the 0.96 measured in real flocks instead of locking at 1.0, and a gentle pull draws birds on a long straggling arm back toward the bulk of their own flock.

## How the splats work

- **Data.** Each splat is four RGBA32F texels in one data texture. Static splats (grass, barn, trees, hills, clouds) are uploaded once. Bird splats are rebuilt and re-uploaded every frame into a dynamic region of the same texture.
- **Birds.** Every bird is posed on the CPU: a body frame from its velocity and bank angle, two-segment wings, and a tail. Near birds use 46 splats, with separate top and underside wing splats that fade by facing, so wings look glossy black from above and grey-brown from below. Distant birds collapse to 3 splats.
- **Barn and silo.** About 22,000 surface splats laid out on panels: board-and-batten walls with per-board weathering, white trim, windows, X-braced sliding doors standing ajar with light inside, a hay-loft door, a rusting standing-seam gambrel roof, a louvred cupola with a weathervane, a stave silo with steel hoops, and a split-rail fence. Each splat's thin axis is the surface normal, so the sunset lights them properly.
- **Trees.** Recursive limbs with sprays of translucent leaf splats at the tips. Leaves glow when backlit by the sun and sway slightly.
- **Meadow.** The grass tiles a 36 m torus centred on the camera, so it follows you in Fly mode. It sways in a periodic wind field, thins out with distance, and fades into a ray-traced ground plane.
- **Sort.** Visible splats are culled against the frustum and ordered with a 16-bit counting sort, linear in depth out to 40 m and logarithmic beyond.
