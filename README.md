# Splatterhouse

A real-time 3D Gaussian splat demo in one HTML file: birds at dusk over a wildflower meadow, above a barn, a silo, fenced fields and a mix of trees. Seven kinds of flier are on the menu, each with its own rules: starlings in a murmuration, dunlin, jackdaws, pigeons, geese in V formation, white storks circling in thermals, and a speculative reconstruction of *Pteranodon*. Flocks split apart and rejoin, and up to four peregrine falcons can attack at once.

Every splat is generated in the browser and drawn the way trained splat scenes are. Each one is an anisotropic 3D Gaussian (position, rotation, scale, colour, opacity), projected to a 2D covariance in the vertex shader, depth-sorted on the CPU each frame and alpha-blended back to front. There is no mesh and no depth buffer.

## Run it

Open `index.html`, or serve the repo with GitHub Pages. It needs WebGL2 and has no dependencies or build step.

## Controls

| | |
|---|---|
| **Meadow** (`1`) | Stand in the grass. The view follows the largest flock and keeps the horizon in shot while the birds are low. Drag to look, scroll or pinch to zoom, tap Meadow again to re-track. |
| **Orbit** (`2`) | Circle all the flocks from below, against the sky. |
| **Ride** (`3`) | Fly behind one bird inside the flock. |
| **Fly** (`4`) | Free flight. `W` `A` `S` `D` move along the view, `Space` climbs, `C` descends, `Shift` goes faster, drag to look, scroll sets speed. Pressing any movement key in another view takes off from where you are. On touch: drag to look, pinch to move. |
| **Falcon** (`F`) | Launch a peregrine. Press again for more, up to four at once. Not offered for geese, storks or *Pteranodon*. |
| **Split** (`X`) | Make the largest flock tear in two now. Not offered for geese, storks or *Pteranodon*. |
| **Slow-mo** (`T`) | 0.2× time, so the 13 Hz wingbeat becomes visible. |
| **Species menu** | Starlings, dunlin, jackdaws, pigeons, geese, white storks or *Pteranodon*. Changing species restarts the birds. |
| **Slider** | How many birds. The range depends on the species: 150 to 2,400 starlings, 7 to 120 geese. |

## The flight model

This section and the next describe the starlings. The other species are variations on it or separate models, described under [Species](#species).

The flock follows StarDisplay (Hildenbrandt, Carere & Hemelrijk 2010, *Behavioral Ecology* 21:1349), which reproduced the shapes and dynamics of real roosting murmurations:

- **Topological neighbours.** Each bird aligns with and is attracted to its 7 nearest neighbours, whatever their distance. Ballerini et al. (2008) measured 6–7 in real flocks. There is a 90° blind cone behind each bird.
- **Separation.** A 0.2 m hard sphere, then a Gaussian falloff. The flock settles at a nearest-neighbour distance of about 1 m.
- **Cohesion weighted by centrality.** Birds on the flock edge are pulled inward more strongly than birds in the interior.
- **Aerodynamics.** Cruise speed is 10 m/s, with relaxation back toward it. Lift scales with v² along the bird's up vector, and gravity acts throughout.
- **Turning only by banking.** Social and roost forces never yaw a bird directly. Their sideways component sets a target roll angle, and the tilted lift vector does the turning. Banked birds lose height in turns, as real ones do.
- **Roost attraction.** Birds turn back toward the roost (the barn and the trees around it) once they stray more than 48 m from it. The preferred altitude drifts slowly, and birds climb over the barn, silo and treetops.
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

## Species

Each species has its own body (built from one parametric splat bird: wingspan, neck, bill, tail, wingtip shape, colours), its own speed and wingbeat, and its own rules. Three of them reuse the starling model with changes suggested by field studies; they are not models fitted to data. Geese and storks are separate models.

| Species | Span | Cruise | Wingbeat | What is different |
|---|---|---|---|---|
| Starling | 0.40 m | 10 m/s | 13 Hz | The model above. |
| Dunlin | 0.36 m | 14 m/s | 12 Hz | Tighter, faster, lower. Turns start every few seconds and pass through the flock with less than half the starlings' copying delay. Brown above, white below, so the flock changes shade as it banks. |
| Jackdaw | 0.70 m | 12 m/s | 5.5 Hz | About 80% of birds have a mate. Mates hold 1.3 m apart and fly each other's course. Each bird attends to 3 neighbours, not 7, and the flock is looser. |
| Pigeon | 0.68 m | 15 m/s | 7 Hz | A small kit circling the barn. Every bird has a rank. A bird weights each neighbour's heading by that neighbour's rank, and high-ranked birds wander more and fly slightly faster, so they end up at the front and the rest follow their turns. Three colour morphs. |
| Goose | 1.6 m | 16 m/s | 3.8 Hz | V formation, below. |
| White stork | 2.0 m | 10.5 m/s | soaring | Thermal soaring, below. |
| *Pteranodon* | 3.8 m and 5.8 m | 12 m/s | soaring | Speculative, below. |

**Dunlin.** The short copying delay stands in for Potts's chorus-line account (1984, *Nature* 309:344), in which a bird times its turn from the wave approaching it and so reacts faster than its own reaction time would allow.

**Jackdaws.** Ling et al. (2019, *Nature Ecology & Evolution* 3:943) tracked wild jackdaws in 3D and found that mated pairs fly together inside the flock and that paired birds interact with fewer neighbours.

**Pigeons.** Nagy et al. (2010, *Nature* 464:890) fitted GPS loggers to homing pigeons and found a hierarchy in who copies whose turns. Here the hierarchy is assigned at random when the kit takes off.

**Geese.** Each goose picks the nearest goose ahead as its guide and flies for a place 1.4 wingspans behind it and 1.06 wingspans to one side, which is where a wingtip leaves rising air (Lissaman & Shollenberger 1970, *Science* 168:1003). The rest is bookkeeping that makes two tidy arms:

- A place belongs to the goose nearest it. A later arrival queues behind that goose, so each arm is a single file.
- A goose keeps to the arm its guide is on. The last goose of the longer arm crosses to the shorter one.
- The front bird steers. After 25 to 55 seconds in front it eases off for a few seconds, the line slides past it and it falls in further back (Voelkl et al. 2015, *PNAS* 112:2115, found ibises share the lead this way).
- Each goose locks its wingbeat to its guide's, delayed by the distance between them, so the flapping runs down the arm as a wave (Portugal et al. 2014, *Nature* 505:399, in ibises).
- More than 29 geese fly as several skeins, each on its own circuit and height.

One part is control engineering and not biology. A goose that matches speed only with the bird ahead amplifies every wobble, and a long arm cracks like a whip in turns. Each goose therefore matches speed partly with its guide and partly with the front bird, using the front bird's course as it was when it passed that point, so the V wheels round together.

**White storks.** Three thermals drift across the field, each lasting about 75 to 120 seconds. A stork circles in one, turning the same way as the rest of the kettle, and climbs at the updraft minus its own sink rate, which grows with bank angle. Near the top it leaves and glides to another thermal, losing height on the way. Bold birds leave first and choose the thermal by its strength and distance; followers leave once much of the kettle has gone and head for the thermal with the most storks (Flack et al. 2018, *Science* 360:911, found leaders and followers of this kind in migrating storks). Specks of thistledown rise in each thermal so the air can be seen. Distances are compressed about tenfold so the cycle fits over the field: real thermals carry storks up a kilometre and the glides run for kilometres, so the glides here are much steeper than a real stork's.

***Pteranodon*.** Nobody has seen a pterosaur fly, so this one is a reconstruction with the guesses marked.

| | Basis |
|---|---|
| Two sizes, two females to each male, large crest on males only | Fossils. Bennett's survey of the Niobrara Chalk specimens found a small class (3.0–4.8 m span) about twice as common as a large class (4.8–6.7 m), differing in crest and pelvis (Bennett 1992, *Journal of Vertebrate Paleontology* 12:422). |
| Thermal soaring: circle up, glide on | Flight modelling. Goto et al. (2022, *PNAS Nexus* 1:pgac023) found *Pteranodon* suited to soaring on updrafts over the sea, as frigatebirds do, though earlier work argued for albatross-style dynamic soaring and the answer depends on uncertain wing drag. |
| Living in groups | Weak, by analogy. Bone beds show some other pterosaurs were gregarious (*Caiuajara*: Manzig et al. 2014, *PLoS ONE* 9:e100005). For *Pteranodon* the skewed sex ratio hints at a colonial, polygynous life like a sea lion's. |
| Leaving a thermal together and gliding in loose files, each behind and beside the one ahead | A guess. Pelicans and frigatebirds do it, and the saving from a neighbour's wake grows with wingspan. |
| Speeds, sink rates, circle radius | Rough figures in the range the flight models give; not fitted. |
| Colours | Invented. Pale below and darker above as in most seabirds, with the male's crest coloured as a display structure. |

The model is the storks' with wider thermals, wider circles and the file-gliding rule added. As with the storks, distances are compressed about tenfold, and there is no sea: a real *Pteranodon* would have been doing this over open water, far from shore.

## How the splats work

- **Data.** Each splat is four RGBA32F texels in one data texture. Static splats (grass, barn, trees, hills, clouds) are uploaded once. Bird splats are rebuilt and re-uploaded every frame into a dynamic region of the same texture.
- **Birds.** Every bird is posed on the CPU: a body frame from its velocity and bank angle, two-segment wings, and a tail. All the fliers come from one parametric body. Near birds use 39 to 63 splats depending on species, with separate top and underside wing splats that fade by facing, so a starling's wings look glossy black from above and grey-brown from below, and a dunlin flashes white when it banks. Distant birds collapse to 3 to 7 splats.
- **Barn and silo.** About 36,000 splats. The shell is surface splats laid out on panels, each with its thin axis along the surface normal so the sunset lights it properly. Fine detail is drawn on top with long, thin splats used as lines: board-and-batten strips, white corner boards, rakes and bargeboards, window casings and glazing bars, standing seams and patched sheets on the gambrel roof, gutters and downspouts.
  - *Front:* X-braced sliding doors standing ajar on a steel track, with light inside and a lantern above; a hay hood with a hoist beam, pulley and rope; hay spilling from the loft door; a fieldstone foundation; Virginia creeper turning red up one corner.
  - *Sides:* windows, some lit from inside; a painted barn quilt on the sunlit wall; a ladder left leaning; a lean-to with a tin roof, posts and stacked straw bales.
  - *Roof:* a louvred cupola with a weathercock, lightning rods with glass balls, and a few starlings perched on the ridge.
  - *Yard:* milk cans, a barrel, bales, an old wagon wheel, and a water trough in the paddock.
  - *Silo:* concrete staves, steel hoops, a ribbed dome, a caged ladder and a filler pipe.
- **Meadow.** The meadow covers the whole field in three nested rings that tile around the camera. Within 17 m every blade, seed head and flower is its own splat. From 14 to 68 m each splat is a tussock or a patch of flowers, and from 55 to 272 m a larger one. Beyond that, and underneath everything, is a ray-traced ground plane in the same colours.
- **World maps.** Two 512 m textures drive all three rings and the ground plane, so they always agree: one holds dryness, flower density, dominant species and grass vigour; the other holds short grass, shade, bare earth and lamplight. Flower splats carry random numbers instead of a colour. The shader turns each into a poppy, oxeye daisy, cornflower, buttercup, knapweed or clover from the species map at its world position, or hides it where the density map is low. That is what makes the flowers grow in drifts.
- **Track, paddock and shadows.** A two-rut farm track runs from the barn doors through a field gate and across the meadow. The white-fenced paddock beside the barn is grazed short. The long sunset shadows of the barn, silo and every tree are painted into the shade map, so they fall across grass, flowers and ground alike.
- **Fences.** Post-and-rail fences enclose the barn field and run off toward the horizon, with a five-bar gate where the track crosses and shrubs along the lines, some with berries.
- **Trees.** Eight kinds, each with its own generator: spreading oaks, orange maples, white-barked birches in clumps, conical spruces with drooping sprays, a row of Lombardy poplars, a weeping willow with hanging withies, a small apple orchard behind the barn, and one dead snag. Leaves are translucent splats that glow when backlit and sway slightly.
- **Sort.** Each meadow ring is stored in a 12×12 grid of cells, so cells outside the ring or the view are skipped without reading their splats. What survives is ordered with a 16-bit counting sort, linear in depth out to 40 m and logarithmic beyond.
