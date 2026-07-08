---
title: "Ambient Status Lights for Claude Code"
date: 2026-07-08 10:00:00 +0200
categories: [Engineering]
tags: [claude-code, hooks, macos, automation]
---

I have three or four Claude Code sessions open at any given moment. One's grinding through a refactor. One's paused on a permission prompt I forgot about. One finished five minutes ago and is just sitting there. I can't tell which is which without Cmd-Tab-ing through a stack of terminal windows like I'm dealing cards.

So I built tbled: one colored tile per session, live on the 2019 MacBook Pro Touch Bar. That strip of glass Apple abandoned and I apparently never will.

🔴 my-api     🟡 webapp     🟢 infra
 working       waiting       ready

- 🔴 red: Claude is mid-turn (thinking, running tools, editing files)
- 🟡 yellow: blocked on me (permission prompt or question)
- 🟢 green: turn finished, session idle
- ⚪ dim: no activity for 20 minutes, probably abandoned

How? Not by scraping terminals. By hooking into Claude Code itself. Each event (SessionStart, PreToolUse, Notification, Stop) triggers a script that writes the session state to a file. The Touch Bar reads and renders it. The tricky part: refusing to lie. A subagent finishing mid-turn or context compaction can't flip a red tile yellow or green.

Hooks work as an ambient event bus. Pipe them anywhere: menu bar, desk lamp, RGB keyboard, Slack. Whatever fits.

Now excuse me. One of my tiles just went yellow.
