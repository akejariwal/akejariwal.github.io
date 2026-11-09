---
layout: post
title: "No Retraining, No Recompiling: Unlocking Hidden Headroom"
date: 2026-11-08
description: "An LLM edited 0.016% of another model’s weights, with no training, and that model’s
              math score rose from 7.9% to 48.1%. Binary optimizers have tuned compiled programs this
              way for decades: BOLT made GCC’s C++ compiler 15% faster on an LLVM build, with no
              recompiling. When do such edits pay off, and where do they stop?"
categories: [weight-space editing, binary optimization, BOLT, compilers, LLMs]
author: Arun Kejariwal
last_modified_at: 2026-11-08
---

**TL;DR**
{: .tldr}

Weight-space editing makes a trained network better without retraining it, much as binary optimization has long made compiled programs faster without recompiling them.

* *\[Weight space\]* A large language model (LLM) edited 0.016% of another model’s weights, with no training, and that model’s score on MATH-500, a competition-math benchmark, <a href="https://rikiyat.github.io/blog/weight-space-programming/" target="_blank" rel="noopener">rose from 7.9% to 48.1%</a>.
* *\[Binary\]* Optimizers such as <a href="https://arxiv.org/pdf/1807.06735" target="_blank" rel="noopener">BOLT</a> (Binary Optimization and Layout Tool) rewrite a finished executable using a profile of how it runs, and gain mostly through code layout. In a recent experiment, BOLT made the C++ compiler in GCC 16.2 (the GNU Compiler Collection) build the LLVM compiler infrastructure 15% faster, with byte-identical output, while executing about 1% fewer instructions ([Appendix A](#appendix-a)).
{: .tldr-list}

<div class="fig-wide fig-hero" style="width: 100%; margin-left: 0; transform: none;">
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/network-and-binary-editing-4k%20v0.5.webp"><img src="{{ site.baseurl }}/assets/images/network-and-binary-editing-4k%20v0.5.webp" width="3840" height="1500" decoding="async" alt="Animation, Weight-space Editing ↔ Binary Optimization: on the left, an edit spreads through a language model’s weights; on the right, BOLT moves a binary’s hot code into one compact region"></a>
</div>

<p class="plot-hint">Click on the animation to zoom in.</p>

Both edit the finished artifact rather than its source, which neither recovers; interpretability researchers even describe a network’s parameters as <a href="https://www.transformer-circuits.pub/2022/mech-interp-essay" target="_blank" rel="noopener">“in some sense a binary computer program”</a>. A weight edit is worked out from the current weights and from how the network behaves on chosen inputs; BOLT works from the binary’s disassembly and a runtime profile, and reorders the code to cut misses in the instruction cache and the instruction translation lookaside buffer (iTLB).

They also share their limits. Both pay off only when one localized bottleneck holds back performance the artifact already has: a repetition loop<sup class="fnref" id="fnref-1"><a href="#fn-1">1</a></sup> in the edited model, or frequently run code scattered across a binary <a href="https://llvm.googlesource.com/llvm-project/+/278d9f76594de1ad0fac669468bf083ad6616a0f/bolt/docs/OptimizingClang.md" target="_blank" rel="noopener">too large for the instruction cache and iTLB</a>. Neither supplies what is missing: a search of the same kind <a href="https://github.com/RikiyaT/lm_understand-logs" target="_blank" rel="noopener">barely moved the score</a> of Gemma 3 270M, a model that rarely loops, and BOLT gains mainly by <a href="https://lists.llvm.org/pipermail/llvm-dev/2020-October/146297.html" target="_blank" rel="noopener">reordering existing code</a>. New capability still comes from retraining or recompiling.

Both are also fragile outside narrow settings. A weight edit can break a model: in <a href="https://arxiv.org/pdf/2402.09656" target="_blank" rel="noopener">one study</a>, a single edit was sometimes enough to wreck a model’s performance across benchmarks. Binary rewriting in general is no safer: when <a href="https://arxiv.org/pdf/2203.13231" target="_blank" rel="noopener">10 general-purpose rewriters</a> were asked to insert extra code into test programs, even the best produced a correctly running program only 70% of the time, and half of the tools never did. Rewriting, the study’s authors concluded, can be made to work for particular kinds of programs but not for all. BOLT stays practical by asking the build for extra information, such as <a href="https://llvm.googlesource.com/llvm-project.git/+/refs/heads/main/bolt/docs/GettingStarted.md" target="_blank" rel="noopener">symbol tables and relocation records</a>, so that it can move code safely.

<div class="table-wrap" style="--tw: 1200px">
<table class="pdf-table t-blog">
  <colgroup><col style="width: 15.8%"><col style="width: 40.0%"><col style="width: 44.2%"></colgroup>
  <thead>
    <tr><th class="vsep-l">Dimension</th><th class="vsep-l">Weight-space editing</th><th>Binary optimization (BOLT)</th></tr>
  </thead>
  <tbody>
    <tr><td class="metric vsep-l">What is edited</td><td class="vsep-l"><a href="https://rikiyat.github.io/blog/weight-space-programming/" target="_blank" rel="noopener">Weights of a trained network</a></td><td><a href="https://arxiv.org/pdf/1807.06735" target="_blank" rel="noopener">Machine code of a linked executable</a></td></tr>
    <tr><td class="metric vsep-l">The missing “source”</td><td class="vsep-l">Training data, objective and training code</td><td>Source code, with its types and high-level control flow</td></tr>
    <tr><td class="metric vsep-l">Guidance signal</td><td class="vsep-l">The network’s <a href="https://github.com/RikiyaT/lm_understand-logs" target="_blank" rel="noopener">outputs and activations on chosen inputs</a></td><td>A profile of real runs, from <a href="https://arxiv.org/pdf/1807.06735" target="_blank" rel="noopener">hardware samples</a> or <a href="https://llvm.googlesource.com/llvm-project.git/+/refs/heads/main/bolt/docs/GettingStarted.md" target="_blank" rel="noopener">exact counts</a></td></tr>
    <tr><td class="metric vsep-l">Edits that work</td><td class="vsep-l">Removing one internal direction, tied to <a href="https://github.com/RikiyaT/lm_understand-logs" target="_blank" rel="noopener">looping</a> or to <a href="https://arxiv.org/pdf/2406.11717" target="_blank" rel="noopener">refusal</a></td><td><a href="https://arxiv.org/pdf/1807.06735" target="_blank" rel="noopener">Reordering basic blocks and functions</a>; separating frequently run code from rarely run code</td></tr>
    <tr><td class="metric vsep-l">Reported gain</td><td class="vsep-l"><a href="https://rikiyat.github.io/blog/weight-space-programming/" target="_blank" rel="noopener">MATH-500 from 7.9% to 48.1%</a> once the looping direction was removed</td><td>LLVM built 15% faster (<a href="#appendix-a">Appendix A</a>); <a href="https://arxiv.org/pdf/1807.06735" target="_blank" rel="noopener">up to 20% faster</a> Clang and GCC in the original BOLT paper</td></tr>
    <tr><td class="metric vsep-l">Adds new capability?</td><td class="vsep-l">Not on its own: <a href="https://github.com/RikiyaT/lm_understand-logs" target="_blank" rel="noopener">barely any gain on Gemma 3 270M</a>, which rarely loops</td><td>No: the same instructions run in a better order (<a href="#appendix-a">Appendix A</a>)</td></tr>
    <tr><td class="metric vsep-l">Understanding the artifact</td><td class="vsep-l">Finding the directions or neurons behind a behaviour, which <a href="https://arxiv.org/pdf/2502.17420" target="_blank" rel="noopener">may span several directions</a></td><td>Disassembling functions and rebuilding their control flow, which <a href="https://arxiv.org/pdf/2007.14266" target="_blank" rel="noopener">relies on heuristics</a></td></tr>
    <tr><td class="metric vsep-l">Reliability</td><td class="vsep-l"><a href="https://arxiv.org/pdf/2402.09656" target="_blank" rel="noopener">One edit can wreck a model</a></td><td>BOLT <a href="https://arxiv.org/pdf/1807.06735" target="_blank" rel="noopener">skips code</a> it cannot analyze with confidence; <a href="https://arxiv.org/pdf/2203.13231" target="_blank" rel="noopener">general-purpose rewriters</a> often break the programs they modify</td></tr>
  </tbody>
</table>
</div>

The analogy breaks on correctness. A binary optimizer must preserve behaviour, and the original binary is the reference to test it against: the optimized compiler in [Appendix A](#appendix-a) produced byte-identical object files. A weight edit changes behaviour on purpose, so there is no reference to test against, and side effects surface only in evaluations beyond the target task ([Section 1.2](#weight-space-limitations)).

The two also differ in scope. A binary optimizer has one job, making a program faster, and never needs to know what the program is for: whether it plays Go or chess, BOLT only needs to know which parts run most. Weight edits serve many goals, from removing a loop to updating a stored fact or stripping out refusal ([Appendix B](#appendix-b)), and the broader idea of weight-space programming also reads what a small network was trained to do and writes new networks from scratch ([Section 1](#weight-space-programming)). On the binary side, those two jobs belong to reverse engineering and to compilers, not to optimization.

## 1. Weight-space Programming {#weight-space-programming}

Almost every network gets its weights from training, though not all: networks have been trained to <a href="https://arxiv.org/pdf/2102.11174" target="_blank" rel="noopener">write other networks’ weights</a>, and programs have been <a href="https://arxiv.org/pdf/2301.05062" target="_blank" rel="noopener">compiled into transformer weights</a>. Weight-space editing changes a trained network’s weights directly, without retraining it, and most methods derive the change from other fine-tuned models or from the network’s own activations ([Appendix B](#appendix-b)).

The newest variant hands the edit to a general-purpose LLM, which decides it from a goal stated in plain language. A <a href="https://rikiyat.github.io/blog/weight-space-programming/" target="_blank" rel="noopener">post by MIT researchers</a> calls the broader idea weight-space programming: the LLM inspects a network’s weights, as numbers in its prompt or through Python, and writes new values. Its experiments span three settings, and only the second is editing in the sense that parallels binary optimization:

* *\[Reading\]* Infer what a network was trained to do from its weights alone.
* *\[Fixing\]* Change a trained network to meet a specification.
* *\[Writing\]* Build a working network from all-zero weights, without training.

In eight experiments, on networks ranging from small game-playing policies to a 0.8-billion-parameter language model, OpenAI’s GPT-6 Astra read, fixed or wrote weights directly, without gradient descent. In that language model, Qwen3.5-0.8B, it subtracted one direction linked to repetitive looping, the edit behind the MATH-500 gain.

How much feedback the LLM gets <a href="https://github.com/RikiyaT/lm_understand-logs" target="_blank" rel="noopener">depends on the size of the network</a>. For networks small enough to fit in the prompt, it read the weights as text and returned its answer or new weights in a single reply, with no tools and without running the network. Larger networks came with a Python sandbox and a budget of forward passes, runs of the network on chosen inputs, which the authors varied to measure what feedback buys.

### 1.1 Can You Just Edit Weights to Make a Model Better at Math? {#editing-for-math}

The evidence says no: the edit removed an obstacle rather than adding mathematical skill.

* *\[It was a loop fix\]* Qwen’s own model card warns that the model is prone to thinking loops, as the <a href="https://rikiyat.github.io/blog/weight-space-programming/" target="_blank" rel="noopener">post</a> notes. Before the edit, 93% of its MATH-500 answers <a href="https://github.com/RikiyaT/lm_understand-logs" target="_blank" rel="noopener">ran out of output tokens unfinished</a>; after it, 33% did.
* *\[Instruction following rose too\]* On IFEval (Instruction-Following Evaluation), which checks whether answers obey <a href="https://arxiv.org/pdf/2311.07911" target="_blank" rel="noopener">verifiable instructions</a> such as “write in more than 400 words”, the score <a href="https://github.com/RikiyaT/lm_understand-logs" target="_blank" rel="noopener">rose from 21% to 62%</a>. Better math would not explain that; removing a general failure would.
* *\[No loop, no gain\]* Gemma 3 270M rarely loops, and the same search <a href="https://github.com/RikiyaT/lm_understand-logs" target="_blank" rel="noopener">barely moved its score</a>: from 6.8% to at most 7.7%.
* *\[Independent replication agrees\]* A reader who reproduced the released edit, in a <a href="https://www.alphaxiv.org/abs/2611.weight-space-programming" target="_blank" rel="noopener">comment on alphaXiv</a>, found accuracy rising from 11% to 62% as unfinished answers fell from 89% to 18%.

Weight edits can add skill only by importing it from a model trained for it. A <a href="https://arxiv.org/pdf/2509.01363" target="_blank" rel="noopener">reasoning vector</a>, the weight difference between two copies of a model fine-tuned on the same data, one with reinforcement learning and one without, raised a compatible 1.5-billion-parameter model’s grade-school math accuracy by 2.6 points. The skill came from training; the edit only moved it.

### 1.2 Limitations {#weight-space-limitations}

The post drew quick pushback, much of it reflected in the table below. Critics noted that reading raw weights without tools was shown only on small networks, while the Qwen fix relied on Python and practice runs \[<a href="https://honnibal.dev/blog/astra-read-write-misleading" target="_blank" rel="noopener">1</a>\]; that naming the games reduces to simple statistics over input channels, and the loop fix follows a familiar recipe for removing a direction \[<a href="https://www.seangoedecke.com/llms-are-not-superhuman-at-reading-and-writing-neural-network-weights/" target="_blank" rel="noopener">2</a>\]; that the task prompt itself said fixing repetition counted \[<a href="https://www.turingpost.com/p/what-is-weight-space-programming-and-why-it-suddenly-blew-up-the-internet" target="_blank" rel="noopener">3</a>\]; and that small networks may not predict how large ones behave \[<a href="https://news.ycombinator.com/item?id=50347685" target="_blank" rel="noopener">4</a>\]. Others asked whether the gains extend beyond math \[<a href="https://www.linkedin.com/posts/varunmayya_a-new-technique-called-weight-space-programming-activity-7523706323444776960-02s-" target="_blank" rel="noopener">5</a>\], warned that the work could accelerate risky AI progress \[<a href="https://venturebeat.com/technology/like-gene-editing-for-neural-networks-mit-researchers-used-gpt-6-astra-to-edit-qwens-open-weights-directly-boosting-math-performance" target="_blank" rel="noopener">6</a>\], and debated whether weights resemble a binary or source code \[<a href="https://news.ycombinator.com/item?id=50341174" target="_blank" rel="noopener">7</a>\]. Two follow-ups tested the claims: a local open model read three of four games from raw weights \[<a href="https://simonwillison.net/2026/Nov/3/qwen-can-interpret-model-weights/" target="_blank" rel="noopener">8</a>\], and a reader’s reproduction of the released Qwen edit, posted on alphaXiv, confirmed its gain \[<a href="https://www.alphaxiv.org/abs/2611.weight-space-programming" target="_blank" rel="noopener">9</a>\].

<div class="table-wrap" style="--tw: 1180px">
<table class="pdf-table t-blog">
  <colgroup><col style="width: 21.2%"><col style="width: 78.8%"></colgroup>
  <thead>
    <tr><th class="vsep-l">Limitation</th><th>Description</th></tr>
  </thead>
  <tbody>
    <tr><td class="metric vsep-l">Elicitation, not new capability</td><td>Gains appear only where a localized obstacle, such as a loop, hides an existing skill (<a href="#editing-for-math">Section 1.1</a>), and loop edits <a href="https://arxiv.org/pdf/2606.13705" target="_blank" rel="noopener">cannot supply missing knowledge</a>.</td></tr>
    <tr><td class="metric vsep-l">Weights-only reading at small scale</td><td>Reading raw weights without tools <a href="https://github.com/RikiyaT/lm_understand-logs" target="_blank" rel="noopener">worked only on small networks</a>; with two larger language models, the LLM could not tell which had been fine-tuned to be sycophantic until it inspected their attention patterns.</td></tr>
    <tr><td class="metric vsep-l">A guided search</td><td>The 48.1% came from a prompt that named repetition as a failure worth fixing; a prompt without that hint <a href="https://www.turingpost.com/p/what-is-weight-space-programming-and-why-it-suddenly-blew-up-the-internet" target="_blank" rel="noopener">scored the same 14.1% as the hinted one</a> on a smaller test budget, with no result reported at the budget that reached 48.1%.</td></tr>
    <tr><td class="metric vsep-l">Harder at scale</td><td>Loop fixes that took two neurons in the smallest Gemma 4 model <a href="https://arxiv.org/pdf/2606.13705" target="_blank" rel="noopener">took over a thousand in the 31-billion-parameter one</a>, and refusal, once traced to a single direction, <a href="https://arxiv.org/pdf/2502.17420" target="_blank" rel="noopener">spans several</a>.</td></tr>
    <tr><td class="metric vsep-l">Collateral damage</td><td>Removing the refusal direction <a href="https://arxiv.org/pdf/2406.11717" target="_blank" rel="noopener">lowers scores on a truthfulness benchmark</a>, and fact edits <a href="https://arxiv.org/pdf/2401.07453" target="_blank" rel="noopener">bleed into other stored facts</a>, including earlier edits.</td></tr>
    <tr><td class="metric vsep-l">Edits do not compose</td><td>Fact edits applied one after another erode a model gradually, then abruptly: with one popular method, a 6-billion-parameter model <a href="https://arxiv.org/pdf/2401.07453" target="_blank" rel="noopener">broke down within about 1,500 edits</a>.</td></tr>
    <tr><td class="metric vsep-l">Dual use</td><td>The same kind of edit strips safety training: removing one direction <a href="https://arxiv.org/pdf/2406.11717" target="_blank" rel="noopener">disables refusal in 13 chat models</a>.</td></tr>
  </tbody>
</table>
</div>

## 2. Binary Optimization {#binary-optimization}

Binary optimizers rework compiled code, most often its layout, guided by a profile of the program on representative inputs. <a href="https://arxiv.org/pdf/1807.06735" target="_blank" rel="noopener">BOLT</a> starts from the finished executable, which makes it the closest analogue to weight-space editing. It disassembles each function, rebuilds its control-flow graph and attaches the profile, then rewrites the layout: it orders basic blocks so that the hot paths, those run most often, execute straight through, places hot functions that call each other side by side, and moves rarely run blocks to a separate section. The hot code then fits in fewer cache lines and memory pages.

The profile is BOLT’s only feedback. It comes either from hardware samples of the running program or from <a href="https://llvm.googlesource.com/llvm-project.git/+/refs/heads/main/bolt/docs/GettingStarted.md" target="_blank" rel="noopener">exact branch counts</a> collected by a copy of the program with counting code added. Profile-guided compilation uses such a profile too, but must map each sample back onto source code, and after inlining and other compiler transformations that mapping is only approximate. BOLT applies the profile to the exact machine code that ran, so it <a href="https://arxiv.org/pdf/1807.06735" target="_blank" rel="noopener">finds gains even in programs already compiled with a profile</a>.

Profile-guided layout predates BOLT, and newer tools apply it at other stages ([Appendix C](#appendix-c)). Propeller, for example, has the compiler emit each basic block as a separately movable piece and lets the linker arrange the pieces by profile, instead of rewriting the finished binary.

### 2.1 Where Does BOLT’s Speedup Come From? {#bolt-speedup}

Not from doing less work. Like the Qwen edit, BOLT removed an obstacle rather than adding capability; the experiment in [Appendix A](#appendix-a) shows how:

* *\[Nearly the same work\]* The optimized compiler executed about 1% fewer instructions but completed 13% more of them per clock cycle.
* *\[Fewer stalls fetching code\]* Misses in the iTLB fell by half and in the instruction cache by 41%, because nearly all of the compiler’s time now falls in 8 MiB of packed hot code instead of being spread over 30 MiB.
* *\[Gains depend on the program\]* BOLT helps most on <a href="https://llvm.googlesource.com/llvm-project/+/278d9f76594de1ad0fac669468bf083ad6616a0f/bolt/docs/OptimizingClang.md" target="_blank" rel="noopener">large programs</a> whose hot code overflows the instruction cache and iTLB; reported gains range from <a href="https://github.com/python/cpython/commit/214eb2cce5caa99f476ae8abd406077e2c293a3c" target="_blank" rel="noopener">1–5% for CPython</a> to <a href="https://arxiv.org/pdf/1807.06735" target="_blank" rel="noopener">up to 20% for Clang and GCC</a>.

A faster algorithm still has to come from the source code: BOLT preserves what the program computes and changes mainly where its code sits.

### 2.2 Limitations {#binary-optimization-limitations}

<div class="table-wrap" style="--tw: 1180px">
<table class="pdf-table t-blog">
  <colgroup><col style="width: 21.2%"><col style="width: 78.8%"></colgroup>
  <thead>
    <tr><th class="vsep-l">Limitation</th><th>Description</th></tr>
  </thead>
  <tbody>
    <tr><td class="metric vsep-l">Layout, not new algorithms</td><td>Code reordering is BOLT’s primary optimization and <a href="https://lists.llvm.org/pipermail/llvm-dev/2020-October/146297.html" target="_blank" rel="noopener">the source of most of its gains</a>, so the instructions themselves change little.</td></tr>
    <tr><td class="metric vsep-l">Missing metadata</td><td>BOLT needs the binary’s symbol table, which release builds often strip, and for full gains the <a href="https://llvm.googlesource.com/llvm-project.git/+/refs/heads/main/bolt/docs/GettingStarted.md" target="_blank" rel="noopener">relocation records</a> that linkers discard by default; without those records it <a href="https://discourse.llvm.org/t/rfc-bolt-enabling-post-link-optimization-for-stripped-elf-binaries/91765" target="_blank" rel="noopener">cannot reorder functions</a>.</td></tr>
    <tr><td class="metric vsep-l">Profile staleness</td><td>A profile describes one version of the code: once the source changes, BOLT drops the profile data that no longer matches and loses much of its gain, and <a href="https://arxiv.org/pdf/2401.17168" target="_blank" rel="noopener">stale-profile matching</a> recovers most, but not all, of it.</td></tr>
    <tr><td class="metric vsep-l">Scale and build integration</td><td>Rewriting a whole binary at once strains memory on very large programs and <a href="https://doi.org/10.1145/3575693.3575727" target="_blank" rel="noopener">fits poorly into distributed builds</a>, which later work such as <a href="https://conf.researchr.org/details/CC-2021/cc-research-papers/2/Lightning-BOLT-Powerful-Fast-and-Scalable-Binary-Optimization" target="_blank" rel="noopener">Lightning BOLT</a> addresses.</td></tr>
    <tr><td class="metric vsep-l">Heuristic control-flow recovery</td><td>Disassembly <a href="https://arxiv.org/pdf/2007.14266" target="_blank" rel="noopener">relies on heuristics</a>, so BOLT skips any function whose control-flow graph it <a href="https://arxiv.org/pdf/1807.06735" target="_blank" rel="noopener">cannot rebuild with confidence</a>.</td></tr>
    <tr><td class="metric vsep-l">Toolchain fragility</td><td>Hand-written assembly <a href="https://lists.llvm.org/pipermail/llvm-dev/2020-October/146108.html" target="_blank" rel="noopener">has tripped up BOLT</a>, and new compiler output can too; CPython marks its BOLT build <a href="https://github.com/python/cpython/commit/214eb2cce5caa99f476ae8abd406077e2c293a3c" target="_blank" rel="noopener">experimental</a> because success depends on toolchain, flags and processor.</td></tr>
  </tbody>
</table>
</div>

## Notes {#notes}

<ul class="notes">
  <li id="fn-1"><span class="fn-num">1</span> A repetition loop is a failure in which the model keeps regenerating the same text or reasoning step until it reaches its output-length limit. <a class="fn-back" href="#fnref-1" aria-label="Back to text">&#8617;&#xFE0E;</a></li>
</ul>

## Appendix A: The BOLT Experiment (GCC 16.2 Building LLVM) {#appendix-a}

This appendix describes the experiment behind the BOLT numbers in the TL;DR and [Section 2.1](#bolt-speedup): GCC 16.2’s C++ compiler, optimized with BOLT, timed against the original while building LLVM.

### A.1 Setup {#appendix-a1}

<div class="fig-wide" id="figure-a1" style="width: 100%; margin-left: 0; transform: none;">
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/Bin%20Opt%20workflow-4k.gif"><img src="{{ site.baseurl }}/assets/images/Bin%20Opt%20workflow-4k.gif" width="3840" height="673" loading="lazy" decoding="async" alt="Figure A1, animated: build GCC 16.2 (51 min), instrument cc1plus, profile GCC compiling GCC in 1,139 runs (42 min), rewrite with BOLT (under 1 min), time five pairs of LLVM builds (5.5 hours), record heatmaps (2 min), save results, analyze"></a>
</div>

<p class="fig-caption with-hint">Figure A1. The experiment’s workflow and the time each stage took.</p>

<p class="plot-hint">Click on the animation to zoom in.</p>

The experiment compares two builds of the same compiler on one cloud virtual machine (VM):

* *\[A\]* GCC 16.2, built from its release source.
* *\[B\]* The same GCC 16.2 install, except that its C++ compiler, `cc1plus`, is replaced by a copy rewritten by BOLT.

The run took about 7 hours, unattended, in three stages:

* *\[Build A\]* A standard build of GCC 16.2.0 from source, linked with `-Wl,-q` so that relocation records survive and BOLT can move functions, and with GCC’s own hot/cold function splitting turned off, as <a href="https://llvm.googlesource.com/llvm-project.git/+/refs/heads/main/bolt/docs/GettingStarted.md" target="_blank" rel="noopener">BOLT’s documentation</a> requires for GCC 8 and later.
* *\[Profile, then build B\]* A copy of `cc1plus` with BOLT’s counting code added compiled GCC’s own sources, one compiler run per source file (the 1,139 runs in [Figure A1](#figure-a1)), each recording exact branch counts. Counting replaced the usual hardware sampling because the VM does not expose the processor’s branch history, the last branch record (LBR). `llvm-bolt` from the LLVM 23.1.4 release then used the merged counts to reorder basic blocks and functions and to split hot code from cold.
* *\[Time\]* Five pairs of clean builds of Clang and LLVM 23.1.4, alternating A and B, each measured with `perf stat` and each running eight parallel jobs, one per virtual CPU. The profile came from compiling GCC and the timing from compiling LLVM, so the profile was tested on code it never saw. For the heatmaps, each compiler also built two of LLVM’s libraries, `LLVMSupport` and `LLVMCore`, under `perf record`.

The VM was a Google Cloud `c4-standard-8`: an Intel Xeon Platinum 8581C with 4 cores × 2 hyperthreads, 30 GB of memory, and a 32 KiB level-1 instruction cache and 2 MiB level-2 cache per core.

### A.2 Results {#appendix-a2}

The BOLT-optimized compiler built LLVM 14.75% faster (95% confidence interval, 14.50–15%): a 34.8-minute build took 30.3 minutes. Every pair gained between 14.6% and 15.1%, and the two compilers produced byte-identical output: all 2,672 object files of the first pair matched. Hardware counters, recorded over each whole build, show where the gain came from ([Figure A2](#figure-a2)).

<div class="fig-wide" id="figure-a2" style="--fw: 860px">
  <img src="{{ site.baseurl }}/assets/images/Bin%20Opt%20counters-4k.png" width="3840" height="1881" loading="lazy" decoding="async" alt="Figure A2: change from A to B in each perf stat count, mean of five pairs: iTLB misses -50.48%, L1 i-cache misses -40.83%, branch misses -15.24%, wall time -12.85%, cycles -12.75%, branches executed -2.82%, instructions executed -1.11%">
</div>

<p class="fig-caption long">Figure A2. Front-end misses fell most while the work barely changed: mean change from A to B in each <code>perf stat</code> count over the five pairs, with 95% confidence intervals as whiskers. A 12.85% drop in wall time is a 14.75% speedup.</p>

### A.3 Where the Time Went {#appendix-a3}

A Linux executable keeps its machine code in a section named `.text`. BOLT does not compact that section in place: it keeps the original, renamed `.bolt.org.text`, for the functions it did not move; writes the optimized hot code into a new `.text` section at a higher address; and puts the cold blocks it split off into `.text.cold`. There is no `.hot.text`; the new `.text` is BOLT’s hot section. [Figure A3](#figure-a3) maps the cycle samples taken, while each compiler (original and BOLT-optimized) built `LLVMSupport` and `LLVMCore`, onto these sections, one 4 KB page at a time: before BOLT, the compiler’s time spread across the whole 29.6 MiB `.text`; after it, 99.6% falls in the new 8.1 MiB `.text`.

<div class="fig-wide" id="figure-a3" style="--fw: 600px">
  <img src="{{ site.baseurl }}/assets/images/Bin%20Opt%20heatmap-4k.png" width="3840" height="3651" loading="lazy" decoding="async" alt="Figure A3: share of cycle samples per 4 KB page of code, original compiler (A) beside the BOLT-optimized one (B); in B, 99.62% of samples fall in the new, compact .text">
</div>

<p class="fig-caption long">Figure A3. Share of cycle samples per 4 KB page of code, on a log scale from icy blue (few samples) to hot red (many); light gray pages have no samples. Each panel’s labels sum to just under 100%: the rest, 0.24% in A and 0.23% in B, falls in the <code>.plt</code>, the stubs through which the compiler calls shared libraries. The panels come from separate runs, so their totals need not match.</p>

Packing the hot code shrinks how much of it the processor must keep in its caches and iTLB. [Table A1](#table-a1) counts the 4 KB pages and 64-byte cache lines that hold a given share of the compiler’s time. Pages shrank by half or more, in line with the halving of iTLB misses; cache lines shrank by a quarter to a third, alongside the 41% drop in instruction-cache misses.

Two moves produce this packing. Splitting moved cold blocks out of the profiled functions, more than halving their hot code; ordering placed the hottest functions together, so the 100 hottest now span less than a tenth of the address range they used to.

<table class="pdf-table t-blog t-narrow" id="table-a1">
  <colgroup><col style="width: 46.9%"><col style="width: 18.8%"><col style="width: 17.2%"><col style="width: 17.2%"></colgroup>
  <thead>
    <tr><th class="vsep-l">Code holding the compiler’s time</th><th class="ctr vsep-l">A original</th><th class="ctr vsep-l">B BOLT</th><th class="ctr">Change</th></tr>
  </thead>
  <tbody>
    <tr><td class="metric vsep-l">4 KB pages for 50% / 99% of samples</td><td class="ctr vsep-l">159 / 2,133</td><td class="ctr vsep-l">55 / 977</td><td class="ctr">−65% / −54%</td></tr>
    <tr><td class="metric vsep-l">4 KB pages touched at all</td><td class="ctr vsep-l">3,300</td><td class="ctr vsep-l">1,658</td><td class="ctr">−50%</td></tr>
    <tr><td class="metric vsep-l">64 B cache lines for 50% / 99% of samples</td><td class="ctr vsep-l">1,510 / 32,076</td><td class="ctr vsep-l">998 / 23,648</td><td class="ctr">−34% / −26%</td></tr>
  </tbody>
</table>

<p class="fig-caption">Table A1. Code that holds the compiler’s time, before and after BOLT.</p>

### A.4 The Profile Carried Over to New Code {#appendix-a4}

The profile came from GCC compiling itself, yet it fit LLVM well: only 0.15% of the compiler’s time on LLVM fell in code that BOLT left in place or marked cold. BOLT’s own statistics predicted 1.2% fewer instructions on the profiled run, and the LLVM build measured 1.1%.

## Appendix B: Prior Work on Weight Editing {#appendix-b}

None of these methods retrains the model being edited. They differ in where the edit comes from: other fine-tuned models, the model’s own activations, or a program compiled into weights.

<div class="table-wrap" style="--tw: 1340px">
<table class="pdf-table t-blog">
  <colgroup><col style="width: 23.1%"><col style="width: 39.1%"><col style="width: 37.8%"></colgroup>
  <thead>
    <tr><th class="vsep-l">Work</th><th class="vsep-l">Idea</th><th>Key result</th></tr>
  </thead>
  <tbody>
    <tr><td class="metric lft vsep-l"><a href="https://arxiv.org/pdf/2212.04089" target="_blank" rel="noopener">Editing Models with Task Arithmetic</a></td><td class="vsep-l">A task vector, fine-tuned minus pre-trained weights, is added to teach a task, negated to unlearn it, or combined by analogy.</td><td>Negation lowers accuracy on the target task with little change on control tasks; adding vectors builds a multi-task model.</td></tr>
    <tr><td class="metric lft vsep-l"><a href="https://arxiv.org/pdf/2509.01363" target="_blank" rel="noopener">Reasoning Vectors: Transferring Chain-of-Thought Capabilities via Task Arithmetic</a></td><td class="vsep-l">Moves reasoning learned by reinforcement learning to another model as a weight difference (<a href="#editing-for-math">Section 1.1</a>).</td><td>For a 1.5B model, +2.6 points on GSM8K (Grade School Math 8K) and +10.2 on BigBenchHard; +4.9 and +12.3 with a step-by-step prompt.</td></tr>
    <tr><td class="metric lft vsep-l"><a href="https://arxiv.org/pdf/2403.13187" target="_blank" rel="noopener">Evolutionary Optimization of Model Merging Recipes</a></td><td class="vsep-l">Evolutionary search over ways to merge existing models’ weights and layers, with no gradient training.</td><td>A merged model scored 52.0% on the Japanese part of Multilingual Grade School Math (MGSM-JA), where no source model scored above 30%.</td></tr>
    <tr><td class="metric lft vsep-l"><a href="https://arxiv.org/pdf/2202.05262" target="_blank" rel="noopener">Locating and Editing Factual Associations in GPT</a></td><td class="vsep-l">Factual recall localizes to mid-layer multilayer-perceptron (MLP) modules, where Rank-One Model Editing (ROME) rewrites a single fact.</td><td>Edits generalize to paraphrases while leaving unrelated subjects unchanged.</td></tr>
    <tr><td class="metric lft vsep-l"><a href="https://arxiv.org/pdf/2210.07229" target="_blank" rel="noopener">Mass-Editing Memory in a Transformer (MEMIT)</a></td><td class="vsep-l">Spreads updates over several layers to change many facts at once.</td><td>Thousands of edits in GPT-J (6B) and GPT-NeoX (20B), orders of magnitude beyond earlier methods.</td></tr>
    <tr><td class="metric lft vsep-l"><a href="https://arxiv.org/pdf/2406.11717" target="_blank" rel="noopener">Refusal in Language Models Is Mediated by a Single Direction</a></td><td class="vsep-l">A refusal direction, found by contrasting activations on harmful and harmless prompts, is projected out of the weights, a step practitioners call abliteration.</td><td>Refusal is disabled across 13 chat models of up to 72B parameters, with most benchmarks changing by under 1%.</td></tr>
    <tr><td class="metric lft vsep-l"><a href="https://arxiv.org/pdf/2606.13705" target="_blank" rel="noopener">When Can One Neuron Fix Repetition Loops in LLMs?</a></td><td class="vsep-l">Locates Gemma 4 enumeration loops with ablation and attribution, then edits the responsible neurons.</td><td>Static edits remove the loops; the residual failure is a knowledge gap that editing cannot fill.</td></tr>
    <tr><td class="metric lft vsep-l"><a href="https://arxiv.org/pdf/2301.05062" target="_blank" rel="noopener">Tracr: Compiled Transformers as a Laboratory for Interpretability</a></td><td class="vsep-l">Compiles programs written in RASP (Restricted Access Sequence Processing) into transformer weights.</td><td>Models whose algorithm is known exactly, but too sparse and inefficient to scale to full language models.</td></tr>
  </tbody>
</table>
</div>

## Appendix C: Prior Work on Binary Optimization {#appendix-c}

Profile-guided code layout predates BOLT; later systems lay code out at link time instead of rewriting the binary, or re-optimize it while the program runs.

<div class="table-wrap" style="--tw: 1310px">
<table class="pdf-table t-blog">
  <colgroup><col style="width: 24.0%"><col style="width: 45.6%"><col style="width: 30.4%"></colgroup>
  <thead>
    <tr><th class="vsep-l">Work</th><th class="vsep-l">Idea</th><th>Key result</th></tr>
  </thead>
  <tbody>
    <tr><td class="metric lft vsep-l"><a href="https://doi.org/10.1145/93542.93550" target="_blank" rel="noopener">Profile Guided Code Positioning</a></td><td class="vsep-l">Orders procedures in the linker and basic blocks in the compiler by execution profile, to cut instruction-cache misses.</td><td>Established profile-guided code layout; the profile-guided function ordering in LLVM’s linker (LLD) is <a href="https://lists.llvm.org/pipermail/llvm-dev/2018-August/125154.html" target="_blank" rel="noopener">a variant of it</a>.</td></tr>
    <tr><td class="metric lft vsep-l"><a href="https://www.usenix.org/legacy/publications/library/proceedings/usenix-nt97/full_papers/cohn/cohn.pdf" target="_blank" rel="noopener">Spike: An Optimizer for Alpha/NT Executables</a></td><td class="vsep-l">Instruments executables and their dynamic-link libraries (DLLs), then applies profile-directed code layout and hot/cold optimization.</td><td>5–20% lower execution time across a wide set of applications.</td></tr>
    <tr><td class="metric lft vsep-l"><a href="https://doi.org/10.1145/349299.349303" target="_blank" rel="noopener">Dynamo: A Transparent Dynamic Optimization System</a></td><td class="vsep-l">Interprets a running binary, selects hot instruction traces, optimizes and caches them, with no recompilation.</td><td>SpecInt95 benchmark programs compiled with -O ran roughly as fast as their -O4 builds.</td></tr>
    <tr><td class="metric lft vsep-l"><a href="https://doi.org/10.1109/MICRO56248.2022.00045" target="_blank" rel="noopener">OCOLOS: Online COde Layout OptimizationS</a></td><td class="vsep-l">Applies BOLT-style layout inside a running process and swaps in the new code online, so the profile cannot go stale.</td><td>Up to 1.41× faster on MySQL, 2.20× on Verilator and 1.14× on a Clang build.</td></tr>
    <tr><td class="metric lft vsep-l"><a href="https://doi.org/10.1145/3575693.3575727" target="_blank" rel="noopener">Propeller: A Profile Guided, Relinking Optimizer for Warehouse-Scale Applications</a></td><td class="vsep-l">Has the compiler emit basic blocks as separately movable pieces and relinks them in profile order, instead of disassembling and rewriting the binary; this fits distributed builds.</td><td>Matches BOLT’s speedups with 30–70% less memory and runs on tens of millions of production cores.</td></tr>
  </tbody>
</table>
</div>

<div class="plot-lightbox" id="plot-lightbox" role="dialog" aria-modal="true" aria-label="Enlarged animation. Press Escape or click to close." hidden>
  <button type="button" class="plot-lightbox-close" aria-label="Close">&times;</button>
  <img id="plot-lightbox-img" src="data:," alt="">
</div>

<style>
.post-content { font-family: Arial, "Helvetica Neue", Helvetica, "Liberation Sans", sans-serif; font-size: 16px; color: #000; }
.post-content a { color: #1155cc; }
.post-content a:hover { text-decoration: underline; }
.post-content h2 { font-family: Arial, "Helvetica Neue", Helvetica, "Liberation Sans", sans-serif; font-size: 20px; font-weight: 400; line-height: 1.3; letter-spacing: 0; color: #434343; margin: 30px 0 8px 0; }
.post-content h3 { font-family: Arial, "Helvetica Neue", Helvetica, "Liberation Sans", sans-serif; font-size: 17.5px; font-weight: 400; line-height: 1.3; color: #666666; margin: 24px 0 8px 0; }
.post-content p, .post-content li { text-align: justify !important; text-justify: inter-word; }

/* Post title: no-JS fallback size that keeps the title on one line; the script below fine-tunes it */
@media screen and (min-width: 801px) {
  .post-header .post-title { font-size: 27px; white-space: normal; }
}

/* TL;DR: an empty line below the label, and a gap before the post begins */
.post-content p.tldr { margin-bottom: 1.6em; }
.post-content ul.tldr-list { margin-bottom: 1.8em; }

.post-content ul ul { list-style-type: circle !important; }

/* Monospace: Courier New, no badge background */
.post-content code { font-family: "Courier New", Courier, "Liberation Mono", monospace; font-size: 1em; color: inherit; background: none; border: 0; border-radius: 0; padding: 0; white-space: nowrap; hyphens: manual; }
.post-content table.pdf-table code { white-space: normal; }

/* Footnote references */
.post-content sup.fnref { font-size: 0.65em; line-height: 0; vertical-align: super; }
.post-content sup.fnref a { color: #000; text-decoration: none; padding: 0 1px; }
.post-content sup.fnref a:hover { color: #1155cc; text-decoration: underline; }

/* Tables: purple bold headers, green first column, light dividers */
.post-content .table-wrap { width: min(var(--tw, 100%), calc(100vw - 48px)); margin-left: 50%; transform: translateX(-50%); overflow-x: auto; }
.post-content table.pdf-table { width: 100%; border-collapse: collapse; border: 0; margin: 8px auto 26px auto; font-family: Arial, "Helvetica Neue", Helvetica, "Liberation Sans", sans-serif; font-size: 14.5px; line-height: 1.35; color: #000; background: none; hyphens: manual; }
.post-content table.t-blog { table-layout: fixed; }
.post-content table.pdf-table tr, .post-content table.pdf-table tr:nth-child(even) { background: none; }
.post-content table.pdf-table th, .post-content table.pdf-table td { border: 0; background: none; padding: 8px 8px; vertical-align: middle; text-align: left; }
.post-content table.pdf-table thead th { font-size: 16px; font-weight: 700; color: #674ea7; line-height: 1.25; border-bottom: 1.5px solid #000; }
.post-content table.pdf-table tbody td { border-bottom: 1px solid #c9d3de; }
.post-content table.pdf-table .metric { color: #38761d; }
.post-content table.t-blog tbody td { text-align: justify; text-justify: inter-word; }
.post-content table.pdf-table th.ctr, .post-content table.t-blog tbody td.ctr { text-align: center; }
.post-content table.t-blog tbody td.lft { text-align: left; }
.post-content table.pdf-table .vsep-l { border-right: 1px solid #c9c9c9; }
.post-content table.t-blog tbody tr:last-child td, .post-content table.t-blog tbody td.last-group { border-bottom: 1px solid #000; }
.post-content table.t-narrow { width: 86.5%; margin-bottom: 6px; }

/* Figures: wide like the tables, centred on the text column; captions below */
.post-content .fig-wide { width: min(var(--fw, 100%), calc(100vw - 48px)); margin: 10px 0 8px 50%; transform: translateX(-50%); }
.post-content .fig-wide img { display: block; width: 100%; height: auto; }
.post-content p.fig-caption { text-align: center !important; font-size: 14px; line-height: 1.4; color: #555555; margin: 6px 0 22px 0; }
.post-content p.fig-caption.long { text-align: justify !important; }
.post-content p.fig-caption.with-hint { margin-bottom: 2px; }

/* Click to zoom: the animations open in a centred viewer, 65% of the screen width */
.post-content .fig-wide a.plot-zoom { display: block; line-height: 0; cursor: zoom-in; }
.post-content p.plot-hint { text-align: center !important; font-size: 0.875rem; font-style: italic; color: #6b7280; margin: 6px 0 24px 0; }
.plot-lightbox { position: fixed; inset: 0; z-index: 9999; display: flex; align-items: center; justify-content: center; background: rgba(17, 17, 17, 0.78); cursor: zoom-out; }
.plot-lightbox[hidden] { display: none; }
.plot-lightbox img { width: 65vw; height: auto; max-height: 94vh; object-fit: contain; background: #ffffff; border-radius: 4px; box-shadow: 0 12px 40px rgba(0, 0, 0, 0.45); }
.plot-lightbox-close { position: absolute; top: 10px; right: 18px; border: 0; background: none; color: #ffffff; font: 36px/1 Arial, sans-serif; cursor: pointer; }

/* Notes (footnotes) */
.post-content ul.notes { font-size: 13px; line-height: 1.5; list-style: none; margin-left: 0; padding-left: 0; }
.post-content ul.notes li { margin-bottom: 6px; padding: 2px 4px; border-radius: 3px; scroll-margin-top: 20px; }
.post-content ul.notes li:target { background: #fff6cc; }
.post-content ul.notes .fn-num { font-size: 0.75em; vertical-align: super; line-height: 0; margin-right: 2px; }
.post-content ul.notes .fn-back { text-decoration: none; margin-left: 2px; }

/* Narrow screens: the narrow table takes the full width; the viewer widens */
@media screen and (max-width: 800px) {
  .post-content table.t-narrow { width: 100%; }
  .plot-lightbox img { width: 94vw; }
}
</style>

<script>
(function () {
  /* Fit the title on one line of the text column, whatever font the reader's OS supplies */
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

<script>
(function () {
  /* Open an animation in a centred viewer; Escape or a click closes it */
  var box = document.getElementById('plot-lightbox');
  var big = document.getElementById('plot-lightbox-img');
  if (!box || !big) { return; }
  document.body.appendChild(box);
  var opener = null;
  var close = function () {
    if (box.hidden) { return; }
    box.hidden = true;
    big.removeAttribute('src');
    document.documentElement.style.overflow = '';
    if (opener) { opener.focus(); opener = null; }
  };
  document.addEventListener('click', function (e) {
    var link = e.target.closest ? e.target.closest('a.plot-zoom') : null;
    if (!link) { return; }
    e.preventDefault();
    var thumb = link.querySelector('img');
    big.src = link.href;
    big.alt = thumb ? thumb.alt : '';
    opener = link;
    box.hidden = false;
    document.documentElement.style.overflow = 'hidden';
    box.querySelector('.plot-lightbox-close').focus();
  });
  box.addEventListener('click', close);
  document.addEventListener('keydown', function (e) { if (e.key === 'Escape') { close(); } });
})();
</script>
