---
title: "6TB of Claude Traffic Leaked from a Chinese LLM Router"
date: 2026-09-15 10:00:00 +0200
categories: [AI, Security]
tags: [claude, security, llm, pi-dev, secrets]
image:
  path: /assets/secureLLM.png
---

Last week a researcher bought a 6TB dump of Claude traffic from a Chinese LLM router. Inside were live SSH keys, VPN configs, cloud API keys and GitLab tokens, enough to compromise systems at several government agencies and companies including Xiaomi and Huawei.

None of this came from hacking the model. The data leaked from the middlemen, the routers and proxies that sit between your AI tool and the model, quietly logging everything that passes through. I've used OpenRouter myself for personal projects, comparing different models at a reasonable price, and I'm happy with it. But this incident is a reminder that any intermediary you route traffic through can see what you send. The secrets got there the boring way. Developers and AI agents pasted logs, config files and command output into prompts, and the keys came along for the ride.

I deal with this in my own work. Before I troubleshoot production logs with an AI assistant, I spend time cleaning them by hand, and I still worry I missed something.

What I've landed on is [secureLLM](https://github.com/vgudzhev/secureLLM), a scrubber that sits in front of the model and checks every piece of text before it leaves my machine. I'm building it with Pi Dev, and the idea is straightforward: secrets and personal data get swapped for consistent placeholders like `[[TOKEN_1]]` or `[[IP_3]]`. The model can still reason about the logs, it just never sees the real values. When the answer comes back or the model wants to run a command, the real values get put back in. If the scrubber is down, nothing gets sent. It works with whatever model I'm using at the time, and the same setup can run as a shared service for a team if you need that.

I'll walk through the whole workflow in upcoming posts.

Full story here: [A researcher buys 6TB of Claude data from a Chinese LLM router](https://wccftech.com/a-researcher-buys-6tb-of-anthropic-claude-data-dump-from-a-china-based-llm-router-finds-enough-ammo-to-hack-xiaomi-huawei-and-chinese-government-agencies/)
