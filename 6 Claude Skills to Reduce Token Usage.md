---
title: "6 Claude Skills to Reduce Token Usage"
source: "https://medium.com/@hii_mohit/6-claude-skills-to-reduce-token-usage-67b68a3cffce"
author:
  - "[[Mohit Vaswani]]"
published: 2026-09-20
created: 2026-09-26
description: "More"
tags:
  - "clippings"
---
![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*S2tqhvTZEYRhTq011NVLmw.png)

Claude Code feels like magic. Then the token bill shows up.

One long session. A couple of big files. A wall of test logs you didn’t even read. And just like that, half your budget is gone and it isn’t even noon yet.

Here’s what nobody says out loud:

Most of that money didn’t go into your project. It went into garbage. Claude reading the same file for the fourth time. A giant command output pasted straight into the chat like anyone asked for it.

The model writing three paragraphs to say “done.” Failed attempts from an hour ago still riding along in the context, charging you rent on every new message.

The good part? Almost all of it is fixable.

Skills are tiny add-ons that change how Claude behaves, and a few of them exist for one job only: stop the leak. I dug through a big pile of them so you don’t have to. Here are six that actually made a dent in my bill, plus a few habits at the end that cost nothing to start.

Think of your context window like a backpack on a long hike. Whatever you stuff in it, you carry on every single step. This whole list is just about packing lighter.

Before going on to the skills, I have something for you

[***AgentsKit***](https://agentskit.co/) ~ Your AI marketing & engineering team in one command

It drops in ***89 specialist agents, 122 skills and 181 slash commands*** with one command, so one person can plan, build, test, ship and market a real product.

Let’s get into the list:

### 1\. Caveman

Start here, because it hits the most obvious leak: Claude talking too much.

Caveman strips out the filler. The “Great question!”, the long recaps, the “let me explain what I just did” paragraphs. It keeps the code and the real answer and throws away the rest. In testing it cut output by around 65%, and output matters twice over, because today’s chatty reply becomes tomorrow’s bloated context.

Why It’s Useful: You pay for every word Claude writes, then pay again to keep those words around. Cutting the fluff is the easiest win on this whole list.

Install: search the plugin marketplace for caveman, or grab it from [***GitHub***](https://github.com/JuliusBrussee/caveman)***.***

### 2\. Context Mode

The second biggest leak is tool output. Run your tests and 400 lines of logs land in the chat. Open a big file and the whole thing just sits there forever.

Context Mode catches that noise before it fills your window. It tucks large outputs, test logs, DOM dumps, MCP payloads into a local index and hands Claude only the part it actually needs. Your sessions stop dying after thirty minutes and start running for hours.

Why It’s Useful: Most context bloat isn’t your code, it’s the junk around it. This keeps the junk out of the room while still letting Claude look it up when it has to.

Install: search the marketplace for context-mode, or find it on [***GitHub***](https://github.com/davesque/claude-context-mode)***.***

### 3\. Token Savior

Claude has a bad habit: it reads the whole file even when it only needed five lines.

Token Savior fixes the greed. It serves code in layers. A summary first, then the exact snippet, then the full file only when Claude is genuinely editing it. Most of the time the summary is enough, and you never pay for the other 300 lines.

Why It’s Useful: Blind full-file reads are one of the quietest token drains there is. Reading less, on purpose, adds up fast across a day.

Install: search the marketplace for token-savior, or check [***GitHub***](https://github.com/Mibayy/token-savior)***.***

### 4\. RTK

This one is for anyone whose logs are a mess. Git, npm, build output, test runs, all of it is 90% noise and 10% signal.

RTK sits between those commands and Claude and squeezes the output down before it ever reaches the context, often by 60 to 99%. Claude still sees the errors that matter. It just doesn’t have to wade through a thousand lines of “installing package…” to find them.

Why It’s Useful: Command output is invisible spend. You didn’t type it, but you’re paying to store every line. RTK deletes the boring parts for you.

Install: it’s a small command-line tool, grab it from its [***GitHub repo***](https://github.com/rtk-ai/rtk) and run your commands through it.

### 5\. code-review-graph

When Claude wants to understand your codebase, it opens files. Then more files. Then the same files again next time you ask a related question.

code-review-graph maps your whole project once, using the same parsing your editor uses, and stores it in a small local database. After that, questions like “what calls this function?” get answered from the map instead of by re-reading half the repo.

Why It’s Useful: Re-reading files you’ve already read is pure repeat spend. Build the map once, answer a hundred questions cheaply.

Install: find it on [***GitHub***](https://github.com/tirth8205/code-review-graph) and run it against your repo.

### 6\. Handoff

Long sessions get heavy. Every message drags the whole history along, and by hour two you’re paying a premium just to keep going.

Handoff compresses the stuff that matters, the decisions, the current state, the next steps, into a short markdown file. You start a fresh session, hand it that file, and you’re back to a light, cheap context without losing the plot.

Why It’s Useful: The cheapest context is a fresh one. Handoff lets you reset without re-explaining your whole project from scratch.

Install: search the marketplace for handoff, or grab it from [***GitHub***](https://github.com/dariohansson/claude-handoff)***.***

### A few habits that save even more

Skills do the heavy lifting, but small habits stack on top:

- Run /context now and then to see exactly what’s eating your window. You can’t cut what you can’t see.
- Use /clear or /rewind to delete failed attempts. A dead-end you keep dragging along is dead weight.
- Keep your CLAUDE.md short, under a page. Long instruction files get re-read on every single turn.
- Add a.claudeignore so Claude skips node\_modules, build folders and giant lock files.
- Point small look-around tasks at a cheaper model like Haiku, and save the expensive model for real work.
- Start a new session for a new task. One giant thread costs more than five clean ones.

### FAQ

***Do these skills change Claude’s answers?***  
Mostly no. They cut waste, not quality. Caveman trims words, the rest trim context. The actual code you get stays the same.

***Which one should I install first?  
***Caveman and Context Mode. Between them they handle the two biggest leaks, chatty replies and noisy tool output.

***Are they free?***  
The skills here are free to install. Some connect to tools with their own pricing, but the skill layer itself costs nothing.

Thanks for reading:)

Token usage is one of those things you ignore until the bill shows up, and then it’s all you can think about. Pick two of these, watch your /context drop, and go build something.

If you’ve got a token-saving trick in your own setup, drop it in the comments, I’m always hunting for the next one.