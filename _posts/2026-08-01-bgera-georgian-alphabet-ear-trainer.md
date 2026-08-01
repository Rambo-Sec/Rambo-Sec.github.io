---
layout: post
title: "Building bgera: A Georgian Alphabet Ear-Trainer"
date: 2026-08-01
categories: [projects]
tags: [javascript, side-project, georgian]
---

Most of what I post here is cybersecurity work. This one isn't. It's something I'm just as proud of, and it comes from a more personal place.

## Why Georgian

My mother fled the invasion of Gagra in 1993. I still have family in Tbilisi. I have a lot of respect for what my family went through, and for the culture and language they carried with them out of it.

A while back I taught myself the Cyrillic alphabet and some basic Russian. It went well enough that I wanted to do the same with Georgian. The problem was I couldn't find a decent tool for it. Mkhedruli, the modern Georgian script, doesn't get the same treatment as Cyrillic or the Latin-adjacent alphabets. The learning tools that exist are either buried inside broader language-course apps, or they just don't cover the alphabet in a focused way. So I decided to build the tool I was looking for.

## What it is

**bgera** (ბგერა, Georgian for "sound") is the core game on [ქართული ანბანი (Georgian Alphabet)](https://learngeorgianletters.com), a small site for learning Mkhedruli by ear.

It's a timed sound-matching game, inspired by a Cyrillic ear-trainer I'd used before. The game plays a real recording of a Georgian letter, not synthesized speech. You have to tap the matching letter on a dial before a countdown ring runs out. Each round narrows the full alphabet down to 8 choices: the correct letter plus 7 random distractors. That keeps it approachable instead of overwhelming for someone just starting out.

Get one wrong, and the game doesn't just move on. It reveals the correct letter and waits for you to hit Continue before advancing, so a mistake actually registers instead of scrolling past unnoticed. There's also a Practice mode: a plain grid of every letter, or filtered to just vowels or just consonants, that you can tap freely to hear. No timer, no score. It's useful for building familiarity before testing yourself for real. The game tracks score, current streak, and best streak. Nothing persists across sessions though; it's meant as a focused drilling tool, not a spaced-repetition system.

## What was actually hard about it

The stack is about as simple as it gets. Vanilla HTML/CSS/JS. No frameworks, no build step, no dependencies. Just one self-contained HTML file plus a sibling `audio/` folder. CSS custom properties handle theming. Noto Sans Georgian makes sure Mkhedruli actually renders correctly. The dial itself is just trigonometry, positioning tiles around a circle with `sin`/`cos`. Each round is a fresh shuffle-and-slice from the full letter pool.

A few things still turned out to be genuinely non-trivial:

- **No clean audio source existed.** There's no single licensed, complete recording set for all 33 Georgian letters freely available online. That was a real discovery partway through the build. The game falls back gracefully to the browser's speech synthesis for any letter missing a real recording. But I ended up sourcing and dropping in all 33 real MP3s myself.
- **Browser autoplay policies.** The first sound on page load was silently failing. Browsers block audio before any user gesture has happened on the page. I fixed it by having the game "arm" the first round (letter picked, dial built) without playing anything, and only starting the clock and audio on the user's first Play click.
- **Linux/Flatpak sandboxing.** Not a code bug at all. While testing locally, Firefox (packaged as a Flatpak on my Bazzite setup) couldn't see the local `audio/` folder. Flatpak apps are sandboxed to specific directories by default. I solved it outside the code entirely, by serving the folder over `localhost` via a Python HTTP server running in a Distrobox container. That sidesteps the filesystem sandbox altogether.

Everything else about the build was refreshingly ordinary. Sometimes the hardest parts of a project have nothing to do with the code.

## Try it

You can play it at [learngeorgianletters.com](https://learngeorgianletters.com). If you're learning Georgian, or just curious what Mkhedruli sounds like, I'd love to know what you think.
