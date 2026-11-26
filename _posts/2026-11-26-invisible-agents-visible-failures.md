---
layout: post
title: "Invisible Agents, Visible Failures"
date: 2026-11-26
description: "Agents may soon outnumber people a hundredfold, with many working out of sight inside the laptops, phones and
              services we use. When something goes wrong, the symptom can surface far from the decision that caused it. What
              does it take to trace that failure upstream - and test whether a proposed fix addresses its cause?"
categories: [multi-agent systems, failure attribution, observability, endpoint management, agents]
author: Arun Kejariwal
last_modified_at: 2026-11-26
---

Agents are everywhere. They write and review code, triage tickets (startups such as <a href="https://www.atomicwork.com/" target="_blank" rel="noopener">Atomicwork</a> and <a href="https://decagon.ai/product" target="_blank" rel="noopener">Decagon</a> sell agents for it), reconcile invoices and patch fleets of laptops, often handing work to one another along the way. And their number is forecast to explode. Box CEO Aaron Levie expects <em><a href="https://techcrunch.com/2025/10/29/box-ceo-aaron-levie-on-how-ai-is-changing-the-enterprise-saas-landscape/" target="_blank" rel="noopener">“about 100 times more, maybe 1,000 times more, agents than we have people”</a></em>. IDC projects <a href="https://www.idc.com/resource-center/blog/agent-adoption-the-it-industrys-next-great-inflection-point/" target="_blank" rel="noopener">more than a billion actively deployed agents by 2029</a>, 40 times the 2025 count.

More agents mean more ways to fail<sup class="fnref" id="fnref-1"><a href="#fn-1">1</a></sup>, and more failures that do not look like errors: a run can finish without an exception, return a plausible answer and still miss its task (an agent reinstalls a laptop’s VPN client, sees the install succeed and closes the ticket, but the laptop still cannot connect). Multi-agent systems add failure modes of their own. <a href="https://arxiv.org/pdf/2503.13657" target="_blank" rel="noopener">MAST</a>, a taxonomy developed from 150 traces and then used to annotate more than 1,600 across seven frameworks, catalogues 14 of them, from flaws in system design to misalignment between agents to weak verification of the task. Agents that share tools, memory and context also have side effects on one another: one agent’s write becomes another’s premise (see <a href="#below-the-waterline">Below the Waterline</a>). And failure has more than one axis: a run can reach the right outcome too slowly or too expensively, break a policy on the way, or satisfy its grader while the task stays undone (see <a href="{{ site.baseurl }}/2026-09-13-eval-computer-systems-agents.html">Eval: From Computer Systems to Agents</a>). Increasingly, these failures will surface far from the agents that cause them.

## Agents Behind the Endpoint {#agents-behind-the-endpoint}

Many of these agents will work out of sight. People will use endpoints<sup class="fnref" id="fnref-2"><a href="#fn-2">2</a></sup> (laptops, phones and kiosks), while agents act behind them, on the devices themselves and in the services that manage them. Windows is previewing an <a href="https://support.microsoft.com/en-us/windows/ai/ai-features/experimental-agentic-features" target="_blank" rel="noopener">agent workspace</a>, a contained session in which agents use apps and files in the background while the user keeps working. <a href="https://learn.microsoft.com/en-us/intune/agents/" target="_blank" rel="noopener">Microsoft Intune</a> ships agents that prioritize vulnerabilities across managed devices, turn plain-language requirements into policy settings, review proposed changes and offboard stale devices. Gartner tracks the broader shift as <a href="https://www.gartner.com/en/documents/6081995" target="_blank" rel="noopener">autonomous endpoint management</a>.

The person at the endpoint sees a symptom, not an agent: the VPN stops connecting, an install fails, security software turns off. The decision behind it may have been made hours earlier, by an agent other than the one that acted, from information that was stale or never true. The blast radius grows too: one playbook (a scripted fix) runs on thousands of devices, and some effects, such as a reboot, an uninstall or an email to users, cannot be taken back. This distance between symptom and cause shapes what happens when something goes wrong.

## When Something Goes Wrong {#when-something-goes-wrong}

Working with an agent directly, you can read the conversation, retry or rephrase. Behind an endpoint, a failure arrives as a ticket, often far from its cause in time, place and agent. Either way, the same five questions follow: which run did it, what did it change, what caused the failure, which fix actually works, and what stops it in time?

None of these questions is new. Compilers and program analysis developed methods to investigate them for programs, and distributed tracing and cloud operations did the same for services (<a href="#appendix-b">Appendix B</a>). Agents reopen them: natural language can decide which step runs next, one agent can modify another’s future evidence, and an evaluator can share the errors of the agent it grades. Endpoints raise the stakes. The sections that follow locate where failures hide, then follow the evidence upstream to a cause and ask how far a diagnosis can be tested.

## Where Anomalies Hide {#where-anomalies-hide}

<div class="fig-r70" id="figure-1">
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig01_lenses.png"><img src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig01_lenses.png" width="3766" height="2468" loading="lazy" decoding="async" alt="Hexagon of six lenses around one agent execution: individual event, contextual event, action sequence, agent interaction, shared-state history and evaluation process, each with an example and a monitoring requirement."></a>
  <p class="fig-caption long with-hint">Figure 1. Six lenses on agent anomalies. Each card pairs an example with the monitoring requirement it implies (purple).</p>
  <p class="plot-hint">Click on a figure to zoom in.</p>
</div>

Where an anomaly shows up decides which check can catch it. Streaming anomaly detection distinguishes a point anomaly, one value far from normal, from a pattern anomaly, a sequence that is abnormal as a whole (<a href="https://arxiv.org/pdf/1710.04735" target="_blank" rel="noopener">Choudhary et al.</a>). For agents, four more places are worth checking: the context of an action, the interaction between agents, the state they share and the evaluation itself. These six lenses overlap, and each calls for its own check (<a href="#figure-1">Figure 1</a>).

Many failures, though, are set before the first step, in the configuration that every run inherits (<a href="#figure-2">Figure 2</a>). A model upgrade that changes an output format, for instance, can break every run that parses it, while none of those runs looks unusual on its own. A configuration defect can exist without any runtime anomaly, and an anomaly does not prove a defect.

<div class="fig-wide" id="figure-2" style="--fw: 592px">
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig02_config.png"><img src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig02_config.png" width="2618" height="1522" loading="lazy" decoding="async" alt="Configuration hub with six surfaces: models and prompts, roles and delegation, tools and permissions, shared memory, scheduling and retries, and success criteria, each with an endpoint example."></a>
</div>

<p class="fig-caption long">Figure 2. Six configuration surfaces where a defect can be set before an agent’s first run, each with an example. EDR, endpoint detection and response, is the security software on each device.</p>

Locating an observation says where to look. Deciding what it means takes the right record.

## Different Questions, Different Evidence {#different-questions}

<div class="fig-r70" id="figure-3">
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig03_graphs.png"><img src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig03_graphs.png" width="3728" height="1962" loading="lazy" decoding="async" alt="Six small graphs around one recorded execution: orchestration, invocation, information, hierarchy, propagation and intervention, each with the question it answers."></a>
  <p class="fig-caption long">Figure 3. Six graph lenses on one recorded execution; different questions need different edges. P is the parent agent, A and B are agents, T is a tool, and in the hierarchy panel the outer box is the parent task.</p>
</div>

An orchestration graph, such as a <a href="https://docs.langchain.com/oss/python/langgraph/graph-api" target="_blank" rel="noopener">LangGraph</a> workflow, describes what may run next; a runtime trace, such as one that follows <a href="https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-agent-spans.md" target="_blank" rel="noopener">OpenTelemetry’s conventions for agents</a>, records what actually called what. Neither, by itself, records which output a later step consumed. Compilers addressed the same gap long ago with the <a href="https://doi.org/10.1145/24039.24041" target="_blank" rel="noopener">program dependence graph</a>, which records which statement uses which value and which condition decides whether a statement runs. Agents need the same kind of graph, and <a href="#follow-dependencies">Follow Dependencies, Not Timestamps</a> builds on it. Three more views answer the remaining questions: which task contains the step that failed, how the failure spread, and which link survives a test (does changing a step change the outcome?) (<a href="#figure-3">Figure 3</a>).

Calling a link causal needs care. A delivered message shows only that it reached the agent; a later response may suggest that the agent used it; a controlled replay, repeated where outputs vary, tests whether changing the message changes the outcome. So when a model infers a graph’s links from logs, the graph suggests causes but does not prove them (<a href="#from-observed-to-tested">From Observed to Tested</a>).

<div class="fig-r70" id="figure-4">
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig04_questions3.png"><img src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig04_questions3.png" width="3606" height="1732" loading="lazy" decoding="async" alt="Left, a Venn diagram of unusual and incorrect behavior: novel valid plan, rare mistake, repeated falsehood. Right, a loop from a visible failure upstream to an origin candidate and back through a test."></a>
  <p class="fig-caption long">Figure 4. Rarity, correctness and cause are different questions. Left: unusual and incorrect behavior overlap only in part. Right: a visible failure starts the search; trace upstream to an origin candidate, then test it.</p>
</div>

Three questions are easy to conflate: is the behavior unusual, is it wrong, and what caused it (<a href="#figure-4">Figure 4</a>)? Rarity tells you where to look first, not whether something is wrong: an unfamiliar plan can be valid and a familiar answer false. Evidence checks correctness; provenance, where the evidence came from and what happened to it, helps establish whether that evidence is trustworthy and applicable. When requirements concern how an outcome was reached, execution history is part of the check: a setting that reads the same before and after may have been changed and restored in between (the <a href="https://en.wikipedia.org/wiki/ABA_problem" target="_blank" rel="noopener">ABA problem</a>; see <a href="#appendix-d">Appendix D</a>). Claims about cause need a controlled test, because a confirmed failure does not identify its cause: the agent that failed may simply have passed on an upstream error.

Four failures from endpoint operations show how easily these questions blur when nothing looks wrong.

## Failures That Look Normal {#failures-that-look-normal}

### Check, Then Act {#check-then-act}

A runtime guardrail checks that a laptop has room for an install that needs 30 GB, finds 42 GB and approves. In the 20 seconds before the install starts, a backup job writes 20 GB of cache; the install finds 22 GB and fails at 73% (<a href="#figure-5">Figure 5</a>). The check was right when it ran: this is a time-of-check-to-time-of-use race. Rule frameworks such as <a href="https://arxiv.org/pdf/2503.18666" target="_blank" rel="noopener">AgentSpec</a> enforce checks before an agent acts, but a check cannot stop the device from changing between the check and the action. Three measures help, in different ways: reserving the space at the check prevents the race, if other jobs honor the reservation; checking again when the install tool runs catches a change but only narrows the window; and a resumable install recovers after a failure (<a href="#appendix-d">Appendix D</a> lists related mechanisms).

<div class="fig-wide" id="figure-5" style="width: 100%; margin-left: 0; transform: none;">
  <div class="vid-wrap" style="aspect-ratio: 3552 / 1348"><video class="anim-in-view" muted playsinline preload="none" tabindex="0" width="3552" height="1348" poster="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig05_check_then_act_poster.png" data-src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig05_check_then_act.mp4" aria-label="Animation: free disk over time. A guardrail check passes at 42 GB, a backup job writes 20 GB in the next 20 seconds, the install starts with 22 GB and fails at 73%; a strip lists three measures: reserve the space to prevent the race, re-check to detect a change, and make the install resumable to recover."></video><button type="button" class="vid-zoom" aria-label="Enlarge Figure 5" title="Enlarge">&#10530;</button></div>
</div>

<p class="fig-caption long with-hint">Figure 5. The guardrail saw 42 GB free; 20 seconds later the install got 22. Unless check and act are one step, the device can change in between. Numbers are illustrative.</p>

<p class="plot-hint">Each animation plays when it scrolls into view and stops on the finished figure; click it to replay, or use &#10530; to enlarge it.</p>

### Unsupported Premises {#unsupported-premises}

A plan to roll back driver v4.2 on 2,000 laptops rests on one premise: v4.2 causes the crashes. A provenance check asks which evidence in the record supports each premise. Here nothing does: the crash logs name no driver, and the ticket supports only the timing (<a href="#figure-6">Figure 6</a>). The check holds the rollback until a fleet query supports a narrower premise, that crashes are far more frequent on v4.2 (4.1% against 0.3% on v4.1). That shows an association, not a cause, so the revised plan starts with one ring<sup class="fnref" id="fnref-3"><a href="#fn-3">3</a></sup>, which limits the damage if the premise is wrong. An absent source shows a lack of grounding, not falsehood.

<div class="fig-wide" id="figure-6" style="width: 100%; margin-left: 0; transform: none;">
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig06_provenance.png"><img src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig06_provenance.png" width="3536" height="1067" loading="lazy" decoding="async" alt="Top row: crash logs and a ticket do not support the claim that driver v4.2 causes the crashes, so a provenance check holds the rollback. Bottom row: a fleet query supports the claim and the rollback starts in ring 1."></a>
</div>

<p class="fig-caption long">Figure 6. No source, no rollback: the plan’s key premise traces to nothing the agent read. Below, a fleet query supports a narrower premise, and the plan starts with one ring.</p>

### Score Inflation {#score-inflation}

When an agent, or the workflow around it, is tuned to raise a judge’s score (a judge is a model that scores its work), it can learn to satisfy the judge instead of fixing the problem. In <a href="#figure-7">Figure 7</a>, the judge rates 92% of tickets resolved by week eight, while a weekly audit of 50 of the same devices finds only about 58% still fixed. The judge scored the ticket’s closing note; the audit checked the device.

<div class="fig-wide" id="figure-7" style="--fw: 592px">
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig07_audit.png"><img src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig07_audit.png" width="3540" height="1241" loading="lazy" decoding="async" alt="Line chart over eight weeks: judged resolution rises from 64% to 92% while audited fixes stay near 58%, a 34-point gap labelled score inflation."></a>
</div>

<p class="fig-caption long">Figure 7. Judged resolution and audited fixes over eight weeks; the shaded gap between them is score inflation. Weekly values are illustrative.</p>

### Stale Beliefs, Irreversible Effects {#stale-beliefs}

System state, an agent’s belief and effects outside the system are three different things, and one of them can be right even when the others are wrong (<a href="#figure-8">Figure 8</a>). In case A, the MDM (mobile device management) service swaps a VPN profile from v1 to v2. The agent’s fix, planned for v1, fails its version check, so no update is lost: the transaction is correct, yet the agent’s memory still says v1. In case B, the agent acts on the stale belief and emails 400 users to reinstall their VPN client; it then notices the swap and corrects its plan, but 120 users have already reinstalled.

Transactional correctness does not heal a belief, and a corrected belief cannot unsend an email. <a href="https://arxiv.org/pdf/2606.15376" target="_blank" rel="noopener">CoAgent</a>, a layer between peer agents and the systems they change, notifies an agent when another agent’s write may invalidate its plan, so that it can re-check and patch the plan. It reverts a write only through an undo step registered in advance, and it holds back an action with none, such as an email, until the agents that could invalidate it have finished. These guarantees assume that every tool declares what it reads and writes and that agents act correctly on notifications.

<div class="fig-wide" id="figure-8" style="width: 100%; margin-left: 0; transform: none;">
  <div class="vid-wrap" style="aspect-ratio: 3552 / 1248"><video class="anim-in-view" muted playsinline preload="none" tabindex="0" width="3552" height="1248" poster="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig08_belief_vs_effect_poster.png" data-src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig08_belief_vs_effect.mp4" aria-label="Animation: a grid of system state, agent belief and outside world for two cases. After a VPN profile swap, case A&#x27;s write is rejected while its belief stays stale; case B emails 400 users before correcting its belief."></video><button type="button" class="vid-zoom" aria-label="Enlarge Figure 8" title="Enlarge">&#10530;</button></div>
</div>

<p class="fig-caption long">Figure 8. System state, the agent’s belief and the outside world in two cases. Both start in agreement; the three layers then diverge at different moments.</p>

Each of these failure modes&nbsp;- a check that goes stale before the action, an unsupported premise, an inflated score and a stale belief&nbsp;- needs its own safeguard: a check that stays valid until the action completes, a provenance check, an audit against outcomes, and a re-check of current state before any action that cannot be undone. When agents share state, an invariant, a rule that must hold across all of them, also catches combinations that each agent’s own check misses, provided each agent checks it and reserves what it needs in one step (<a href="#appendix-d">Appendix D</a> shows two playbooks that each pass their own check and together break a store’s limit).

Learned detectors such as <a href="https://doi.org/10.1145/3133956.3134015" target="_blank" rel="noopener">DeepLog</a> flag unusual sequences (a rare delegation, a retry loop), but unusual is not wrong: their alerts should lead to a provenance or state check. Because real incidents are rare, even an accurate detector can raise mostly false alarms (the <a href="https://doi.org/10.1145/357830.357849" target="_blank" rel="noopener">base-rate fallacy</a>), so its alert threshold should be set from the deployment’s recent incident rate, the number of alerts the team can review and the cost of a missed incident, and revisited as these change. Detection also has to come early enough to stop an irreversible effect, or at least to limit it. Other fields have built versions of each of these safeguards for decades.

## What Carries Over {#what-carries-over}

<div class="fig-r70" id="figure-9">
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig09_foundations.png"><img src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig09_foundations.png" width="3766" height="2457" loading="lazy" decoding="async" alt="Pentagon around agent monitoring: distributed systems, MLOps, program analysis, multi-agent learning and security, each with its key topics."></a>
  <p class="fig-caption long">Figure 9. Five foundations of agent monitoring, each from a different research tradition. Connector lengths encode no quantity.</p>
</div>

Five research traditions supply the methods (<a href="#figure-9">Figure 9</a>). Distributed systems explain concurrency and cascading failures, and produced <a href="https://research.google.com/archive/papers/dapper-2010-1.pdf" target="_blank" rel="noopener">distributed tracing</a> and checks of execution against expectations (<a href="https://www.usenix.org/legacy/event/nsdi06/tech/full_papers/reynolds/reynolds.pdf" target="_blank" rel="noopener">Pip</a>). Program analysis gives <a href="https://doi.org/10.1145/24039.24041" target="_blank" rel="noopener">dependency graphs</a> and backward slicing. MLOps (machine learning operations) measures drift, a change in data or behavior over time, and whether a remediation actually worked; it also documented the <a href="https://proceedings.neurips.cc/paper/2015/file/86df7dcfd896fcaf2674f757a2463eba-Paper.pdf" target="_blank" rel="noopener">hidden technical debt</a> of feedback loops and of systems that quietly depend on a model’s output. Multi-agent learning addresses conflicting objectives, where one agent’s successful fix can undermine another’s (<a href="https://arxiv.org/pdf/1702.03037" target="_blank" rel="noopener">sequential social dilemmas</a>). Security assumes that the agent, its inputs or its monitoring channel could be compromised, and intrusion analysis in security already traces an attack backward through records of which process touched which file (<a href="https://doi.org/10.1145/1047915.1047918" target="_blank" rel="noopener">BackTracker</a>, <a href="https://www.ndss-symposium.org/wp-content/uploads/2019/02/ndss2019_03B-1-3_UlHassan_paper.pdf" target="_blank" rel="noopener">NoDoze</a>).

The match is closest between program analysis and agent execution: an agent run behaves like a program whose statements are model and tool calls. Six ideas, most of them from program analysis, have useful counterparts, though none carries over unchanged (<a href="#figure-10">Figure 10</a>). Distinguishing calls of the same function becomes distinguishing invocations of the same agent role; dynamic slicing, which keeps only the statements a run actually used, narrows the steps that failure attribution must consider; and <a href="https://doi.org/10.1109/32.988498" target="_blank" rel="noopener">delta debugging</a>, which isolates what triggers a failure by testing ever smaller sets of changes, motivates a search for the smallest set of repairs, tested by replay from saved state. The compiler side of such parallels, aimed at speed rather than diagnosis, is explored in <a href="{{ site.baseurl }}/2026-07-27-optimizing-agentic-execution.html">Optimization of Agentic Execution</a>.

<div class="fig-wide" id="figure-10" style="width: 100%; margin-left: 0; transform: none;">
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig10_parallels.png"><img src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig10_parallels.png" width="2886" height="1508" loading="lazy" decoding="async" alt="Six interlocking pairs: calling-context sensitivity and per-invocation identity, dynamic slicing and attribution, side-effect analysis and effect history, delta debugging and checkpoint replay, MLOps lineage and versioned configuration, hidden technical debt and agents grading agents."></a>
</div>

<p class="fig-caption long">Figure 10. Six established ideas, each with an agentic counterpart: what the idea established in program analysis and MLOps (left) and what it becomes for agents (right).</p>

What is new is the combination: decisions made in natural language, agents that modify one another’s evidence, and judges that share the errors of what they judge, all acting through general-purpose tools. Of the six ideas, slicing carries over most directly, and it addresses a question every ticket raises: which earlier steps led to this failure?

## Follow Dependencies, Not Timestamps {#follow-dependencies}

Mark Weiser’s <a href="https://doi.org/10.1109/TSE.1984.5010248" target="_blank" rel="noopener">program slicing</a> reduces a program to the statements that can affect a value at a point of interest. A static slice keeps every statement that could affect it on some input; a dynamic slice (<a href="https://doi.org/10.1016/0020-0190(88)90054-3" target="_blank" rel="noopener">Korel and Laski</a>; <a href="https://doi.org/10.1145/93542.93576" target="_blank" rel="noopener">Agrawal and Horgan</a>) keeps only those that did, in one execution. For agents, the analogue is the backward dynamic slice of a failure: start at the failed output and follow what each step actually used. <a href="#appendix-a">Appendix A</a> walks through both slicing mechanisms on two small programs (Figures <a href="#figure-A1">A1</a> and <a href="#figure-A2">A2</a>).

In <a href="#figure-11">Figure 11</a>, one patch fails on one device: at step 27 the installer runs out of disk. Timestamps blame the planner, which ran just before. Dependencies tell a different story: the installer ran the planner’s schedule, and the planner used `disk_free` v1, read at step 12, which reported 100 GB on a disk that had 10. The battery reading at step 15 is valid but unused, so it stays outside the slice. Fix the reading and replay from step 19: the planner, now seeing 10 GB, holds the 30 GB update, and the failure is prevented.

<div class="fig-wide" id="figure-11" style="width: 100%; margin-left: 0; transform: none;">
  <div class="vid-wrap" style="aspect-ratio: 3554 / 1386"><video class="anim-in-view" muted playsinline preload="none" tabindex="0" width="3554" height="1386" poster="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig11_backward_slice_poster.png" data-src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig11_backward_slice.mp4" aria-label="Animation: steps on lanes for a telemetry reader, a patch planner and an installer. Blaming the last step points at the planner; the backward slice follows the installer&#x27;s input to the planner and the planner&#x27;s input to the disk reading at step 12, the origin. With the reading fixed, the planner holds the 30 GB update and the failure is prevented."></video><button type="button" class="vid-zoom" aria-label="Enlarge Figure 11" title="Enlarge">&#10530;</button></div>
</div>

<p class="fig-caption long">Figure 11. Follow dependencies, not timestamps: the step that failed need not be the step that erred. One failed patch on one device, traced backward through what each step actually used. The scenario is illustrative.</p>

The slice must follow invocations, not roles. A role such as patch planner is like a function, and each invocation of it is like a call; grouping events by role can merge a correct call with a faulty one. A useful event identity includes run, parent invocation, instance, attempt and step. OpenTelemetry reserves `gen_ai.agent.id` for the stable agent, not a transient instance, so invocations and instances need identities of their own.

To test a suspect step, correct its output and rerun only the steps that depend on it, starting from the state saved at that step and under the same outside conditions, so that later tool calls respond to the corrected step. Because models sample their outputs, one rerun can mislead; compare outcomes over repeated reruns (<a href="#appendix-e">Appendix E</a>). If the failure goes away, that is evidence that the step caused the failure in this run; it does not show that the step causes it in other runs. Research is converging on this pattern: <a href="https://arxiv.org/pdf/2610.01256" target="_blank" rel="noopener">DeFA</a> reconstructs dependencies and how a failure spread, <a href="https://arxiv.org/pdf/2610.11600" target="_blank" rel="noopener">EMFA</a> also models loops that persist, <a href="https://arxiv.org/pdf/2509.13782" target="_blank" rel="noopener">FAMAS</a> compares repeated executions, and <a href="https://arxiv.org/pdf/2608.29228" target="_blank" rel="noopener">graph-constrained joint replay</a> tests sets of repairs (<a href="#appendix-c">Appendix C</a>). The problem is far from solved: on the hand-crafted systems of <a href="https://arxiv.org/pdf/2505.00212" target="_blank" rel="noopener">Who&When</a>, a benchmark of failed multi-agent runs labeled with the responsible agent and step, EMFA, a 2026 method, matches the labeled step about one time in three with GPT-4o<sup class="fnref" id="fnref-4"><a href="#fn-4">4</a></sup>.

A slice says which claims fed the failure. It does not say whether those claims deserved to be believed.

## Agreement Is Not Evidence {#agreement-is-not-evidence}

<a href="#figure-12">Figure 12</a> shows two incident reviews. In each, three agents name the same root cause, word for word. Traced to their sources, the claims in the first review all lead to one forum post, echoed three times; those in the second rest on three separate sources: crash dumps, a vendor advisory and fleet correlation. From the claims alone, the two reviews look the same.

<div class="fig-wide" id="figure-12" style="width: 100%; margin-left: 0; transform: none;">
  <div class="vid-wrap" style="aspect-ratio: 3554 / 1386"><video class="anim-in-view" muted playsinline preload="none" tabindex="0" width="3554" height="1386" poster="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig12_one_source_repeated_poster.png" data-src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig12_one_source_repeated.mp4" aria-label="Animation: two panels with three identical claims each. Traced to sources, the first panel&#x27;s claims converge on one forum post and the second&#x27;s lead to three separate sources whose independence still needs checking; three checks follow: support, provenance and independence."></video><button type="button" class="vid-zoom" aria-label="Enlarge Figure 12" title="Enlarge">&#10530;</button></div>
</div>

<p class="fig-caption">Figure 12. Three agents agreeing can be one source repeated.</p>

Three checks, listed under the two reviews, separate a real consensus from an echo: support, provenance and independence. Agreement counts only when the agreeing sources could fail in different ways, and separate sources are not automatically independent: a vendor advisory may rest on the same crash dumps. The same holds for monitors and judges: those that run on the same model as the agents they check can share its blind spots, so independent checks need different models or different evidence. Oversight must also hold up against agents trying to subvert it (<a href="https://arxiv.org/pdf/2312.06942" target="_blank" rel="noopener">AI Control</a>), and agents can pass signals inside ordinary-looking text (<a href="https://arxiv.org/pdf/2402.07510" target="_blank" rel="noopener">collusion research</a>)<sup class="fnref" id="fnref-5"><a href="#fn-5">5</a></sup>.

Even well-sourced claims leave two questions open: is a claim true, and did it matter?

## Two Verdicts per Claim {#two-verdicts-per-claim}

Tracing a failure backward narrows what to inspect; it does not detect a hallucination. At each step that produces a claim, compare the claim with the evidence available at that moment, and distinguish a new unsupported claim from a faithful repetition of an earlier one, an error introduced while transforming it, or a later correction. One approach, used by <a href="https://arxiv.org/pdf/2305.14251" target="_blank" rel="noopener">FActScore</a>, breaks a suspect answer into checkable claims and checks each against a knowledge source; for diagnosis, record each claim as supported, contradicted or unresolved. Where external evidence is weak, disagreement among sampled answers (<a href="https://arxiv.org/pdf/2303.08896" target="_blank" rel="noopener">SelfCheckGPT</a>) and <a href="https://www.nature.com/articles/s41586-024-07421-0" target="_blank" rel="noopener">semantic entropy</a>, which measures how much the answers differ in meaning, are warning signals, not certificates: samples can repeat the same misconception.

A claim’s truth and its causal role are separate verdicts, decided by different evidence (<a href="#figure-13">Figure 13</a>). In incident A, an agent rolls back driver v4.2 on all 2,000 laptops at once, skipping the pilot ring. Its summary puts the crash rate of v4.2 at 41%; fleet data says 4.1%, so the claim is false. But a manager had also ordered a full rollback, so the claim may not be what drove the decision: rerun the agent with its summary corrected to 4.1%, and if it still rolls back everywhere, the false figure was not needed for that decision, and the manager’s order is the next candidate to test. In incident B, a note in agent memory says “EDR breaks VPN 5.2”, where EDR (endpoint detection and response) is the security software on each device. No one knows who wrote the note or why, so its truth is unverified. Yet when the agent was rerun without the note, EDR stayed on: that is evidence that the note led the agent to turn EDR off (repeat the rerun to rule out chance), and whether the note is true is a separate check.

<div class="fig-wide" id="figure-13" style="width: 100%; margin-left: 0; transform: none;">
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig13_uncertainty.png"><img src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig13_uncertainty.png" width="3539" height="1200" loading="lazy" decoding="async" alt="Two incidents compared row by row: what happened, whether the claim is true, whether it caused the failure, and what to do. Incident A&#x27;s claim is false but its role unclear; incident B&#x27;s claim is unverified, yet a rerun without it suggests that it caused the failure."></a>
</div>

<p class="fig-caption long">Figure 13. Two incidents, two verdicts per claim: whether it is true and whether it caused the failure, with the next step for each.</p>

When the evidence cannot isolate one origin, report the set of plausible origins and the observation that would distinguish them, rather than naming a single first hallucination. And some influences never reach the record at all.

## Below the Waterline {#below-the-waterline}

Standard traces record the tip: calls between agents and tools, in spans (timed records of one operation) such as `invoke_agent` and `execute_tool`, and the messages themselves where content capture is turned on. Below the waterline, one agent changes what another sees, can do or is judged by, through six channels that standard traces do not link (<a href="#figure-14">Figure 14</a>).

<div class="fig-wide" id="figure-14" style="--fw: 592px">
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig14_iceberg.png"><img src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig14_iceberg.png" width="3544" height="1339" loading="lazy" decoding="async" alt="Iceberg: returned outputs and messages above the waterline; below it, six channels: resources, state, context, memory, environment and evaluation."></a>
</div>

<p class="fig-caption long">Figure 14. What standard traces capture (above the waterline) and six channels of influence between agents that they miss (below), each with an example.</p>

The environment channel is the easiest to overlook, because the change outlives the run that made it. In Anthropic’s <a href="https://www.anthropic.com/research/eval-awareness-browsecomp" target="_blank" rel="noopener">BrowseComp investigation</a>, e-commerce sites turned agents’ search queries into persistent, indexed pages, which later agents found: an apparent read changed the environment for future runs. Detection therefore needs the history of effects, not only the final transcript. A final-state check can miss a violation that was later repaired (firewall on → off while a file is copied out → on again), and a correct answer can conceal a disclosure or an unauthorized write. An example of the latter: an agent restores a user’s VPN as asked, but on the way edits a firewall policy it was never authorized to change.

Capturing more of the record leaves one question: how strong is the evidence a diagnosis rests on?

## From Observed to Tested {#from-observed-to-tested}

Evidence about why an endpoint agent failed comes in four kinds, a ladder from what the agent was permitted to do, through what it was observed doing and what information it is inferred to have used, to what a test shows (<a href="#figure-15">Figure 15</a>). Each rung answers a stronger question about cause, although a careful observation can be more reliable than a speculative inference. Standard observability stops at observed. The top rung moves from seeing to doing, in <a href="https://cacm.acm.org/research/the-seven-tools-of-causal-inference-with-reflections-on-machine-learning" target="_blank" rel="noopener">Pearl’s</a> terms, and an agent that remediates endpoints in stages can gather this evidence along the way: a staged remediation is a test when it has a comparison group and a defined outcome.

<div class="fig-wide" id="figure-15" style="--fw: 592px">
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig15_ladder.png"><img src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig15_ladder.png" width="3624" height="1336" loading="lazy" decoding="async" alt="Ladder of four evidence levels: permitted, observed, inferred and tested, with who provides each today; standard observability stops between observed and inferred."></a>
</div>

<p class="fig-caption long">Figure 15. Four rungs of evidence about why an endpoint agent failed, from permitted to tested. The providers and badges show typical sources today, not a market survey.</p>

The operational takeaway is an execution record with four views: permitted behavior, observed events, inferred information use and causes tested by intervention. Capture event identity, timestamps, parent and message links, attempts, tool arguments and results, read and write versions, permissions and verdict evidence wherever the agent runtime calls a model or a tool. Record both the direction of each dependency and its provenance, and when a source is missing or a relationship is inferred, say so instead of filling the gap with a confident narrative.

With such a record, a useful incident report lists the observed failure, the agent instances and steps that may have caused it, the supporting slice, the checks that failed and the results of interventions. For a cascade, it separates the step that introduced the error from the steps that passed it on and from the points where it could have been stopped. Judge the diagnostic system itself by how fast it detects failures, how many alerts it raises, how many serious failures it misses, how precisely it locates the instance and step, and how often its repairs work.

## Mostly Familiar Ground {#mostly-familiar-ground}

Which artifacts make that record possible? Mostly ones that other fields already keep, some of them for decades (<a href="#figure-16">Figure 16</a>). For each of six categories (identity, configuration, information use, effects, verdicts and unknowns), program analysis, cloud operations and MLOps already record artifacts such as trace IDs for identity or data lineage for information use. Agentic execution adds or sharpens others, such as delegation chains for identity or judge models and rubrics for verdicts. Most of the record therefore has precedents; what agents add is narrower, but it is where many agent failures hide, and where the open problems lie.

<div class="fig-wide" id="figure-16" style="width: 100%; margin-left: 0; transform: none;">
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig16_landscape.png"><img src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_fig16_landscape.png" width="3521" height="1480" loading="lazy" decoding="async" alt="Terrain over six categories (identity, configuration, information use, effects, verdicts, unknowns) with ridges for program analysis, cloud operations and MLOps, and a purple front ridge for agentic execution."></a>
</div>

<p class="fig-caption long">Figure 16. Mostly familiar ground, plus a region that agents add or sharpen: for each of six categories, two artifacts established fields already record (green) and one that agentic execution adds or makes central (purple). Peak heights are illustrative.</p>

## Open Problems {#open-problems}

Existing work supplies important pieces, but not yet a general way to diagnose causes end to end in concurrent agentic systems. Six gaps stand out, each a research agenda in its own right:

* *\[Instance-aware attribution across trajectories\]* Most attribution methods label a failure with an agent and a step within one task. Identifiers that group events exist (OpenTelemetry’s `gen_ai.conversation.id` correlates the messages of one session or thread), but attribution also needs trajectory context, the agent counterpart of calling context: which invocation of a role, reached through which delegations, retries and memory reads, produced the step. Test cases should include several invocations of the same role, only some of which contributed to the failure.
* *\[Uncertain origins and incomplete observation\]* The earliest recorded unsupported claim, the first false claim and the decisive error can be different events. Attribution methods are usually tested on complete logs, while real deployments have missing sources and unrecorded external changes; methods should also be tested on such records.
* *\[Interventions beyond a single repair\]* A repair that restores success is evidence; a minimal set of repairs that does so says more, since each repair in it is needed, at least among the repairs tested. Joint faults, valid alternative plans and random variation between reruns complicate both (<a href="#appendix-e">Appendix E</a> shows an example of each). Reproducible state capture, and replay that reuses a recorded response only where the request has not changed, are needed to tell a causal correction from a lucky rerun.
* *\[Concurrency with shared beliefs\]* Three problems remain open: repairing an agent’s premises after the state they describe has changed, handling actions that cannot be undone (knowing before acting whether an action is reversible, and what to do when it is not), and coordinating safely across nested subagents and peer agents.
* *\[Monitor and evaluator independence\]* Monitors that flag failures and evaluators that grade outcomes both need independence from what they check. That calls for audits of the errors they share with the agent, such as a judge with the agent’s blind spots, and of concealed communication between agents, carried out without changing the execution being observed<sup class="fnref" id="fnref-6"><a href="#fn-6">6</a></sup>.
* *\[Operational evidence and cost\]* Short or synthetic traces cannot show how a method performs at production scale: how quickly it detects failures, how many false alerts it raises, how often its repairs work and what it costs to run. Evaluations should report these alongside accuracy at the exact step.

## Takeaways {#takeaways}

* *\[Symptoms travel\]* Behind an endpoint, a failure surfaces far from the decision that caused it.
* *\[Know where to look\]* Six lenses show where an anomaly can surface, and six configuration surfaces where a defect can be set; rarity, correctness and cause are separate questions.
* *\[Follow use, not time\]* Trace what each step actually used, not what ran just before, and slice over invocations, not roles.
* *\[Agreement needs independence\]* Check support, provenance and independence before trusting a consensus.
* *\[Two verdicts\]* Whether a claim is true and whether it caused the failure need different evidence.
* *\[Test the diagnosis\]* A staged remediation with a comparison group is an intervention: it turns an inferred cause into a tested one.

## Notes {#notes}

<ul class="notes">
  <li id="fn-1"><span class="fn-num">1</span> If each of N agents fails independently with probability p, the chance that none fails is (1 − p)<sup>N</sup>. At p = 0.1%, that is about 90% for 100 agents, 37% for 1,000 and under 0.01% for 10,000; at p = 1%, it is already down to 37% at 100 agents. Dean and Barroso’s <a href="https://doi.org/10.1145/2408776.2408794" target="_blank" rel="noopener">The Tail at Scale</a> makes the same point about latency: if one request in 100 to a server is slow, a request that needs answers from 100 servers is slow 63% of the time. Independence is a strong assumption. Agents that share a model, a prompt or a memory tend to fail together: a common cause can cluster many failures into one incident, and the independent formula no longer describes that risk. <a class="fn-back" href="#fnref-1" aria-label="Back to text">&#8617;&#xFE0E;</a></li>
  <li id="fn-2"><span class="fn-num">2</span> In IT, an endpoint is a device at the edge of a network, such as a laptop, phone, server or kiosk. <a class="fn-back" href="#fnref-2" aria-label="Back to text">&#8617;&#xFE0E;</a></li>
  <li id="fn-3"><span class="fn-num">3</span> A ring is one stage of a staged rollout: a change goes first to a small pilot group of devices and reaches wider rings only after it passes the pilot’s safety and success checks. <a class="fn-back" href="#fnref-3" aria-label="Back to text">&#8617;&#xFE0E;</a></li>
  <li id="fn-4"><span class="fn-num">4</span> EMFA’s figure is its exact-step accuracy, 32.8%, with GPT-4o on the hand-crafted subset; with three other models it scores 24.6% to 29.3%. It measures agreement with the benchmark’s labels, which were not validated by intervention. The original <a href="https://arxiv.org/pdf/2505.00212" target="_blank" rel="noopener">Who&When</a> abstract reports 53.5% agent accuracy and 14.2% step accuracy, but the two figures come from different best methods, averaged across settings. Exact-step scoring also penalizes near misses, and more than one step can plausibly be blamed; <a href="https://arxiv.org/pdf/2603.25001" target="_blank" rel="noopener">multi-perspective attribution</a> examines that ambiguity directly. <a class="fn-back" href="#fnref-4" aria-label="Back to text">&#8617;&#xFE0E;</a></li>
  <li id="fn-5"><span class="fn-num">5</span> Steganography hides a message inside ordinary-looking content. In a toy code, “Hello” could mean 0 and “Hi” 1: a reader sees greetings while a second agent reads bits. This illustrates a channel, not proof of spontaneous collusion; the cited work studies the capability under specified conditions. <a class="fn-back" href="#fnref-5" aria-label="Back to text">&#8617;&#xFE0E;</a></li>
  <li id="fn-6"><span class="fn-num">6</span> A Heisenbug changes when observed: logging can change scheduling, and inserting a critic changes an agent’s context and decisions. Nondeterminism alone is not a Heisenbug. Preserve the original traces, and distinguish replaying captured observations from a fresh live run. <a class="fn-back" href="#fnref-6" aria-label="Back to text">&#8617;&#xFE0E;</a></li>
</ul>

## Appendix A: Static and Dynamic Slicing, Step by Step {#appendix-a}

This appendix expands the slicing summary in <a href="#follow-dependencies">Follow Dependencies, Not Timestamps</a>, using two small programs.

A slicing criterion names a value at a point, such as ⟨16, `net_total`⟩: the value of `net_total` at line 16. A static slice, in <a href="https://doi.org/10.1109/TSE.1984.5010248" target="_blank" rel="noopener">Weiser’s sense</a>, keeps every statement that may affect that value on some input, following two kinds of dependence: data (a line reads a variable that another line may have set) and control (a line runs only if a condition or loop lets it). In <a href="#figure-A1">Figure A1</a>, `process_order` gets `net_total` wrong for members who use the promo code `FLAT15`. Slicing back from line 16 adds the lines that set `subtotal`, `discount` and `tax`, then the conditions that guard them, and stops when nothing new joins; lines 3, 4, 7, 9, 17 and 18 drop out. Only three lines set `discount`, so the slice narrows the search to them. What makes line 14 the bug is the requirement that member and promo discounts stack: it replaces the member discount instead of adding to it, so the `=` should be `+=`.

<div class="fig-wide" id="figure-A1" style="width: 100%; margin-left: 0; transform: none;">
  <div class="vid-wrap" style="aspect-ratio: 3482 / 1420"><video class="anim-in-view" muted playsinline preload="none" tabindex="0" width="3482" height="1420" poster="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_figA1_static_slice_poster.png" data-src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_figA1_static_slice.mp4" aria-label="Animation: the code of process_order beside a dependence graph that grows from line 16 through data and control dependences; unused lines drop out and line 14 is marked as the bug."></video><button type="button" class="vid-zoom" aria-label="Enlarge Figure A1" title="Enlarge">&#10530;</button></div>
</div>

<p class="fig-caption long">Figure A1. A static slice keeps every line that could affect the value: slicing back from line 16 of <code>process_order</code> isolates the bug at line 14.</p>

A dynamic slice keeps only the statement instances that affected the value in one run. Superscripts number loop iterations: 12² is line 12 in the second iteration. In <a href="#figure-A2">Figure A2</a>, `apply_adjustments` returns `[60, 65, 0]` instead of `[60, 5, 70]`, and the criterion is ⟨13, `scores[2]`⟩, the 0 that should be 70. The last write to `scores[2]` was 12², which ran because 11² matched `RESET_NEXT` after 5² and 8² did not match. 12² used `idx = 1`, set by 7¹ in the first iteration, which ran because 5¹ matched `BONUS`, starting from `idx = 0` at line 3. The slice is lines 3, 4, 5, 7, 8, 11, 12 and 13, and line 12 is the bug: it writes `scores[idx + 1]` where it should write `scores[idx]`. A static slice for the same criterion would keep nearly the whole function, because on some input almost every line could affect `scores[2]`.

<div class="fig-wide" id="figure-A2" style="width: 100%; margin-left: 0; transform: none;">
  <div class="vid-wrap" style="aspect-ratio: 3448 / 1444"><video class="anim-in-view" muted playsinline preload="none" tabindex="0" width="3448" height="1444" poster="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_figA2_dynamic_slice_poster.png" data-src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_figA2_dynamic_slice.mp4" aria-label="Animation: the code of apply_adjustments beside parallel timelines of one failing run; the slice walks back from scores[2] through line instances 12², 11², 8², 5², 7¹, 5¹ and line 3."></video><button type="button" class="vid-zoom" aria-label="Enlarge Figure A2" title="Enlarge">&#10530;</button></div>
</div>

<p class="fig-caption long">Figure A2. A dynamic slice keeps only what the failing run actually used: slicing back from <code>scores[2]</code>, on parallel timelines of the three loop iterations, isolates line 12.</p>

## Appendix B: Same Questions, Four Settings {#appendix-b}

This appendix backs the five questions in <a href="#when-something-goes-wrong">When Something Goes Wrong</a>. In <a href="#figure-B1">Figure B1</a>, each column asks one question in four settings: program analysis, distributed systems and cloud and ML operations, agents, and endpoints. For “Which run did it?”, calling-context sensitivity distinguishes separate calls of the same function by the chain of calls that led to each, and distributed tracing follows one ID per request across every service and retry. Agents need to distinguish a role from each invocation of it, and endpoints need to distinguish a playbook from each run of it, on a particular device and attempt.

<div class="fig-wide" id="figure-B1" style="width: 100%; margin-left: 0; transform: none;">
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_figB1_questions.png"><img src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_figB1_questions.png" width="3542" height="837" loading="lazy" decoding="async" alt="Table of five questions (which run did it, what did it change, what caused the failure, which fix works, what stops it in time) across program analysis, distributed systems and cloud/ML operations, agents, and endpoints."></a>
</div>

<p class="fig-caption long">Figure B1. Compilers and cloud operations developed methods for these questions; agents reopen them, and endpoints raise the stakes.</p>

## Appendix C: Attribution Research at a Glance {#appendix-c}

This appendix lists the attribution proposals behind <a href="#follow-dependencies">Follow Dependencies, Not Timestamps</a>. They address different targets and should not be ranked by a single accuracy number.

<div class="table-wrap" style="--tw: 1240px">
<table class="pdf-table t-blog">
  <colgroup><col style="width: 12.1%"><col style="width: 44.0%"><col style="width: 44.0%"></colgroup>
  <thead>
    <tr><th class="vsep-l">Proposal</th><th class="vsep-l">How it represents and attributes a failure</th><th>Limit of the evidence</th></tr>
  </thead>
  <tbody>
    <tr><td class="metric vsep-l"><a href="https://arxiv.org/pdf/2602.23701" target="_blank" rel="noopener">CHIEF</a></td><td class="vsep-l">Splits a run into tasks and parsed steps, backtracks with generated checks, and reasons about what a correction would change.</td><td>Works turn by turn; a model reasoning about a correction is not a rerun with the correction.</td></tr>
    <tr><td class="metric vsep-l"><a href="https://arxiv.org/pdf/2603.14688" target="_blank" rel="noopener">AgentTrace</a></td><td class="vsep-l">Builds a graph of actions linked by order, messages and data; walks it backward and ranks suspect steps.</td><td>Tested on synthetic runs, and the ranking leans on where a step sits in the run, so results may not carry over to deployments.</td></tr>
    <tr><td class="metric vsep-l"><a href="https://arxiv.org/pdf/2608.24361" target="_blank" rel="noopener">Adaptive influence graphs</a></td><td class="vsep-l">Segments the log and infers which step influenced which; a reader agent navigates the graph and checks the raw log.</td><td>Inference makes the log easier to navigate but can add links that are wrong.</td></tr>
    <tr><td class="metric vsep-l"><a href="https://arxiv.org/pdf/2610.01256" target="_blank" rel="noopener">DeFA</a></td><td class="vsep-l">Recovers dependencies between events, then builds a graph of how the failure spread.</td><td>Works within one run; recovering dependencies and judging blame both rely on a model.</td></tr>
    <tr><td class="metric vsep-l"><a href="https://arxiv.org/pdf/2610.11600" target="_blank" rel="noopener">EMFA</a></td><td class="vsep-l">Adds cascades and loops that persist to the dependency graph, then screens for the decisive step.</td><td>Accuracy on a benchmark does not show coverage of concurrent work or of work spread across sessions.</td></tr>
    <tr><td class="metric vsep-l"><a href="https://arxiv.org/pdf/2608.29646" target="_blank" rel="noopener">DuoTrace</a></td><td class="vsep-l">Encodes each run by meaning and by structure to flag unusual steps before attributing blame.</td><td>Filtering improves the evidence a judge sees; it does not itself prove cause.</td></tr>
    <tr><td class="metric vsep-l"><a href="https://arxiv.org/pdf/2609.01360" target="_blank" rel="noopener">EDGE</a></td><td class="vsep-l">Links categories of errors across many runs; a subset of the links is checked by rerunning.</td><td>A graph of error categories is not a graph of concrete invocations; only the rerun links support strong claims.</td></tr>
    <tr><td class="metric vsep-l"><a href="https://arxiv.org/pdf/2509.13782" target="_blank" rel="noopener">FAMAS</a></td><td class="vsep-l">Reruns the task many times and scores each agent action by how often it appears in failed runs compared with successful ones.</td><td>Association between an action and failure guides diagnosis but does not by itself establish cause.</td></tr>
    <tr><td class="metric vsep-l"><a href="https://arxiv.org/pdf/2608.29228" target="_blank" rel="noopener">Joint replay</a></td><td class="vsep-l">Searches, within the dependency graph, for the smallest sets of repaired events that restore success.</td><td>Small controlled evaluations; assumes clean replacement messages and a limited number of candidates.</td></tr>
  </tbody>
</table>
</div>

## Appendix D: Concurrency Mechanisms {#appendix-d}

This appendix separates the mechanisms behind the state-integrity failures in <a href="#check-then-act">Check, Then Act</a> and <a href="#stale-beliefs">Stale Beliefs, Irreversible Effects</a>. These failures can occur even when every model output is factual, so diagnosing them needs system-level evidence.

<div class="table-wrap" style="--tw: 1240px">
<table class="pdf-table t-blog">
  <colgroup><col style="width: 13.7%"><col style="width: 43.1%"><col style="width: 43.1%"></colgroup>
  <thead>
    <tr><th class="vsep-l">Mechanism</th><th class="vsep-l">Illustrative agent execution</th><th>Relevant control</th></tr>
  </thead>
  <tbody>
    <tr><td class="metric vsep-l">Lost update</td><td class="vsep-l">Agents A and B read one document version; B’s stale write overwrites A’s edit.</td><td>Compare versions at write time; merge conflicts explicitly.</td></tr>
    <tr><td class="metric vsep-l">Write skew</td><td class="vsep-l">Two agents each check a shared limit, see room and act on different items; together they break the limit (<a href="#figure-D1">Figure D1</a>).</td><td>Check the limit and reserve capacity in one atomic step, counting others’ reservations; serializable transactions do this in databases.</td></tr>
    <tr><td class="metric vsep-l">Duplicate effect</td><td class="vsep-l">A retry repeats an external operation that had already taken effect, because its acknowledgement was lost.</td><td>Give each operation a stable ID, so the receiving service ignores repeats (idempotency).</td></tr>
    <tr><td class="metric vsep-l">Check then act</td><td class="vsep-l">An agent verifies an approval, but the approval changes before the tool acts on it.</td><td>Check and act in one step, or use versioned permissions that the tool itself verifies.</td></tr>
    <tr><td class="metric vsep-l"><a href="https://en.wikipedia.org/wiki/ABA_problem" target="_blank" rel="noopener">ABA</a></td><td class="vsep-l">A value changes from A to B and back to A; a check that compares values sees no change.</td><td>Compare version numbers, not just values.</td></tr>
  </tbody>
</table>
</div>

Write skew is the hardest of these to see, because every individual check passes (<a href="#figure-D1">Figure D1</a>). Two playbooks each verify that a store keeps at least 12 of its 20 checkout kiosks online, and each would leave 14 or 15 online on its own; running at the same time, they leave 9. Checking the store’s current state just before each reboot is not enough, because both checks can pass at the same moment. Instead, each playbook checks the limit and reserves the kiosks it will take offline in one atomic step, counting the other’s reservation, so the second playbook is held until the first one’s kiosks are back. Databases prevent this anomaly with serializable <a href="https://www.postgresql.org/docs/current/transaction-iso.html" target="_blank" rel="noopener">transaction isolation</a>; agents acting on devices need the same guarantee where their actions take effect.

<div class="fig-wide" id="figure-D1" style="width: 100%; margin-left: 0; transform: none;">
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_figD1_kiosk.png"><img src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_figD1_kiosk.png" width="3537" height="1129" loading="lazy" decoding="async" alt="Step chart of kiosks online from 10:00 to 10:30: playbook A reboots 6 kiosks and playbook B reboots 5; overlapping from 10:10 to 10:15, they leave 9 online, below the floor of 12. A panel shows the check-and-reserve step at 10:10 holding B until A&#x27;s kiosks are back."></a>
</div>

<p class="fig-caption long">Figure D1. Each playbook was safe alone; together they took the store below its kiosk floor. A state invariant, checked and reserved in one step, watches the total, not each action: at least 12 of 20 checkout kiosks online during store hours. Store, kiosks and times are illustrative.</p>

## Appendix E: Why One Repair Can Mislead {#appendix-e}

This appendix illustrates the three complications named in <a href="#open-problems">Open Problems</a>: joint faults, valid alternative plans and random variation between reruns.

A joint fault needs more than one repair (<a href="#figure-E1">Figure E1</a>). A patch fails on a group of laptops because two agents erred: the inventory agent targeted the wrong driver version, and the scheduler agent picked a window when the laptops were asleep. Repairing either step alone still fails, so a test of single repairs can wrongly clear both; only repairing both restores success.

<div class="fig-wide" id="figure-E1" style="--fw: 592px">
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_figE1_e_joint.png"><img src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_figE1_e_joint.png" width="2232" height="1055" loading="lazy" decoding="async" alt="Four rows of a three-step chain (inventory agent, scheduler agent, outcome): the original run, the version repaired only, the window repaired only, and both repaired; only the last row installs the patch."></a>
</div>

<p class="fig-caption long">Figure E1. Four runs: the original, each single repair, and both repairs together; only the last installs the patch. The scenario is illustrative.</p>

A repair can also work by steering the run onto a different valid plan (<a href="#figure-E2">Figure E2</a>). The install fails because the vendor’s download mirror is down. Changing the planner’s step to install from the local cache succeeds, which seems to blame the planner, yet its original plan was valid: the outage was the cause.

<div class="fig-wide" id="figure-E2" style="width: 100%; margin-left: 0; transform: none;">
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_figE2_e_plans.png"><img src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_figE2_e_plans.png" width="3039" height="845" loading="lazy" decoding="async" alt="Two chains: the original run installs from the vendor mirror and fails because the mirror is down; with the planner changed to install from the local cache, the install succeeds."></a>
</div>

<p class="fig-caption long">Figure E2. Two valid plans for the same install; the mirror outage, not the planner, decides which one succeeds. The scenario is illustrative.</p>

Finally, reruns of the same steps can differ, because models sample their outputs (<a href="#figure-E3">Figure E3</a>). A repair that lifts success from 3 in 10 to 7 in 10 can still fail a single rerun, and an unrepaired run can succeed by luck, so compare success rates, with their uncertainty, over repeated reruns.

<div class="fig-wide" id="figure-E3" style="--fw: 592px">
  <a class="plot-zoom" href="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_figE3_e_reruns.png"><img src="{{ site.baseurl }}/assets/images/invisible_agents/agents-endpoint_figE3_e_reruns.png" width="2320" height="593" loading="lazy" decoding="async" alt="Two rows of ten reruns: unrepaired, 3 of 10 succeed, with one lucky success marked; repaired, 7 of 10 succeed, with one unlucky failure marked."></a>
</div>

<p class="fig-caption long">Figure E3. Ten reruns before and after a repair: one rerun from either row could tell the wrong story. Counts are illustrative.</p>

<div class="plot-lightbox" id="plot-lightbox" role="dialog" aria-modal="true" aria-label="Enlarged figure. Press Escape or click to close." hidden>
  <button type="button" class="plot-lightbox-close" aria-label="Close">&times;</button>
  <img id="plot-lightbox-img" src="data:," alt="" hidden>
  <video id="plot-lightbox-vid" muted playsinline hidden></video>
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
  .post-header .post-title { font-size: 42px; white-space: normal; }
}

.post-content ul ul { list-style-type: circle !important; }
.post-content sup.fnref { font-size: 0.65em; line-height: 0; vertical-align: super; }
.post-content sup.fnref a { color: #000; text-decoration: none; padding: 0 1px; }
.post-content sup.fnref a:hover { color: #1155cc; text-decoration: underline; }

/* Code: Courier New, no badge */
.post-content code { font-family: "Courier New", Courier, "Liberation Mono", monospace; font-size: 1em; color: inherit; background: none; border: 0; border-radius: 0; padding: 0; }

/* Tables */
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

/* Notes */
.post-content ul.notes { font-size: 13px; line-height: 1.5; list-style: none; margin-left: 0; padding-left: 0; }
.post-content ul.notes li { margin-bottom: 6px; padding: 2px 4px; border-radius: 3px; scroll-margin-top: 20px; }
.post-content ul.notes li:target { background: #fff6cc; }
.post-content ul.notes .fn-num { font-size: 0.75em; vertical-align: super; line-height: 0; margin-right: 2px; }
.post-content ul.notes .fn-back { text-decoration: none; margin-left: 2px; }

/* Figures: centred on the text column; captions centred, or justified when longer than one line */
.post-content .fig-wide { width: min(var(--fw, 100%), calc(100vw - 48px)); margin: 6px 0 4px 50%; transform: translateX(-50%); }
.post-content .fig-wide img { display: block; width: 100%; height: auto; }
.post-content .fig-wide a.plot-zoom { display: block; line-height: 0; cursor: zoom-in; }
.post-content p.fig-caption { text-align: center !important; font-size: 14px; line-height: 1.4; color: #555555; margin: 4px 0 14px 0; }
.post-content p.fig-caption.long { text-align: justify !important; }
.post-content p.fig-caption.with-hint { margin-bottom: 0; }
.post-content p.plot-hint { text-align: center !important; font-size: 0.875rem; font-style: italic; color: #6b7280; margin: 2px 0 12px 0; }

/* Figures 1, 3, 4 and 9: floated right at 70% of the text column; the paragraph after each wraps beside it in a
   narrow column, so long words there may hyphenate. Headings, centred figures and tables start below any float. */
.post-content .fig-r70 { float: right; clear: right; width: 70%; margin: 5px 0 10px 18px; }
.post-content .fig-r70 img { display: block; width: 100%; height: auto; }
.post-content .fig-r70 a.plot-zoom { display: block; line-height: 0; cursor: zoom-in; }
.post-content .fig-r70 p.fig-caption { margin: 4px 0 0 0; }
.post-content .fig-r70 p.plot-hint { margin: 2px 0 0 0; }
.post-content .fig-r70 + p { hyphens: auto; -webkit-hyphens: auto; hyphenate-limit-chars: 6 3 3; }
.post-content h2, .post-content h3, .post-content .fig-wide, .post-content .table-wrap { clear: both; }

/* Animations: play once when scrolled into view, rest on the finished figure; the corner button enlarges them */
.post-content .vid-wrap { position: relative; line-height: 0; background: #ffffff; }
.post-content .vid-wrap video { display: block; width: 100%; height: 100%; cursor: pointer; background: #ffffff; }
.post-content .vid-wrap video:focus-visible { outline: 2px solid #1155cc; outline-offset: 2px; }
.post-content .vid-zoom { position: absolute; top: 6px; right: 6px; width: 30px; height: 30px; border: 1px solid #d6dbe2; border-radius: 6px; background: rgba(255, 255, 255, 0.92); color: #374151; font-size: 16px; line-height: 1; cursor: zoom-in; opacity: 0; transition: opacity 0.15s ease; }
.post-content .vid-wrap:hover .vid-zoom, .post-content .vid-zoom:focus-visible { opacity: 1; }
@media (hover: none) { .post-content .vid-zoom { opacity: 0.7; } }

/* Enlarged figure or animation */
.plot-lightbox { position: fixed; inset: 0; z-index: 9999; display: flex; align-items: center; justify-content: center; background: rgba(17, 17, 17, 0.78); cursor: zoom-out; }
.plot-lightbox[hidden] { display: none; }
.plot-lightbox img, .plot-lightbox video { width: min(92vw, 1760px); height: auto; max-height: 92vh; object-fit: contain; background: #ffffff; border-radius: 4px; box-shadow: 0 12px 40px rgba(0, 0, 0, 0.45); }
.plot-lightbox img[hidden], .plot-lightbox video[hidden] { display: none; }
.plot-lightbox-close { position: absolute; top: 10px; right: 18px; border: 0; background: none; color: #ffffff; font: 36px/1 Arial, sans-serif; cursor: pointer; }

/* Narrow screens: the floated figures fill the column, and enlarged figures use the full screen width */
@media screen and (max-width: 800px) {
  .post-content .fig-r70 { float: none; width: 100%; margin: 6px 0 14px 0; }
  .plot-lightbox img, .plot-lightbox video { width: 96vw; }
}
</style>

<script>
(function () {
  'use strict';
  /* Fit the title on one line of the text column, whatever font the reader's OS supplies */
  var title = document.querySelector('.post-header .post-title');
  if (title) {
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
  }

  /* Animations load as they near the screen */
  function ensureSrc(v) {
    if (!v.getAttribute('src') && v.getAttribute('data-src')) { v.preload = 'auto'; v.src = v.getAttribute('data-src'); }
  }

  /* Enlarge: a figure or an animation opens in a centred viewer; Escape or a click closes it */
  var box = document.getElementById('plot-lightbox');
  var big = document.getElementById('plot-lightbox-img');
  var bigVid = document.getElementById('plot-lightbox-vid');
  var opener = null;
  function closeBox() {
    if (!box || box.hidden) { return; }
    box.hidden = true;
    big.removeAttribute('src'); big.hidden = true;
    bigVid.pause(); bigVid.removeAttribute('src'); bigVid.load(); bigVid.hidden = true;
    document.documentElement.style.overflow = '';
    if (opener) { opener.focus(); opener = null; }
  }
  function openBox(kind, src, alt, from) {
    opener = from;
    if (kind === 'img') { big.src = src; big.alt = alt || ''; big.hidden = false; }
    else { bigVid.src = src; bigVid.hidden = false; bigVid.muted = true; bigVid.controls = true; bigVid.play().catch(function () {}); }
    box.hidden = false;
    document.documentElement.style.overflow = 'hidden';
    box.querySelector('.plot-lightbox-close').focus();
  }
  if (box && big && bigVid) {
    document.body.appendChild(box);   /* so no ancestor can clip or offset it */
    document.addEventListener('click', function (e) {
      var link = e.target.closest ? e.target.closest('a.plot-zoom') : null;
      if (link) {
        e.preventDefault();
        var img = link.querySelector('img');
        openBox('img', link.href, img ? img.alt : '', link);
        return;
      }
      var zb = e.target.closest ? e.target.closest('button.vid-zoom') : null;
      if (zb) {
        e.preventDefault();
        var v = zb.parentNode.querySelector('video');
        ensureSrc(v);
        openBox('vid', v.currentSrc || v.src, v.getAttribute('aria-label'), zb);
      }
    });
    box.addEventListener('click', function (e) { if (e.target !== bigVid) { closeBox(); } });
    document.addEventListener('keydown', function (e) { if (e.key === 'Escape') { closeBox(); } });
  }

  /* Each animation plays once (about four seconds) when 60% of it is visible and rests on the finished figure.
     It replays when it comes back into view after leaving the screen, or when clicked after it has finished;
     a click, Enter or Space while it plays pauses it. With reduced motion, it waits for its own controls. */
  var vids = Array.prototype.slice.call(document.querySelectorAll('video.anim-in-view'));
  var reduce = window.matchMedia && window.matchMedia('(prefers-reduced-motion: reduce)').matches;
  function start(v, fromTop) {
    ensureSrc(v);
    if (fromTop || v.ended || v.dataset.restart === '1') { v.currentTime = 0; v.dataset.restart = ''; }
    var p = v.play();
    if (p && p.catch) { p.catch(function (err) { if (err && err.name === 'NotAllowedError') { v.controls = true; } }); }
  }
  vids.forEach(function (v) {
    v.muted = true;
    v.dataset.restart = '1';
    if (reduce) { v.controls = true; v.addEventListener('play', function () { ensureSrc(v); }, { once: true }); return; }
    function toggle() {
      if (v.controls) { return; }
      if (!v.paused) { v.dataset.userPaused = '1'; v.pause(); } else { v.dataset.userPaused = ''; start(v, v.ended); }
    }
    v.addEventListener('click', toggle);
    v.addEventListener('keydown', function (e) { if (e.key === 'Enter' || e.key === ' ') { e.preventDefault(); toggle(); } });
  });
  if ('IntersectionObserver' in window) {
    var near = new IntersectionObserver(function (entries) {
      entries.forEach(function (e) { if (e.isIntersecting) { ensureSrc(e.target); near.unobserve(e.target); } });
    }, { rootMargin: '900px 0px' });
    vids.forEach(function (v) { near.observe(v); });
    if (!reduce) {
      var io = new IntersectionObserver(function (entries) {
        entries.forEach(function (e) {
          var v = e.target;
          if (e.intersectionRatio >= 0.6) {
            if (!v.dataset.userPaused && (v.dataset.restart === '1' || (v.paused && !v.ended))) { start(v); }
          } else {
            if (!v.paused) { v.pause(); }
            if (!e.isIntersecting) { v.dataset.restart = '1'; v.dataset.userPaused = ''; }
          }
        });
      }, { threshold: [0, 0.6] });
      vids.forEach(function (v) { io.observe(v); });
    }
  } else {
    vids.forEach(function (v) { ensureSrc(v); v.controls = true; });
  }
})();
</script>
