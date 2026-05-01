---
title: "vRO AI Studio: Local LLMs, Cost, and Privacy"
date: 2026-05-01 12:00:00 +0200
categories: [VMware, AI]
tags: [vro, aria-orchestrator, vscode, claude, local-llm, privacy, automation]
image:
  path: /assets/Gemini_Generated_Image_eoakxdeoakxdeoak.png
---

In the last few weeks I've been seeing more and more discussions around token costs — and it is starting to feel like this has become a bigger concern than data privacy and security.

Two camps are emerging. Many organizations are still debating whether to allow AI tools at all — even just as a chat window — because of security and data privacy concerns. Others that have already adopted AI in their development workflow are now facing a different problem: the bill is growing faster than the value they can demonstrate.

That's where local LLMs come in — models that run entirely on your own machine or infrastructure, with no data sent to external servers and no per-token billing. They're not as powerful as the frontier models, but for many everyday coding tasks they're good enough, and the trade-off in cost and privacy is significant.

That thinking drove a recent update I made to vRO AI Studio. I added support for both OpenAI-compatible APIs and the native local LLM API, and tested it running Qwen2.5 Coder locally on AnythingLLM. No cloud calls, no token costs, no data leaving the machine.

The next step is proper AI orchestration — knowing when to call the big model and when the local one is good enough. To get the best of both worlds you can combine them with approaches like Anthropic's advisor tool pattern and projects like [ai-orchestrator](https://github.com/vgudzhev/ai-orchestrator) that route work intelligently between Claude and local models.

Cost, security, and capability don't have to be a trade-off. They just require a bit more architecture.
