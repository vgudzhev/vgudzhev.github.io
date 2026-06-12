---
title: "Introducing vRO AI Studio: Bringing AI to VMware Aria Orchestrator Development"
date: 2026-03-31 12:00:00 +0200
categories: [VMware, AI]
tags: [vro, aria-orchestrator, vscode, claude, typescript, automation]
---

As part of my Claude AI training, I built something I've been wanting to exist for years.

**vRO AI Studio** is an AI-powered VS Code extension that brings Anthropic's Claude directly into [VMware vRealize Build Tools (vRBT)](https://github.com/vmware/build-tools-for-vmware-aria) development.

Automating a complex VMware environment is hard. The ecosystem is powerful, but writing, reviewing, and maintaining vRO automation code manually is slow, error-prone, and a nightmare to onboard new engineers into.

So I connected Claude to the vRBT toolchain. The result is an assistant that understands your TypeScript actions, your vCenter SDK, and your Maven project structure, and helps you build automation without spending half your day on boilerplate.

Over the next 5 posts, I'll go through each capability:

**1. Generate:** Describe what you need in plain English. Add your coding standards, libraries, and existing projects as context, and Claude writes production-ready TypeScript tailored to your environment, matching your style and APIs without guessing.

**2. Review:** Paste your vRO action and get an instant code review. Blocking synchronous calls, memory issues, security gaps, legacy API usage, all flagged and color-coded by severity.

**3. Test:** Claude generates a complete Jasmine `.spec.ts` file covering happy paths, null inputs, edge cases, and API errors, ready to run.

**4. Explain:** Select any vRO action and get a plain-English summary of what it does and why. Useful for documentation and onboarding.

**5. Onboard:** A conversational setup assistant that detects missing Maven profiles, Java versions, and vRO connection issues, then walks you through fixing each one step by step.

- Code repo: [github.com/vgudzhev/vro-ai-studio](https://github.com/vgudzhev/vro-ai-studio)
- Release v0.1.0: [github.com/vgudzhev/vro-ai-studio/releases/tag/v0.1.0](https://github.com/vgudzhev/vro-ai-studio/releases/tag/v0.1.0)

Follow along for the full series. If you're working on VMware automation or AI-assisted DevOps, reach out.
