---
title: "Introducing vRO AI Studio: Bringing AI to VMware Aria Orchestrator Development"
date: 2026-03-31 12:00:00 +0200
categories: [VMware, AI]
tags: [vro, aria-orchestrator, vscode, claude, typescript, automation]
---

As part of my Claude AI training, I built something I've been wanting to exist for years.

**vRO AI Studio** — an AI-powered VS Code extension that brings Anthropic's Claude directly into [VMware vRealize Build Tools (vRBT)](https://github.com/vmware/build-tools-for-vmware-aria) development.

Automating and orchestrating a complex VMware environment is a challenging task — the ecosystem is powerful, but writing, reviewing, and maintaining vRO automation code manually is slow, error-prone, and a nightmare to onboard new engineers into.

To address some of these pain points, I connected Claude to the vRBT toolchain. The result is an assistant that understands your TypeScript actions, your vCenter SDK, and your Maven project structure — and actively helps you build better automation, faster.

Over the next 5 posts, I'll do a deep dive into each capability:

**1. Generate** — Describe what you need in plain English. Add your existing coding standards, libraries, and projects as context — and Claude will write production-ready TypeScript code tailored to your environment. No boilerplate, no guessing APIs, no style mismatches.

**2. Review** — Paste your vRO action and get an instant code review: blocking synchronous calls, memory issues, security gaps, legacy API usage — all color-coded by severity.

**3. Test** — Claude generates a complete Jasmine `.spec.ts` file covering happy paths, null inputs, edge cases, and API errors. One click, ready to run.

**4. Explain** — Select any vRO action and get a plain-English summary of what it does and why. Perfect for documentation and onboarding.

**5. Onboard** — A conversational setup assistant that detects missing Maven profiles, Java versions, and vRO connection issues, then walks you through fixing each one step by step.

- Code Repo: [github.com/vgudzhev/vro-ai-studio](https://github.com/vgudzhev/vro-ai-studio)
- Release v0.1.0: [github.com/vgudzhev/vro-ai-studio/releases/tag/v0.1.0](https://github.com/vgudzhev/vro-ai-studio/releases/tag/v0.1.0)

Follow along for the full series. And if you're working on VMware automation or AI-assisted DevOps — let's connect.
