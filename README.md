
CONNECTOME COMBAT LAB
A full working guide: the dataset, the 3D world, the neural nets, the genetic algorithm, and the code that ties them together
Built on 139,248 real, classified neurons from the published male Drosophila (fruit fly) whole-brain connectome
139,248
real neurons	7 → 9 → 3
network shape
 
Contents
Contents	1
1. What this project actually is	1
1.1 The honest version, up front	1
2. The dataset, explained simply	1
2.1 The columns that matter for this project	1
2.2 Neurons by functional class	1
2.3 Neurotransmitters — the brain’s chemical signals	1
2.4 The specific neuron types used in the game	1
3. How to use it — a step-by-step walkthrough	1
3.1 Starting your first duel	1
3.2 Auto-run, speed, and mutation controls	1
3.3 The four sidebar tabs	1
4. The 3D world, explained	1
4.1 The arena	1
4.2 The flies	1
4.3 The neuron nebula	1
4.4 Camera and controls	1
5. The neural network "brain," explained simply	1
5.1 The seven sensory inputs	1
5.2 The hidden layer and the decision	1
5.3 Turning outputs into movement	1
6. Combat rules — how a fight actually plays out	1
7. The genetic algorithm — how the flies get better	1
7.1 Crossover	1
7.2 Mutation	1
7.3 What this does and doesn’t guarantee	1
8. Code walkthrough	1
8.1 The embedded real data	1
8.2 The Brain class — the neural network itself	1
8.3 stepAgent() — turning brain output into movement	1
8.4 resolveCombat() — deciding damage	1
8.5 endDuel() — declaring a winner and evolving the loser	1
8.6 The render loop	1
8.7 Camera, layout, and data-driven UI	1
9. Limitations and things worth knowing	1
10. Ideas for extending it further	1

 
1. What this project actually is
Connectome Combat Lab is a browser game that turns a real scientific dataset into something you can watch, poke at, and play with. Two flies fight in a 3D arena. Neither fly is remote-controlled by you — each one is steered entirely by its own small neural network, and after every duel the loser’s network is rebuilt from a mix of the winner’s wiring and its own, then nudged with a bit of random mutation. Run enough duels and the two "lineages" of flies visibly get better at fighting, without a single line of hand-written fight logic.
The dataset behind it is the real supplemental annotation file from a published male Drosophila (fruit fly) whole-brain connectome — the complete map of every neuron in the fly’s brain, produced by imaging and reconstructing the brain at synapse-level resolution. That file supplies real neuron identities, real cell-type counts, and real 3D soma (cell body) positions, all of which show up directly in the game: as the point-cloud "brain nebula" floating behind the arena, and as the actual names labelling each sensory input and motor output of the fighting flies’ tiny neural nets.
This guide walks through every part of that in plain language: what the dataset contains, how the 3D scene is built, how the neural network "brain" makes decisions, how the genetic algorithm evolves the flies over time, and finally a section-by-section explanation of the actual code that runs all of it.
1.1 The honest version, up front
What’s real
Every neuron identity, cell-type name, cell-type count, and 3D position used anywhere in the game comes directly from the real annotation file — nothing there is invented.	What’s simulated
The file contains neuron annotations, not synapse-level wiring, so there is no real connectivity data to drive behaviour. The connection weights that actually control each fly are evolved by the genetic algorithm, not measured from the real brain.
Keep this distinction in mind through the rest of the guide: "real" always refers to identity, classification, and position data; "evolved" or "simulated" always refers to the numbers that decide behaviour.
2. The dataset, explained simply
The uploaded file is a tab-separated table with 139,248 rows — one row per neuron — and columns describing what kind of neuron it is, what chemical signal it sends, which side of the brain it sits on, and where its cell body is physically located in 3D space. Think of it as a census of every cell in the brain, rather than a wiring diagram of how they connect.
2.1 The columns that matter for this project
Column	What it means	How the game uses it
super_class	The neuron’s broad functional category — optic, sensory, central, descending, motor, and so on.	Colours every point in the 3D brain nebula and drives the Dataset tab’s breakdown chart.
cell_type	A specific, named neuron type (for example LC12 or DNg12_b), grouping neurons that repeat in a stereotyped way across the brain.	Supplies the real names attached to each sensory input and motor output in the Circuit tab.
flow	Whether signals mostly arrive (afferent), stay local (intrinsic), or leave (efferent) at that neuron.	Shown in the dataset notes as context on how information moves through the real brain.
top_nt	The neurotransmitter that neuron most likely uses to signal — acetylcholine, GABA, glutamate, dopamine, serotonin, or octopamine.	Colour-codes the neurotransmitter donut chart and the nebula’s palette choices.
pos_x / pos_y / pos_z	The real 3D coordinates of that neuron’s position in the imaged brain volume.	Directly plotted, after rescaling, as the point cloud floating behind the arena.
side	Which side of the brain the neuron is on — left, right, or center.	Available in the raw data; not currently separated visually in the scene.
2.2 Neurons by functional class
The single biggest category by far is "optic" — 77,541 neurons dedicated to processing what the fly sees, which makes sense given how much of a fly’s brain is devoted to vision.
 "Central" neurons (32,383) handle internal processing, and
 "sensory" neurons (16,907) cover touch, smell, taste, and other senses beyond vision. At the far end, "descending" neurons (1,303) are the ones that carry decisions out of the brain toward the body’s motor circuits — these are the real neurons the game borrows names from for its motor-output layer.
 
Figure 1 — every neuron in the file, grouped by its real super_class label.
2.3 Neurotransmitters — the brain’s chemical signals
Every neuron in the dataset is also tagged with its most likely neurotransmitter — the chemical it releases to pass a signal to the next neuron. Acetylcholine dominates, which is typical for excitatory signalling in insect brains; GABA and glutamate (mostly inhibitory) make up most of the rest, with smaller populations of dopamine, serotonin, and octopamine handling more specialised, modulatory roles.
 
Figure 2 — the real neurotransmitter mix across all classified neurons in the file.
2.4 The specific neuron types used in the game
Rather than inventing fictional "brain regions," the game reaches into the real data and pulls out the most common real cell types within two specific categories — visual_projection neurons (which relay processed visual information onward) for its sensory-input labels, and descending neurons (which carry decisions toward the body) for its motor-output labels.
 
Figure 3 — the ten most common real visual-projection cell types; the top seven name the fly’s sensory inputs.
 
Figure 4 — the ten most common real descending cell types; the top three name the fly’s motor outputs.
The pairing of a real neuron type to a game function (for example, labelling one input "distance to opponent" and tagging it LC12) is illustrative — it borrows a real, verified identity and cell count, but is not a claim that this specific neuron type actually performs that function in a real fly.
3. How to use it — a step-by-step walkthrough
3.1 Starting your first duel
●	Open the file in any modern browser. The 3D arena loads immediately with two flies waiting at opposite ends — cyan for Lineage A, magenta for Lineage B.
●	Click "Start duel" in the bottom-left of the footer bar. The flies begin moving on their own, driven entirely by their neural nets.
●	Watch the health bars in the top corners drain as flies land hits, and the centre banner announce a winner once one fly’s HP reaches zero or the 22-second timer runs out.
●	Click "Next duel" to run another round — the loser’s brain will already reflect what it learned from the previous fight.
3.2 Auto-run, speed, and mutation controls
Control	What it does
Auto-run toggle	Chains duels back-to-back automatically, pausing about a second between fights so you can read the result banner.
Speed slider (0.5×–3×)	Scales how fast simulated time passes — handy for watching many generations evolve quickly.
Mutation rate slider (2%–40%)	Controls how much randomness is injected into a loser’s new network on each evolution step. Low values converge slowly but steadily; high values explore more but can be erratic.
Reset lineages	Wipes both brains back to random starting weights and resets generation counters, win counts, and fitness history.
3.3 The four sidebar tabs
●	Duel — this duel’s damage dealt/avoided, survival time, a fitness-history line chart, and the lineage record.
●	Brain — switch between inspecting Lineage A or B and watch its sensory inputs, hidden-layer activity, and motor outputs update live, frame by frame.
●	Circuit — the real neuron types and cell counts behind every sensory input and motor output.
●	Dataset — the full real-data breakdown by super_class and neurotransmitter, plus the "what’s real vs simulated" note.
4. The 3D world, explained
The entire visual scene is built with Three.js, a JavaScript library for rendering 3D graphics in a web browser using WebGL. Nothing is pre-rendered or a video — every frame is computed and drawn live, which is what lets you freely drag to orbit the camera and zoom in and out.
4.1 The arena
The floor is a flat circular disc 15 units in radius, with a polar grid (concentric rings and radial spokes, like a radar screen) laid over it so motion and distance are easy to judge. A thin glowing ring traces the arena’s boundary — flies that drift too close to it get gently pushed back toward the centre and turned inward, which is also fed back into their brains as a "wall proximity" sensory signal.
4.2 The flies
Each fly is built from simple 3D shapes rather than an imported model: a stretched sphere for the thorax, a smaller sphere for the head, another for the abdomen, and two flat, semi-transparent wing panels that flap on a simple sine-wave animation tied to the clock. A small coloured point light is attached to each fly’s body, so it visibly glows — and that glow intensifies sharply for the brief window when a fly commits to a lunge attack, giving a clear, readable cue for the game’s single most important action.
4.3 The neuron nebula
The glowing field of points drifting slowly above and behind the arena is not decoration — it is a real sample of neurons from the connectome file, plotted using their real 3D coordinates. Because the dataset is naturally very flat along one axis (the brain volume is thin in that direction), that axis is stretched about four times when placed in the scene, purely so the cloud reads as a 3D shape rather than a flat sheet; every other position value is used as measured. Each point is coloured by that neuron’s real super_class, using the same colour key as the Dataset tab’s charts.
4.4 Camera and controls
The camera orbits a fixed look-at point just above the arena centre. Dragging with the pointer adjusts azimuth (left/right) and elevation (up/down) angles directly; scrolling moves the camera closer or further along its current viewing direction. There is no automatic camera movement beyond this — what you see is always exactly where you’ve pointed it.
5. The neural network "brain," explained simply
Each fly carries one small feed-forward neural network — a fixed pipeline of numbers flowing one direction through three layers, with no memory of previous frames. Every single frame, the network is fed a fresh snapshot of the fly’s situation and immediately produces a fresh decision; nothing is stored between frames except the connection weights themselves.
 
Figure 5 — the fly brain’s architecture: 7 sensory inputs, 9 hidden units, 3 motor outputs, fully connected layer to layer.
5.1 The seven sensory inputs
On every frame, the game computes seven numbers describing the fly’s situation relative to its opponent and the arena, each scaled to sit roughly between -1 and 1:
●	Distance to opponent — how close the other fly is.
●	Bearing angle — which direction the opponent is relative to the fly’s current heading.
●	Own vitality — the fly’s own remaining HP, as a fraction.
●	Opponent vitality — the opponent’s remaining HP, as a fraction.
●	Opponent lunging — a flag for whether the opponent is mid-attack right now.
●	Wall proximity — how close the fly is to the arena boundary.
●	Energy reserve — how much stamina is left for the next lunge.
Each of these seven inputs is labelled in the Brain and Circuit tabs with the name and real cell count of an actual visual-projection neuron type from the dataset (for example, "distance to opponent" is tagged with the real type LC12). This is a naming choice built on real, verified neuron identities — not a claim that LC12 neurons literally compute opponent distance in a real fly.
5.2 The hidden layer and the decision
Those seven numbers are multiplied by a grid of connection weights and summed into nine "hidden units," each squashed through a tanh function (which forces any value into a smooth range between -1 and 1). Those nine hidden values are then combined again, through a second grid of weights, into exactly three output numbers: turn, thrust, and strike. The Brain tab’s bar chart shows these nine hidden values live, one bar per unit, which is the closest thing the game has to visualising the fly "thinking."
5.3 Turning outputs into movement
Output	Real neuron label	Effect in the arena
Turn	DNg12_b	Rotates the fly’s heading left or right, scaled by a fixed turn rate.
Thrust	DNpe008	Sets forward speed, up to a maximum cruising speed.
Strike / lunge	DNge091	Above a threshold, and if enough energy and cooldown allow it, triggers a fast, short lunge that can land damage on contact.
6. Combat rules — how a fight actually plays out
Underneath the visuals, combat is a small, deterministic set of physics rules applied every frame:
●	Both flies move continuously based on their own network’s turn and thrust outputs — there is no turn-taking.
●	A lunge is a short (about a fifth of a second) burst of much higher speed, triggered when the strike output crosses a threshold and the fly has enough energy and isn’t still cooling down from its last lunge.
●	Landing a lunge costs energy immediately, and a short cooldown prevents lunge-spamming; energy slowly regenerates over time either way.
●	When the flies come within striking range of each other, the outcome depends on who is lunging: a lunging fly hitting a non-lunging one deals solid damage; two flies lunging into each other simultaneously both take a smaller, shared "trade" of damage; two non-lunging flies bumping into each other deal no damage.
●	A duel ends the instant either fly’s HP reaches zero, or automatically after 22 simulated seconds if neither has been finished off — in which case the fly with the stronger combination of remaining HP and damage dealt is declared the winner.
7. The genetic algorithm — how the flies get better
This is the part of the project that makes repeated play worthwhile: nobody hand-tunes the flies’ tactics. Instead, after every single duel, the game runs a small evolutionary step on the losing lineage.
 
Figure 6 — the evolution loop that runs after every duel.
7.1 Crossover
The loser’s entire network — every connection weight and bias, in both layers — is rebuilt by walking through the winner’s matching weights one at a time and, for each individual number, keeping the winner’s value about two-thirds of the time and falling back to the loser’s own previous value the rest of the time. This is a classic genetic-algorithm technique called crossover: the new network is a patchwork inheriting mostly from the stronger parent, with some of the loser’s own prior wiring surviving through.
7.2 Mutation
After crossover, every one of those weights has an independent chance — set by the mutation-rate slider — of being jolted by a random nudge. This keeps the population from getting stuck: without mutation, a lineage could only ever recombine weights that already existed, and would plateau. With it, genuinely new tactics can appear and, if they work, get inherited going forward.
7.3 What this does and doesn’t guarantee
Because only the loser is replaced each time, a lineage on a losing streak evolves quickly while a winning lineage stays completely untouched — which is why win streaks in the Duel tab are worth watching: a long streak means one lineage has pulled meaningfully ahead, at least against this particular opponent. There’s no guarantee of steady, monotonic improvement — evolution here, as in nature, can wander, regress, or overfit to whatever the immediate opponent happens to be doing.
 
8. Code walkthrough
The whole game is one self-contained HTML file: a small amount of CSS for the layout and HUD, a block of embedded real data (the neuron sample and summary statistics extracted from the uploaded file), and one JavaScript file that does everything else. This section walks through that JavaScript, piece by piece, in the order it actually runs.
8.1 The embedded real data
Before any game logic runs, two JavaScript constants are defined directly in the page from data extracted out of the original file with a small Python script: NEURON_META, a compact object of summary statistics (totals, breakdowns by class and neurotransmitter, and the top real cell-type names), and NEURON_POINTS, an array of roughly 950 individually sampled real neurons with their class, neurotransmitter, and 3D position. Keeping this as static, pre-extracted data — rather than trying to load and parse the full 139,000-row file in the browser — is what keeps the page small and fast to open.
const NEURON_META = { total_neurons: 139248, super_class_counts: {...}, ... };
const NEURON_POINTS = [ {x:120496, y:80359, z:5027, sc:"descending", nt:"na", ct:"DNg02_b"}, ... ];
8.2 The Brain class — the neural network itself
Brain is a small class with four connection matrices: w1/b1 for the input-to-hidden layer, and w2/b2 for the hidden-to-output layer. Its forward() method is the entire "thought process" of a fly for one frame — multiply, sum, squash, repeat once more:
forward(inputs){
  for (let h = 0; h < N_HID; h++) {
    let s = this.b1[h];
    for (let i = 0; i < N_IN; i++) s += this.w1[h][i] * inputs[i];
    this.hiddenAct[h] = tanh(s);
  }
  for (let o = 0; o < N_OUT; o++) {
    let s = this.b2[o];
    for (let h = 0; h < N_HID; h++) s += this.w2[o][h] * this.hiddenAct[h];
    this.outAct[o] = tanh(s);
  }
  return this.outAct;
}
The same class also holds the two genetic-algorithm operations described in Section 7: a static crossover(winner, loser) that builds a brand-new Brain by picking each weight from the winner about 65% of the time, and a mutate(rate) method that jitters each weight with a probability set by the mutation-rate slider.
8.3 stepAgent() — turning brain output into movement
Every frame, stepAgent() is called once per fly. It first computes that fly’s seven sensory inputs relative to its opponent (via computeInputs()), passes them through that fly’s Brain, and then applies the resulting turn/thrust/strike numbers directly to its position and heading — including the logic that decides whether a lunge should begin this frame, based on the strike output, remaining energy, and lunge cooldown.
const inputs = computeInputs(self, foe);
const [turnOut, thrustOut, strikeOut] = self.brain.forward(inputs);
self.heading += turnOut * TURN_RATE * dt;
if (strikeOut > 0.45 && self.lungeCd <= 0 && self.energy > 14) {
  self.lungeT = LUNGE_DUR; self.lungeCd = LUNGE_CD; self.energy -= 14;
}
8.4 resolveCombat() — deciding damage
After both flies have moved for the frame, resolveCombat() measures the distance between them and, if they’re within striking range, checks each fly’s current lunge state to work out the three outcomes described in Section 6: one-sided hit, mutual trade, or no damage at all.
8.5 endDuel() — declaring a winner and evolving the loser
When a duel finishes, endDuel() picks the winner (by HP, with damage dealt as the tiebreaker at a timeout), then immediately calls Brain.crossover() on the loser’s lineage and mutates the result — this is the exact moment Section 7’s evolution loop executes. It also updates the win-streak tracker, pushes a new point onto the fitness-history array that feeds the Duel tab’s line chart, and — if auto-run is on — schedules the next duel to start automatically.
8.6 The render loop
A single requestAnimationFrame loop, animate(), runs continuously regardless of duel state. Each call: advances duel physics if a duel is running (Sections 5–7), moves the visible 3D meshes to match the underlying fly positions, flaps wings, brightens a fly’s glow while it’s lunging, slowly rotates the neuron nebula, redraws the two small canvas charts (hidden-layer activity and fitness history), updates every HUD number and bar in the DOM, and finally asks Three.js to render the frame. Keeping physics and rendering in the same loop, rather than on separate timers, is what keeps the speed slider able to cleanly scale the whole simulation at once.
8.7 Camera, layout, and data-driven UI
Camera dragging is handled with plain pointer events (pointerdown/pointermove/pointerup) adjusting two angles and a distance, with no external orbit-control library — this keeps the file dependency-free beyond Three.js itself. Most of the sidebar content — the dataset bar charts, the sensory/motor neuron lists, and the input/output bar rows in the Brain tab — is built once at start-up in refreshStaticPanels(), directly from the NEURON_META and label arrays, so the real dataset numbers you see in the UI are generated from the same real data structures the rest of the game uses, not hand-typed into the page.
9. Limitations and things worth knowing
●	The uploaded file is an annotation table, not a connectivity/synapse table — so there is no real "wiring" for the game to use, even in principle, without a separate connectivity export.
●	The mapping from a real neuron type to a specific game input or output is a naming choice for grounding and flavour, not a scientific claim about that neuron’s function.
●	The 3D nebula shows a random sample of roughly 950 neurons out of the full dataset, chosen to keep the page lightweight rather than to be statistically representative of every sub-category.
●	Combat and movement rules (speeds, damage numbers, cooldowns) are game-design choices tuned for readability and fun, not derived from any biological measurement.
10. Ideas for extending it further
●	Import a real NeuPrint/FlyWire connectivity export and use actual synapse counts to initialise the starting weights, instead of random values.
●	Add a third scenario (for example, a predator both flies must avoid) so cooperative or avoidance tactics can evolve alongside combat tactics.
●	Let a fly’s glow or wing colour subtly shift with its dominant hidden-unit activation, turning the Brain tab’s bar chart into an in-arena visual cue as well.
●	Add a side-by-side replay mode comparing an early-generation brain against the current one on an identical scripted opponent, to make improvement visible rather than just inferred from win streaks.
