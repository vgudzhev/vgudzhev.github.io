---
title: "vRO AI Studio: Action Generation"
date: 2026-04-07 12:00:00 +0200
categories: [VMware, AI]
tags: [vro, aria-orchestrator, vscode, claude, typescript, automation]
---

In my last post I gave an overview of vRO AI Studio. Today I want to go deep on action generation, and the real problem it's trying to solve.

Every cloud automation team reaches the same point. The work gets complex enough that you need shared utilities, typed wrappers, and documented conventions. The toolchain changes, the problem doesn't.

**The tool is only as good as what you feed it.** Generic AI generates generic code. But feed vRO AI Studio your internal utilities, your existing actions, your coding standards, and every action it generates uses your patterns, calls your utilities, follows your conventions.

## For experienced teams

If you already have a mature library, plug it in and start immediately. The tool knows your wrappers, your clients, your conventions. Senior engineers fix gaps, cut releases, and the tool reflects that improvement automatically. The library you spent years building becomes the foundation every developer writes from, including the ones who joined last week.

> "Create a config element handler using our standard wrapper"
>
> "Call the Aria Automation deployment API and map the response using our existing client"
>
> "Get context variables for this VM creation event the way we always do it"

## For new adopters

If you're starting from scratch, you don't have to. vRO AI Studio ships with an opinionated starter project that gives your team a real foundation from day one:

- An extended HTTP library covering all request types
- A context library for all kinds of properties
- Typed getters and setters for every vRO element type
- Pipeline logic execution patterns
- Coding standards decided upfront: naming conventions, casing, class structure
- Logging and error handling baked in

The decisions that experienced teams took years to converge on are made for you before you write your first action. From there the library is yours. It grows with your project and improves with every gap your team finds.

---

- Code repo: [github.com/vgudzhev/vro-ai-studio](https://github.com/vgudzhev/vro-ai-studio)
- Release v0.2.0: [github.com/vgudzhev/vro-ai-studio/releases/tag/v0.2.0](https://github.com/vgudzhev/vro-ai-studio/releases/tag/v0.2.0)
- Example coding standards file: [lnkd.in/dBG4shuH](https://lnkd.in/dBG4shuH)
- Example common libraries file: [lnkd.in/d-jxAdB5](https://lnkd.in/d-jxAdB5)

If you've tried it or have features in mind, I'd like to hear it.
