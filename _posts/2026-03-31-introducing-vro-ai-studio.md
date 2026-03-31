---
title: "Introducing vRO AI Studio: Bringing AI to VMware Aria Orchestrator Development"
date: 2026-03-31 12:00:00 +0200
categories: [VMware, AI]
tags: [vro, aria-orchestrator, vscode, claude, typescript, automation]
---

If you've spent time writing automation for VMware Aria Orchestrator (vRO), you know the friction. The platform has its own TypeScript runtime, its own APIs (`System`, `Server`, `VcPlugin`, `RESTHost`), its own quirks — and almost none of that context lives in general-purpose AI tools. You paste code into ChatGPT and get back generic JavaScript that won't run inside vRO. You ask for tests and get mocks that don't know what `System.log()` is.

vRO AI Studio is a VS Code extension built to close that gap. It embeds vRO-specific AI assistance directly in your editor — no context switching, no copy-pasting, no re-explaining the platform on every prompt.

## The Problem It Solves

vRO development has a high knowledge floor. You need to know which APIs are available in the runtime, how to structure actions and modules, what patterns to avoid in the vRO scripting engine, and how to write testable code against a proprietary platform. Onboarding new developers is slow, code review is manual, and writing Jasmine tests for vRO actions is tedious enough that many teams skip it entirely.

vRO AI Studio attacks all of these pain points at once with five focused tools.

## What It Does

**Generate Action** — Describe what you need in plain English. The extension generates production-ready TypeScript using proper vRO APIs, ES5-compatible and ready to run inside the vRO runtime. Input parameters, return types, and platform-specific patterns are all handled automatically.

**Review This File** — A single click triggers a vRO-aware code review. The AI knows to look for the things that matter on this platform: blocking calls in async contexts, hardcoded credentials, incorrect use of vRO APIs, deprecated patterns, and security issues. Issues surface directly in VS Code's Problems panel with severity levels.

**Generate Tests** — Produces a complete Jasmine spec file with proper spies and mocks for vRO global APIs (`System`, `Server`, `VcPlugin`, `RESTHost`). It covers the happy path, null inputs, empty results, and API errors — the full coverage most teams never get around to writing manually.

**Explain Code** — Translates any action or workflow into plain English, Markdown documentation, a README section, or an ops runbook. Useful for onboarding, handoffs, and documentation that actually stays up to date.

**Onboarding Assistant** — Scans your local environment (Java, Maven, Node versions, Maven settings), detects configuration issues that would block `mvn vro:push`, and walks new developers through the five setup stages interactively.

## How It's Built

The extension is a standard VS Code webview extension written in TypeScript. It calls the Claude API directly using Node's built-in `https` module — no external runtime dependencies, which matters for air-gapped enterprise environments. The API endpoint is configurable, so teams running an internal Claude proxy can point the extension there without any code changes.

Every feature uses a system prompt carefully tuned for the vRO platform — not a generic "write TypeScript" prompt, but one that knows the APIs, the runtime constraints, and the common failure modes specific to Aria Orchestrator. That specificity is what makes the output usable without heavy editing.

## What's Next

The first release covers the core developer loop — write, review, test, document, onboard. Each of these features has a deeper story worth telling on its own, and that's exactly what the next series of posts will do: one feature at a time, with real examples and the reasoning behind the design decisions.

If you work with vRO and want to try it, the source is at [github.com/vgudzhev/vro-ai-studio](https://github.com/vgudzhev/vro-ai-studio) and the `.vsix` is in the releases.

---
*This is the first post in a series on vRO AI Studio. Upcoming posts will cover each of the five features in depth.*
