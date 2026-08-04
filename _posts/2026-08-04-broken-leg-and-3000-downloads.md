---
title: "A Broken Leg and 3,000 Downloads"
date: 2026-08-04 10:00:00 +0300
categories: [Engineering]
tags: [typescript, npm, open-source]
image:
  path: /assets/broken-leg-3000-downloads.png
---

In the summer of 2017 I broke my leg. For weeks I couldn't do much besides lie in bed with a laptop balanced on a pillow. Bored, medicated, and looking for something to build, I wrote a small npm package called [bg-egn-helper](https://github.com/vgudzhev/bg-egn-helper) — a utility to validate, generate, and parse Bulgarian EGN (ЕГН) numbers. I published it and mostly forgot about it.

## What is an EGN?

For those outside Bulgaria: the EGN (Единен граждански номер) is a 10-digit personal identification number assigned to every Bulgarian citizen. It encodes date of birth, sex, and region of birth, with a check digit at the end. If you've ever worked on a Bulgarian government system, banking app, or anything that touches citizen data, you've had to deal with EGN validation.

## Seven years later

Fast forward to 2024. I checked npm one day and discovered that bg-egn-helper had quietly crossed 3,000 downloads. For a niche library serving a country of 7 million people, that number surprised me. People were actually using this thing — the thing I wrote while drifting in and out of painkiller-induced naps.

So I went back and looked at the code. It was... rough.

## The bugs

Here's a highlight reel of what I found:

- **The exports were broken.** Parts of the public API simply weren't accessible.
- **`validateList()` always returned `true`.** The one function you'd reach for when you have a batch of EGNs to check — and it just nodded along regardless.
- **July was spelled "Jule."** In my defense, I was on painkillers.

There were seven bugs in total. Seven bugs in a library with three core functions. Not my finest work.

## The rewrite

I decided to give bg-egn-helper the rewrite it deserved. Version 2.0.0 is a ground-up rebuild:

- **TypeScript** — full type safety, no more guessing what a function returns.
- **Dual ESM/CJS** — works in modern bundlers and legacy Node setups alike.
- **54 tests, 97% coverage** — every edge case I could think of, including the ones I missed the first time.
- **All 7 bugs fixed** — exports work, `validateList()` actually validates, and July is spelled correctly.

The API surface stayed small on purpose. You can validate an EGN, generate test EGNs for a given date, and parse one into its component parts. That's it. That's the library.

## Thank you

To everyone who pulled bg-egn-helper into their projects over the past seven years — thank you. Every download means a lot, especially for something this niche. I hope v2.0.0 is the version you deserved from the start.

Check out [v2.0.0 on GitHub](https://github.com/vgudzhev/bg-egn-helper).
