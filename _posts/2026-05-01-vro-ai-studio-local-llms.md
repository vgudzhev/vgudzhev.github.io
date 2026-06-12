---
title: "vRO AI Studio: Local LLMs, Cost, and Privacy"
date: 2026-05-01 12:00:00 +0200
categories: [VMware, AI]
tags: [vro, aria-orchestrator, vscode, claude, local-llm, privacy, automation]
image:
  path: /assets/Gemini_Generated_Image_eoakxdeoakxdeoak.png
---

In the last few weeks I've been seeing more discussions around token costs, and it's starting to feel like that concern is overtaking data privacy and security.

Two camps are emerging. Many organizations are still debating whether to allow AI tools at all, even just as a chat window, because of security and data privacy concerns. Others that have already adopted AI in their development workflow are facing a different problem: the bill is growing faster than the value they can demonstrate.

Local LLMs run entirely on your own machine or infrastructure, with nothing sent to external servers and no per-token billing. They're not as capable as the frontier models, but for everyday coding tasks they're often good enough, and the cost and privacy difference is significant.

That thinking drove a recent update to vRO AI Studio. I added support for both OpenAI-compatible APIs and the native local LLM API, and tested it running Qwen2.5 Coder locally on AnythingLLM. Everything stays on the machine.

The next step is proper AI orchestration: knowing when to call the big model and when the local one is good enough. You can combine them with approaches like Anthropic's advisor tool pattern and projects like [ai-orchestrator](https://github.com/vgudzhev/ai-orchestrator) that route work intelligently between Claude and local models.

Cost, security, and capability are usually presented as a trade-off. With a bit more architecture, they don't have to be.

---

**Resources:**
- [Anthropic advisor tool pattern](https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool)
- [ai-orchestrator](https://github.com/vgudzhev/ai-orchestrator)
- [Qwen2.5 Coder on Ollama](https://ollama.com/library/qwen2.5-coder)
- [Code Repo](https://github.com/vgudzhev/vro-ai-studio)
- [Release v0.3.1](https://github.com/vgudzhev/vro-ai-studio/releases/tag/v0.3.1)
