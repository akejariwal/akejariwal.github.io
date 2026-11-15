---
layout: post
title: "AI Memory: The Cost of Remembering"
date: 2026-11-15
description: "A 10M-token context can demand over a terabyte of KV cache,
              even for a standard 8B model. Notes, persistent caches and
              learned weights offer other ways to retain experience, shifting
              costs across serving, offline compute and storage. Which
              approaches pay off as agents work for days, and how much does
              the answer depend on reuse?"
categories: [agent memory, KV cache, long context, continual learning, AI infrastructure]
author: Arun Kejariwal
last_modified_at: 2026-11-15
---

Twenty-five years after my last visit, I can still recollect the seating inside the <a href="https://en.wikipedia.org/wiki/Lotus_Temple" target="_blank" rel="noopener">Lotus Temple</a>, without a photograph or other persisted artifact to search. Could an AI agent keep its experience that long? Not by default: <a href="https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents" target="_blank" rel="noopener">each new session starts with no memory of the last</a>. To remember, an agent must preserve a trace, in an external record or in its learned parameters.

For a research agent on a week-long project, the challenge arrives within days: it must resume after a pause, incorporate new evidence and remember why an earlier approach failed, often while delegating to other agents. Documents can be retrieved on demand; what matters is whether the experience survives. Three approaches compete to retain it: a longer context window, organized memory such as notes or a wiki, and learned memory in the model’s weights, each shifting cost to a different part of the infrastructure.

## What a Longer Window Buys {#what-a-longer-window-buys}

Context length describes how many tokens a model can examine at once. The window is working memory: experience reaches a later session only if saved, as transcripts, notes or curated files, and reloaded. Windows grew from <a href="https://arxiv.org/pdf/2005.14165" target="_blank" rel="noopener">GPT-3’s 2,048 tokens</a> in 2020 to <a href="https://www.anthropic.com/news/claude-2-1" target="_blank" rel="noopener">Claude’s 200K</a> in 2023 and <a href="https://arxiv.org/pdf/2403.05530" target="_blank" rel="noopener">Gemini’s 1M</a> in 2024; Anthropic made <a href="https://claude.com/resources/articles/1m-context-ga" target="_blank" rel="noopener">1M generally available</a> in March 2026. <a href="https://ai.meta.com/blog/llama-4-multimodal-intelligence/" target="_blank" rel="noopener">Meta</a> and <a href="https://mirendil.com/news/introducing-fern/" target="_blank" rel="noopener">Mirendil</a> report 10M support, and <a href="https://magic.dev/blog/100m-token-context-windows" target="_blank" rel="noopener">Magic</a> reported 100M. But capacity is not use. At 10M tokens, Google reported over 99% recall of a single planted fact; Mirendil’s base model scored 100% on a similar test but 45% on multi-hop questions, which chain several facts. Neither shows whether an agent revises a conclusion that new evidence contradicts.

A longer window’s main appeal is that it preserves evidence whose importance becomes clear only later: a clause in a contract review, an anomalous experiment, or a constraint in a code migration. It can delay summarization and let the model compare more source material directly.

<div class="table-wrap">
<table class="pdf-table t-blog">
  <colgroup><col style="width: 44.3%"><col style="width: 55.7%"></colgroup>
  <thead>
    <tr><th class="vsep-l">Benefit of a longer window</th><th>Remaining limitation</th></tr>
  </thead>
  <tbody>
    <tr><td class="metric vsep-l">Revisit details that an early summary would omit.</td><td>More active tokens increase memory traffic and processing cost.</td></tr>
    <tr><td class="metric vsep-l">Compare evidence spread across a large collection.</td><td>Placement matters: <a href="https://arxiv.org/pdf/2307.03172" target="_blank" rel="noopener">Lost in the Middle</a> found weaker use of facts in the middle of long inputs.</td></tr>
    <tr><td class="metric vsep-l">Retain earlier findings alongside later corrections.</td><td>Extra space does not establish which account is current or correct.</td></tr>
  </tbody>
</table>
</div>

## From Retained Evidence to Reusable Knowledge {#from-retained-evidence-to-reusable-knowledge}

Organized memory takes a different route: a <a href="https://www.anthropic.com/research/components-of-agent-memory" target="_blank" rel="noopener">recent study</a> uses dreaming, an offline review of past sessions, to correct notes and build wiki pages without changing model weights. A <a href="https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f" target="_blank" rel="noopener">wiki pattern</a> describes this representation: interlinked pages compiled from retained raw sources. On 25 questions over a 420M-token corpus, wiki plus search met 61% of graded criteria versus 53% for search alone, after 128 dreaming sessions. Those sessions are offline compute, spent before any question arrives; the infrastructure question is how often their output is reused.

<div class="fig-right" id="figure-1">
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/Fig1_memory_at_three_speeds_4K_v2.png"><img src="{{ site.baseurl }}/assets/images/Fig1_memory_at_three_speeds_4K_v2.png" width="3840" height="3102" loading="lazy" decoding="async" alt="Figure 1: a three-level memory stack. New input enters the context window, is absorbed into fast memory, and during sleep is distilled into slow, durable memory; a dream loop rehearses from slow memory back into fast memory. An arrow beside the stack marks memory getting slower and more durable from top to bottom."></a>
  <p class="fig-caption">Figure 1. Memory at three speeds, after Hope and the sleep framework. Sleep distills fast memory into slow, durable layers.</p>
  <p class="plot-hint">Click on the figure to zoom in.</p>
</div>

Learned memory goes a step further, from notes to weights: through distillation, a model is trained to reproduce useful behavior without the original material in context. A <a href="https://arxiv.org/pdf/2606.03979" target="_blank" rel="noopener">sleep framework</a> applies this to <a href="https://arxiv.org/pdf/2512.24695" target="_blank" rel="noopener">Hope</a>, a model whose memory is a hierarchy: fast components adapt quickly and slow ones change rarely, preserving knowledge ([Figure 1](#figure-1)). During sleep, the model distills fast, recent memories into new low-rank parameters in slower layers, then dreams, using reinforcement learning to generate practice data that rehearses new knowledge and refines existing skills. On <a href="https://arxiv.org/pdf/2406.10149" target="_blank" rel="noopener">BABILong’s synthetic tests of chaining scattered facts</a>, a Hope model with sleep, fine-tuned for the benchmark, stayed near-perfect up to 10M tokens: retention on one task, not recall of arbitrary experience.

Learning and reasoning can reinforce each other. <a href="https://arxiv.org/pdf/2506.10943" target="_blank" rel="noopener">SEAL</a> rewards self-edits, the training data and update settings a model writes for itself, when the resulting update improves later answers, including on selected <a href="https://github.com/fchollet/ARC-AGI" target="_blank" rel="noopener">ARC puzzles</a>. <a href="https://arxiv.org/pdf/2601.19897" target="_blank" rel="noopener">Self-Distillation Fine-Tuning</a> trains a model on its own answers, steering them toward what the same model says after seeing expert demonstrations. Because the targets are its own outputs, updates stay close to current behavior, which <a href="https://arxiv.org/pdf/2509.04259" target="_blank" rel="noopener">RL’s Razor</a> links to less forgetting. Taught science questions, tool use and medical questions in turn, the model kept each earlier skill; with ordinary fine-tuning, each skill degraded once training moved on. The risk is self-bias: the authors note that the model can pick up spurious patterns from its own guided answers, and repeated training on generated text can cause <a href="https://www.nature.com/articles/s41586-024-07566-y" target="_blank" rel="noopener">model collapse</a>, where rare content vanishes. A <a href="https://arxiv.org/pdf/2610.05872" target="_blank" rel="noopener">newer study</a> argues that on-policy training is not required: it fine-tunes a checkpoint from before post-training and grafts the scaled weight change onto the deployed model, learning new tasks while matching or improving old ones.<sup class="fnref" id="fnref-1"><a href="#fn-1">1</a></sup>

Notes, wikis and learned weights cut rereading, but they keep only what seemed worth keeping; an unanticipated question may hinge on a detail they dropped. In the <a href="https://www.anthropic.com/research/components-of-agent-memory" target="_blank" rel="noopener">dreaming study</a> described above, the wiki alone met 48% of criteria; adding search over the raw corpus lifted that to 61%. Long context helps such questions mainly through coverage, not deduction: in a <a href="https://arxiv.org/pdf/2407.16833" target="_blank" rel="noopener">Google DeepMind comparison</a>, long-context models beat retrieval on average when given enough budget, as retrieval missed the evidence for multi-step, general and implicit queries. Yet, <a href="https://arxiv.org/pdf/2406.10149" target="_blank" rel="noopener">BABILong</a> found that models effectively used only 10-20% of their context, and accuracy fell as reasoning steps grew. The choice shifts cost between serving each request and offline compute.

## Infrastructure Implications {#infrastructure-implications}

An agent’s full history can sit in durable storage; only the tokens in active use need GPU memory, and 10M of them change the economics. As a model reads, it caches the attention keys and values of every token in every layer, so they need not be recomputed for each new token. For an illustrative 8B model, this key/value (KV) cache takes 131,072 bytes per token (keys and values: 2&nbsp;&times; 32 layers&nbsp;&times; 8 KV heads&nbsp;&times; 128 dimensions&nbsp;&times; 2 bytes): *131 GB at 1M tokens* and *1.31 TB at 10M* for one conversation, excluding weights, more than the combined memory of sixteen 80-GB H100s.

With the same architecture and precision, going from 1M to 10M tokens means roughly *100&times; the attention arithmetic during prefill* (reading the prompt) and *10&times; the attention work and KV reads per generated token*; these are scaling estimates, not latency predictions. Systems that run prefill and decoding (generating the answer) on <a href="https://arxiv.org/pdf/2401.09670" target="_blank" rel="noopener">separate GPUs</a> must also copy each request’s cache between them: 1.31 TB at 10M tokens. A paused agent ties up its cache too; evicting it frees GPU memory, but resuming then means reloading it from CPU memory or SSD, if it was saved there, or recomputing it from the text. Three practices keep these costs in check:

* *\[Use GPU memory selectively\]* <a href="https://arxiv.org/pdf/2309.06180" target="_blank" rel="noopener">Prefix sharing</a> keeps one copy of a history that several requests share; each branch needs its own entries once it diverges. <a href="https://arxiv.org/pdf/2402.02750" target="_blank" rel="noopener">Quantization</a> to 2 bits per value shrinks the cache.
* *\[Treat the KV cache as a memory tier\]* A saved cache lets a paused agent resume without rereading its history. <a href="https://arxiv.org/pdf/2609.19969" target="_blank" rel="noopener">DeepSeek-V4.1-Flash</a>, built for input-heavy agent workloads, shares cache entries across layers and stores them in 4 bits, cutting its GPU cache to 890 bytes per token, under 1% of the 8B example’s. It keeps caches on SSD for at least 72 hours; a request that hits one reuses the full prefix. For a pool of coding agents, CPU memory sized to their reusable context <a href="https://arxiv.org/pdf/2609.33762" target="_blank" rel="noopener">cut recomputed prompt tokens by 93%</a>. Letting agents <a href="https://arxiv.org/pdf/2610.28577" target="_blank" rel="noopener">drop old tool outputs from their cache</a>, rather than summarize their history and force a re-read, cut cost by 27.7% and slightly raised scores.
* *\[Budget compute and network bandwidth\]* <a href="https://arxiv.org/pdf/2205.14135" target="_blank" rel="noopener">FlashAttention</a> reduces GPU memory traffic but keeps attention’s quadratic arithmetic. <a href="https://arxiv.org/pdf/2310.01889" target="_blank" rel="noopener">Ring Attention</a> spreads one long sequence across many GPUs, which must exchange data as they compute. Placement and interconnect bandwidth matter alongside GPU capacity.

<a href="https://arxiv.org/pdf/2506.06266" target="_blank" rel="noopener">Cartridges</a> shows the tradeoff between paying once, offline, and paying on every query. With model weights frozen, it trains a compact KV cache for one corpus. Matching in-context quality on its benchmarks, it used *38.6&times; less cache memory* and reached *26.4&times; higher peak throughput*. But training one cartridge for an 8B model took about 30 minutes on eight H100s in an unoptimized implementation, and its test corpora spanned 100K to 484K tokens, far short of 10M. The one-time training pays off when many queries hit a rarely changing corpus, such as a textbook or signed contracts; each change means retraining.

Learned memory pays in training instead. <a href="https://arxiv.org/pdf/2506.10943" target="_blank" rel="noopener">SEAL</a> applies each self-edit as a LoRA update to small adapter weights; its training loop took 30-45 seconds to fine-tune and evaluate each candidate. Any weight update, even to adapters, brings training into the serving stack: backward passes to compute gradients, plus activation memory, <a href="https://arxiv.org/pdf/1910.02054" target="_blank" rel="noopener">gradient and optimizer state</a> and saved adapter versions. The dreaming study’s offline reviews add background inference; the sleep framework adds training and new parameters every cycle. Updates also age stored KV caches: in a <a href="https://arxiv.org/pdf/2611.14051" target="_blank" rel="noopener">study of cache reuse across model updates</a>, old caches kept useful task information but did not faithfully reproduce the updated model’s behavior, so caches must be versioned with the weights.

When agents share memory, they must coordinate writes. In a <a href="https://www.anthropic.com/research/components-of-agent-memory" target="_blank" rel="noopener">contract experiment</a> from the same dreaming study, agents writing to one shared table overwrote one another’s notes, and only 41 of 57 graded facts survived; one page and one writer per contract, with terms quoted verbatim, preserved all 57. Longer context cannot fix write conflicts. Separate workspaces and controlled merging protect shared state; source links and observation dates distinguish an old finding from a current fact.

## Looking Forward {#looking-forward}

The right yardstick is total cost per successfully completed task, with one-time offline compute spread over the tasks that reuse it: the cost-controlled evaluation that <a href="https://arxiv.org/pdf/2407.01502" target="_blank" rel="noopener">AI Agents That Matter</a> calls for. Test systems on a task like the week-long research project described at the start: pause and resume with new evidence, revisit a decision, combine agents’ findings. Score memory with <a href="https://arxiv.org/pdf/2410.10813" target="_blank" rel="noopener">LongMemEval</a>’s knowledge-update and multi-session accuracy, and with <a href="https://arxiv.org/pdf/2510.17776" target="_blank" rel="noopener">per-question forgetting and backward transfer</a>: answers that flip from right to wrong (forgetting) or from wrong to right (backward transfer) after an update. Score serving with time-to-first-token (TTFT), time-per-output-token (TPOT), <a href="https://arxiv.org/pdf/2401.09670" target="_blank" rel="noopener">goodput</a> (requests meeting both targets), KV cache hit rate, peak GPU memory and interconnect traffic as history grows.

## Takeaways {#takeaways}

* *\[Context is working memory\]* Only what is saved, cached or learned outlives the session.
* *\[Retrieval is not reasoning\]* Finding a fact in 10M tokens is not revising a conclusion.
* *\[Each design moves the bill\]* Long context pays at serving time; notes, caches and weight updates pay in offline compute and storage.
* *\[The KV cache is becoming a memory tier\]* Stored for days, it lets agents resume without rereading, until the weights change.
* *\[Reuse picks the winner\]* Offline compute pays off only for stable material queried again and again.

## Notes {#notes}

<ul class="notes">
  <li id="fn-1"><span class="fn-num">1</span> Continual learning also requires preserving the ability to learn. <a href="https://www.nature.com/articles/s41586-024-07711-7" target="_blank" rel="noopener">Loss of plasticity in deep continual learning</a> showed that ability deteriorating in vision and reinforcement-learning experiments, not LLM reasoning. The newer language-model studies establish bounded experimental gains, not decades of reliable memory. <a class="fn-back" href="#fnref-1" aria-label="Back to text">&#8617;&#xFE0E;</a></li>
</ul>

<div class="plot-lightbox" id="plot-lightbox" role="dialog" aria-modal="true" aria-label="Enlarged figure. Press Escape or click to close." hidden>
  <button type="button" class="plot-lightbox-close" aria-label="Close">&times;</button>
  <img id="plot-lightbox-img" src="data:," alt="">
</div>

<style>
.post-content { font-family: Arial, "Helvetica Neue", Helvetica, "Liberation Sans", sans-serif; font-size: 16px; color: #000; }
.post-content a { color: #1155cc; }
.post-content a:hover { text-decoration: underline; }
.post-content h2 { font-family: Arial, "Helvetica Neue", Helvetica, "Liberation Sans", sans-serif; font-size: 20px; font-weight: 400; line-height: 1.3; letter-spacing: 0; color: #434343; margin: 22px 0 8px 0; }
.post-content h3 { font-family: Arial, "Helvetica Neue", Helvetica, "Liberation Sans", sans-serif; font-size: 17.5px; font-weight: 400; line-height: 1.3; color: #666666; margin: 18px 0 6px 0; }
.post-content p, .post-content li { text-align: justify !important; text-justify: inter-word; }

/* Post title: no-JS fallback size that keeps the title on one line; the script below fine-tunes it */
@media screen and (min-width: 801px) {
  .post-header .post-title { font-size: 40px; white-space: normal; }
}

.post-content ul ul { list-style-type: circle !important; }
/* Footnote references */
.post-content sup.fnref { font-size: 0.65em; line-height: 0; vertical-align: super; }
.post-content sup.fnref a { color: #000; text-decoration: none; padding: 0 1px; }
.post-content sup.fnref a:hover { color: #1155cc; text-decoration: underline; }
/* Tables: purple bold headers, green first column, light dividers */
.post-content .table-wrap { width: min(var(--tw, 100%), calc(100vw - 48px)); margin: -3px 0 14px 50%; transform: translateX(-50%); overflow-x: auto; }
.post-content h2 + .table-wrap, .post-content h3 + .table-wrap { margin-top: 6px; }
.post-content table.pdf-table { width: 100%; border-collapse: collapse; border: 0; margin: 0 auto; font-family: Arial, "Helvetica Neue", Helvetica, "Liberation Sans", sans-serif; font-size: 14.5px; line-height: 1.35; color: #000; background: none; hyphens: manual; }
.post-content table.t-blog { table-layout: fixed; }
.post-content table.pdf-table tr, .post-content table.pdf-table tr:nth-child(even) { background: none; }
.post-content table.pdf-table th, .post-content table.pdf-table td { border: 0; background: none; padding: 8px 8px; vertical-align: middle; text-align: left; }
.post-content table.pdf-table thead th { font-size: 16px; font-weight: 700; color: #674ea7; line-height: 1.25; border-bottom: 1.5px solid #000; }
.post-content table.pdf-table tbody td { border-bottom: 1px solid #c9d3de; }
.post-content table.pdf-table .metric { color: #38761d; }
.post-content table.t-blog tbody td { text-align: justify; text-justify: inter-word; }
.post-content table.pdf-table .vsep-l { border-right: 1px solid #c9c9c9; }
.post-content table.t-blog tbody tr:last-child td, .post-content table.t-blog tbody td.last-group { border-bottom: 1px solid #000; }
/* Figure 1: floats right at a quarter of the text width; the text wraps around it */
.post-content .fig-right { float: right; width: 40%; margin: 5px 0 8px 22px; }
.post-content .fig-right a.plot-zoom { display: block; line-height: 0; cursor: zoom-in; }
.post-content .fig-right img { display: block; width: 100%; height: auto; }
.post-content .fig-right p.fig-caption { font-size: 12px; line-height: 1.4; color: #555555; text-align: justify !important; hyphens: auto; margin: 6px 0 0 0; }
.post-content .fig-right p.plot-hint { font-size: 12px; font-style: italic; color: #6b7280; text-align: center !important; margin: 4px 0 0 0; }
/* Click to zoom: the figure opens in a centred viewer, 65% of the screen width */
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
/* Narrow screens: the figure stops floating and the viewer widens */
@media screen and (max-width: 600px) {
  .post-content .fig-right { float: none; width: 75%; margin: 6px auto 14px auto; }
}
@media screen and (max-width: 800px) {
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
  /* Open the figure in a centred viewer; Escape or a click closes it */
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
