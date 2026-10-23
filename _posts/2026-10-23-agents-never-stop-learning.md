---
layout: post
title: "Agents That Never Stop Learning: Beyond the Bitter Lesson"
date: 2026-10-23
description: "Sutton’s Bitter Lesson says search and learning beat human knowledge. Scaled with compute, both have delivered stunning
              results, yet generalization to new tasks and settings remains uneven. In the multi-agent era, can agents learn from
              experience to generalize, including from each other?"
categories: [experiential learning, continual learning, agents, imitation learning, tacit knowledge]
author: Arun Kejariwal
last_modified_at: 2026-10-23
---

<a href="https://x.com/dwarkesh_sp" target="_blank" rel="noopener">Dwarkesh</a>’s <a href="https://www.dwarkesh.com/p/richard-sutton" target="_blank" rel="noopener">podcast with Rich Sutton</a> is a fantastic listen. The conversation touches upon Sutton’s <a href="https://www.incompleteideas.net/IncIdeas/BitterLesson.html" target="_blank" rel="noopener">The Bitter Lesson</a> essay, world models, generalization and evolving from building LLMs (and their derivatives) to experiential/continual learning based systems. A few call outs from the podcast and the essay that caught my attention:

* Infants learn mainly by trial and error; imitation is at most a small layer on top – this was a surprise as there’s a lot of literature in child development psychology that says otherwise.
* Learning is not about training, it is an active process!
* Search and Learning scale arbitrarily with increased computation.
* Knowledge representation and the difference between context (which, from contemporary perspective, maps to the context window of LLMs) and tacit knowledge.<sup class="fnref" id="fnref-1"><a href="#fn-1">1</a></sup>
* <em>“First there’s dust, it ends with stars. Stars make planets. The planets can give rise to life. Now we’re giving rise to designed entities.”</em>
* <em>“… AIs will design other AIs, and everything will be done by design and construction rather than by replication.”</em>

The podcast had a lot to think over. A few things:

## Imitation Learning {#imitation-learning}

I was quite surprised to hear Sutton’s take that imitation plays at most a small role in how infants learn. I had read otherwise based on prior work in developmental psychology. A few works are tabulated below:

<div class="table-wrap" style="--tw: 1392px">
<table class="pdf-table t-blog t-dev">
  <colgroup><col style="width: 7.2%"><col style="width: 6.0%"><col style="width: 8.9%"><col style="width: 15.4%"><col style="width: 18.0%"><col style="width: 19.4%"><col style="width: 25.1%"></colgroup>
  <thead>
    <tr><th class="vsep-l">Develop&shy;mental stage</th><th class="vsep-l">Typical age</th><th class="vsep-l">Retention</th><th class="vsep-l">Primary mechanism</th><th class="vsep-l">Characteristic performance</th><th class="vsep-l">Relevant literature</th><th>Notes</th></tr>
  </thead>
  <tbody>
    <tr><td class="metric vsep-l">Neonatal</td><td class="vsep-l">2–3 weeks (as tested)</td><td class="vsep-l">Up to 24 hours by 6 weeks (<a href="https://www.sciencedirect.com/science/article/abs/pii/0163638394900248" target="_blank" rel="noopener">Meltzoff &amp; Moore, 1994</a>)</td><td class="vsep-l">Supramodal matching; later formalized as Active Intermodal Mapping (AIM) (<a href="https://onlinelibrary.wiley.com/doi/abs/10.1002/(SICI)1099-0917(199709/12)6:3/4%3C179::AID-EDP157%3E3.0.CO;2-R" target="_blank" rel="noopener">Meltzoff &amp; Moore, 1997</a>)</td><td class="vsep-l">Matches tongue protrusion, mouth opening, lip protrusion and finger movements.</td><td class="vsep-l"><a href="https://doi.org/10.1126/science.897687" target="_blank" rel="noopener">Imitation of Facial and Manual Gestures by Human Neonates</a></td><td>Contested: a longitudinal study of 106 infants (<a href="https://www.cell.com/current-biology/fulltext/S0960-9822(16)30257-3" target="_blank" rel="noopener">Oostenbroek et al., 2016</a>) found no specific matching at 1–9 weeks.</td></tr>
    <tr><td class="metric vsep-l">Early infancy</td><td class="vsep-l">6 months</td><td class="vsep-l">24 hours</td><td class="vsep-l">Non-verbal encoding</td><td class="vsep-l">Reproduces part of a 3-step sequence after 24 h; needs a doubled demonstration.</td><td class="vsep-l"><a href="https://doi.org/10.1016/S0163-6383(96)90015-6" target="_blank" rel="noopener">Developmental Changes in Deferred Imitation by 6- to 24-Month-Old Infants</a></td><td>12- to 24-month-olds succeed with the standard demonstration.</td></tr>
    <tr><td class="metric vsep-l">Late infancy</td><td class="vsep-l">14 months</td><td class="vsep-l">1 week</td><td class="vsep-l">Explicit declarative retrieval</td><td class="vsep-l">Re-enacts novel, low-probability acts (head-touch) after a 1-week delay; selectively imitates rational means.</td><td class="vsep-l"><a href="https://doi.org/10.1037/0012-1649.24.4.470" target="_blank" rel="noopener">Infant Imitation After a 1-Week Delay: Long-Term Memory for Novel Acts and Multiple Stimuli</a><br><a href="https://doi.org/10.1038/415755a" target="_blank" rel="noopener">Rational Imitation in Preverbal Infants</a></td><td>Head-touch: 67% repeated it a week later; no infant did who hadn’t seen it demonstrated. 69% copied it when the adult’s hands were free, vs 21% when her hands were busy holding a blanket.</td></tr>
    <tr><td class="metric vsep-l">Early toddlerhood</td><td class="vsep-l">18 to 24 months</td><td class="vsep-l">Immediate (in cited study)</td><td class="vsep-l">Intent reading &amp; teleological evaluation</td><td class="vsep-l">Extrapolates intended goals from failed attempts.</td><td class="vsep-l"><a href="https://doi.org/10.1037/0012-1649.31.5.838" target="_blank" rel="noopener">Understanding the Intentions of Others: Re-Enactment of Intended Acts by 18-Month-Old Children</a></td><td>60% pulled the toy apart after watching a person try and fail, vs 10% after watching a mechanical pincer make the same slipping motions: toddlers read intent into people, not machines.</td></tr>
    <tr><td class="metric vsep-l">Preschool</td><td class="vsep-l">3 to 4 years</td><td class="vsep-l">Immediate</td><td class="vsep-l">Faithful copying of the demonstrated method</td><td class="vsep-l">Copies causally irrelevant steps even when they are visibly useless (over-imitation).</td><td class="vsep-l"><a href="https://doi.org/10.1007/s10071-004-0239-6" target="_blank" rel="noopener">Causal Knowledge and Imitation/Emulation Switching in Chimpanzees and Children</a></td><td>Chimpanzees skip the useless steps once they can see the mechanism; children don’t.</td></tr>
  </tbody>
</table>
</div>

So, where’s the disconnect? The disconnect is largely definitional. In machine learning, “imitation learning” means copying an expert’s actions as supervised targets (behavioral cloning); in developmental psychology, “imitation” mostly means inferring and reproducing goals from observed outcomes. Sutton’s own examples are of the first kind: <em>“There’s no imitation for how they move their eyes around or even the sounds they make.”</em>

* At the level of raw muscle contractions, eye saccades, or vocal tract adjustments, an infant receives no direct target or demonstration. An infant cannot directly observe the internal motor commands or muscle activations of an adult model. Therefore, learning how to control one’s own body (“how do I move my arm to touch my forehead?”) requires self-exploration and trial-and-error (often called “motor babbling” in robotics; <a href="https://onlinelibrary.wiley.com/doi/abs/10.1002/(SICI)1099-0917(199709/12)6:3/4%3C179::AID-EDP157%3E3.0.CO;2-R" target="_blank" rel="noopener">Meltzoff and Moore (1997)</a> call it “body babbling” and argue it begins before birth).
* What the empirical research suggests is that infants do not rely on brute-force trial-and-error to learn what actions to perform or what goals to achieve. Instead, they use visual input to set high-level representational targets:
  * <strong>Active Intermodal Mapping (AIM):</strong> Neonates translate visual target gestures into proprioceptive motor outputs via an innate common representational space.
  * <strong>Goal &amp; Rationality Inference:</strong> Toddlers emulate the intended goal (e.g., pulling a dumbbell apart) rather than copying the failed attempts, and selectively evaluate whether an action was rational given physical constraints.

Robotics and ML reached a compatible conclusion. <a href="https://www.cell.com/cognitive-sciences/abstract/S1364-6613(99)01327-3" target="_blank" rel="noopener">Schaal (1999)</a> showed that a demonstration can speed up trial-and-error learning by shrinking the space to explore, but that imitation alone cannot improve on the teacher without an explicit reward signal. <a href="https://proceedings.mlr.press/v15/ross11a/ross11a.pdf" target="_blank" rel="noopener">Ross, Gordon and Bagnell (2011)</a> showed that pure behavioral cloning compounds its own errors unless the learner gathers experience in the states it actually visits: ML’s version of Sutton’s point. Meanwhile, children’s readiness to over-imitate (<a href="https://doi.org/10.1007/s10071-004-0239-6" target="_blank" rel="noopener">Horner &amp; Whiten, 2005</a>) is the strongest evidence that humans are unusually imitative learners.

## Search, Learning and Knowledge Representation {#search-learning-knowledge-representation}

There’s a rich body of work on each topic. The following tabulates some of the seminal works:

<div class="table-wrap" style="--tw: 1300px">
<table class="pdf-table t-blog t-works">
  <colgroup><col style="width: 8.5%"><col style="width: 26.2%"><col style="width: 15.8%"><col style="width: 49.6%"></colgroup>
  <thead>
    <tr><th class="vsep-l">Dimension</th><th class="vsep-l">Paper</th><th class="vsep-l">Authors · Year</th><th>Key thesis and relevance</th></tr>
  </thead>
  <tbody>
    <tr><td class="metric vsep-l" rowspan="3">Learning as search</td><td class="vsep-l"><a href="https://doi.org/10.1016/0004-3702(82)90040-6" target="_blank" rel="noopener">Generalization as Search</a></td><td class="vsep-l">Mitchell · 1982</td><td>Learning is search through a space of hypotheses; methods differ in how they generate and prune candidates that fit the examples.</td></tr>
    <tr><td class="vsep-l"><a href="https://doi.org/10.1016/0004-3702(83)90016-4" target="_blank" rel="noopener">A Theory and Methodology of Inductive Learning</a></td><td class="vsep-l">Michalski · 1983</td><td>Induction is heuristic search over symbolic descriptions, steered by background knowledge and quality criteria.</td></tr>
    <tr><td class="vsep-l"><a href="https://link.springer.com/content/pdf/10.1007/BF00116251.pdf" target="_blank" rel="noopener">Induction of Decision Trees</a></td><td class="vsep-l">Quinlan · 1986</td><td>ID3 (Iterative Dichotomiser 3) grows a decision tree by greedily picking the most informative attribute, showing how a search bias makes learning tractable.</td></tr>
    <tr><td class="metric vsep-l" rowspan="5">Search &amp; learning that scale with compute</td><td class="vsep-l"><a href="https://doi.org/10.1147/rd.33.0210" target="_blank" rel="noopener">Some Studies in Machine Learning Using the Game of Checkers</a></td><td class="vsep-l">Samuel · 1959</td><td>Look-ahead search plus an evaluation function learned through play: an early form of TD learning and the original Bitter Lesson recipe.</td></tr>
    <tr><td class="vsep-l"><a href="https://doi.org/10.1145/360018.360022" target="_blank" rel="noopener">Computer Science as Empirical Inquiry: Symbols and Search</a></td><td class="vsep-l">Newell &amp; Simon · 1976</td><td>Intelligence is heuristic search over symbol structures: problems are solved by generating and pruning candidate solutions.</td></tr>
    <tr><td class="vsep-l"><a href="https://link.springer.com/chapter/10.1007/11871842_29" target="_blank" rel="noopener">Bandit Based Monte-Carlo Planning</a></td><td class="vsep-l">Kocsis &amp; Szepesvári · 2006</td><td>UCT (Upper Confidence bounds applied to Trees) guides Monte Carlo Tree Search (MCTS) by sampling instead of hand-built evaluation: the search that broke open computer Go.</td></tr>
    <tr><td class="vsep-l"><a href="https://doi.org/10.1126/science.aar6404" target="_blank" rel="noopener">A General Reinforcement Learning Algorithm that Masters Chess, Shogi, and Go through Self-Play</a></td><td class="vsep-l">Silver et al. · 2018</td><td>AlphaZero masters all three games from self-play alone, with search guided by a learned network: the Bitter Lesson’s showcase.</td></tr>
    <tr><td class="vsep-l"><a href="https://arxiv.org/pdf/2001.08361" target="_blank" rel="noopener">Scaling Laws for Neural Language Models</a></td><td class="vsep-l">Kaplan et al. · 2020</td><td>Language-model loss falls as a power law in parameters, data and compute, so the gains from scale become predictable.</td></tr>
    <tr><td class="metric vsep-l" rowspan="5">Learning: foundations and mechanisms</td><td class="vsep-l"><a href="https://dl.acm.org/doi/pdf/10.1145/1968.1972" target="_blank" rel="noopener">A Theory of the Learnable</a></td><td class="vsep-l">Valiant · 1984</td><td>Introduces PAC (Probably Approximately Correct) learning: which concepts can be learned, to a given accuracy and confidence, in polynomial time.</td></tr>
    <tr><td class="vsep-l"><a href="https://dl.acm.org/doi/pdf/10.1145/76359.76371" target="_blank" rel="noopener">Learnability and the Vapnik–Chervonenkis Dimension</a></td><td class="vsep-l">Blumer, Ehrenfeucht, Haussler &amp; Warmuth · 1989</td><td>Finite VC (Vapnik–Chervonenkis) dimension characterizes distribution-free PAC learnability: a class’s capacity, not its size, sets how many examples are needed.</td></tr>
    <tr><td class="vsep-l"><a href="https://www.nature.com/articles/323533a0" target="_blank" rel="noopener">Learning Representations by Back-Propagating Errors</a></td><td class="vsep-l">Rumelhart, Hinton &amp; Williams · 1986</td><td>Backpropagation trains hidden layers, so useful internal representations emerge from errors instead of being hand-designed.</td></tr>
    <tr><td class="vsep-l"><a href="https://link.springer.com/content/pdf/10.1007/BF00115009.pdf" target="_blank" rel="noopener">Learning to Predict by the Methods of Temporal Differences</a></td><td class="vsep-l">Sutton · 1988</td><td>TD (Temporal-Difference) learning updates a prediction from the next prediction, so learning starts before the outcome arrives. Also see <a href="https://mitpress.mit.edu/9780262039246/reinforcement-learning/" target="_blank" rel="noopener">Sutton &amp; Barto (2018)</a>.</td></tr>
    <tr><td class="vsep-l"><a href="https://www.cs.rhul.ac.uk/~chrisw/thesis.html" target="_blank" rel="noopener">Learning from Delayed Rewards</a> (PhD thesis)</td><td class="vsep-l">Watkins · 1989</td><td>Introduces Q-learning: learning optimal action values directly from experience, with no model of the environment. Convergence proved in <a href="https://link.springer.com/content/pdf/10.1007/BF00992698.pdf" target="_blank" rel="noopener">Watkins &amp; Dayan (1992)</a>.</td></tr>
    <tr><td class="metric vsep-l last-group" rowspan="7">Knowledge representation</td><td class="vsep-l"><a href="https://www-formal.stanford.edu/jmc/mcc59/mcc59.html" target="_blank" rel="noopener">Programs with Common Sense</a></td><td class="vsep-l">McCarthy · 1959</td><td>The Advice Taker represents facts as logical sentences and reasons over them: the founding paper of knowledge representation.</td></tr>
    <tr><td class="vsep-l"><a href="https://hdl.handle.net/1721.1/6089" target="_blank" rel="noopener">A Framework for Representing Knowledge</a></td><td class="vsep-l">Minsky · 1974/1975</td><td>Frames package slots, defaults and expectations for familiar situations; understanding means selecting a frame and adapting it.</td></tr>
    <tr><td class="vsep-l"><a href="https://doi.org/10.1016/0004-3702(80)90014-4" target="_blank" rel="noopener">A Logic for Default Reasoning</a></td><td class="vsep-l">Reiter · 1980</td><td>Default rules let a system draw conclusions from incomplete knowledge and withdraw them when contrary evidence appears.</td></tr>
    <tr><td class="vsep-l"><a href="https://doi.org/10.1016/0167-2789(90)90087-6" target="_blank" rel="noopener">The Symbol Grounding Problem</a></td><td class="vsep-l">Harnad · 1990</td><td>Symbols defined only by other symbols mean nothing to the system; meaning must bottom out in sensorimotor experience. A standard lens on whether text-trained models “understand”.</td></tr>
    <tr><td class="vsep-l"><a href="https://www.sciencedirect.com/science/article/abs/pii/000437029190053M" target="_blank" rel="noopener">Intelligence without Representation</a></td><td class="vsep-l">Brooks · 1991</td><td>Robots built from layered behaviors coupled directly to the world need no central world model: the embodied view closest to Polanyi and Moravec.</td></tr>
    <tr><td class="vsep-l"><a href="https://doi.org/10.1006/knac.1993.1008" target="_blank" rel="noopener">A Translation Approach to Portable Ontology Specifications</a></td><td class="vsep-l">Gruber · 1993</td><td>Ontologies, as explicit shared vocabularies, let knowledge be reused across systems and representation languages.</td></tr>
    <tr><td class="vsep-l"><a href="https://ojs.aaai.org/index.php/aimagazine/article/view/1029" target="_blank" rel="noopener">What Is a Knowledge Representation?</a></td><td class="vsep-l">Davis, Shrobe &amp; Szolovits · 1993</td><td>A representation plays five roles: surrogate for reality, set of ontological commitments, theory of reasoning, medium for computation, and medium of human expression.</td></tr>
  </tbody>
</table>
</div>

## Tacit Knowledge {#tacit-knowledge}

Polanyi’s ideas directly intersect with a famous AI principle called Moravec’s Paradox. As Moravec put it in <a href="https://www.hup.harvard.edu/books/9780674576186" target="_blank" rel="noopener"><em>Mind Children</em> (1988)</a>: <em>“it is comparatively easy to make computers exhibit adult level performance on intelligence tests or playing checkers, and difficult or impossible to give them the skills of a one-year-old when it comes to perception and mobility.”</em>

<div class="table-wrap" style="--tw: 1300px">
<table class="pdf-table t-blog t-moravec">
  <colgroup><col style="width: 23.1%"><col style="width: 15.4%"><col style="width: 32.6%"><col style="width: 28.9%"></colgroup>
  <thead>
    <tr><th class="vsep-l">Task type</th><th class="vsep-l">Human effort</th><th class="vsep-l">AI effort</th><th>Polanyi connection</th></tr>
  </thead>
  <tbody>
    <tr><td class="metric vsep-l">Advanced logic (e.g., calculus, chess, data analytics)</td><td class="vsep-l"><strong>Hard</strong> (requires conscious, high-level school learning)</td><td class="vsep-l"><strong>Easy</strong> (the rules are explicit, and computers win through search and computation rather than codified human expertise)</td><td><strong>Explicit knowledge:</strong> easily written down and turned into algorithms.</td></tr>
    <tr><td class="metric vsep-l">Sensorimotor skills (e.g., walking, avoiding obstacles, reading micro-expressions)</td><td class="vsep-l"><strong>Easy</strong> (instinctive, effortless to a toddler)</td><td class="vsep-l"><strong>Hard</strong> (requires immense compute and complex robotics)</td><td><strong>Tacit knowledge:</strong> evolved over millions of years; we possess no internal conscious script of how we do it.</td></tr>
  </tbody>
</table>
</div>

Since around 2012 the paradox has partly eroded. Deep networks trained on large datasets now see and hear well, learning perceptual skills from data rather than from rules anyone could write down: the Bitter Lesson at work. Dexterous manipulation and robust real-world mobility remain hard, largely because physical-interaction data is scarce and slow to collect.

For transferring a person’s tacit expertise, the most direct starting points are behavioral cloning, inverse RL (Reinforcement Learning) / reward learning, and preference learning. Example works in each are tabulated below:

<div class="table-wrap" style="--tw: 1300px">
<table class="pdf-table t-blog t-works">
  <colgroup><col style="width: 11.5%"><col style="width: 21.7%"><col style="width: 9.1%"><col style="width: 57.7%"></colgroup>
  <thead>
    <tr><th class="vsep-l">Category</th><th class="vsep-l">Paper</th><th class="vsep-l">Authors · Year</th><th>Key contribution and connection to tacit knowledge</th></tr>
  </thead>
  <tbody>
    <tr><td class="metric vsep-l" rowspan="2">Behavioral cloning &amp; direct imitation: copying expert actions</td><td class="vsep-l"><a href="https://proceedings.neurips.cc/paper_files/paper/1988/file/812b4ba287f5ee0bc9d43bbf5bbe87fb-Paper.pdf" target="_blank" rel="noopener">ALVINN: An Autonomous Land Vehicle in a Neural Network</a></td><td class="vsep-l">Pomerleau · 1988</td><td>Learns to steer by mapping camera images to a human driver’s steering. The driver never states a rule; the network absorbs the skill. Simple and direct, but brittle once it drifts into situations the driver never showed.</td></tr>
    <tr><td class="vsep-l"><a href="https://proceedings.neurips.cc/paper_files/paper/2016/file/cc7e2b878868cbae992d1fb743995d8f-Paper.pdf" target="_blank" rel="noopener">Generative Adversarial Imitation Learning</a></td><td class="vsep-l">Ho &amp; Ermon · 2016</td><td>Trains a policy until a discriminator can’t tell it apart from the expert. It skips recovering a reward and copies behavior far more faithfully than plain cloning.</td></tr>
    <tr><td class="metric vsep-l" rowspan="3">Inverse RL / reward learning: inferring objectives from expert behavior</td><td class="vsep-l"><a href="https://ai.stanford.edu/~ang/papers/icml00-irl.pdf" target="_blank" rel="noopener">Algorithms for Inverse Reinforcement Learning</a></td><td class="vsep-l">Ng &amp; Russell · 2000</td><td>Infers the reward under which an expert’s behavior makes sense, so experts never have to articulate their objectives. It also exposes an ambiguity: many rewards explain the same behavior.</td></tr>
    <tr><td class="vsep-l"><a href="https://dl.acm.org/doi/10.1145/1015330.1015430" target="_blank" rel="noopener">Apprenticeship Learning via Inverse Reinforcement Learning</a></td><td class="vsep-l">Abbeel &amp; Ng · 2004</td><td>Matches the expert’s average feature counts instead of recovering their exact reward. Apprenticeship succeeds without knowing precisely what the expert values.</td></tr>
    <tr><td class="vsep-l"><a href="https://proceedings.mlr.press/v48/finn16.pdf" target="_blank" rel="noopener">Guided Cost Learning: Deep Inverse Optimal Control via Policy Optimization</a></td><td class="vsep-l">Finn, Levine &amp; Abbeel · 2016</td><td>Learns a neural cost from demonstrations while learning to act under unknown dynamics, refining it with the robot’s own experience. No hand-engineered features or dynamics model needed.</td></tr>
    <tr><td class="metric vsep-l last-group" rowspan="2">Preference learning: learning from human judgments</td><td class="vsep-l"><a href="https://proceedings.neurips.cc/paper/2017/file/d5e2c0adad503c91f91df240d0cd4e49-Paper.pdf" target="_blank" rel="noopener">Deep Reinforcement Learning from Human Preferences</a></td><td class="vsep-l">Christiano et al. · 2017</td><td>Learns a reward from people’s comparisons of short behavior clips, then optimizes it with RL. It transfers judgments people can make more easily than explain.</td></tr>
    <tr><td class="vsep-l"><a href="https://papers.neurips.cc/paper_files/paper/2020/file/1f89885d556929e98d3ef9b86448f951-Paper.pdf" target="_blank" rel="noopener">Learning to Summarize from Human Feedback</a></td><td class="vsep-l">Stiennon et al. · 2020</td><td>Brings preference-based reward learning to summarization, capturing judgments that text-matching metrics miss. Optimizing the learned reward too hard, though, lowers real human-rated quality.</td></tr>
  </tbody>
</table>
</div>

Note the tension with Sutton: every method here learns from human-provided targets (demonstrations or judgments), which is exactly what he argues cannot be the foundation of intelligence (<a href="https://storage.googleapis.com/deepmind-media/Era-of-Experience%20/The%20Era%20of%20Experience%20Paper.pdf" target="_blank" rel="noopener">Silver &amp; Sutton, 2025</a>).

## World Models and Experiential Learning {#world-models-experiential-learning}

Which brings us back to Sutton’s bet: intelligence comes from an agent’s own experience, not from human data. The lineage is older than LLMs:

* <a href="https://doi.org/10.1145/122344.122377" target="_blank" rel="noopener">Dyna (Sutton, 1991)</a>: learn a model of the world from experience, then plan with it.
* <a href="https://doi.org/10.1038/s41586-020-03051-4" target="_blank" rel="noopener">MuZero (Schrittwieser et al., 2020)</a>: a learned model plus search mastered Atari, Go, chess and shogi without being told the rules.
* <a href="https://doi.org/10.1038/s41586-024-07711-7" target="_blank" rel="noopener">Loss of plasticity (Dohare et al., 2024)</a>: standard deep networks gradually lose the ability to learn, which is the core obstacle to truly continual learning.
* <a href="https://storage.googleapis.com/deepmind-media/Era-of-Experience%20/The%20Era%20of%20Experience%20Paper.pdf" target="_blank" rel="noopener">The Era of Experience (Silver &amp; Sutton, 2025)</a>: the manifesto for agents that learn from streams of their own experience, grounded in real-world signals.

Sutton doubts LLMs have world models at all; Dwarkesh thinks they do. Either way, the next frontier isn’t a bigger context window. It’s agents that never stop learning.

## Notes {#notes}

<ul class="notes">
  <li id="fn-1"><span class="fn-num">1</span> In <a href="https://press.uchicago.edu/ucp/books/book/chicago/T/bo6035368.html" target="_blank" rel="noopener"><em>The Tacit Dimension</em> (1966)</a>, Polanyi famously summarized the tacit dimension as <em>“we can know more than we can tell”</em>. LLMs may well have a tacit dimension of their own (they, too, know more than they can tell), but they lack the kind humans have: rooted in evolution, grounded in real-world interaction, and continually acquired through experience. <a class="fn-back" href="#fnref-1" aria-label="Back to text">&#8617;&#xFE0E;</a></li>
</ul>

<style>
/* ---------- typography: mirror the PDF (Arial body 11pt -> 16px; headings 14pt / 12pt) ---------- */
.post-content {
  font-family: Arial, "Helvetica Neue", Helvetica, "Liberation Sans", sans-serif;
  font-size: 16px;
  color: #000;
}
.post-content a { color: #1155cc; }
.post-content a:hover { text-decoration: underline; }
.post-content h2 {
  font-family: Arial, "Helvetica Neue", Helvetica, "Liberation Sans", sans-serif;
  font-size: 20px;              /* 14pt */
  font-weight: 400;
  line-height: 1.3;
  letter-spacing: 0;
  color: #434343;
  margin: 30px 0 8px 0;
}

/* Fully justify body text, including bullet items (the site stylesheet left-aligns lists) */
.post-content p,
.post-content li { text-align: justify !important; text-justify: inter-word; }

/* Post title: shrink just enough to sit on one line within the text column (desktop; minima keeps its own size <= 800px) */
@media screen and (min-width: 801px) {
  .post-header .post-title { font-size: 28px; white-space: normal; }
}

/* Sub-bullets: unfilled circles */
.post-content ul ul { list-style-type: circle !important; }

/* Footnote references */
.post-content sup.fnref { font-size: 0.65em; line-height: 0; vertical-align: super; }
.post-content sup.fnref a { color: #000; text-decoration: none; padding: 0 1px; }
.post-content sup.fnref a:hover { color: #1155cc; text-decoration: underline; }

/* ---------- tables: purple bold headers, green first column, as in the reference post ---------- */
/* Tables break out of the ~740px text column (up to their own width, or the screen) so most cells wrap to <= 2 lines */
.post-content .table-wrap {
  width: min(var(--tw, 100%), calc(100vw - 48px));
  margin-left: 50%;
  transform: translateX(-50%);
  overflow-x: auto;
}
.post-content table.t-blog { table-layout: fixed; }
.post-content table.pdf-table {
  width: 100%;
  border-collapse: collapse;
  border: 0;
  margin: 8px auto 26px auto;
  font-family: Arial, "Helvetica Neue", Helvetica, "Liberation Sans", sans-serif;
  font-size: 14.5px;            /* 10pt */
  line-height: 1.35;
  color: #000;
  background: none;
  hyphens: manual;
}
.post-content table.pdf-table tr,
.post-content table.pdf-table tr:nth-child(even) { background: none; }
.post-content table.pdf-table th,
.post-content table.pdf-table td {
  border: 0;
  background: none;
  padding: 8px 8px;
  vertical-align: middle;
  text-align: left;
}
.post-content table.pdf-table thead th {
  font-size: 16px;              /* 11pt */
  font-weight: 700;
  color: #674ea7;
  line-height: 1.25;
  border-bottom: 1.5px solid #000;
}
.post-content table.pdf-table tbody td { border-bottom: 1px solid #c9d3de; }
.post-content table.pdf-table .metric { color: #38761d; }
.post-content table.t-blog tbody td { text-align: justify; text-justify: inter-word; }
.post-content table.pdf-table .vsep-l { border-right: 1px solid #c9c9c9; }
.post-content table.t-blog tbody tr:last-child td,
.post-content table.t-blog tbody td.last-group { border-bottom: 1px solid #000; }
.post-content table.t-dev { min-width: 900px; }

/* ---------- Notes (footnotes) ---------- */
.post-content ul.notes { font-size: 13px; line-height: 1.5; list-style: none; margin-left: 0; padding-left: 0; }
.post-content ul.notes li { margin-bottom: 6px; padding: 2px 4px; border-radius: 3px; scroll-margin-top: 20px; }
.post-content ul.notes li:target { background: #fff6cc; }
.post-content ul.notes .fn-num { font-size: 0.75em; vertical-align: super; line-height: 0; margin-right: 2px; }
.post-content ul.notes .fn-back { text-decoration: none; margin-left: 2px; }
</style>

<script>
(function () {
  /* Fine-tune the title size so it fits on one line of the text column, whatever font the reader's OS supplies */
  var title = document.querySelector('.post-header .post-title');
  if (!title) { return; }
  var fit = function () {
    title.style.fontSize = '';
    if (!window.matchMedia('(min-width: 801px)').matches) { return; }
    var size = 42;                      /* minima's desktop title size: shrink from here until it fits */
    title.style.fontSize = size + 'px';
    var oneLine = function () { return title.getBoundingClientRect().height < size * 1.5; };
    while (!oneLine() && size > 22) { size -= 0.5; title.style.fontSize = size + 'px'; }
  };
  fit();
  window.addEventListener('resize', fit);
})();
</script>
