---
layout: post
title: "Edge AI for Wearables: The Gearbox"
date: 2026-09-28
description: "AI on wearables is subject to tight constraints: answer in under half a second, on about a watt, with a sliver of
              the memory a phone has. Getting there takes many knobs across models, data, compression and inference, turning
              together like gears. Which gears matter most, and where are they still stuck?"
categories: [edge AI, wearables, on-device inference, model compression]
author: Arun Kejariwal
last_modified_at: 2026-09-28
---

<p class="post-byline" id="post-byline">with <a href="https://www.linkedin.com/in/bhargav-bhushanam-b5876125/" target="_blank" rel="noopener">Bhargav Bhushanam</a></p>

Running AI on glasses is a different problem from running it in the data center. Peak memory, battery, heat and a sub-500 ms
response budget change what "good" means across data, models and inference, and UX is part of the design, not an afterthought.

The work in this space has exploded over the last two years. To make it easier to find your way in, we mapped the key research
directions as a set of gears, with a companion table that links every technique to its source and calls out open problems.

<figure class="gearbox-figure">
  <a href="{{ site.baseurl }}/assets/images/Edge_AI_Gearbox_16K.png" id="gearbox-img-link" title="Click to view in full screen">
    <img id="gearbox-img" src="{{ site.baseurl }}/assets/images/Edge_AI_Gearbox_16K.png" alt="The Edge AI for Wearables gearbox: interlocking gears for models, data, compression and inference, each labelled with its knobs" decoding="async">
  </a>
  <figcaption>Click the image to see in full screen</figcaption>
</figure>

<div class="gearbox-legend" aria-hidden="true">
  <span><i class="sw sw-mod"></i>Models</span>
  <span><i class="sw sw-data"></i>Data (under Models)</span>
  <span><i class="sw sw-comp"></i>Compression (under Models)</span>
  <span><i class="sw sw-inf"></i>Inference</span>
</div>

<div class="gearbox-wrap">
<table class="gearbox"><colgroup><col class="c-l1"><col class="c-l2"><col class="c-l3"><col class="c-knob"><col class="c-what"><col class="c-op"></colgroup><thead><tr><th scope="col">Level 1</th><th scope="col">Level 2</th><th scope="col">Level 3</th><th scope="col">Knob / technique</th><th scope="col">What it is</th><th scope="col">Open problems</th></tr></thead><tbody>
<tr class="t-mod grp-start"><th scope="rowgroup" class="l1 l1-mod" rowspan="94"><span>MODELS</span></th><th scope="rowgroup" class="l2" colspan="2" rowspan="8"><span>Dense SLMs</span></th><td class="knob"><a href="https://arxiv.org/pdf/2402.14905" target="_blank" rel="noopener"><strong>0.1–1B params</strong></a></td><td class="what">The size band that fits on the wearable itself: at 4-bit, a 1B model needs about 0.5 GB of weights, which fits glasses-class memory. 2–4B models (e.g., <a href="https://arxiv.org/pdf/2505.09388" target="_blank" rel="noopener">Qwen3-4B</a>) run only on the companion phone.</td><td class="op op-mod" rowspan="39"><ul><li><strong>MoE prefill:</strong> multi-token prompts activate most experts, erasing MoE's advantage on time to first token.</li><li><strong>Low-bit reasoning:</strong> small reasoning models degrade sharply below 4 bits; recipes that keep chain-of-thought intact at 2–3 bits are missing.</li><li><strong>Temporal memory:</strong> all-day egocentric streams need bounded state that can still recall an event from minutes ago, within ~1–2 W.</li><li><strong>Visual evidence:</strong> token pruning and low resolution can drop exactly the detail (text, small objects) a task needs.</li><li><strong>Honest benchmarks:</strong> static QA and warm tokens/s omit capture, speech, missed events and long-session power.</li></ul></td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2402.14905" target="_blank" rel="noopener"><strong>Deep &amp; thin</strong></a></td><td class="what">Below ~1B parameters, more layers at narrower width beat wide-and-shallow at equal size (<a href="https://arxiv.org/pdf/2402.14905" target="_blank" rel="noopener">MobileLLM</a>: +2.7 / +4.3 points at 125M / 350M), at some cost in serial latency.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/1608.05859" target="_blank" rel="noopener"><strong>Tied embeds</strong></a></td><td class="what">The input embedding (token → vector) and the output projection (final vector → a score for each vocabulary word) share one matrix. Embeddings are a large share of a small model's weights, so tying frees memory for more layers.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2305.13245" target="_blank" rel="noopener"><strong>GQA</strong></a></td><td class="what">Grouped-query attention: groups of query heads share one key/value head, shrinking the KV cache 4–8× with close to full-attention quality. Standard in current small models.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2402.14905" target="_blank" rel="noopener"><strong>Weight sharing</strong></a></td><td class="what">Adjacent layers reuse one transformer block's weights (<a href="https://arxiv.org/pdf/2402.14905" target="_blank" rel="noopener">MobileLLM-LS</a> runs each block twice): more depth without more storage, and little added latency because the weights are already in cache.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2310.07707" target="_blank" rel="noopener"><strong>MatFormer</strong></a></td><td class="what">A nested model (<a href="https://developers.googleblog.com/en/introducing-gemma-3n-developer-guide/" target="_blank" rel="noopener">Gemma 3n</a>): a smaller sub-model lives inside the larger one, so one download serves several sizes; switching between them at run time, by device load, is described as future work.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://developers.googleblog.com/en/introducing-gemma-3n-developer-guide/" target="_blank" rel="noopener"><strong>PLE offload</strong></a></td><td class="what">Per-layer embeddings (PLE): <a href="https://developers.googleblog.com/en/introducing-gemma-3n-developer-guide/" target="_blank" rel="noopener">Gemma 3n</a> and <a href="https://arxiv.org/pdf/2607.02770" target="_blank" rel="noopener">Gemma 4</a> keep these in CPU memory instead of accelerator memory, so a model with ~5B raw parameters runs with a ~2B-class footprint (E2B), which is still above most glasses budgets today.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2404.16710" target="_blank" rel="noopener"><strong>Early exit</strong></a></td><td class="what">Stop at an intermediate layer when the prediction is already confident (<a href="https://arxiv.org/pdf/2404.16710" target="_blank" rel="noopener">LayerSkip</a>), saving compute on easy tokens; the early layers can double as a self-speculative drafter.</td></tr>
<tr class="t-mod grp-start"><th scope="rowgroup" class="l2" colspan="2" rowspan="9"><span>MoE (mixture of experts)</span></th><td class="knob"><a href="https://arxiv.org/pdf/2605.27358" target="_blank" rel="noopener"><strong># experts</strong></a></td><td class="what">How many expert feed-forward blocks a layer holds. Each token runs only its top-k experts, so adding experts adds capacity without adding per-token compute; all of them must still be stored; <a href="https://arxiv.org/pdf/2605.27358" target="_blank" rel="noopener">MobileMoE</a> finds moderate sparsity best on-device.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/1701.06538" target="_blank" rel="noopener"><strong>Top-k routing</strong></a></td><td class="what">Each token is sent to its k highest-scoring experts (e.g., top-2, top-4). Larger k improves quality but raises compute and the number of experts that must be loaded.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2605.27358" target="_blank" rel="noopener"><strong>Granularity</strong></a></td><td class="what">Split experts into many smaller ones (fine-grained experts). <a href="https://arxiv.org/pdf/2605.27358" target="_blank" rel="noopener">MobileMoE</a> uses 60 routed experts with top-4 routing, which gives more diverse routing at equal compute.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2401.06066" target="_blank" rel="noopener"><strong>Shared expert</strong></a></td><td class="what">One always-on expert processes every token alongside the routed ones; it captures common knowledge and improves quality at fixed compute (<a href="https://arxiv.org/pdf/2605.27358" target="_blank" rel="noopener">MobileMoE</a>, <a href="https://arxiv.org/pdf/2401.06066" target="_blank" rel="noopener">DeepSeek</a>).</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2605.27358" target="_blank" rel="noopener"><strong>Active / total</strong></a></td><td class="what">Compute scales with active parameters, memory with total. <a href="https://arxiv.org/pdf/2605.27358" target="_blank" rel="noopener">MobileMoE</a>: 0.3–0.9B active of 1.3–5.3B total. A 30B-A3B model still needs ~15 GB at 4-bit.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2308.14352" target="_blank" rel="noopener"><strong>Expert cache</strong></a></td><td class="what">Keep recently or frequently used experts in RAM and stream the rest from flash (<a href="https://arxiv.org/pdf/2308.14352" target="_blank" rel="noopener">EdgeMoE</a>, <a href="https://arxiv.org/pdf/2312.17238" target="_blank" rel="noopener">Mixtral-offloading</a>). The cache hit rate largely sets decode speed.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2312.17238" target="_blank" rel="noopener"><strong>Prefetch</strong></a></td><td class="what">Predict the next layer's experts and load them while the current layer computes (<a href="https://arxiv.org/pdf/2507.20984" target="_blank" rel="noopener">routers placed before attention</a>, <a href="https://arxiv.org/pdf/2312.17238" target="_blank" rel="noopener">next-layer gate speculation</a>), hiding flash I/O behind compute.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2308.14352" target="_blank" rel="noopener"><strong>Mixed-bit</strong></a></td><td class="what">Store less important experts at lower precision, 2–4 bits by importance (<a href="https://arxiv.org/pdf/2308.14352" target="_blank" rel="noopener">EdgeMoE</a>, <a href="https://arxiv.org/pdf/2411.01433" target="_blank" rel="noopener">HOBBIT</a>); shrinks memory and flash traffic with limited quality loss.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2402.14800" target="_blank" rel="noopener"><strong>Prune / merge</strong></a></td><td class="what">Delete rarely useful experts or merge similar ones (<a href="https://arxiv.org/pdf/2402.14800" target="_blank" rel="noopener">NAEE</a>, <a href="https://arxiv.org/pdf/2310.01334" target="_blank" rel="noopener">MC-SMoE</a>), or slim every expert and distill (<a href="https://arxiv.org/pdf/2506.18349" target="_blank" rel="noopener">SlimMoE</a>: 41.9B → 7.6B total).</td></tr>
<tr class="t-mod grp-start"><th scope="rowgroup" class="l2" colspan="2" rowspan="7"><span>Hybrid &amp; recurrent</span></th><td class="knob"><a href="https://arxiv.org/pdf/2511.23404" target="_blank" rel="noopener"><strong>Conv + attn</strong></a></td><td class="what">About two-thirds to three-quarters of layers are gated short convolutions, the rest attention (<a href="https://huggingface.co/LiquidAI/LFM2-1.2B" target="_blank" rel="noopener">LFM2-1.2B</a>: 10 of 16; <a href="https://huggingface.co/LiquidAI/LFM2-2.6B" target="_blank" rel="noopener">LFM2-2.6B</a>: 22 of 30). <a href="https://arxiv.org/pdf/2511.23404" target="_blank" rel="noopener">LFM2 reports</a> up to 2× faster CPU prefill and decode than similar-size transformers.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2405.21060" target="_blank" rel="noopener"><strong>SSM (Mamba-2)</strong></a></td><td class="what">State-space model (SSM): recurrent layers (e.g., <a href="https://arxiv.org/pdf/2405.21060" target="_blank" rel="noopener">Mamba-2</a>) compress the past into a fixed-size state instead of a growing KV cache, so memory stays flat with sequence length (<a href="https://www.ibm.com/new/announcements/ibm-granite-4-0-hyper-efficient-high-performance-hybrid-models" target="_blank" rel="noopener">Granite 4.0-H</a> mixes them 9:1 with attention).</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2411.13676" target="_blank" rel="noopener"><strong>Attn+SSM heads</strong></a></td><td class="what">Attention and SSM heads run in parallel inside each layer (<a href="https://arxiv.org/pdf/2411.13676" target="_blank" rel="noopener">Hymba</a>): an 11.67× smaller cache and 3.49× the throughput of Llama-3.2-3B, with higher accuracy.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2412.06464" target="_blank" rel="noopener"><strong>Gated DeltaNet</strong></a></td><td class="what">A gated linear-attention layer with delta-rule state updates; <a href="https://huggingface.co/Qwen/Qwen3.5-0.8B" target="_blank" rel="noopener">Qwen3.5</a> small models interleave three of these with each full-attention block to bound long-context cost.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2503.14456" target="_blank" rel="noopener"><strong>RWKV-7</strong></a></td><td class="what">Receptance Weighted Key Value (RWKV): a fully recurrent RNN-style language model with constant memory and time per token and no KV cache; competitive up to ~3B parameters.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2402.18668" target="_blank" rel="noopener"><strong>Fixed-size state</strong></a></td><td class="what">The key edge property of recurrent layers: memory does not grow with context. State size and precision still matter, and recall of distant details can suffer.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2403.19887" target="_blank" rel="noopener"><strong>Few attn layers</strong></a></td><td class="what">Keep only a handful of full-attention layers for precise retrieval and make everything else cheap. Those layers still need a KV cache that grows with context.</td></tr>
<tr class="t-mod grp-start"><th scope="rowgroup" class="l2" colspan="2" rowspan="8"><span>VLM (vision-language)</span></th><td class="knob"><a href="https://arxiv.org/pdf/2412.13303" target="_blank" rel="noopener"><strong>Encoder cost</strong></a></td><td class="what">The vision encoder often dominates time to first token. Efficient encoders such as <a href="https://arxiv.org/pdf/2412.13303" target="_blank" rel="noopener">FastViTHD (FastVLM)</a> and <a href="https://developers.googleblog.com/en/introducing-gemma-3n-developer-guide/" target="_blank" rel="noopener">MobileNet-V5 (Gemma 3n)</a> cut encoding time sharply.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2501.03895" target="_blank" rel="noopener"><strong>Visual tokens</strong></a></td><td class="what">Every image token must be prefilled by the language model, so token count drives latency: from 1 (<a href="https://arxiv.org/pdf/2501.03895" target="_blank" rel="noopener">LLaVA-Mini</a>) to 144 (<a href="https://arxiv.org/pdf/2402.03766" target="_blank" rel="noopener">MobileVLM v2</a>) to thousands at high resolution.</td></tr>
<tr class="t-mod"><td class="knob"><strong><strong>Resolution</strong></strong></td><td class="what">Higher resolution helps OCR and small objects but multiplies tokens and encoder cost; tiling one large image can add hundreds of tokens.</td></tr>
<tr class="t-mod"><td class="knob"><strong><strong>ROI crops</strong></strong></td><td class="what">Encode a small high-resolution crop of the region that matters (text, hands) plus a coarse view of the scene, instead of the whole frame at full resolution.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2402.03766" target="_blank" rel="noopener"><strong>Projector</strong></a></td><td class="what">The module that maps encoder features into language-model tokens; pooling projectors (<a href="https://arxiv.org/pdf/2402.03766" target="_blank" rel="noopener">LDPv2</a>, <a href="https://arxiv.org/pdf/2504.05299" target="_blank" rel="noopener">pixel shuffle</a>) compress 576 tokens to 144 or fewer.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2403.06764" target="_blank" rel="noopener"><strong>Token pruning</strong></a></td><td class="what">Drop or merge low-importance visual tokens after encoding (<a href="https://arxiv.org/pdf/2403.06764" target="_blank" rel="noopener">FastV</a>, <a href="https://arxiv.org/pdf/2410.04417" target="_blank" rel="noopener">SparseVLM</a>, <a href="https://arxiv.org/pdf/2412.04467" target="_blank" rel="noopener">VisionZip</a>, <a href="https://arxiv.org/pdf/2210.09461" target="_blank" rel="noopener">ToMe</a>), often halving compute; can remove evidence needed for counting or OCR.</td></tr>
<tr class="t-mod"><td class="knob"><strong><strong>Frame rate</strong></strong></td><td class="what">For video, process fewer frames or only frames that change. Cuts compute and power, at the risk of missing brief events.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2510.09608" target="_blank" rel="noopener"><strong>Video KV cap</strong></a></td><td class="what">Streaming video grows the KV cache without bound; cap it with attention sinks plus a window (<a href="https://arxiv.org/pdf/2510.09608" target="_blank" rel="noopener">StreamingVLM</a>) or redundancy removal (<a href="https://arxiv.org/pdf/2506.15745" target="_blank" rel="noopener">InfiniPot-V</a>: up to −94% memory).</td></tr>
<tr class="t-mod grp-start"><th scope="rowgroup" class="l2" colspan="2" rowspan="7"><span>VRM (visual reasoning)</span></th><td class="knob"><a href="https://arxiv.org/pdf/2505.09388" target="_blank" rel="noopener"><strong>Think / no-think</strong></a></td><td class="what">Hybrid models (<a href="https://arxiv.org/pdf/2505.09388" target="_blank" rel="noopener">Qwen3</a>, <a href="https://arxiv.org/pdf/2509.18154" target="_blank" rel="noopener">MiniCPM-V 4.5</a>, <a href="https://www.microsoft.com/en-us/research/blog/phi-4-reasoning-vision-and-the-lessons-of-training-a-multimodal-reasoning-model/" target="_blank" rel="noopener">Phi-4-reasoning-vision</a>) switch between answering directly and chain-of-thought, skipping reasoning on plain perception tasks.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2501.19393" target="_blank" rel="noopener"><strong>CoT budget</strong></a></td><td class="what">Cap the number of thinking tokens (budget forcing, as in <a href="https://arxiv.org/pdf/2501.19393" target="_blank" rel="noopener">s1</a>) so reasoning fits the latency and energy budget.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2505.13417" target="_blank" rel="noopener"><strong>Adaptive length</strong></a></td><td class="what">Train the model to reason only as long as needed (<a href="https://arxiv.org/pdf/2505.13417" target="_blank" rel="noopener">AdaptThink</a>: −53% length at 1.5B) or stop once confident (<a href="https://arxiv.org/pdf/2504.15895" target="_blank" rel="noopener">DEER</a>: 19–80% shorter chains of thought).</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2507.13348" target="_blank" rel="noopener"><strong>Re-look image</strong></a></td><td class="what">Instead of long textual reasoning, look again at a higher-resolution view only when needed (<a href="https://arxiv.org/pdf/2507.13348" target="_blank" rel="noopener">VisionThink</a> starts at 1/4 resolution).</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2504.07615" target="_blank" rel="noopener"><strong>RL (GRPO)</strong></a></td><td class="what">Group Relative Policy Optimization (GRPO): reinforcement learning with rule-based rewards (<a href="https://arxiv.org/pdf/2504.07615" target="_blank" rel="noopener">VLM-R1</a> with GRPO; <a href="https://arxiv.org/pdf/2503.07536" target="_blank" rel="noopener">LMM-R1</a> at 3B with PPO) builds visual reasoning in small models more robustly than supervised fine-tuning.</td></tr>
<tr class="t-mod"><td class="knob"><a href="https://arxiv.org/pdf/2502.12143" target="_blank" rel="noopener"><strong>Short-CoT KD</strong></a></td><td class="what">Distill a large reasoner's traces into short, correct chains for a small student, keeping the accuracy gain while cutting output tokens.</td></tr>
<tr class="t-mod"><td class="knob"><strong><strong>Cloud offload</strong></strong></td><td class="what">Deep visual reasoning rarely fits a wearable: 1,000 tokens at 20 tok/s ≈ 50 s. Send it to the companion or the cloud and keep a short-answer path on the wearable.</td></tr>
<tr class="op-row t-mod"><td colspan="6" class="op op-mod"><div class="op-title">Open problems · Models</div><ul><li><strong>MoE prefill:</strong> multi-token prompts activate most experts, erasing MoE's advantage on time to first token.</li><li><strong>Low-bit reasoning:</strong> small reasoning models degrade sharply below 4 bits; recipes that keep chain-of-thought intact at 2–3 bits are missing.</li><li><strong>Temporal memory:</strong> all-day egocentric streams need bounded state that can still recall an event from minutes ago, within ~1–2 W.</li><li><strong>Visual evidence:</strong> token pruning and low resolution can drop exactly the detail (text, small objects) a task needs.</li><li><strong>Honest benchmarks:</strong> static QA and warm tokens/s omit capture, speech, missed events and long-session power.</li></ul></td></tr>
<tr class="t-data grp-start"><th scope="rowgroup" class="l2 l2-band" rowspan="22"><span>DATA</span></th><th scope="rowgroup" class="l3" rowspan="7"><span>Real data</span></th><td class="knob"><a href="https://facebookresearch.github.io/projectaria_tools/docs/tech_spec/device_calibration" target="_blank" rel="noopener"><strong>Calibration &amp; sync</strong></a></td><td class="what"><p>Each sensor's own parameters, the relative pose between sensors, and their timestamps must line up before video, IMU, audio and gaze can be fused.</p><ul><li>Sensor calibration: <a href="https://facebookresearch.github.io/projectaria_tools/docs/tech_spec/device_calibration" target="_blank" rel="noopener">Aria Gen 1 ships intrinsics and extrinsics</a> for 5 cameras and 2 IMUs; magnetometer, barometer and microphones have intrinsics only, with extrinsics taken from the CAD design.</li><li>Time sync: <a href="https://facebookresearch.github.io/projectaria_tools/docs/tech_insights/temporal_alignment_of_sensor_data" target="_blank" rel="noopener">Aria uses the SLAM camera's mid-exposure time</a> as reference; RGB rolling-shutter readout takes 16.26 ms, and audio, barometer and GPS offsets are still undetermined.</li><li>Multi-device sync: <a href="https://arxiv.org/pdf/2510.16134" target="_blank" rel="noopener">Aria Gen 2</a> aligns several devices to under 1 ms over sub-GHz radio.</li></ul></td><td class="op op-data" rowspan="22"><ul><li><strong>Commercial gap:</strong> the big egocentric datasets are non-commercial, so there is no shared public baseline for shipped glasses.</li><li><strong>Bystander consent at scale:</strong> blurring misses voices, reflections, gaze and heart-rate signals; anonymizing on the device before storage is unsolved.</li><li><strong>Sensor-faithful generation:</strong> generators rarely reproduce rolling shutter, fisheye lenses, low-light noise or IMU drift, or keep video, IMU, audio and gaze consistent.</li><li><strong>Small models and synthetic data:</strong> the right real:synthetic ratio for ≤1B models is unknown, and on-device personalization risks recursive collapse.</li><li><strong>Compute-aware benchmarks:</strong> egocentric benchmarks score accuracy only, never the &lt; 500 ms / ~2 W regime.</li></ul></td></tr>
<tr class="t-data"><td class="knob"><a href="https://arxiv.org/pdf/2110.07058" target="_blank" rel="noopener"><strong>Collection protocol</strong></a></td><td class="what"><p>Scripted vs unscripted activity, who wears the device, where, for how long, and with what consent; together these decide how well a model generalizes.</p><ul><li><a href="https://arxiv.org/pdf/2110.07058" target="_blank" rel="noopener">Ego4D</a>: 3,670 h from 931 wearers in 74 locations across 9 countries.</li><li><a href="https://arxiv.org/pdf/2311.18259" target="_blank" rel="noopener">Ego-Exo4D</a>: 1,286 h, 740 participants, 13 cities, skilled activities of 1–42 min.</li><li><a href="https://arxiv.org/pdf/2503.03803" target="_blank" rel="noopener">EgoLife</a>: 300 h from 6 people living together for one week.</li><li><a href="https://arxiv.org/pdf/2502.04144" target="_blank" rel="noopener">HD-EPIC</a>: 41 h of unscripted recording in 9 kitchens.</li></ul></td></tr>
<tr class="t-data"><td class="knob"><a href="https://arxiv.org/pdf/2406.09905" target="_blank" rel="noopener"><strong>Multimodal capture</strong></a></td><td class="what"><p>Wearables need synchronized video, IMU, spatial audio, gaze, hand and body pose and physiological signals, ideally with precise ground truth.</p><ul><li><a href="https://arxiv.org/pdf/2406.09905" target="_blank" rel="noopener">Nymeria</a>: 300 h, 264 participants, 50 locations, full-body motion ground truth plus Aria video, gaze and IMU.</li><li><a href="https://arxiv.org/pdf/2510.16134" target="_blank" rel="noopener">Aria Gen 2 pilot</a>: 800 Hz IMUs, 8-channel audio, 128 Hz PPG, on-device VIO, hand tracking and gaze.</li><li>Ground truth: <a href="https://arxiv.org/pdf/2306.06362" target="_blank" rel="noopener">Aria Digital Twin</a> has 200 sequences with 398 objects; <a href="https://arxiv.org/pdf/2402.13349" target="_blank" rel="noopener">Aria Everyday Activities</a> has 143 sequences with SLAM trajectories, gaze and transcripts.</li></ul></td></tr>
<tr class="t-data"><td class="knob"><a href="https://arxiv.org/pdf/2502.04144" target="_blank" rel="noopener"><strong>Annotation</strong></a></td><td class="what"><p>Timestamped narrations and dense action, audio and QA labels are the main cost of real data; LLMs can densify cheaper weak labels.</p><ul><li><a href="https://arxiv.org/pdf/2502.04144" target="_blank" rel="noopener">HD-EPIC</a>: 263 annotations per minute of video (59K actions, 51K audio events).</li><li><a href="https://arxiv.org/pdf/2006.13256" target="_blank" rel="noopener">EPIC-KITCHENS-100</a>: 100 h, 700 videos, 90K actions.</li><li><a href="https://arxiv.org/pdf/2311.18259" target="_blank" rel="noopener">Ego-Exo4D</a> adds expert commentary from coaches and teachers.</li><li>LLM-assisted: <a href="https://arxiv.org/pdf/2212.04501" target="_blank" rel="noopener">LaViLa</a>, using half of Ego4D's human narrations, beat baselines (+10.1% EGTEA, +5.9% EK-100 retrieval).</li></ul></td></tr>
<tr class="t-data"><td class="knob"><a href="https://arxiv.org/pdf/2308.13093" target="_blank" rel="noopener"><strong>Privacy &amp; consent</strong></a></td><td class="what"><p>Glasses record people who never agreed to it; faces and licence plates must be anonymized, and biometric laws apply to wearers and bystanders.</p><ul><li><a href="https://arxiv.org/pdf/2308.13093" target="_blank" rel="noopener">EgoBlur</a>: Aria's face and licence-plate blurring, evaluated for fairness across demographics.</li><li><a href="https://arxiv.org/pdf/2110.07058" target="_blank" rel="noopener">Ego4D</a>: consenting participants plus de-identification.</li><li>Law: <a href="https://www.ilga.gov/Legislation/ILCS/Articles?ActID=3004&amp;ChapterID=57" target="_blank" rel="noopener">Illinois BIPA</a>, <a href="https://eur-lex.europa.eu/eli/reg/2016/679/oj" target="_blank" rel="noopener">GDPR</a>; gaze and heart-rate (PPG) data are biometric-adjacent.</li><li>Gap: blurring does not cover voices, reflections or physiological signals.</li></ul></td></tr>
<tr class="t-data"><td class="knob"><a href="https://arxiv.org/pdf/2305.18465" target="_blank" rel="noopener"><strong>Federated</strong></a></td><td class="what"><p>Real user data stays on the device; training, evaluation and analytics run federated, ideally with differential privacy.</p><ul><li><a href="https://arxiv.org/pdf/2305.18465" target="_blank" rel="noopener">Google trained and deployed 20+ Gboard language models</a> with federated learning and differential privacy (DP-FTRL).</li><li><a href="https://machinelearning.apple.com/research/federated-personalization" target="_blank" rel="noopener">Apple runs federated evaluation and tuning</a> for on-device personalization; the data stays on devices.</li><li>Not yet shown for multimodal video + IMU on battery- and heat-limited wearables.</li></ul></td></tr>
<tr class="t-data"><td class="knob"><a href="https://arxiv.org/pdf/2006.13256" target="_blank" rel="noopener"><strong>Domain shift</strong></a></td><td class="what"><p>Models degrade across devices (sensor, sampling rate, placement), users (gait, height, handedness), environments and time; personalization data closes part of the gap.</p><ul><li><a href="https://arxiv.org/pdf/2006.13256" target="_blank" rel="noopener">EPIC-KITCHENS-100</a> includes an unsupervised domain-adaptation challenge and a 'test of time' split recorded two years later.</li><li>IMU activity recognition shifts with sensor placement, sampling rate and the wearer's movement style.</li><li>Factory calibration drifts as glasses deform, heat up and age.</li></ul></td></tr>
<tr class="t-data grp-start"><th scope="rowgroup" class="l3" rowspan="8"><span>Synthetic data</span></th><td class="knob"><a href="https://arxiv.org/pdf/2501.03575" target="_blank" rel="noopener"><strong>World models</strong></a></td><td class="what"><p>Video diffusion and world models generate controllable scenes, rare events and 'what if' variations (lighting, objects, weather).</p><ul><li><a href="https://arxiv.org/pdf/2501.03575" target="_blank" rel="noopener">NVIDIA Cosmos</a>: open platform with video curation, tokenizers, pre-trained world models and post-training recipes.</li><li><a href="https://deepmind.google/blog/genie-3-a-new-frontier-for-world-models/" target="_blank" rel="noopener">Genie 3</a>: interactive worlds at 720p / 24 fps that stay consistent for a few minutes.</li><li>Egocentric: <a href="https://arxiv.org/pdf/2511.18173" target="_blank" rel="noopener">EgoControl</a> (video driven by 3D body pose), <a href="https://arxiv.org/pdf/2512.04515" target="_blank" rel="noopener">EgoLCD</a> (long-context egocentric video).</li></ul></td></tr>
<tr class="t-data"><td class="knob"><a href="https://arxiv.org/pdf/2407.10910" target="_blank" rel="noopener"><strong>LoRA generators</strong></a></td><td class="what"><p>Fine-tune a diffusion model with LoRA on a few real images so its output matches the device's camera and classes.</p><ul><li><a href="https://arxiv.org/pdf/2407.10910" target="_blank" rel="noopener">DataDream</a>: LoRA-tuned generator from few-shot real images; beats the few-shot state of the art on 7 of 10 datasets.</li><li><a href="https://arxiv.org/pdf/2210.07574" target="_blank" rel="noopener">Text-to-image synthetic data helps</a> zero-shot, few-shot and pre-training, but with clear limits.</li></ul></td></tr>
<tr class="t-data"><td class="knob"><a href="https://arxiv.org/pdf/2405.14062" target="_blank" rel="noopener"><strong>Scenario gen.</strong></a></td><td class="what"><p>An LLM or program writes scenario descriptions, including edge cases, which are compiled into simulator code.</p><ul><li><a href="https://arxiv.org/pdf/2405.14062" target="_blank" rel="noopener">ChatScene</a> (driving, CARLA): 15% more collisions found than baselines; fine-tuning on them cut collisions by 9%.</li><li>Transfer to egocentric scenes (falls, hazards, social situations) is still an extrapolation.</li></ul></td></tr>
<tr class="t-data"><td class="knob"><a href="https://facebookresearch.github.io/projectaria_tools/docs/open_datasets/aria_synthetic_environments_dataset" target="_blank" rel="noopener"><strong>3D sim &amp; render</strong></a></td><td class="what"><p>Physically rendered scenes give free, pixel-accurate labels (depth, segmentation, 6-DoF pose); randomizing layouts and assets helps transfer to real data.</p><ul><li><a href="https://facebookresearch.github.io/projectaria_tools/docs/open_datasets/aria_synthetic_environments_dataset" target="_blank" rel="noopener">Aria Synthetic Environments</a>: 100K indoor scenes, ~8K 3D objects, simulated Aria camera and lens.</li><li><a href="https://arxiv.org/pdf/2206.06994" target="_blank" rel="noopener">ProcTHOR</a>: procedurally generated houses for embodied AI.</li><li><a href="https://arxiv.org/pdf/2401.08739" target="_blank" rel="noopener">EgoGen</a>: virtual humans moving by their own egocentric view; localization recall 66.9% → 76.7%, EgoBody pose error −20.7%.</li></ul></td></tr>
<tr class="t-data"><td class="knob"><a href="https://arxiv.org/pdf/2006.05675" target="_blank" rel="noopener"><strong>Sensor sim.</strong></a></td><td class="what"><p>Turn other modalities into IMU, audio or raw-camera signals that match wearable hardware.</p><ul><li>IMU from video (<a href="https://arxiv.org/pdf/2006.05675" target="_blank" rel="noopener">IMUTube</a>) or from text via motion synthesis (<a href="https://arxiv.org/pdf/2402.01049" target="_blank" rel="noopener">IMUGPT 2.0</a>, which halves generation effort with diversity metrics).</li><li><a href="https://arxiv.org/pdf/2506.07612" target="_blank" rel="noopener">Virtual IMU beat classic augmentation</a> on PAMAP2 with 10% real data: +163% vs +35.5%.</li><li>Audio: <a href="https://github.com/facebookresearch/sound-spaces" target="_blank" rel="noopener">SoundSpaces 2.0</a> renders room impulse responses.</li><li>Camera: <a href="https://arxiv.org/pdf/1811.11127" target="_blank" rel="noopener">'Unprocessing'</a> creates realistic raw noisy images; 14–38% lower denoising error.</li></ul></td></tr>
<tr class="t-data"><td class="knob"><a href="https://arxiv.org/pdf/2306.11644" target="_blank" rel="noopener"><strong>LLM distillation</strong></a></td><td class="what"><p>A large teacher model writes pre-training, instruction or multimodal QA data for small models.</p><ul><li><a href="https://arxiv.org/pdf/2306.11644" target="_blank" rel="noopener">phi-1</a> (1.3B): 6B filtered web tokens plus 1B GPT-3.5 synthetic tokens, 50.6% on HumanEval.</li><li><a href="https://huggingface.co/datasets/HuggingFaceTB/cosmopedia" target="_blank" rel="noopener">Cosmopedia</a>: 25B tokens generated by Mixtral-8x7B, used to train the Cosmo-1B small model.</li><li><a href="https://arxiv.org/pdf/2304.08485" target="_blank" rel="noopener">LLaVA</a>: 158K GPT-4-generated multimodal instruction samples (conversations, descriptions, reasoning).</li></ul></td></tr>
<tr class="t-data"><td class="knob"><a href="https://arxiv.org/pdf/1706.00527" target="_blank" rel="noopener"><strong>Augmentation</strong></a></td><td class="what"><p>Cheap, label-preserving transforms of real signals; still the first thing to try.</p><ul><li>IMU: rotation, permutation and time-warp <a href="https://arxiv.org/pdf/1706.00527" target="_blank" rel="noopener">raised accuracy from 77.5% to 86.9%</a> on Parkinson's data; jitter and scaling did not help.</li><li>Images and video: crops, colour, blur and mixup; audio: noise and reverberation.</li></ul></td></tr>
<tr class="t-data"><td class="knob"><a href="https://arxiv.org/pdf/2305.15560" target="_blank" rel="noopener"><strong>Private synthesis</strong></a></td><td class="what"><p>Differentially private synthetic data, and replacing bystanders in always-on footage.</p><ul><li><a href="https://arxiv.org/pdf/2305.15560" target="_blank" rel="noopener">Private Evolution</a>: DP synthetic data through model APIs alone, FID ≤ 7.9 at ε = 0.67 on CIFAR-10; <a href="https://arxiv.org/pdf/2502.05505" target="_blank" rel="noopener">a later version uses simulators</a>.</li><li><a href="https://arxiv.org/pdf/2211.09454" target="_blank" rel="noopener">DeepPrivacy2</a>: GAN-based anonymization of full bodies and faces.</li><li>Formal guarantees for synthetic egocentric video are still open.</li></ul></td></tr>
<tr class="t-data grp-start"><th scope="rowgroup" class="l3" rowspan="7"><span>Curation &amp; eval</span></th><td class="knob"><a href="https://arxiv.org/pdf/2402.01049" target="_blank" rel="noopener"><strong>Filter &amp; dedup</strong></a></td><td class="what"><p>Remove duplicates, low-quality and unsafe samples, and verify synthetic samples before use; small models cannot absorb noisy data.</p><ul><li>Verification filters for generated data (e.g., <a href="https://arxiv.org/pdf/2402.01049" target="_blank" rel="noopener">IMUGPT's motion filter</a>) catch physically implausible samples.</li><li>Deduplicate across real and synthetic sources to avoid leaking eval data into training.</li></ul></td></tr>
<tr class="t-data"><td class="knob"><a href="https://arxiv.org/pdf/2509.24945" target="_blank" rel="noopener"><strong>Data selection</strong></a></td><td class="what"><p>Quality scoring and mixture design decide what a small model sees; data quality can substitute for data volume.</p><ul><li><a href="https://arxiv.org/pdf/2509.24945" target="_blank" rel="noopener">MobileLLM-R1</a>: ~2T high-quality tokens were enough; 4.2T pre-training tokens via resampling (~11.7% of <a href="https://arxiv.org/pdf/2505.09388" target="_blank" rel="noopener">Qwen3's 36T</a>); the 950M model scores 15.5 on AIME.</li></ul></td></tr>
<tr class="t-data"><td class="knob"><a href="https://www.nature.com/articles/s41586-024-07566-y.pdf" target="_blank" rel="noopener"><strong>Real : synth mix</strong></a></td><td class="what"><p>The ratio of real to synthetic data matters; synthetic data should add to real data, not replace it.</p><ul><li>Model collapse: training recursively on generated data erodes the tails of the distribution (<a href="https://www.nature.com/articles/s41586-024-07566-y.pdf" target="_blank" rel="noopener">Shumailov et al., Nature 2024</a>).</li><li>Accumulating synthetic data alongside real data avoids collapse (<a href="https://arxiv.org/pdf/2404.01413" target="_blank" rel="noopener">Gerstgrasser et al., 2024</a>).</li><li>The right ratio for models of 1B parameters or less is unknown.</li></ul></td></tr>
<tr class="t-data"><td class="knob"><a href="https://arxiv.org/pdf/2006.13256" target="_blank" rel="noopener"><strong>Long-tail coverage</strong></a></td><td class="what"><p>Everyday egocentric data is dominated by a few common actions; rare but important events (falls, hazards, unusual objects) are sparse.</p><ul><li>Report tail-class and unseen-participant results separately (as <a href="https://arxiv.org/pdf/2006.13256" target="_blank" rel="noopener">EPIC-KITCHENS-100</a> does).</li><li>Target synthetic generation and extra collection at the tail.</li></ul></td></tr>
<tr class="t-data"><td class="knob"><a href="https://arxiv.org/pdf/2308.09126" target="_blank" rel="noopener"><strong>Egocentric evals</strong></a></td><td class="what"><p>Held-out real benchmarks for long-horizon memory and question answering, which is what an assistant on glasses needs.</p><ul><li><a href="https://arxiv.org/pdf/2308.09126" target="_blank" rel="noopener">EgoSchema</a>: 5,000+ questions over 3-minute clips; humans ~76%, models under 33% at release.</li><li><a href="https://arxiv.org/pdf/2411.04998" target="_blank" rel="noopener">HourVideo</a>: 12,976 questions over 20–120 min videos; experts 85.0%, Gemini 1.5 Pro 37.3%.</li><li><a href="https://arxiv.org/pdf/2502.04144" target="_blank" rel="noopener">HD-EPIC VQA</a>: 26K questions; Gemini Pro 38.5%. <a href="https://arxiv.org/pdf/2503.03803" target="_blank" rel="noopener">EgoLifeQA</a>: memory recall and habit questions.</li></ul></td></tr>
<tr class="t-data"><td class="knob"><strong><strong>Compute-aware eval</strong></strong></td><td class="what"><p>None of the egocentric benchmarks measure latency or energy; wearables need task success within the budget.</p><ul><li>Report accuracy together with time to first response (&lt; 500 ms), energy per task and behaviour under thermal throttling.</li><li>Measure on the actual device, including capture, speech and preprocessing.</li></ul></td></tr>
<tr class="t-data"><td class="knob"><a href="https://www.projectaria.com/datasets/aea/license/" target="_blank" rel="noopener"><strong>Provenance</strong></a></td><td class="what"><p>Track where each sample came from and what its licence allows; most egocentric corpora are research-only.</p><ul><li><a href="https://www.projectaria.com/datasets/aea/license/" target="_blank" rel="noopener">Aria Everyday Activities licence</a>: non-commercial, no identifying people, no redistribution.</li><li><a href="https://arxiv.org/pdf/2510.16134" target="_blank" rel="noopener">Aria Gen 2 pilot data</a>: CC BY-NC-SA 4.0.</li><li>Product teams must collect their own data under their own consent.</li></ul></td></tr>
<tr class="op-row t-data"><td colspan="6" class="op op-data"><div class="op-title">Open problems · Data</div><ul><li><strong>Commercial gap:</strong> the big egocentric datasets are non-commercial, so there is no shared public baseline for shipped glasses.</li><li><strong>Bystander consent at scale:</strong> blurring misses voices, reflections, gaze and heart-rate signals; anonymizing on the device before storage is unsolved.</li><li><strong>Sensor-faithful generation:</strong> generators rarely reproduce rolling shutter, fisheye lenses, low-light noise or IMU drift, or keep video, IMU, audio and gaze consistent.</li><li><strong>Small models and synthetic data:</strong> the right real:synthetic ratio for ≤1B models is unknown, and on-device personalization risks recursive collapse.</li><li><strong>Compute-aware benchmarks:</strong> egocentric benchmarks score accuracy only, never the &lt; 500 ms / ~2 W regime.</li></ul></td></tr>
<tr class="t-comp grp-start"><th scope="rowgroup" class="l2 l2-band" rowspan="33"><span>COMPRESSION</span></th><th scope="rowgroup" class="l3" rowspan="10"><span>Quantization</span></th><td class="knob"><strong><strong>FP16</strong></strong></td><td class="what">16-bit reference precision: 2 bytes per weight (a 2B model is 4 GB). Keep it for sensitive parts or as the baseline.</td><td class="op op-comp" rowspan="33"><ul><li><strong>Portable low-bit execution:</strong> 2-bit and ternary storage often lacks equally efficient arithmetic and kernels on mobile NPUs.</li><li><strong>Kernel fragmentation:</strong> group-wise scales, 2:4 sparsity and rotations speed up GPUs but often not mobile or wearable NPUs.</li><li><strong>Well-trained small models:</strong> the harm from post-training quantization grows with training data, so the smallest, best-trained models are the hardest to compress.</li><li><strong>Compression order:</strong> distill → prune → QAT → adapters is still largely empirical; changing one step invalidates the others.</li><li><strong>Adapters on quantized bases:</strong> keeping many adapters accurate after merging and quantization, and invalidating cached KV when weights change.</li></ul></td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2208.07339" target="_blank" rel="noopener"><strong>INT8</strong></a></td><td class="what">8-bit weights and/or activations (<a href="https://arxiv.org/pdf/2208.07339" target="_blank" rel="noopener">LLM.int8()</a>, <a href="https://arxiv.org/pdf/2211.10438" target="_blank" rel="noopener">SmoothQuant</a> W8A8): close to lossless, half the memory, and a good match for integer NPUs.</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2210.17323" target="_blank" rel="noopener"><strong>INT4 · default</strong></a></td><td class="what">4-bit weights (<a href="https://arxiv.org/pdf/2210.17323" target="_blank" rel="noopener">GPTQ</a>, <a href="https://arxiv.org/pdf/2306.00978" target="_blank" rel="noopener">AWQ</a>, or QAT as in <a href="https://developers.googleblog.com/en/gemma-3-quantized-aware-trained-state-of-the-art-ai-to-consumer-gpus/" target="_blank" rel="noopener">Gemma 3</a> / <a href="https://ai.meta.com/blog/meta-llama-quantized-lightweight-models/" target="_blank" rel="noopener">Llama 3.2</a>): about 4× smaller than FP16 with small loss. The safe post-training default.</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2404.14047" target="_blank" rel="noopener"><strong>3-bit</strong></a></td><td class="what">Post-training quantization starts to hurt (about −4 points on average for <a href="https://arxiv.org/pdf/2404.14047" target="_blank" rel="noopener">3-bit AWQ on Llama-3-8B</a>); small and reasoning models degrade most (<a href="https://arxiv.org/pdf/2504.04823" target="_blank" rel="noopener">R1-Distill-1.5B</a>: −18.6 points with 3-bit AWQ).</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2401.06118" target="_blank" rel="noopener"><strong>2-bit + QAT</strong></a></td><td class="what">Post-training 2-bit collapses; with QAT or vector quantization plus fine-tuning (<a href="https://arxiv.org/pdf/2401.06118" target="_blank" rel="noopener">AQLM</a>, <a href="https://arxiv.org/pdf/2405.14852" target="_blank" rel="noopener">PV-Tuning</a>, <a href="https://arxiv.org/pdf/2502.02631" target="_blank" rel="noopener">ParetoQ</a>) it becomes Pareto-optimal. <a href="https://arxiv.org/pdf/2507.13575" target="_blank" rel="noopener">Apple's on-device model</a> uses 2-bit QAT.</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2504.12285" target="_blank" rel="noopener"><strong>1.58-bit</strong></a></td><td class="what">Ternary weights (−1, 0, +1) trained natively (<a href="https://arxiv.org/pdf/2504.12285" target="_blank" rel="noopener">BitNet b1.58 2B4T</a>): 0.4 GB of non-embedding weights and 0.028 J per token, but it needs custom kernels.</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2306.07629" target="_blank" rel="noopener"><strong>Mixed-bit</strong></a></td><td class="what">Give sensitive layers or channels more bits and the rest fewer (<a href="https://arxiv.org/pdf/2306.07629" target="_blank" rel="noopener">SqueezeLLM</a>, <a href="https://arxiv.org/pdf/2405.14917" target="_blank" rel="noopener">SliM-LLM</a>); nested formats (<a href="https://arxiv.org/pdf/2502.06786" target="_blank" rel="noopener">MatQuant</a>) serve int8, int4 and int2 from one model.</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2405.04532" target="_blank" rel="noopener"><strong>W4A8 / W4A16</strong></a></td><td class="what">Weight vs activation bits. W4A16 keeps activations at 16-bit (simple, accurate); W4A8 uses integer activations for faster NPU math (<a href="https://arxiv.org/pdf/2405.04532" target="_blank" rel="noopener">QServe</a>), at some accuracy risk.</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2509.23324" target="_blank" rel="noopener"><strong>Group size</strong></a></td><td class="what">One scale per group of 32–128 weights preserves accuracy, but some NPUs support only per-channel scales (<a href="https://arxiv.org/pdf/2509.23324" target="_blank" rel="noopener">Hexagon</a>: Llama-3.2-1B scores 2.1% vs 15.9% on MATH500).</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2404.00456" target="_blank" rel="noopener"><strong>Rotations</strong></a></td><td class="what">Multiply by rotation matrices (<a href="https://arxiv.org/pdf/2404.00456" target="_blank" rel="noopener">QuaRot</a>, <a href="https://arxiv.org/pdf/2405.16406" target="_blank" rel="noopener">SpinQuant</a>) to spread activation outliers, enabling 4-bit weights, activations and KV cache; the runtime must support or fuse them.</td></tr>
<tr class="t-comp grp-start"><th scope="rowgroup" class="l3" rowspan="8"><span>Pruning &amp; sparsity</span></th><td class="knob"><a href="https://arxiv.org/pdf/2305.11627" target="_blank" rel="noopener"><strong>Structured</strong></a></td><td class="what">Remove whole channels, heads or blocks, then retrain; the result is a smaller dense model that standard kernels accelerate (<a href="https://arxiv.org/pdf/2305.11627" target="_blank" rel="noopener">LLM-Pruner</a>, <a href="https://arxiv.org/pdf/2401.15024" target="_blank" rel="noopener">SliceGPT</a>).</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2301.00774" target="_blank" rel="noopener"><strong>Unstructured</strong></a></td><td class="what">Zero individual weights (<a href="https://arxiv.org/pdf/2301.00774" target="_blank" rel="noopener">SparseGPT</a>: 50–60% in one shot; <a href="https://arxiv.org/pdf/2306.11695" target="_blank" rel="noopener">Wanda</a> needs no weight updates); saves little on devices unless the kernel actually skips the zeros.</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2104.08378" target="_blank" rel="noopener"><strong>N:M (2:4)</strong></a></td><td class="what">At most 2 non-zeros in each group of 4 weights; fast on NVIDIA sparse tensor cores, but support on mobile NPUs is limited.</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/1711.02782" target="_blank" rel="noopener"><strong>Block</strong></a></td><td class="what">Zero whole tiles (e.g., 16×16) so kernels can skip them; the speedup depends on block size, sparsity level and device support.</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/1905.10650" target="_blank" rel="noopener"><strong>Channel / head</strong></a></td><td class="what">Remove feed-forward channels or attention heads and physically shrink the tensors; keep dimensions aligned to the accelerator's tile sizes.</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2403.03853" target="_blank" rel="noopener"><strong>Layer drop</strong></a></td><td class="what">Remove whole layers ranked by low importance (<a href="https://arxiv.org/pdf/2403.03853" target="_blank" rel="noopener">ShortGPT</a>); simple and stacks with quantization, but deep cuts need recovery training.</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2407.14679" target="_blank" rel="noopener"><strong>Width / depth</strong></a></td><td class="what">Prune hidden width and depth together, then distill from the original (<a href="https://arxiv.org/pdf/2407.14679" target="_blank" rel="noopener">Minitron</a>: 40× fewer training tokens than training from scratch).</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2310.17157" target="_blank" rel="noopener"><strong>Activation sp.</strong></a></td><td class="what">Activation sparsity: skip neurons whose activations are near zero for the current input (<a href="https://arxiv.org/pdf/2310.17157" target="_blank" rel="noopener">Deja Vu</a>, <a href="https://arxiv.org/pdf/2408.14690" target="_blank" rel="noopener">TEAL</a>, <a href="https://arxiv.org/pdf/2402.13516" target="_blank" rel="noopener">ProSparse</a> at ~89%); pays off with CPU/NPU co-design or flash offload (<a href="https://arxiv.org/pdf/2406.06282" target="_blank" rel="noopener">PowerInfer-2</a>).</td></tr>
<tr class="t-comp grp-start"><th scope="rowgroup" class="l3" rowspan="8"><span>Fine-tuning (PEFT)</span></th><td class="knob"><a href="https://arxiv.org/pdf/2106.09685" target="_blank" rel="noopener"><strong>LoRA</strong></a></td><td class="what">Low-Rank Adaptation: freeze the base model and learn a low-rank update B·A (rank 8–64, under 1% of the weights); merge it into the weights or keep it as a swappable adapter.</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2305.14314" target="_blank" rel="noopener"><strong>QLoRA (NF4)</strong></a></td><td class="what">Quantized LoRA: LoRA on a 4-bit NormalFloat (NF4) base with double quantization and paged optimizers. A training-side technique: adapters are built off-device, then shipped to the wearable. NF4 is a training format, not an NPU format.</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2402.09353" target="_blank" rel="noopener"><strong>DoRA</strong></a></td><td class="what">Weight-Decomposed Low-Rank Adaptation: splits each weight into magnitude and direction and applies LoRA only to the direction; consistently beats LoRA and merges with no inference cost.</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2501.04315" target="_blank" rel="noopener"><strong>RoRA (α/√r)</strong></a></td><td class="what">Reliability Optimization for Rank Adaptation: changes LoRA's scaling from α/r to α/√r so accuracy keeps rising with rank; notable gains on pruned models (+5.7% over LoRA on <a href="https://arxiv.org/pdf/2310.06694" target="_blank" rel="noopener">Sheared-LLaMA-1.3B</a>).</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2309.14717" target="_blank" rel="noopener"><strong>QA-LoRA</strong></a></td><td class="what">Quantization-Aware LoRA: a LoRA variant whose update merges cleanly into an INT4 model, avoiding a precision mismatch after merging.</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2310.08659" target="_blank" rel="noopener"><strong>LoftQ</strong></a></td><td class="what">LoRA-Fine-Tuning-aware Quantization: initializes LoRA jointly with quantization so the adapter starts by correcting quantization error; strongest at 2-bit and mixed 2/4-bit.</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2312.03732" target="_blank" rel="noopener"><strong>Rank r</strong></a></td><td class="what">The adapter's size and capacity; higher rank adds parameters and memory. With standard scaling, gains can plateau as rank grows (the motivation for <a href="https://arxiv.org/pdf/2501.04315" target="_blank" rel="noopener">RoRA</a>).</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://machinelearning.apple.com/research/introducing-apple-foundation-models" target="_blank" rel="noopener"><strong>Adapter swap</strong></a></td><td class="what">Keep one quantized base model and load small task adapters on demand (<a href="https://machinelearning.apple.com/research/introducing-apple-foundation-models" target="_blank" rel="noopener">Apple: rank-16 adapters</a> of tens of MB) instead of storing several full models.</td></tr>
<tr class="t-comp grp-start"><th scope="rowgroup" class="l3" rowspan="7"><span>Distillation</span></th><td class="knob"><a href="https://arxiv.org/pdf/1503.02531" target="_blank" rel="noopener"><strong>Logit KD</strong></a></td><td class="what">Train the student to match the teacher's output probability distribution, which carries more signal than hard labels.</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/1412.6550" target="_blank" rel="noopener"><strong>Feature KD</strong></a></td><td class="what">Also match intermediate hidden states or attention maps; useful after pruning, when student and teacher share structure.</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/1606.07947" target="_blank" rel="noopener"><strong>Teacher outputs</strong></a></td><td class="what">Fine-tune on answers generated by a larger model (sequence-level distillation); simple and effective for narrow tasks.</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2502.12067" target="_blank" rel="noopener"><strong>Short CoT</strong></a></td><td class="what">Distill concise reasoning traces so the student reasons in fewer tokens, directly cutting decode time and energy.</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2301.12726" target="_blank" rel="noopener"><strong>Task narrowing</strong></a></td><td class="what">Train a small specialist for the device's actual tasks (intents, captions, OCR) rather than a generalist; often a bigger win than another bit of quantization.</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2407.06023" target="_blank" rel="noopener"><strong>System 2 → 1</strong></a></td><td class="what">Distill answers that needed chain-of-thought into direct answers (<a href="https://arxiv.org/pdf/2407.06023" target="_blank" rel="noopener">Meta, 2024</a>), baking repeated reasoning into the weights; works for many tasks, not multi-step math.</td></tr>
<tr class="t-comp"><td class="knob"><a href="https://arxiv.org/pdf/2407.14679" target="_blank" rel="noopener"><strong>Prune + KD</strong></a></td><td class="what">Prune the model, then distill from the unpruned original to recover quality (<a href="https://arxiv.org/pdf/2407.14679" target="_blank" rel="noopener">Minitron</a>, <a href="https://arxiv.org/pdf/2506.18349" target="_blank" rel="noopener">SlimMoE</a>); <a href="https://arxiv.org/pdf/2310.06694" target="_blank" rel="noopener">Sheared-LLaMA</a> recovers with continued pre-training instead.</td></tr>
<tr class="op-row t-comp"><td colspan="6" class="op op-comp"><div class="op-title">Open problems · Compression</div><ul><li><strong>Portable low-bit execution:</strong> 2-bit and ternary storage often lacks equally efficient arithmetic and kernels on mobile NPUs.</li><li><strong>Kernel fragmentation:</strong> group-wise scales, 2:4 sparsity and rotations speed up GPUs but often not mobile or wearable NPUs.</li><li><strong>Well-trained small models:</strong> the harm from post-training quantization grows with training data, so the smallest, best-trained models are the hardest to compress.</li><li><strong>Compression order:</strong> distill → prune → QAT → adapters is still largely empirical; changing one step invalidates the others.</li><li><strong>Adapters on quantized bases:</strong> keeping many adapters accurate after merging and quantization, and invalidating cached KV when weights change.</li></ul></td></tr>
<tr class="t-inf grp-start"><th scope="rowgroup" class="l1 l1-inf" rowspan="45"><span>INFERENCE</span></th><th scope="rowgroup" class="l2" colspan="2" rowspan="9"><span>Attention</span></th><td class="knob"><a href="https://arxiv.org/pdf/1706.03762" target="_blank" rel="noopener"><strong>MHA</strong></a></td><td class="what">Multi-head attention: every query head has its own key/value head. Best quality, largest KV cache.</td><td class="op op-inf" rowspan="45"><ul><li><strong>Router calibration:</strong> query-value estimators trained on chat data must handle egocentric, multimodal intents under battery, thermal and network shifts.</li><li><strong>Speculation energy:</strong> speedups can raise energy per token when acceptance drops; wearable-scale evidence is thin.</li><li><strong>KV eviction vs recall:</strong> bounded caches forget, and there are no guarantees for recalling earlier scenes.</li><li><strong>Wearable–companion–cloud split:</strong> radio start-up, queueing and disconnects dominate tail latency; per-token cross-device protocols are fragile.</li><li><strong>No glasses-class measurements:</strong> tokens/s, joules per token and thermal curves for AR1-class chips remain unpublished.</li></ul></td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/1911.02150" target="_blank" rel="noopener"><strong>MQA</strong></a></td><td class="what">Multi-query attention: all query heads share a single key/value head. Smallest cache, some quality loss.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2305.13245" target="_blank" rel="noopener"><strong>GQA</strong></a></td><td class="what">Grouped-query attention: groups of query heads share key/value heads (e.g., 32 → 8, a 4× smaller cache); existing models can be <a href="https://arxiv.org/pdf/2305.13245" target="_blank" rel="noopener">uptrained with ~5% of pretraining compute</a>.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2405.04434" target="_blank" rel="noopener"><strong>MLA</strong></a></td><td class="what">Multi-head latent attention (<a href="https://arxiv.org/pdf/2405.04434" target="_blank" rel="noopener">DeepSeek</a>): cache one small compressed vector per token (−93% KV); existing models can be converted (<a href="https://arxiv.org/pdf/2502.14837" target="_blank" rel="noopener">MHA2MLA</a>, <a href="https://arxiv.org/pdf/2502.07864" target="_blank" rel="noopener">TransMLA</a>).</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2503.19786" target="_blank" rel="noopener"><strong>Local : global</strong></a></td><td class="what">Interleave sliding-window layers with occasional full-attention layers (<a href="https://arxiv.org/pdf/2503.19786" target="_blank" rel="noopener">Gemma 3</a>: 5:1 with a 1,024-token window), cutting KV memory overhead from ~60% to under 15%.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2004.05150" target="_blank" rel="noopener"><strong>Window size</strong></a></td><td class="what">How many recent tokens a local layer sees (e.g., 512–4,096). Smaller windows save memory and compute; long-range recall then relies on the global layers.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2502.11089" target="_blank" rel="noopener"><strong>Sparse (NSA)</strong></a></td><td class="what"><a href="https://arxiv.org/pdf/2502.11089" target="_blank" rel="noopener">Native sparse attention</a>: each token attends to a compressed summary, a few selected blocks and a local window; the model is trained sparse from the start.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2502.13189" target="_blank" rel="noopener"><strong>Block (MoBA)</strong></a></td><td class="what"><a href="https://arxiv.org/pdf/2502.13189" target="_blank" rel="noopener">Mixture of block attention</a>: a gate picks the top-k context blocks for each query, like MoE routing over the context. Saves compute, not KV storage.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2406.15786" target="_blank" rel="noopener"><strong>Skip attn</strong></a></td><td class="what">Remove attention from some layers entirely (<a href="https://arxiv.org/pdf/2406.15786" target="_blank" rel="noopener">dropping half of Llama-2-70B's</a>: 48% faster for −2.4%, shown at server scale); <a href="https://arxiv.org/pdf/2603.15954" target="_blank" rel="noopener">MobileLLM-Flash</a> searches for where skipping is safe.</td></tr>
<tr class="t-inf grp-start"><th scope="rowgroup" class="l2" colspan="2" rowspan="9"><span>KV cache</span></th><td class="knob"><a href="https://arxiv.org/pdf/2507.13575" target="_blank" rel="noopener"><strong>8-bit KV</strong></a></td><td class="what">Store keys and values in 8-bit: half the KV memory with negligible loss. The shipped default in <a href="https://arxiv.org/pdf/2507.13575" target="_blank" rel="noopener">Apple's on-device model</a>; <a href="https://arxiv.org/pdf/2607.02770" target="_blank" rel="noopener">the Gemma 4 report</a> sizes memory with an int8 KV cache.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2504.04823" target="_blank" rel="noopener"><strong>4-bit KV</strong></a></td><td class="what">Another 2× saving; near-lossless even for reasoning models <a href="https://arxiv.org/pdf/2504.04823" target="_blank" rel="noopener">in reported tests</a>, with extra packing and dequantization overhead.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2402.02750" target="_blank" rel="noopener"><strong>2-bit KV</strong></a></td><td class="what">Aggressive (<a href="https://arxiv.org/pdf/2402.02750" target="_blank" rel="noopener">KIVI</a>: keys per channel, values per token): 2.6× less peak memory, but more sensitive on retrieval-heavy tasks.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2309.17453" target="_blank" rel="noopener"><strong>Sinks + window</strong></a></td><td class="what">Keep the first few 'attention sink' tokens plus a recent window (<a href="https://arxiv.org/pdf/2309.17453" target="_blank" rel="noopener">StreamingLLM</a>): bounded memory for always-on streams, but older details are dropped.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2306.14048" target="_blank" rel="noopener"><strong>Eviction</strong></a></td><td class="what">Keep only important tokens (<a href="https://arxiv.org/pdf/2306.14048" target="_blank" rel="noopener">H2O</a> heavy hitters, <a href="https://arxiv.org/pdf/2404.14469" target="_blank" rel="noopener">SnapKV</a>, <a href="https://arxiv.org/pdf/2406.02069" target="_blank" rel="noopener">PyramidKV</a> at 12% of the cache) and drop the rest; selection overhead and changing queries can hurt recall.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2603.04428" target="_blank" rel="noopener"><strong>Prefix reuse</strong></a></td><td class="what">Cache the KV of recurring prompts (system prompt, agent context) and reload it; <a href="https://arxiv.org/pdf/2603.04428" target="_blank" rel="noopener">one study</a> restored a context in 577 ms instead of 15.7 s.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2405.12981" target="_blank" rel="noopener"><strong>KV sharing</strong></a></td><td class="what">Adjacent layers reuse one KV cache (<a href="https://arxiv.org/pdf/2405.12981" target="_blank" rel="noopener">CLA</a>, <a href="https://developers.googleblog.com/en/introducing-gemma-3n-developer-guide/" target="_blank" rel="noopener">Gemma 3n</a> and <a href="https://arxiv.org/pdf/2607.02770" target="_blank" rel="noopener">Gemma 4</a>, <a href="https://arxiv.org/pdf/2507.13575" target="_blank" rel="noopener">Apple's on-device model</a>: −37.5% KV); must be trained in, not bolted on.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2609.21172" target="_blank" rel="noopener"><strong>Flash tiering</strong></a></td><td class="what">Keep hot KV in RAM and move cold KV to flash or a low-rank form (<a href="https://arxiv.org/pdf/2609.21172" target="_blank" rel="noopener">TierKV</a>: up to 17.6× faster prefill; phone-measured).</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://developer.apple.com/forums/thread/806542" target="_blank" rel="noopener"><strong>Context ≤ 4K</strong></a></td><td class="what">Shipped assistants on phones and browsers cap context near 4K tokens (<a href="https://developer.apple.com/forums/thread/806542" target="_blank" rel="noopener">Apple on-device model: 4,096</a>; <a href="https://groups.google.com/a/chromium.org/g/chrome-ai-dev-preview-discuss/c/WO2NIK_9Ue4" target="_blank" rel="noopener">Gemini Nano in Chrome</a>: 4,096 per session); wearable budgets are tighter. A 128K advertised context is a ceiling, not a default.</td></tr>
<tr class="t-inf grp-start"><th scope="rowgroup" class="l2" colspan="2" rowspan="9"><span>Speculative decoding</span></th><td class="knob"><a href="https://arxiv.org/pdf/2211.17192" target="_blank" rel="noopener"><strong>Draft model</strong></a></td><td class="what">A small model proposes several tokens and the target verifies them in one pass; needs a second model and its KV cache resident in memory.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2404.16710" target="_blank" rel="noopener"><strong>Self-spec</strong></a></td><td class="what">Draft with the target's own early layers and verify with the rest (<a href="https://arxiv.org/pdf/2404.16710" target="_blank" rel="noopener">LayerSkip</a>, <a href="https://arxiv.org/pdf/2309.08168" target="_blank" rel="noopener">Draft &amp; Verify</a>), so no second model is needed in memory.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://research.google/blog/accelerating-gemini-nano-models-on-pixel-with-frozen-multi-token-prediction/" target="_blank" rel="noopener"><strong>MTP heads</strong></a></td><td class="what">Multi-token prediction (MTP): extra heads predict several future tokens at once; <a href="https://research.google/blog/accelerating-gemini-nano-models-on-pixel-with-frozen-multi-token-prediction/" target="_blank" rel="noopener">Gemini Nano v3 on Pixel</a> is 50%+ faster than with a separate drafter and saves 130 MB (phone-measured).</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2406.16858" target="_blank" rel="noopener"><strong>EAGLE heads</strong></a></td><td class="what">Extrapolation Algorithm for Greater Language-model Efficiency (EAGLE): a light head drafts from the target's hidden features (<a href="https://arxiv.org/pdf/2406.16858" target="_blank" rel="noopener">EAGLE-2</a> and <a href="https://arxiv.org/pdf/2503.01840" target="_blank" rel="noopener">EAGLE-3</a>: 3–6.5×, GPU-measured; gains on phones are smaller); a strong drafter with a small memory footprint.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://github.com/apoorvumang/prompt-lookup-decoding" target="_blank" rel="noopener"><strong>Prompt lookup</strong></a></td><td class="what">Draft by copying n-grams from the prompt or context; free, and effective for summarizing, editing and grounded answers.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://ieeexplore.ieee.org/document/10812936" target="_blank" rel="noopener"><strong>Token trees</strong></a></td><td class="what">Verify a tree of candidate continuations in one pass instead of a single chain (<a href="https://ieeexplore.ieee.org/document/10812936" target="_blank" rel="noopener">EdgeLLM</a>, <a href="https://arxiv.org/pdf/2406.16858" target="_blank" rel="noopener">EAGLE-2</a>), raising the tokens accepted per step.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2211.17192" target="_blank" rel="noopener"><strong>Draft length k</strong></a></td><td class="what">Tokens proposed per round; longer drafts pay off only when acceptance is high and verification cost grows slowly with k.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2211.17192" target="_blank" rel="noopener"><strong>Accept rate</strong></a></td><td class="what">The fraction of drafted tokens the target accepts: the main driver of speedup, and of whether energy per token goes up or down.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2505.21594" target="_blank" rel="noopener"><strong>Cross-device</strong></a></td><td class="what">Draft on the wearable and verify on the companion or in the cloud (−35% latency in <a href="https://arxiv.org/pdf/2505.21594" target="_blank" rel="noopener">one study</a>); every round adds radio and queueing delay.</td></tr>
<tr class="t-inf grp-start"><th scope="rowgroup" class="l2" colspan="2" rowspan="8"><span>Routing</span></th><td class="knob"><a href="https://arxiv.org/pdf/2406.18665" target="_blank" rel="noopener"><strong>Query value</strong></a></td><td class="what">Estimate how much a bigger model would improve this request, weighed against latency, energy and privacy; learned routers (<a href="https://arxiv.org/pdf/2406.18665" target="_blank" rel="noopener">RouteLLM</a>) <a href="https://www.lmsys.org/blog/2024-07-01-routellm/" target="_blank" rel="noopener">cut cost by over 85% on MT-Bench</a> while keeping 95% of GPT-4's quality.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2404.14618" target="_blank" rel="noopener"><strong>Difficulty est.</strong></a></td><td class="what">Predict up front whether the small model will do well enough (<a href="https://arxiv.org/pdf/2404.14618" target="_blank" rel="noopener">Hybrid LLM</a>: 40% fewer large-model calls with no quality drop).</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2305.05176" target="_blank" rel="noopener"><strong>Cascades</strong></a></td><td class="what">Try the small model first and escalate if a check fails (<a href="https://arxiv.org/pdf/2305.05176" target="_blank" rel="noopener">FrugalGPT</a>, <a href="https://arxiv.org/pdf/2310.12963" target="_blank" rel="noopener">AutoMix</a>): cheap on easy queries, but hard queries pay for both models.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2505.21600" target="_blank" rel="noopener"><strong>Token-level</strong></a></td><td class="what">Send only the critical tokens to the large model (<a href="https://arxiv.org/pdf/2505.21600" target="_blank" rel="noopener">R2R</a>: a 1.5B + 32B pair is 2.8× faster than the 32B model alone at similar accuracy; the 32B model sits off-device).</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2505.09388" target="_blank" rel="noopener"><strong>Think budget</strong></a></td><td class="what">Decide how much reasoning a query deserves (<a href="https://arxiv.org/pdf/2505.09388" target="_blank" rel="noopener">thinking vs non-thinking modes</a>, <a href="https://arxiv.org/pdf/2501.19393" target="_blank" rel="noopener">token budgets</a>) as part of the same routing decision.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2404.10136" target="_blank" rel="noopener"><strong>Confidence gate</strong></a></td><td class="what">Escalate or stop <a href="https://arxiv.org/pdf/2404.10136" target="_blank" rel="noopener">based on the model's confidence</a>; thresholds need calibrating on the deployed model, since quantization and distillation change its confidence.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2508.11291" target="_blank" rel="noopener"><strong>Where to run</strong></a></td><td class="what">Choose among the wearable, its companion phone and the cloud, weighing network latency, availability and privacy; keep an offline path for time-critical actions.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2603.23640" target="_blank" rel="noopener"><strong>Battery-aware</strong></a></td><td class="what">Factor battery level and temperature into the decision; sustained load can cut throughput ~40% (<a href="https://arxiv.org/pdf/2603.23640" target="_blank" rel="noopener">iPhone 16 Pro: 40.5 → 23.7 tok/s</a>, phone-measured; glasses throttle sooner).</td></tr>
<tr class="t-inf grp-start"><th scope="rowgroup" class="l2" colspan="2" rowspan="10"><span>Input &amp; system</span></th><td class="knob"><a href="https://arxiv.org/pdf/2407.05858" target="_blank" rel="noopener"><strong>NPU placement</strong></a></td><td class="what">Run supported operators on the NPU, DSP or GPU and keep large graph segments on one processor to avoid transfers and synchronization.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://developers.google.com/edge/litert/performance/gpu" target="_blank" rel="noopener"><strong>No CPU fallback</strong></a></td><td class="what">An unsupported operator forces a fallback to the CPU; <a href="https://developers.google.com/edge/litert/performance/gpu" target="_blank" rel="noopener">fragmented execution can be slower than running everything on the CPU</a>. Build only from supported ops.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2205.14135" target="_blank" rel="noopener"><strong>Op fusion</strong></a></td><td class="what">Merge adjacent operations (e.g., matrix multiply + activation, fused attention) into one kernel, cutting memory traffic and dispatch overhead.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/1712.05877" target="_blank" rel="noopener"><strong>BN folding</strong></a></td><td class="what">Fold batch normalization into the preceding convolution's weights and bias; mathematically exact, and removes a separate pass over the tensor.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://docs.pytorch.org/executorch/stable/index.html" target="_blank" rel="noopener"><strong>AOT compile</strong></a></td><td class="what">Compile and optimize the graph ahead of time for the target runtime (<a href="https://docs.pytorch.org/executorch/stable/index.html" target="_blank" rel="noopener">ExecuTorch</a>, <a href="https://docs.qualcomm.com/bundle/publicresource/topics/80-63442-50/introduction.html" target="_blank" rel="noopener">QNN</a>, <a href="https://developer.apple.com/documentation/coreml" target="_blank" rel="noopener">Core ML</a>), reducing startup and dispatch work.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2001.03288" target="_blank" rel="noopener"><strong>Buffer reuse</strong></a></td><td class="what">Preallocate and reuse memory buffers instead of allocating on every inference; lowers peak memory and allocator overhead.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://ieeexplore.ieee.org/document/9408206" target="_blank" rel="noopener"><strong>Fewer copies</strong></a></td><td class="what">Remove redundant copies and layout conversions between camera, preprocessing and model; these can cost more than the model itself.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://arxiv.org/pdf/2205.14135" target="_blank" rel="noopener"><strong>Tiling</strong></a></td><td class="what">Split computation into tiles that fit in on-chip memory, reducing slow and energy-hungry external-memory traffic.</td></tr>
<tr class="t-inf"><td class="knob"><a href="https://machinelearning.apple.com/research/introducing-apple-foundation-models" target="_blank" rel="noopener"><strong>Shorter context</strong></a></td><td class="what">Fewer prompt tokens mean faster prefill and a smaller KV cache; at <a href="https://machinelearning.apple.com/research/introducing-apple-foundation-models" target="_blank" rel="noopener">0.6 ms per prompt token</a>, 1K tokens alone cost ~600 ms.</td></tr>
<tr class="t-inf"><td class="knob"><strong><strong>Fewer calls</strong></strong></td><td class="what">Run the model less often (event triggers, batching, lower duty cycle). On wearables, average power depends more on how often you run than on per-call speed.</td></tr>
<tr class="op-row t-inf"><td colspan="6" class="op op-inf"><div class="op-title">Open problems · Inference</div><ul><li><strong>Router calibration:</strong> query-value estimators trained on chat data must handle egocentric, multimodal intents under battery, thermal and network shifts.</li><li><strong>Speculation energy:</strong> speedups can raise energy per token when acceptance drops; wearable-scale evidence is thin.</li><li><strong>KV eviction vs recall:</strong> bounded caches forget, and there are no guarantees for recalling earlier scenes.</li><li><strong>Wearable–companion–cloud split:</strong> radio start-up, queueing and disconnects dominate tail latency; per-token cross-device protocols are fragile.</li><li><strong>No glasses-class measurements:</strong> tokens/s, joules per token and thermal curves for AR1-class chips remain unpublished.</li></ul></td></tr>
</tbody></table>
</div>

<style>
/* ---------- subtitle ---------- */
.post-byline {
  font-size: 1.125rem;
  color: #4b5563;
  margin: -6px 0 18px 0;
  text-align: left !important;
}
.post-byline a { font-weight: 600; }

/* ---------- gearbox image + full-screen view ---------- */
.gearbox-figure { margin: 28px 0 8px 0; }
.gearbox-figure a { display: block; cursor: zoom-in; }
.gearbox-figure img {
  display: block;
  width: 100%;
  height: auto;
  border: 1px solid #e5e7eb;
  border-radius: 6px;
}
.gearbox-figure figcaption {
  margin-top: 8px;
  text-align: center;
  font-size: 0.875rem;
  font-style: italic;
  color: #6b7280;
}
#gearbox-img:fullscreen {
  width: 100vw; height: 100vh; max-width: none;
  object-fit: contain; background: #ffffff;
  border: 0; border-radius: 0; cursor: zoom-out;
}
#gearbox-img:-webkit-full-screen {
  width: 100vw; height: 100vh; max-width: none;
  object-fit: contain; background: #ffffff;
  border: 0; border-radius: 0; cursor: zoom-out;
}
/* Fallback viewer for browsers without the Fullscreen API (e.g. iPhone Safari) */
.gearbox-lightbox {
  position: fixed; inset: 0; z-index: 9999;
  background: #ffffff;
  display: flex; align-items: center; justify-content: center;
  overflow: auto; cursor: zoom-out;
}
.gearbox-lightbox img { max-width: 100%; max-height: 100%; object-fit: contain; }

/* ---------- legend ---------- */
.gearbox-legend {
  display: flex; flex-wrap: wrap; gap: 6px 20px;
  margin: 36px 0 10px 0;
  font-size: 0.8125rem; color: #4b5563;
  text-align: left !important;
}
.gearbox-legend span { display: inline-flex; align-items: center; }
.gearbox-legend .sw { display: inline-block; width: 12px; height: 12px; border-radius: 3px; margin-right: 6px; }
.sw-mod { background: #c62828; } .sw-data { background: #b7791f; }
.sw-comp { background: #26282b; } .sw-inf { background: #2f5f7a; }

/* ---------- table: break out of the 740px text column so it stays readable ---------- */
.gearbox-wrap {
  width: min(1560px, calc(100vw - 48px));
  margin-left: calc(50% - min(780px, 50vw - 24px));
  margin-bottom: 32px;
}
.post-content table.gearbox {
  width: 100%;
  table-layout: fixed;
  border-collapse: separate;
  border-spacing: 0;
  border: 0;
  margin: 0;
  font-size: 14px;
  line-height: 1.5;
  color: #313032;
  background: #ffffff;
  text-align: left !important;
  hyphens: manual;
}
.gearbox col.c-l1   { width: 62px; }
.gearbox col.c-l2   { width: 132px; }
.gearbox col.c-l3   { width: 118px; }
.gearbox col.c-knob { width: 150px; }
.gearbox col.c-op   { width: 27%; }

.post-content table.gearbox th,
.post-content table.gearbox td {
  padding: 9px 12px;
  border: 0;
  vertical-align: top;
  background: transparent;
}
.post-content table.gearbox tr:nth-child(even) td { background: transparent; }

/* header row */
.post-content table.gearbox thead th {
  position: sticky; top: 0; z-index: 3;
  background: #1d1d1f;
  color: #ffffff;
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  padding: 11px 12px;
  line-height: 1.35;
}
.post-content table.gearbox thead th:first-child { padding: 11px 4px 11px 8px; letter-spacing: 0.04em; }

/* level 1 band (vertical label) */
.post-content table.gearbox th.l1 { padding: 18px 0; text-align: center !important; color: #ffffff; }
.gearbox th.l1 span {
  position: sticky; top: 56px;
  display: inline-block;
  writing-mode: vertical-rl;
  transform: rotate(180deg);
  font-size: 17px; font-weight: 800; letter-spacing: 0.3em;
}
.post-content table.gearbox th.l1-mod { background: #c62828; }
.post-content table.gearbox th.l1-inf { background: #2f5f7a; }

/* level 2 / level 3 labels */
.post-content table.gearbox th.l2,
.post-content table.gearbox th.l3 {
  font-size: 14px; font-weight: 700; line-height: 1.35;
  padding-top: 11px;
  border-top: 1px solid rgba(255,255,255,0.9);
}
.gearbox th.l2 span, .gearbox th.l3 span { position: sticky; top: 50px; display: block; }
.post-content table.gearbox tr.t-mod  th.l2 { background: #fceaea; color: #c62828; }
.post-content table.gearbox tr.t-inf  th.l2 { background: #e7eff4; color: #1f4a63; }
.post-content table.gearbox th.l2.l2-band { color: #ffffff; letter-spacing: 0.06em; }
.post-content table.gearbox tr.t-data th.l2-band { background: #b7791f; }
.post-content table.gearbox tr.t-comp th.l2-band { background: #26282b; }
.post-content table.gearbox tr.t-data th.l3 { background: #f5e2c2; color: #8a580e; }
.post-content table.gearbox tr.t-comp th.l3 { background: #e7e8ea; color: #26282b; }

/* knob + description */
.post-content table.gearbox td.knob,
.post-content table.gearbox td.what {
  background: #ffffff;
  border-bottom: 1px solid #ecebe6;
}
.post-content table.gearbox tbody tr:nth-child(even) td.knob,
.post-content table.gearbox tbody tr:nth-child(even) td.what { background: #fafaf7; }
.post-content table.gearbox td.knob {
  font-weight: 700;
  font-size: 13.5px;
  border-left: 4px solid #c62828;
  overflow-wrap: anywhere;
}
.post-content table.gearbox tr.t-data td.knob { border-left-color: #b7791f; }
.post-content table.gearbox tr.t-comp td.knob { border-left-color: #3d4146; }
.post-content table.gearbox tr.t-inf  td.knob { border-left-color: #2f5f7a; }
.post-content table.gearbox tr.grp-start td.knob,
.post-content table.gearbox tr.grp-start td.what { border-top: 1.5px solid #9ca3af; }
.post-content table.gearbox tr.t-mod.grp-start  td.what,
.post-content table.gearbox tr.t-mod.grp-start  td.knob { border-top-color: #c62828; }
.post-content table.gearbox tr.t-data.grp-start td.what,
.post-content table.gearbox tr.t-data.grp-start td.knob { border-top-color: #b7791f; }
.post-content table.gearbox tr.t-comp.grp-start td.what,
.post-content table.gearbox tr.t-comp.grp-start td.knob { border-top-color: #26282b; }
.post-content table.gearbox tr.t-inf.grp-start  td.what,
.post-content table.gearbox tr.t-inf.grp-start  td.knob { border-top-color: #2f5f7a; }

.gearbox a { color: #1f4e8c; text-decoration: underline; text-underline-offset: 2px; }
.gearbox a:hover { color: #2563eb; }

/* the two rightmost columns are fully justified, as in the PDF */
.post-content table.gearbox td.what,
.post-content table.gearbox td.op {
  text-align: justify !important;
  text-justify: inter-word;
}
.post-content table.gearbox td.what ul,
.post-content table.gearbox td.what li,
.post-content table.gearbox td.op ul,
.post-content table.gearbox td.op li {
  text-align: justify !important;
  text-justify: inter-word;
  hyphens: auto;
  -webkit-hyphens: auto;
}
.gearbox td.what p { margin: 0 0 4px 0; }
.gearbox td ul { margin: 2px 0 0 0; padding-left: 16px; list-style: disc; }
.gearbox td li { margin: 0 0 2px 0; }
.gearbox td li:last-child { margin-bottom: 0; }

/* open problems column */
.post-content table.gearbox td.op { padding: 14px 16px; font-size: 13.5px; }
.gearbox td.op ul { padding-left: 14px; }
.gearbox td.op li { margin-bottom: 8px; }
.post-content table.gearbox td.op-mod  { background: #fceaea; color: #313032; }
.post-content table.gearbox td.op-data { background: #b7791f; color: #ffffff; }
.post-content table.gearbox td.op-comp { background: #26282b; color: #ffffff; }
.post-content table.gearbox td.op-inf  { background: #e7eff4; color: #313032; }
.gearbox td.op strong { font-weight: 700; }
.gearbox .op-title {
  font-size: 11px; font-weight: 700; letter-spacing: 0.08em;
  text-transform: uppercase; margin-bottom: 8px;
}
.post-content table.gearbox tr.op-row { display: none; }

/* medium screens: tighter label columns */
@media (max-width: 1180px) {
  .gearbox col.c-l1   { width: 44px; }
  .gearbox col.c-l2   { width: 104px; }
  .gearbox col.c-l3   { width: 96px; }
  .gearbox col.c-knob { width: 120px; }
  .gearbox col.c-op   { width: 25%; }
  .post-content table.gearbox th.l1 span { font-size: 14px; }
}

/* phones and small tablets: stack each row so nothing needs zooming or sideways scrolling */
@media (max-width: 899px) {
  .gearbox-wrap { width: auto; margin-left: 0; }
  .post-content table.gearbox,
  .post-content table.gearbox tbody,
  .post-content table.gearbox tr,
  .post-content table.gearbox th,
  .post-content table.gearbox td { display: block; width: auto; }
  .post-content table.gearbox thead,
  .post-content table.gearbox colgroup { display: none; }
  .post-content table.gearbox { font-size: 15px; }
  .gearbox th.l1 span, .gearbox th.l2 span, .gearbox th.l3 span { position: static; }
  .post-content table.gearbox th.l1 { padding: 10px 12px; text-align: left !important; margin-top: 18px; border-radius: 4px 4px 0 0; }
  .gearbox th.l1 span { writing-mode: horizontal-tb; transform: none; font-size: 15px; letter-spacing: 0.2em; }
  .post-content table.gearbox th.l2,
  .post-content table.gearbox th.l3 { padding: 8px 12px; border-top: 0; }
  .post-content table.gearbox tr > td.op { display: none; }
  .post-content table.gearbox tr.op-row { display: block; }
  .post-content table.gearbox tr.op-row td.op { display: block; font-size: 14.5px; margin-bottom: 8px; }
  .post-content table.gearbox td.knob { font-size: 15px; padding: 10px 12px 2px 12px; border-bottom: 0; }
  .post-content table.gearbox td.what { padding: 2px 12px 12px 16px; border-left: 4px solid #c62828; }
  .post-content table.gearbox tr.t-data td.what { border-left-color: #b7791f; }
  .post-content table.gearbox tr.t-comp td.what { border-left-color: #3d4146; }
  .post-content table.gearbox tr.t-inf  td.what { border-left-color: #2f5f7a; }
  .post-content table.gearbox tr.grp-start td.what { border-top: 0; }
}
</style>

<script>
(function () {
  /* Subtitle: show "with ..." directly under the post title */
  var byline = document.getElementById('post-byline');
  var title = document.querySelector('.post-header .post-title');
  if (byline && title) { title.insertAdjacentElement('afterend', byline); }

  /* Full-screen view of the gearbox image; Esc (or another click) returns to the post */
  var link = document.getElementById('gearbox-img-link');
  var img = document.getElementById('gearbox-img');
  if (!link || !img) { return; }
  var fsElement = function () { return document.fullscreenElement || document.webkitFullscreenElement; };
  var exitFs = function () {
    if (document.exitFullscreen) { document.exitFullscreen(); }
    else if (document.webkitExitFullscreen) { document.webkitExitFullscreen(); }
  };
  var openLightbox = function () {
    var box = document.createElement('div');
    box.className = 'gearbox-lightbox';
    box.setAttribute('role', 'dialog');
    box.setAttribute('aria-label', 'Gearbox image, full screen. Press Escape to close.');
    var big = document.createElement('img');
    big.src = img.currentSrc || img.src;
    big.alt = img.alt;
    box.appendChild(big);
    var close = function () {
      document.removeEventListener('keydown', onKey);
      if (box.parentNode) { box.parentNode.removeChild(box); }
      document.documentElement.style.overflow = '';
    };
    var onKey = function (e) { if (e.key === 'Escape') { close(); } };
    box.addEventListener('click', close);
    document.addEventListener('keydown', onKey);
    document.documentElement.style.overflow = 'hidden';
    document.body.appendChild(box);
  };
  link.addEventListener('click', function (e) {
    e.preventDefault();
    if (fsElement()) { exitFs(); return; }
    var req = img.requestFullscreen || img.webkitRequestFullscreen;
    if (req) {
      var p = req.call(img);
      if (p && p.catch) { p.catch(openLightbox); }
    } else {
      openLightbox();
    }
  });
})();
</script>
