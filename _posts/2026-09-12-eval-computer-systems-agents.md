---
layout: post
title: "Eval: From Computer Systems to Agents"
date: 2026-09-13
description: "As adoption of open-weight models grows, robust Eval design becomes increasingly important. Agent Eval shares several 
              challenges with Computer Systems Eval, from attributing improvements to validating the measurements themselves. 
              Which lessons carry over, and where do agents introduce new challenges?"
categories: [eval, agents, computer systems, performance analysis]
usemath: true
author: Arun Kejariwal
last_modified_at: 2026-09-13
---

As agents take on complex tasks, evaluation provides evidence that they meet specified requirements under deployment conditions. With 
growing adoption of open-weight models, robust Eval design becomes increasingly important: it guides improvements to narrow the gap with 
frontier models on a company’s target workloads. Assessing these improvements requires distinguishing the contributions of model capability,
harness design, and execution budget, then verifying that the benefits persist in production.

Interestingly, there are parallels with Computer Systems Eval, where profiling helps investigate performance limits across interacting 
components, and the measurements themselves require validation. Agents add a further complication: the grader can be consistently wrong, 
and the criteria can reward behavior that misses the intended goal. Drawing on my experience across the two domains, I penned down these 
parallels, where they hold, and where Agent Eval introduces new challenges. Feedback welcome!

Thanks [Bikash](https://www.linkedin.com/in/bikashsharma/){:target="_blank" rel="noopener"} for perusing the early versions of the essay.

---

{% include pdf-embed.html file="Eval - From Computer Systems to Agents.pdf" %}
