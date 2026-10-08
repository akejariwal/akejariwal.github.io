---
layout: post
title: "Optimizing Agentic Execution: Putting the Ideas to the Test"
date: 2026-10-08
description: "Can compiler-inspired ideas make AI agents more efficient? Across ten Terminal-Bench tasks, a combined intervention
              cut cost by 21% and execution time by 24% among successful runs—but earlier stopping exposed a trade-off.
              Here’s what worked, what varied, and where the savings came from."
categories: [agentic execution, harness optimization, eval, agents]
author: Arun Kejariwal
last_modified_at: 2026-10-08
---

An earlier post, [Optimization of Agentic Execution]({{ site.baseurl }}/2026-07-27-optimizing-agentic-execution.html), mapped compiler optimizations to agentic execution. Here we test whether six compiler-inspired directives, combined with a Bash-output cap, reduce cost and execution time on the following ten [Terminal-Bench 4.0](https://www.tbench.ai/news/terminal-bench-4-0){:target="_blank" rel="noopener"} tasks:

* `coq-block-bound`
* `embedding-drift-monitor`
* `risk-scorer-replay`
* `sound-change-cascade`
* `layout-config-recreation2`
* `telecom-entity-resolution`
* `retro-console-soc`
* `payments-pipeline-fix`
* `pretrain-shard-corruption`
* `takens-embedding-lean`

The agent is Claude Code 2.1.289 in headless mode, backed by claude-opus-5-5, the only model recorded in any run. It ran on a cloud VM: a KVM guest with 4 vCPUs (Intel Xeon @ 2.10 GHz), 15.7 GiB RAM and a 252 GiB disk, running Ubuntu 24.04.4 LTS and Docker 29.6.2. Every shell command ran inside the task’s own container, under the task’s resource caps (2-4 CPUs and 4-8 GiB; 10 GiB for `takens-embedding-lean`) and with no time limit. Each task ran five times in each of two arms. The *baseline* arm had no directives. The *optimized* arm had the six directives - inspired from the mapping between compiler optimizations and agentic execution shared in an earlier [post]({{ site.baseurl }}/2026-07-27-optimizing-agentic-execution.html) - in `CLAUDE.md` and a 12,000-character cap on Bash output. We evaluate these changes as one combined intervention; this experiment does not isolate the contributions of individual directives or the output cap. Runs were independent rather than paired (run k of one arm has no special link to run k of the other), and the arm that ran first alternated between repeats: baseline first on odd repeats, optimized first on even ones.<sup class="fnref" id="fnref-1"><a href="#fn-1">1</a></sup> Each run was graded by the task’s own verifier.

## Metrics {#metrics}

For each run, we monitored the following metrics:

<table class="pdf-table t-defs">
  <colgroup><col style="width: 18%"><col style="width: 82%"></colgroup>
  <thead>
    <tr><th class="vsep">Metric</th><th>Definition</th></tr>
  </thead>
  <tbody>
    <tr><td class="metric vsep">Cost</td><td>List-price cost reported by Claude Code.</td></tr>
    <tr><td class="metric vsep">Wall minutes</td><td>Wall-clock time of the agent session, from launch to exit; excludes container setup and grading.</td></tr>
    <tr><td class="metric vsep">Input tokens</td><td>All input tokens, cached or not.</td></tr>
    <tr><td class="metric vsep">Output tokens</td><td>All generated tokens, including reasoning.</td></tr>
    <tr><td class="metric vsep">Cache-hit rate</td><td>Share of input tokens read from the prompt cache.</td></tr>
    <tr><td class="metric vsep">Uncached input</td><td>Input not served from the prompt cache (fresh input plus cache writes).</td></tr>
    <tr><td class="metric vsep">Reasoning share</td><td>Reasoning tokens as a share of output tokens.</td></tr>
    <tr><td class="metric vsep">Turns</td><td>Model turns, summed over all wake-ups of the session.</td></tr>
  </tbody>
</table>

## Analysis Methodology {#analysis-methodology}

We compare the baseline and optimized configurations using successful runs only. For each task and metric, we estimate the effect of the combined intervention with the [two-sample Hodges–Lehmann (HL) shift on the log scale](https://projecteuclid.org/journals/annals-of-mathematical-statistics/volume-34/issue-2/Estimates-of-Location-Based-on-Rank-Tests/10.1214/aoms/1177704172.full){:target="_blank" rel="noopener"}, i.e., the median of all pairwise log-ratios between optimized and baseline runs, reported as a percentage change.<sup class="fnref" id="fnref-2"><a href="#fn-2">2</a></sup> For the two rates, cache-hit rate and reasoning share, we report differences in percentage points. Changes quoted for individual tasks are these HL estimates; the plots show arm medians, a different estimator.

To summarize across tasks, we average the per-task log-shifts. This is the geometric mean<sup class="fnref" id="fnref-3"><a href="#fn-3">3</a></sup> of the per-task ratios [<a href="https://doi.org/10.1145/5666.5673" target="_blank" rel="noopener">1</a>], so every task carries equal weight (see [Appendix A](#appendix-a)). We flag a run as anomalous when its robust z-score, based on the median absolute deviation (MAD), exceeds 2.5 in magnitude within its task and arm [<a href="https://doi.org/10.1016/j.jesp.2013.03.013" target="_blank" rel="noopener">2</a>]. Flagged runs are reported but never removed.<sup class="fnref" id="fnref-4"><a href="#fn-4">4</a></sup>

To find what drives time and cost, we fit run-level Gamma generalized linear models (GLMs) with a log link and task fixed effects [<a href="https://doi.org/10.2307/2344614" target="_blank" rel="noopener">3</a>]: <code class="eq">log E[y] = α<sub>task</sub> + β·log x</code>. The coefficient <code class="eq">β</code> is a within-task elasticity, the percentage change in y per 1% change in x. The two rates enter unlogged, in percentage points (0–100): <code class="eq">log E[y] = α<sub>task</sub> + β·r</code>, so each one-point increase changes y by the same <code class="eq">100 × (exp(β) − 1)%</code>. Fit is reported as the share of within-task deviance that the predictor explains, D².<sup class="fnref" id="fnref-5"><a href="#fn-5">5</a></sup>

## Results {#results}

The table below summarizes the combined intervention across the ten tasks: relative changes are geometric means of the per-task HL ratios, and changes in the two rates are in percentage points.

<table class="pdf-table t-summary">
  <colgroup><col style="width: 63%"><col style="width: 37%"></colgroup>
  <thead>
    <tr><th class="vsep">Metric</th><th class="num">% change</th></tr>
  </thead>
  <tbody>
    <tr class="topline"><td class="metric vsep">Cost</td><td class="num">−20.9%</td></tr>
    <tr class="topline"><td class="metric vsep">Wall minutes</td><td class="num">−24.0%</td></tr>
    <tr><td class="metric vsep">Input tokens</td><td class="num">−25.5%</td></tr>
    <tr><td class="metric vsep">Output tokens</td><td class="num">−20.1%</td></tr>
    <tr><td class="metric vsep">Cache-hit rate</td><td class="num">−0.3 pts</td></tr>
    <tr><td class="metric vsep">Uncached input</td><td class="num">−19.8%</td></tr>
    <tr><td class="metric vsep">Reasoning share</td><td class="num">−1.3 pts</td></tr>
    <tr><td class="metric vsep">Turns</td><td class="num">−15.7%</td></tr>
  </tbody>
</table>

Among successful runs, the combined intervention was associated with reductions of 20.9% in cost and 24.0% in wall time, the two topline metrics (in bold). The under-the-hood metrics overlap and fell by varying amounts, input and output tokens the most. Cache-hit rate and reasoning share barely moved, so the savings came from the agent doing less work, not from better caching or less reasoning per token. The Kiviat plots below compare the two arms on each task. In each plot,

* Each vertex is the per-metric median over that arm's successful runs.
* The token and turn axes are scaled to the larger of the two arms, so the larger arm sits at 100%.
* Cache-hit rate and reasoning share are plotted as they are, on a 0-100% scale.
* The numbers under each axis label are the baseline / optimized medians.
* These plots show arm medians for descriptive comparison. A ratio of two medians is a different estimator from the HL effect, the median of all pairwise ratios, and can differ from it substantially, even in sign.

<div class="plot-grid plot-grid--kiviat">
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/coq-block-bound_kiviat_8k.png"><img src="{{ site.baseurl }}/assets/images/coq-block-bound_kiviat_8k.png" width="7680" height="4320" loading="lazy" decoding="async" alt="Kiviat plot for coq-block-bound: baseline vs optimized medians"></a>
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/embedding-drift-monitor_kiviat_8k.png"><img src="{{ site.baseurl }}/assets/images/embedding-drift-monitor_kiviat_8k.png" width="7680" height="4320" loading="lazy" decoding="async" alt="Kiviat plot for embedding-drift-monitor: baseline vs optimized medians"></a>
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/risk-scorer-replay_kiviat_8k.png"><img src="{{ site.baseurl }}/assets/images/risk-scorer-replay_kiviat_8k.png" width="7680" height="4320" loading="lazy" decoding="async" alt="Kiviat plot for risk-scorer-replay: baseline vs optimized medians"></a>
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/sound-change-cascade_kiviat_8k.png"><img src="{{ site.baseurl }}/assets/images/sound-change-cascade_kiviat_8k.png" width="7680" height="4320" loading="lazy" decoding="async" alt="Kiviat plot for sound-change-cascade: baseline vs optimized medians"></a>
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/layout-config-recreation2_kiviat_8k.png"><img src="{{ site.baseurl }}/assets/images/layout-config-recreation2_kiviat_8k.png" width="7680" height="4320" loading="lazy" decoding="async" alt="Kiviat plot for layout-config-recreation2: baseline vs optimized medians"></a>
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/telecom-entity-resolution_kiviat_8k.png"><img src="{{ site.baseurl }}/assets/images/telecom-entity-resolution_kiviat_8k.png" width="7680" height="4320" loading="lazy" decoding="async" alt="Kiviat plot for telecom-entity-resolution: baseline vs optimized medians"></a>
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/retro-console-soc_kiviat_8k.png"><img src="{{ site.baseurl }}/assets/images/retro-console-soc_kiviat_8k.png" width="7680" height="4320" loading="lazy" decoding="async" alt="Kiviat plot for retro-console-soc: baseline vs optimized medians"></a>
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/payments-pipeline-fix_kiviat_8k.png"><img src="{{ site.baseurl }}/assets/images/payments-pipeline-fix_kiviat_8k.png" width="7680" height="4320" loading="lazy" decoding="async" alt="Kiviat plot for payments-pipeline-fix: baseline vs optimized medians"></a>
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/pretrain-shard-corruption_kiviat_8k.png"><img src="{{ site.baseurl }}/assets/images/pretrain-shard-corruption_kiviat_8k.png" width="7680" height="4320" loading="lazy" decoding="async" alt="Kiviat plot for pretrain-shard-corruption: baseline vs optimized medians"></a>
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/takens-embedding-lean_kiviat_8k.png"><img src="{{ site.baseurl }}/assets/images/takens-embedding-lean_kiviat_8k.png" width="7680" height="4320" loading="lazy" decoding="async" alt="Kiviat plot for takens-embedding-lean: baseline vs optimized medians"></a>
</div>

<p class="plot-hint">Click on a plot to zoom in.</p>

Three patterns stand out (changes are HL estimates).

* On `payments-pipeline-fix` and `retro-console-soc` the agent did less of everything: turns fell by 57% and 43%, and input tokens by 78% and 57%.
* On `embedding-drift-monitor`, `risk-scorer-replay` and `telecom-entity-resolution` the cut is mostly in output and uncached input (output −49%, −18% and −12%; uncached input −27%, −20% and −22%), with turns falling less.
* On the remaining five tasks the token changes are small or mixed, and on `takens-embedding-lean` the optimized arm even used more turns (+22%) and input (+20%).

Across all ten tasks the two rate axes hardly move: median cache-hit rate stays between 94% and 99%, and reasoning share shifts by at most six points. The topline savings therefore come through different levers on different tasks: fewer turns on some, less output and fresh input on others and, for wall time on `layout-config-recreation2` and `telecom-entity-resolution`, less time running commands in the container (tool/other minutes −53% and −40%), which no token axis captures.

The driver models, which quantify within-task associations with the topline metrics, show why cost and model time move together. Model minutes are proportional to output tokens (β = 1.00, D² = 0.996), and cost depends on output and input jointly (β<sub>out</sub> = 0.60, β<sub>in</sub> = 0.34, D² = 0.99). Tokens explain wall time less well (D² = 0.68 for input and output together), because token counts do not capture time spent running commands in the container. Lower output volume was associated with lower model time and cost, while wall-time savings also depended on tool execution and other overhead. Note that the under-the-hood metrics are not independent: a run with more turns also re-reads more input and writes more output. The GLM does not separate these on its own, so we fit each predictor alone, which captures its total association with the response, and input and output together, where each coefficient holds the other fixed. This is why output’s elasticity for cost falls from 1.05 alone to 0.60 jointly. Task fixed effects account for stable differences between tasks; within-task associations may still reflect shared causes.

The tables below fit wall minutes and cost to each of the six metrics in the Kiviat plots, one at a time.

<table class="pdf-table t-drivers t-wall">
  <caption class="table-note">* For the two rates, which enter unlogged in percentage points, the entry is 100 × (exp(β) − 1)%: the exact change in the response per one-point increase in the rate, the same at every level of the rate.</caption>
  <colgroup><col style="width: 36%"><col style="width: 28%"><col style="width: 36%"></colgroup>
  <thead>
    <tr><th class="vsep-l">Predictor<br>(wall minutes)</th><th class="ctr vsep-l">Elasticity</th><th class="ctr">Within-task<br>deviance explained</th></tr>
  </thead>
  <tbody>
    <tr><td class="metric vsep-l">Input tokens</td><td class="ctr vsep-l">0.75</td><td class="ctr">66%</td></tr>
    <tr><td class="metric vsep-l">Cache-hit rate</td><td class="ctr vsep-l">+39% per pt*</td><td class="ctr">35%</td></tr>
    <tr><td class="metric vsep-l">New (uncached) input</td><td class="ctr vsep-l">1.11</td><td class="ctr">53%</td></tr>
    <tr><td class="metric vsep-l">Output tokens</td><td class="ctr vsep-l">1.08</td><td class="ctr">59%</td></tr>
    <tr><td class="metric vsep-l">Reasoning share</td><td class="ctr vsep-l">+1.4% per pt</td><td class="ctr">2%</td></tr>
    <tr><td class="metric vsep-l">Turns</td><td class="ctr vsep-l">1.11</td><td class="ctr">61%</td></tr>
  </tbody>
</table>

Turns, output and uncached input each scale wall time slightly more than one-for-one (elasticities of 1.08-1.11) and explain 53-61% of its within-task variation. Total input explains the most (66%) but scales less than proportionally (0.75), as most of it is re-read from the prompt cache, which is fast and cheap. Reasoning share explains almost nothing (2%): output volume is more strongly associated with wall time than reasoning share in these runs. The positive cache-hit coefficient most likely reflects session length rather than a lever, since longer sessions re-read more of their own cached history and so raise their hit rate.

<table class="pdf-table t-drivers t-cost">
  <colgroup><col style="width: 36%"><col style="width: 28%"><col style="width: 36%"></colgroup>
  <thead>
    <tr><th class="vsep-l">Predictor<br>(cost)</th><th class="ctr vsep-l">Elasticity</th><th class="ctr">Within-task<br>deviance explained</th></tr>
  </thead>
  <tbody>
    <tr><td class="metric vsep-l">Input tokens</td><td class="ctr vsep-l">0.69</td><td class="ctr">91%</td></tr>
    <tr><td class="metric vsep-l">Cache-hit rate</td><td class="ctr vsep-l">+28% per pt</td><td class="ctr">33%</td></tr>
    <tr><td class="metric vsep-l">New (uncached) input</td><td class="ctr vsep-l">1.16</td><td class="ctr">93%</td></tr>
    <tr><td class="metric vsep-l">Output tokens</td><td class="ctr vsep-l">1.05</td><td class="ctr">93%</td></tr>
    <tr><td class="metric vsep-l">Reasoning share</td><td class="ctr vsep-l">+2.3% per pt</td><td class="ctr">8%</td></tr>
    <tr><td class="metric vsep-l">Turns</td><td class="ctr vsep-l">0.98</td><td class="ctr">78%</td></tr>
  </tbody>
</table>

Tokens explain cost far better than wall time, 91–93% of its within-task variation against 53–66%, as expected of a bill computed from token counts. Output tokens and uncached input move cost about one-for-one (1.05 and 1.16), while total input moves it less than proportionally (0.69), because cache reads are billed at a fraction of the price of fresh input. Turns explain 78%, with cost rising in proportion (0.98). The two rates again matter little: reasoning share explains 8%, and the cache-hit coefficient most likely reflects session length, as above. Cost, then, tracks what the agent writes and the fresh input it reads; wall time also tracks what it runs in the container.

## Things to Note {#things-to-note}

Beyond the averages, a few observations stand out.

* Single runs are unreliable. On `pretrain-shard-corruption`, the first optimized run took 149 minutes against 77 for the baseline, yet over five repeats the two arms' medians were almost identical (53.3 and 52.9 minutes), which is why we report repeat-level estimates. The variation across runs, for each task, is reported in [Appendix B](#appendix-b). Anomalous runs are reported in [Appendix C](#appendix-c).
* Both optimized failures stopped after a weak self-check: `layout-config-recreation2` scored 97.99% pixel similarity against a 98% bar, and `payments-pipeline-fix` validated a fix with a mock test only, wrongly concluding that no Kafka broker was available, and stopped after 4.5 minutes. This fits the directive that tells the agent to stop once one check passes; a refinement would require that check to exercise the real system the task provides. Weak self-checks also occurred in the baseline: the one baseline failure, on `retro-console-soc`, validated against a reference emulator the agent wrote itself.
* On `layout-config-recreation2`, the optimized arm wrote more output (+15%) yet finished faster, because it spent less time waiting on long-running search scripts in the container: it stopped once a render check passed, while the baseline kept searching for a closer match. Taken together, the shorter successful runs and the optimized arm’s one failure on this task suggest a possible trade-off between earlier stopping and sufficient validation.

## Closing Thoughts {#closing-thoughts}

The directives evaluated here are `CLAUDE.md` advice that the model may or may not follow on a given step, and they are high level by necessity, since the models expose no low-level API for requesting a specific behavior. A harness-based evaluation is the logical next step ([Appendix D](#appendix-d) sketches a stop hook that runs the real check before the agent may finish), and the field is already moving in that direction.

Since the beginning of ’26, there has been a flurry of research in harness optimization [<a href="https://lilianweng.github.io/posts/2026-07-04-harness/" target="_blank" rel="noopener">4</a>], much of it on harnesses that adapt automatically: auto-compaction [<a href="https://arxiv.org/pdf/2610.02163" target="_blank" rel="noopener">5</a>, <a href="https://arxiv.org/pdf/2609.26779" target="_blank" rel="noopener">6</a>], meta-reasoning [<a href="https://arxiv.org/pdf/2609.38147" target="_blank" rel="noopener">7</a>, <a href="https://arxiv.org/pdf/2610.02525" target="_blank" rel="noopener">8</a>], loop-based setups and self-evolving harnesses [<a href="https://a16z.com/knowing-when-to-stop-the-art-of-making-a-loop-converge/" target="_blank" rel="noopener">9</a>, <a href="https://arxiv.org/pdf/2603.28052" target="_blank" rel="noopener">10</a>] and, more recently, recursive self-improvement (RSI) applied to the harness itself [<a href="https://arxiv.org/pdf/2609.14857" target="_blank" rel="noopener">11</a>].

How do the compiler-inspired directives interplay with these directions? Harness search has independently discovered mechanisms analogous to some of the directives, including fusing an edit with its test into one tool call and archiving long outputs behind a handle [<a href="https://arxiv.org/pdf/2609.20519" target="_blank" rel="noopener">12</a>]; another study found that jointly evolving tools, middleware and memory alongside prompts outperformed prompt-only approaches [<a href="https://arxiv.org/pdf/2604.25850" target="_blank" rel="noopener">13</a>]. Seeding such a search with the directives, and compiling the survivors into the harness rather than `CLAUDE.md`, would be a natural follow-up.

Evaluating that follow-up raises a broader question: how much does each optimization add beyond what the harness already does, and at what cost? Isolating that incremental contribution requires a common baseline and a consistent reporting protocol. A shared reference harness, evaluated on held-out tasks with success rate, cost and execution time reported together, would make those contributions easier to assess, helping the community identify complementary ideas, reduce duplication and advance the cost-performance frontier.

## Notes {#notes}

<ul class="notes">
  <li id="fn-1"><span class="fn-num">1</span> Agent runs are stochastic, so we repeat each configuration five times. Alternating the arm order reduces systematic order effects but does not guarantee equivalent prompt-cache state. <a class="fn-back" href="#fnref-1" aria-label="Back to text">&#8617;&#xFE0E;</a></li>
  <li id="fn-2"><span class="fn-num">2</span> Run times and token counts are right-skewed, and a single extreme run can dominate a mean or a t-test at n = 5. HL uses all pairwise ratios, so one outlier moves only a few of them. The log scale turns multiplicative effects into shifts. <a class="fn-back" href="#fnref-2" aria-label="Back to text">&#8617;&#xFE0E;</a></li>
  <li id="fn-3"><span class="fn-num">3</span> The geometric mean keeps long or expensive tasks from dominating the summary. <a class="fn-back" href="#fnref-3" aria-label="Back to text">&#8617;&#xFE0E;</a></li>
  <li id="fn-4"><span class="fn-num">4</span> The median and MAD are themselves robust, so an outlier cannot mask itself by inflating the spread, as it can with mean ± k·SD. HL already bounds each run’s influence, so removing flagged runs is unnecessary. <a class="fn-back" href="#fnref-4" aria-label="Back to text">&#8617;&#xFE0E;</a></li>
  <li id="fn-5"><span class="fn-num">5</span> Time and cost are positive, right-skewed, and their conditional variance is modeled as proportional to the square of the conditional mean, which is the Gamma family’s assumption. Task fixed effects remove differences between tasks, so <code>β</code> is estimated from variation within tasks across all successful runs. <a class="fn-back" href="#fnref-5" aria-label="Back to text">&#8617;&#xFE0E;</a></li>
</ul>

## Appendix {#appendix}

### A. Geometric mean of % changes {#appendix-a}

1. Turn each task's % change into a ratio: r = 1 + p/100. For example, −68.5% becomes 0.315 and +3.1% becomes 1.031.
2. Take the log of each ratio and average the logs across the 10 tasks.
3. Convert back: 100 \* \[exp(mean) − 1\]%.

### B. Variation across runs {#appendix-b}

<div class="plot-grid plot-grid--bars">
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/coq-block-bound_bars_8k.png"><img src="{{ site.baseurl }}/assets/images/coq-block-bound_bars_8k.png" width="7680" height="4320" loading="lazy" decoding="async" alt="Cost and wall minutes per run, baseline vs optimized, for coq-block-bound"></a>
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/embedding-drift-monitor_bars_8k.png"><img src="{{ site.baseurl }}/assets/images/embedding-drift-monitor_bars_8k.png" width="7680" height="4320" loading="lazy" decoding="async" alt="Cost and wall minutes per run, baseline vs optimized, for embedding-drift-monitor"></a>
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/risk-scorer-replay_bars_8k.png"><img src="{{ site.baseurl }}/assets/images/risk-scorer-replay_bars_8k.png" width="7680" height="4320" loading="lazy" decoding="async" alt="Cost and wall minutes per run, baseline vs optimized, for risk-scorer-replay"></a>
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/sound-change-cascade_bars_8k.png"><img src="{{ site.baseurl }}/assets/images/sound-change-cascade_bars_8k.png" width="7680" height="4320" loading="lazy" decoding="async" alt="Cost and wall minutes per run, baseline vs optimized, for sound-change-cascade"></a>
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/layout-config-recreation2_bars_8k.png"><img src="{{ site.baseurl }}/assets/images/layout-config-recreation2_bars_8k.png" width="7680" height="4320" loading="lazy" decoding="async" alt="Cost and wall minutes per run, baseline vs optimized, for layout-config-recreation2"></a>
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/telecom-entity-resolution_bars_8k.png"><img src="{{ site.baseurl }}/assets/images/telecom-entity-resolution_bars_8k.png" width="7680" height="4320" loading="lazy" decoding="async" alt="Cost and wall minutes per run, baseline vs optimized, for telecom-entity-resolution"></a>
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/retro-console-soc_bars_8k.png"><img src="{{ site.baseurl }}/assets/images/retro-console-soc_bars_8k.png" width="7680" height="4320" loading="lazy" decoding="async" alt="Cost and wall minutes per run, baseline vs optimized, for retro-console-soc"></a>
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/payments-pipeline-fix_bars_8k.png"><img src="{{ site.baseurl }}/assets/images/payments-pipeline-fix_bars_8k.png" width="7680" height="4320" loading="lazy" decoding="async" alt="Cost and wall minutes per run, baseline vs optimized, for payments-pipeline-fix"></a>
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/pretrain-shard-corruption_bars_8k.png"><img src="{{ site.baseurl }}/assets/images/pretrain-shard-corruption_bars_8k.png" width="7680" height="4320" loading="lazy" decoding="async" alt="Cost and wall minutes per run, baseline vs optimized, for pretrain-shard-corruption"></a>
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/takens-embedding-lean_bars_8k.png"><img src="{{ site.baseurl }}/assets/images/takens-embedding-lean_bars_8k.png" width="7680" height="4320" loading="lazy" decoding="async" alt="Cost and wall minutes per run, baseline vs optimized, for takens-embedding-lean"></a>
</div>

<p class="plot-hint">Click on a plot to zoom in.</p>

### C. Anomalous runs {#appendix-c}

A run's value on wall time, model time, cost, input or output tokens is flagged when its robust z-score exceeds 2.5 in magnitude within its task and arm; flagged runs stay in every analysis.

* 39 values are flagged, on 19 of the 97 successful runs (11 baseline, 8 optimized) across 7 of the 10 tasks; `pretrain-shard-corruption`{:.code-green}, `risk-scorer-replay`{:.code-green} and `sound-change-cascade`{:.code-green} have none. Five runs are flagged on three or more metrics at once, since time, tokens and cost tend to move together.
* Most flagged values are modest: 23 of 39 lie within ±35% of their arm's median. A large z-score often reflects how tightly the other runs cluster rather than an extreme value; the largest, z = −12.7 for output on `layout-config-recreation2`{:.code-green} (optimized, run 5), is only 24% below its arm's median. The biggest departures are baseline runs: `embedding-drift-monitor`{:.code-green} run 2 took 3.1 times its arm's median wall time (11.8 vs 3.8 minutes), and `payments-pipeline-fix`{:.code-green} run 5 took 2.2 times (85 vs 39 minutes).
* The flags lean in the direction of the effect: baseline flags are mostly unusually high values (12 of 19), and optimized flags mostly unusually low ones (14 of 20). Dropping all 19 flagged runs narrows the gap only slightly: the cost effect moves from −20.9% to −18.9%, wall time from −24.0% to −21.6%, and no driver elasticity changes by more than 0.05.

### D. A stop hook that runs the real check {#appendix-d}

The directive that tells the agent to stop once one check passes leaves the choice of check to the agent, and both optimized failures stopped after a weak self-check: a mock test instead of the real Kafka pipeline, and a render that scored 97.99% against a 98% bar. A stop hook moves the choice of check into the harness.

* *\[When it runs\]* Claude Code runs the hook, registered under `Stop` in `.claude/settings.json`, every time the agent tries to finish, and passes it the event as JSON.
* *\[What it does\]* It runs the task’s check where the agent’s commands run. If the check passes, the agent may stop. If it fails, the hook blocks the stop, and Claude receives the reason, with the tail of the check’s output, as its next instruction.
* *\[Which check\]* Only one that the task exposes to the agent: its existing tests, a provided reproduction or smoke script, or a health check against the real services. Never the hidden grader, which would leak the benchmark’s answer key.
* *\[Safeguards\]* After three blocks in a row, the hook lets the agent stop anyway, so it can never trap the agent. A check that runs longer than 900 seconds is killed.

<div class="plot-lightbox" id="plot-lightbox" role="dialog" aria-modal="true" aria-label="Enlarged plot. Press Escape or click to close." hidden>
  <button type="button" class="plot-lightbox-close" aria-label="Close">&times;</button>
  <img id="plot-lightbox-img" src="data:," alt="">
</div>

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
.post-content h3 {
  font-family: Arial, "Helvetica Neue", Helvetica, "Liberation Sans", sans-serif;
  font-size: 17.5px;            /* 12pt */
  font-weight: 400;
  line-height: 1.3;
  color: #666666;
  margin: 24px 0 8px 0;
}

/* Fully justify body text, including bullet items (the site stylesheet left-aligns lists) */
.post-content p,
.post-content li { text-align: justify !important; text-justify: inter-word; }

/* Monospace, as in the PDF: Courier New, no badge background */
.post-content code {
  font-family: "Courier New", Courier, "Liberation Mono", monospace;
  font-size: 1em;
  color: inherit;
  background: none;
  border: 0;
  border-radius: 0;
  padding: 0;
  white-space: nowrap;
  hyphens: manual;
}
.post-content code.code-green { color: #188038; }
.post-content code.eq { font-size: 1.09em; }      /* equations are 12pt in the PDF */
.post-content code.eq sub { font-size: 0.6em; }
.post-content sub { line-height: 0; }

/* Footnote references */
.post-content sup.fnref { font-size: 0.65em; line-height: 0; vertical-align: super; }
.post-content sup.fnref a { color: #000; text-decoration: none; padding: 0 1px; }
.post-content sup.fnref a:hover { color: #1155cc; text-decoration: underline; }

/* ---------- tables: purple bold headers, green metric names, as in the PDF ---------- */
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
.post-content table.t-defs td.metric { white-space: nowrap; }
.post-content table.pdf-table tr.topline td { font-weight: 700; }
.post-content table.pdf-table th.num,
.post-content table.pdf-table td.num { text-align: right; padding-right: 2px; }
.post-content table.pdf-table th.ctr,
.post-content table.pdf-table td.ctr { text-align: center; }
.post-content table.pdf-table .vsep { border-right: 1px solid #000; }
.post-content table.pdf-table .vsep-l { border-right: 1px solid #c9c9c9; }
.post-content table.t-defs tbody tr:last-child td,
.post-content table.t-cost tbody tr:last-child td { border-bottom: 1px solid #000; }
.post-content table.t-summary { width: 34%; min-width: 250px; }
.post-content table.t-drivers { width: 66%; min-width: 320px; }
.post-content table.pdf-table caption.table-note {
  caption-side: bottom;
  text-align: justify;
  font-size: 11px;
  line-height: 1.35;
  color: #000;
  padding-top: 4px;
}

/* ---------- plot grids: two columns, as tabulated in the PDF ---------- */
.plot-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px 4%;
  margin: 16px auto 6px auto;
}
.plot-grid--kiviat { width: 94%; }
.plot-grid--bars { width: 100%; gap: 16px 2%; }
.plot-grid a.plot-zoom { display: block; line-height: 0; cursor: zoom-in; }
.plot-grid img { display: block; width: 100%; height: auto; }
.post-content p.plot-hint {
  text-align: center !important;
  font-size: 0.875rem;
  font-style: italic;
  color: #6b7280;
  margin: 6px 0 24px 0;
}

/* ---------- Notes (footnotes) ---------- */
.post-content ul.notes { font-size: 13px; line-height: 1.5; }
.post-content ul.notes li { margin-bottom: 6px; padding: 2px 4px; border-radius: 3px; scroll-margin-top: 20px; }
.post-content ul.notes li:target { background: #fff6cc; }
.post-content ul.notes .fn-num { font-size: 0.75em; vertical-align: super; line-height: 0; margin-right: 2px; }
.post-content ul.notes .fn-back { text-decoration: none; margin-left: 2px; }

/* ---------- zoomed plot: centred, 50% of the screen width ---------- */
.plot-lightbox {
  position: fixed;
  inset: 0;
  z-index: 9999;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(17, 17, 17, 0.78);
  cursor: zoom-out;
}
.plot-lightbox[hidden] { display: none; }
.plot-lightbox img {
  width: 50vw;
  height: auto;
  max-height: 94vh;
  object-fit: contain;
  background: #ffffff;
  border-radius: 4px;
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.45);
}
.plot-lightbox-close {
  position: absolute;
  top: 10px;
  right: 18px;
  border: 0;
  background: none;
  color: #ffffff;
  font: 36px/1 Arial, sans-serif;
  cursor: pointer;
}

/* Narrow screens: 50% of a phone is too small to read, so widen the zoom and stack wide tables */
@media screen and (max-width: 800px) {
  .plot-lightbox img { width: 94vw; }
  .post-content table.t-summary,
  .post-content table.t-drivers { width: 100%; min-width: 0; }
}
@media screen and (max-width: 480px) {
  .plot-grid { grid-template-columns: 1fr; }
  .plot-grid--kiviat { width: 100%; }
}
</style>

<script>
(function () {
  var box = document.getElementById('plot-lightbox');
  var big = document.getElementById('plot-lightbox-img');
  if (!box || !big) { return; }
  /* Move the viewer to <body> so no ancestor can clip or offset it */
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
