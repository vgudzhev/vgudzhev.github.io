---
title: "Ambient status lights for Claude Code, on a Touch Bar nobody uses"
date: 2026-07-08 10:00:00 +0200
categories: [Engineering]
tags: [claude-code, hooks, macos, automation]
image:
  path: /assets/keyled.png
---

I have three or four Claude Code sessions open at any given moment. One's grinding through a refactor. One's paused on a permission prompt I forgot about. One finished five minutes ago and is just sitting there. I can't tell which is which without Cmd-Tab-ing through a stack of terminal windows like I'm dealing cards.

So I built glowbar: one colored tile per session, live on the 2019 MacBook Pro Touch Bar. That strip of glass Apple abandoned and I apparently never will.

🔴 my-api      🟡 webapp      🟢 infra
 working        waiting        ready

- 🔴 red — Claude is mid-turn (thinking, running tools, editing files)
- 🟡 yellow — blocked on me (permission prompt or question)
- 🟢 green — turn finished, session idle
- ⚪ dim — no activity for 20 minutes, probably abandoned

## How it works

Not by scraping terminals. By hooking into Claude Code itself. Each event (SessionStart, PreToolUse, Notification, Stop) triggers a script that writes the session state to a file. The Touch Bar reads and renders it.

The tricky part is refusing to lie. A subagent finishing mid-turn, or context compaction firing while Claude is still working, can't be allowed to flip a red tile yellow or green. Most of the bugs I fixed weren't "it doesn't light up" — they were "it lit up the wrong color for half a second," which is worse, because you learn to distrust it. A status light you don't trust is just a light.

## Then the tiles started doing more than glow

They have names. Each tile is labeled with the session's actual name — not the directory, the session. The one /rename sets. So two sessions in the same repo stop looking like identical twins, which was the whole problem on bad days.

They're links. Tap a tile and the terminal tab running that session jumps to the front. No more Cmd-Tab card-dealing — see yellow, tap yellow, you're there. (It matches the tab by the session's TTY, which turns out to be the one thing that reliably survives.)

They're greedy. By default glowbar takes over the app region of the Touch Bar — the left part that normally shows Terminal's color swatches or VS Code's debug buttons. I have never once needed a color swatch on a strip of glass below my screen. I constantly need to know which session is waiting on me. Easy trade.

One word turns it off. tb. On, off, done. Because ambient is great until you're screen-sharing.

No Touch Bar? It mirrors to the menu bar — the same colored dots and names, up in the corner. Which means it also works on every Mac Apple shipped after they gave up on the Touch Bar. Including the one I'll eventually replace this with.

## The part worth stealing

The Touch Bar isn't the point. Hooks as an ambient event bus is the point. Once each session emits structured state into a file, you can render it anywhere: a menu bar, a desk lamp, an RGB keyboard, a Slack status, a smart bulb that turns the whole room yellow when Claude needs you. The hard, careful part — the part I'd actually copy — is the little state machine that refuses to lie about what's happening.

![LED keyboard with session status indicators](/assets/keyled.png){: width="1376" height="268"}

Now excuse me. One of my tiles just went yellow. And I can tap it.

---

**glowbar** is open source: [github.com/vgudzhev/glowbar](https://github.com/vgudzhev/glowbar)
