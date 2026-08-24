---
title: "I Built a Tool That Records AI Agent Failures So I Can Replay Them"
date: 2026-08-18 10:00:00 +0200
categories: [AI, Engineering]
tags: [claude-code, agents, testing, debugging, open-source]
image:
  path: /assets/Gemini_Generated_Image_repro.jpeg
---

Last month I was using Claude Code to fix a bug in a TypeScript project. The project had auto-generated API types in `src/gen/`. Claude looked at the type error, found the generated file, and edited it directly. The fix worked. The tests passed. I almost merged it.

<!-- TODO: Screenshot of the git diff showing Claude's edit inside src/gen/ — the "crime scene" that started this -->

The problem: `src/gen/` is generated from an OpenAPI schema. The next time someone runs the codegen script, Claude's fix disappears. The real fix should have gone into the schema or the codegen config. But Claude doesn't know that — it saw a file with a type error and fixed the file.

I caught it during review. But this kind of failure is hard to study because you can't reproduce it. The model changes, the prompt changes, the system prompt gets updated, the agent takes a different path. By the time you try again, the agent does something completely different and the failure is gone.

What I wanted was straightforward: record the failure, replay it without an API key, assert on what went wrong, and commit it as a test.

## repro

[repro](https://github.com/vgudzhev/repro-md) is a CLI tool that sits between your AI agent and the model API as an HTTP proxy. During recording, it captures every request and response. During replay, it serves the recorded responses back. The agent doesn't know the difference.

```
Agent  <-->  repro proxy  <-->  Anthropic API    (recording)
Agent  <-->  repro proxy                          (replay, no network)
```

You don't need the model to be deterministic. You just need to record what it said and play it back. The agent runs for real — same binary, same filesystem, same tool calls — but model responses come from the recording.

### Recording

```bash
$ repro record --auth plan -- claude --print "fix the type error in getUserProfile"
repro: recording r-7f3a91
repro: auth=plan
repro: proxy listening on http://127.0.0.1:54321
repro: completed after 41 events
repro: saved r-7f3a91
```

`--auth plan` means I'm using my claude.ai subscription, so no API key is needed. repro strips API keys from the agent's environment and captures everything through the proxy.

<!-- TODO: Screenshot of `repro record` running in the terminal — show the proxy starting up and events being captured live -->

### Replay

```bash
$ repro run r-7f3a91
repro: replaying r-7f3a91 (41 events)
repro: mode: strict
repro: reproduced — 41 events, 0 API calls, 0 API keys
```

Zero API calls. Zero cost. The agent runs against a git worktree checked out to the same commit, so the filesystem matches the original run. Requests are matched by normalized content hash — if the agent sends a different request than what was recorded, the mismatch is detected immediately.

<!-- TODO: Screenshot of `repro run` completing — show the "reproduced" output with 0 API calls -->

### Assertions

```bash
$ repro save r-7f3a91 --title "agent edits generated types" \
    --assertion forbidden_path:src/gen/**
```

`forbidden_path` fires if any tool call in the trace touches a file matching `src/gen/**`. No model needed to judge correctness — it checks structure and behavior, not output quality.

Other assertions:
- `no_repeat:3` fails if the same tool call repeats more than 3 times (catches loops)
- `max_calls:15` fails if the session exceeds 15 API calls (catches runaway sessions)
- `command:"test -f output.txt"` runs any shell command after replay

### CI

```yaml
- run: npx repro test
```

One line in your GitHub Actions workflow. No API key needed, no inference cost. Replays every saved failure and exits non-zero on regression.

<!-- TODO: Screenshot of a GitHub Actions run showing `repro test` passing/failing — a real CI log -->

## Common failure patterns

These are scenarios I've been testing with that come up in real projects:

1. The agent edits generated code. You ask it to add a field to a type. It edits the generated file instead of the source. `forbidden_path:src/gen/**` catches it.

2. The agent deletes the failing test. You ask it to make the test suite pass. It removes the test instead of fixing the code. `command:test -f tests/math.test.js` catches it.

3. The agent loops. You ask it to fix a test. It runs the test, edits, runs the test, edits — same cycle, no progress. `no_repeat:3` catches it.

4. The runaway session. You ask it to review the codebase. A simple question becomes a 40-turn session. `max_calls:15` catches it.

5. The agent reads your secrets. You ask it to check the database config. It opens `.env` to "understand the configuration." `forbidden_path:.env*` catches it.

<!-- TODO: Screenshot of `repro run` hitting a forbidden_path assertion — show what a caught failure looks like in the terminal -->

## How it works under the hood

Requests are matched by SHA-256 hash, computed after stripping volatile fields (timestamps, cache hints, model name, tool definitions, system prompt). Minor environmental differences between recording and replay don't cause false mismatches. Parallel tool calls are sorted canonically so ordering differences don't matter either.

Secrets are scrubbed at capture time, before anything touches disk. Environment variable values, API key patterns, JWTs, and PEM blocks are all replaced with `[[redacted:env:<hash>]]` markers. Common non-secret variables (`PATH`, `HOME`, `PWD`) are excluded from redaction so file paths in responses don't get corrupted.

Claude Code uses streaming (SSE). repro reassembles streamed responses on capture and re-chunks them on replay. The agent sees the same streaming behavior it would from the real API.

`repro minimize` uses the ddmin algorithm to find the smallest set of inputs (context messages, tool definitions, files) that still trigger the failure. This costs money — it makes real API calls — but it can reduce 47 context items down to 3.

## What doesn't work yet

Codex. I spent a day trying to get this working. Codex v0.147.0 uses WebSocket transport exclusively (`wss://api.openai.com/v1/responses`) and completely ignores the `OPENAI_BASE_URL` environment variable. HTTP proxy interception is impossible. Blocked on upstream changes from OpenAI.

Side effects. repro records what the agent tells the API it did (tool calls and results), not what actually happened on disk. If the agent's tool call says "wrote file X" but something else happened, repro doesn't know. This is a v0.1 limitation.

Context compaction. If Claude Code compacts its context mid-session (drops older messages to stay within the context window), the compacted content is different on replay because the filesystem state is slightly different. This can cause hash mismatches in very long sessions.

## The trace format

The `.repro/` directory and `REPRO.md` manifest are plain files you commit to git. Anyone who clones the repo can replay the failures without installing anything beyond Node.js and the agent CLI. `repro inspect <id>` shows a timeline in the terminal, `repro diff <a> <b>` aligns two traces for comparison.

<!-- TODO: Screenshot of `repro inspect` output — the timeline view of a recorded trace -->

The goal is a portable failure format, not a platform. The CLI is one implementation. If someone wants to build a VS Code extension or a web viewer on top of the same format, the data is all there.

## Try it

```bash
npm install -g repro-md
cd your-project
repro init
repro record --auth plan -- claude --print "your prompt here"
repro run <id>
```

If you have a claude.ai subscription, you can record immediately. No API credits needed. MIT licensed.

- npm: [repro-md](https://www.npmjs.com/package/repro-md)
- GitHub: [vgudzhev/repro](https://github.com/vgudzhev/repro-md)
- Website: [repro.md](https://repro.md)
