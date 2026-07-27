---
layout: post
title: "Optimization of Agentic Execution"
date: 2026-07-27
description: "Optimization of model harnesses has started to gain traction. Interestingly, there are a lot of parallels between 
              problems that have been addressed in the realm of compiler-driven program optimization and optimization of agentic 
              execution. Like what? "
categories: [agentic execution, harness optimization, compilers, program optimization]
usemath: true
---

Recently, there has been a lot of chatter around harness engineering. This, in part, stems from the fact the unit of work has evolved from simple prompts to complex, high-level tasks (as exemplified by agentic autoresearch), which then need to be broken down into smaller tasks and then mapped to multi-agent workflows. Optimization of model harnesses has started to gain traction. There are opportunities galore - say, how to construct a task graph from a complex high level specification, how to determine task granularity so as to not overrun the context window, how to mix-and-match smaller and larger models, how to “optimally” map a task graph to a multi-turn multi-agent workflow. 
 Interestingly, there are a lot of parallels between problems that have been addressed in the realm of compiler-driven program optimization - e.g., auto-parallelization, auto-vectorization and profile-guided optimization - and optimization of agentic execution. To this end, I penned down the parallel along the different axes. Feedback welcome!

Thanks [Vivek](https://www.linkedin.com/in/vivek-haldar-25261b3a/){:target="_blank" rel="noopener"} and [Milind](https://www.linkedin.com/in/milind-girkar-838a595/){:target="_blank" rel="noopener"} for the feedback on the early versions of the writeup.

Thanks  for the feedback on the early versions of the writeup. 

---

{% include pdf-embed.html file="Agentic Execution Optimization.pdf" %}
