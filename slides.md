---
theme: seriph
title: Training AI on Your Own Code
info: Brian "bdougie" Douglas — The Paper Compute Company
class: text-left
transition: slide-left
mdc: true
fonts:
  provider: none
drawings:
  persist: false
---

<!--
============================================================
 PACING: ~60 min. ~30s per content slide + 4 live demos.
 BRAND: Paper Compute — warm paper bg, ink text, brick-red accent,
        Archivo display + Space Mono eyebrows. Editorial, left-aligned.
 Layouts: default | section (chapter="…", dark) | demo | two-cols
 Notes live in HTML comments — press `p` for presenter mode.
============================================================
-->

# Training AI <span class="hl">on Your Own Code</span>

<div class="eyebrow red mt-2">A TALK ABOUT</div>

<p class="mt-6 text-xl ink" style="max-width:46ch">
What happens when you stop throwing away agent session data — from speedrunning Pokémon Red to fine-tuning a model on your own work.
</p>

<div class="mt-10">
  <div class="eyebrow">PRESENTED BY</div>
  <div class="mt-1" style="font-family:var(--font-display);font-weight:800;font-size:1.4rem">Brian "bdougie" Douglas</div>
  <div class="eyebrow mt-1">THE PAPER COMPUTE COMPANY · b.dougie.dev</div>
</div>

<div class="abs-br m-12 flex gap-2">
  <span class="tag fill">TAPES</span>
  <span class="tag">STEREOS</span>
  <span class="tag red">SWEEPER</span>
</div>

<!--
This is Brian Douglas — bdougie on the internet. Ex-GitHub (~5 yrs). Now building The Paper Compute
Company. Nothing to sell — this is all open source from the last couple months. Booth's upstairs.
-->

---
layout: two-cols
---

# Who's <span class="hl">talking</span>

<div class="mt-2">

- **bdougie** on the internet
- 5 years at **GitHub** — content & community
- Now: **The Paper Compute Company**
- Background: finance. Guilty pleasure: SEC filings.

</div>

::right::

<div class="card mt-16">
<div class="eyebrow red">DISCLAIMER</div>
<p class="mt-3 ink">I have <strong>nothing</strong> to sell you.</p>
<p class="ink">Everything here is open source.</p>
<p class="dim mt-2 text-sm">The only way to pay us is a ⭐⭐⭐⭐⭐ review. (And water at the end.)</p>
</div>

<!--
GitHub ~5 years, content & community. Now Paper Compute. I have nothing to sell — open source stuff
from the last couple months. Come talk to us at the booth.
-->

---

<div class="eyebrow red">HOW THIS GOES</div>

# This is a <span class="hl">speed run.</span>

<p class="mt-6 text-xl dim" style="max-width:40ch">Not a workshop. No step 1 → 2 → 3.<br/>Just my journey learning how to do this.</p>

<!--
Don't expect a tidy tutorial — I'm sharing how I actually learned this. There IS a blog post with
step 1/2/3, I'll share it later. Also: this got me banned from Claude. Use at your own discretion.
-->

---
layout: section
chapter: CHAPTER 01
---

# It starts with <span class="hl">Pokémon.</span>

<!--
I'll start with Pokémon. Who knows Pokémon? Good. This all connects — bear with me.
-->

---
layout: two-cols
---

# Pokémon Red <span class="hl">/ the game</span>

<div class="mt-2">

- Game Boy, **1996**
- I'm an elder millennial — I spent *a lot* of time here
- It was a big deal

</div>

<div class="card soft mt-6">
I read a paper, and wanted to test what we're building <strong>at scale.</strong> An intense amount of scale.
</div>

::right::

<img src="/hero2.png" class="mt-12" style="border:2px solid var(--ink);box-shadow:6px 6px 0 var(--ink)" />

<!--
Came out when I was a kid, big deal. I read a paper (Karpathy-style auto-research, more later) and
wanted to validate our open-source stuff at a serious amount of scale.
-->

---

# Pokémon Red <span class="hl">/ the goal</span>

<div class="grid grid-cols-3 gap-5 mt-8">

<div class="card">
<div class="stat xl">1,000</div>
<p class="mt-1 ink">turns per session</p>
</div>

<div class="card">
<div class="stat xl">1 epoch</div>
<p class="mt-1 ink">= 1 session, captured</p>
</div>

<div class="card">
<div class="stat xl">∞ loop</div>
<p class="mt-1 ink">learn, feed back, repeat</p>
</div>

</div>

<p class="mt-8 text-xl">Speedrun Pokémon → capture every run → <strong>a self-healing loop.</strong></p>

<!--
Speedrun Pokémon, 1000 turns per session. Each session is an epoch. Every run captured, then I learn
from it and feed it back. That's where the self-healing loop starts.
-->

---

# PyBoy <span class="hl">/ Game Boy, headless</span>

```python
from pyboy import PyBoy

pyboy = PyBoy("rom/pokemon_red.gb", window="null")  # no display
while pyboy.tick():            # ~100× real-time, no 60fps cap
    state = read_memory(pyboy) # battle / HP / map / coords from RAM
    action = decide(state)
    press(pyboy, action)       # up / down / left / right / A / B
```

A Python library to run **any** Game Boy game with **no display server.** ~100× real-time.

<!--
PyBoy runs Game Boy emulators in Python. ROMs are flashed games. window=null = headless = I don't
watch it. No rendering, no 60fps cap, ~100x real-time. Read memory, decide, press, tick.
-->

---

<div class="eyebrow red">THE UNLOCK</div>

# Claude Code is just a <span class="hl">harness.</span>

<p class="mt-6 text-xl dim" style="max-width:42ch">It says "code" — but it's a <strong class="ink">general-purpose</strong> harness. You can drive almost anything with it. Even a Game Boy.</p>

<!--
Then it clicked: use Claude Code (or Codex) to drive PyBoy. Claude Code says "code" but it's a
general-purpose harness — you can do almost anything with it.
-->

---

# Pokémon Red <span class="hl">/ the setup</span>

```
stereOS VM (/workspace)
┌──────────────────────────────────────────────────┐
│  PyBoy (headless, window="null")                  │
│    ↓ memory addresses                             │
│  MemoryReader → BattleState / OverworldState      │
│    ↓                                              │
│  Strategy Engine (heuristic or LLM)               │
│    ↓ button inputs                                │
│  GameController → PyBoy                            │
│                                                   │
│  paperd ← proxies LLM calls, records sessions     │
└──────────────────────────────────────────────────┘
```

<!--
Each turn: tick PyBoy, read memory at known addresses (battle type, HP, moves, map, coords, badges),
decide, send buttons. paperd proxies the LLM calls and records every session.
-->

---

# Two open-source <span class="hl">pieces</span>

<div class="grid grid-cols-2 gap-6 mt-8">

<div class="card">
<span class="tag fill">TAPES.DEV</span>
<h2 class="mt-3">Capture sessions</h2>
<p class="dim">Turn your dead tokens into skills. A log for agents.</p>
</div>

<div class="card">
<span class="tag red">STEREOS</span>
<h2 class="mt-3">A runtime for agents</h2>
<p class="dim">Like a sandbox — but batteries included.</p>
</div>

</div>

<p class="mt-8 dim">Most folks run agents on their MacBook. The skilled ones use a cloud sandbox. We're all early.</p>

<!--
Two things at Paper Compute, both open source. tapes.dev captures sessions. stereOS is a runtime for
agents — a sandbox with batteries included. If you have that problem, come find us.
-->

---
layout: section
chapter: CHAPTER 02
---

# The agent <span class="hl">wasn't learning.</span>

<!--
Now the part where it all goes wrong — and that's the interesting part.
-->

---

# Pokémon Red <span class="hl">/ the logs</span>

<p class="ink text-lg">A screenshot every <strong>10 turns</strong> — so the agent could always go back and <strong>see what happened.</strong></p>

<div class="grid grid-cols-4 gap-3 mt-6">
  <div class="card soft text-center dim text-sm">turn 0</div>
  <div class="card soft text-center dim text-sm">turn 10</div>
  <div class="card soft text-center dim text-sm">turn 20</div>
  <div class="card soft text-center dim text-sm">turn 30</div>
</div>

<p class="dim text-sm mt-4">(swap these for your real frames/ screenshots before the talk)</p>

<!--
Every 10 turns, take a screenshot, so the agent can go back and learn what happened — especially
when it got blocked crossing a bridge.
-->

---
layout: two-cols
---

# Goal <span class="hl">Zero</span>

<div class="mt-4 text-lg">

- Wake up late
- Walk to **Professor Oak's lab**
- Pick a starter

</div>

::right::

<div class="mt-20 card soft">
<p class="ink text-xl">No battling. No conflict.</p>
<p class="ink text-xl"><strong>Just get there.</strong></p>
</div>

<!--
First goal: you start as a kid, wake up late, go to Oak's lab, get your first Pokémon. No battles,
no conflict. I just wanted it to reach that point.
-->

---

# It ran for <span class="hl">hours.</span>

<h1 class="lead mt-4">And it would <span class="hl">not</span> get anywhere.</h1>

<!--
I ran this for hours. It would not get to anything.
-->

---

<div class="eyebrow red">THE FAILURE MODE</div>

# It was politely <span class="hl">hallucinating progress.</span>

<div class="mt-8 card" style="display:inline-block">
<p class="ink">✅ "We finished 1,000 turns. We did it!"</p>
<p class="dim mt-1">— absolutely right!</p>
</div>

<!--
The agent celebrated fake wins. "We finished 1000 turns, we did it!" — that "absolutely right!"
energy. This was February: ~Codex 5.4 and an early Opus.
-->

---
layout: two-cols
---

# The bug was <span class="hl">me</span>

<p class="text-xl mt-2 ink">I forgot to tell the agent to <strong>talk to people.</strong></p>

::right::

<div class="card red mt-10">
The first context clue in the game: <em>"talk to your mom"</em> → she tells you exactly where to go.<br/><br/>
An agent has <strong>no idea</strong> to do that.
</div>

<!--
The first clue in the game: talk to your mom, she tells you exactly where to go. An agent would never
know to do that. That was the whole blocker.
-->

---

<div class="eyebrow red">THE ONE RULE</div>

# No <span class="hl">internet.</span>

<p class="mt-6 text-xl dim" style="max-width:42ch">No looking up how to play Pokémon. Everything <strong class="ink">self-learned</strong> inside the world I gave it.</p>

<!--
My one rule: it can't use the internet to learn how to play. It has to be self-learned within the
ecosystem I give it. That constraint is the whole point.
-->

---
layout: demo
---

# Pokémon agent, live

- Boot the agent headless on Pokémon Red
- Watch it read memory → decide → press buttons
- Show a screenshot from `frames/` mid-run

```bash
mb up && mb attach          # stereOS VM
# or local:
uv run scripts/agent.py rom/pokemon_red.gb --strategy heuristic
```

<!--
DEMO 1 (~5 min): boot the pokemon-kafka agent, attach, narrate a few turns, show a frames/ screenshot.
Have a pre-warmed run ready in case boot is slow.
-->

---
layout: section
chapter: CHAPTER 03
---

# Serving context <span class="hl">back.</span>

<!--
So how do you fix "the agent has no awareness of the world"? You serve context back.
-->

---
layout: two-cols
---

# Tapes <span class="hl">/ the database</span>

<div class="mt-2">

- All the raw data — **a log for agents**
- **SQLite → Postgres** *(SQLite is single-writer)*
- **10 agents × 1,000 turns**, simultaneously
- Constantly learning, feeding back

</div>

::right::

```
sessions
  └─ turns
       ├─ prompts
       ├─ tool calls
       ├─ tokens (in / out)
       └─ skill invocations
```

<!--
I don't review sessions on Fridays — good for you if you do. I dump context into a tapes DB. SQLite
first, then Postgres because SQLite is single-writer and this was 10 agents writing at once, 1000
turns each, all learning and feeding back.
-->

---
layout: two-cols
---

# Observational <span class="hl">memory</span>

A `memory/` folder of **human-readable** observations. Written by the agent, in Markdown.

<p class="dim text-sm mt-2">(concept borrowed from Mastra)</p>

::right::

<div class="card mt-10">
<span class="tag">memory/world.md</span>
<blockquote class="mt-3">
NPCs exist. We've talked to <strong>none</strong> of them. We have no awareness of the world. We should build that into the loop.
</blockquote>
</div>

<!--
Like a journal entry at the end of the day. Observations at the end of sessions. The agent writes
them; they're human-readable Markdown I can actually read and act on.
-->

---

# observer_state.json

The agent learned the **physics of the world.**

```json
{
  "position": { "x": 4, "y": 7, "map_id": "PALLET_TOWN" },
  "facing": "down",
  "learned": {
    "door_cooldown_seconds": 7,
    "note": "can't spam back and forth through doors / caves"
  }
}
```

<p class="mt-3">Found via the observer + <strong>Socratic chat</strong>: <em>doors have a 7-second cooldown</em>. <span class="dim">(remember this number.)</span></p>

<!--
Very Pokémon-specific — in your codebase this lives somewhere else. The discovery: a door has a ~7s
cooldown so you can't spam back and forth to farm. Found it reading observer.md and Socratic-chatting
with the agent. REMEMBER THIS NUMBER — it comes back in the autotune section.
-->

---

# 3 seconds to <span class="hl">Pokémon</span>

<p class="dim">Game start → spam <span class="kbd">A</span> (name char, name Pokémon) → reach Oak.</p>

```diff
- spam A to pick your starter
+ hit B — Oak asks yes/no, and A loops you back
```

<p class="mt-5 text-xl">A self-healing loop got it down to <strong>3 seconds.</strong> The observer caught the last trap.</p>

<!--
A vicious self-healing loop gets it to 3 seconds to Pokémon. Spam A for names, run to Oak. But you
DON'T hit A to pick your starter — you hit B, because he asks yes/no and A loops you. Tiny nuance,
huge difference. The observer found it.
-->

---

# pokemon-kafka

10 agents streaming gameplay events through **Kafka**, **Flink** for real-time anomaly detection.

```
agent ×10  ──►  Kafka topics  ──►  Flink  ──►  anomalies
                                              ├─ stuck at a door
                                              └─ HP low → eat a berry / switch
```

<p class="mt-4 dim">Anomalies aren't just bugs — they're <em>learnings</em>. "Below X HP, you should switch."</p>

<!--
Runs live on pokemon-kafka — 10 agents streaming through Kafka, Flink doing anomaly detection.
Anomalies like not going through doors, or in battle: below a certain HP, eat a berry or switch.
-->

---
layout: section
chapter: CHAPTER 04
---

# The value is <span class="hl">being lost.</span>

<!--
Now zoom out from Pokémon to all of us in this room.
-->

---
layout: two-cols
---

# 30 days. <span class="hl">Then deleted.</span>

<p class="mt-4 text-lg dim" style="max-width:34ch">Claude Code and Codex store your sessions on your machine for <strong class="ink">30 days</strong> — then they're gone.</p>

::right::

<div class="mt-16 card red">
<div class="stat xl">$200<span class="text-base">/mo</span></div>
<p class="mt-2 ink">A 5-hour window. A weekly allowance. And <strong>no output</strong> back.</p>
</div>

<!--
Claude Code and Codex store sessions on your machine for 30 days, then delete. We get a 5-hour window
and a weekly allowance, pay ~$200/mo (£180?), and extract no lasting value. We rent the tokens.
-->

---

# Brute-force discovery is <span class="hl">*good*</span>, actually

<div class="mt-2">

- Search & grep to understand a codebase
- Identify the files you should know
- I like **writing blog posts about codebases I don't know** — I won't write the code, but I need to navigate it and discuss it with engineers

</div>

<p class="mt-6 dim">The problem isn't the exploration. It's that the exploration <strong class="ink">evaporates.</strong></p>

<!--
Brute-force discovery is a great way to understand a codebase — grep, find files, write a blog post
about code you don't know. One of the more fun things I do. The problem is all of it disappears.
-->

---
layout: section
chapter: CHAPTER 05
---

# Sweeper <span class="hl">Agent</span>

<!--
All the Pokémon work turned into this: Sweeper Agent. Funny to say out loud.
-->

---
layout: two-cols
---

# Sweeper <span class="hl">/ what it does</span>

Sweeps a codebase and **massively fixes things.**

<div class="mt-3">

- Old vibe-coded repo I never touched again
- **< 1 hour:** fixed every lint error I'd avoided for *years*
- ~100 users on it; brought 6-month-old code to modern standard

</div>

::right::

<div class="card mt-12">
<span class="tag red">HOW</span>
<p class="mt-3 ink">Each agent gets its own <strong>stereOS VM</strong> + a <strong>tape session.</strong></p>
<p class="dim mt-2">→ 10 tape sessions of data to generate skills, docs, and dev context.</p>
</div>

<!--
Sweeper sweeps a codebase. I had garbage vibe-coded code from last summer I never touched. Ran it,
fixed all the lint errors in under an hour on a project with ~100 users. Or: 10 parallel agents
writing docs and dev context. Each agent = its own stereOS VM, each recording a tape session.
-->

---

<div class="eyebrow red">⚠ INCIDENT REPORT</div>

# This got me <span class="hl">banned from Claude.</span>

<p class="mt-6 text-xl dim" style="max-width:44ch">10 parallel Sweeper agents. Wrote a blog post + emailed an apology → <strong class="ink">unblocked in ~12 hours.</strong></p>
<p class="dim mt-2 text-sm">(All robots answered the email. A blog post was apparently good enough.)</p>

<!--
Running 10 parallel Sweeper agents got my Claude account banned. Turns out: write a blog post, email
an apology, unblocked in ~12 hours. All robots answer the email, but it worked.
-->

---

# Cheaper models, <span class="hl">at scale</span>

<div class="grid grid-cols-2 gap-6 mt-8">

<div class="card">
<h2>Sweeper is bespoke &amp; rudimentary</h2>
<p class="dim mt-2">up / down / left / right works. Lint fixes. Docs.</p>
</div>

<div class="card red">
<h2>So use <span class="hl">Haiku</span> ×10</h2>
<p class="dim mt-2">Maybe slower — but it's a background agent running overnight. None the wiser.</p>
</div>

</div>

<p class="mt-8 dim">Agents don't always need a better model. Opus 4.8 is great — but not for <em>this.</em></p>

<!--
Agents don't always need a better model. We just got Opus 4.8 — great. But Sweeper is bespoke and
rudimentary, so I run Haiku at 10x scale and crush it. Background agents overnight — none the wiser.
-->

---
layout: demo
---

# Sweeper, live

- Point Sweeper at a crusty repo
- Fan out N agents in parallel VMs
- Watch lint errors drop to zero — each run recorded as a tape

```bash
sweeper run --target "fix all lint" --agents 6 --model haiku
```

<!--
DEMO 2 (~5 min): run Sweeper on a repo with known lint errors, show the parallel agents and the
before/after. Mention each agent is recording a tape.
-->

---
layout: section
chapter: CHAPTER 06
---

# Anomalies cut <span class="hl">both ways.</span>

---

# Failure *and* success are <span class="hl">anomalies</span>

<div class="grid grid-cols-2 gap-8 mt-8">

<div class="card red">
<span class="tag red">FAILURE</span>
<p class="mt-3 ink">10 tool calls failed.<br/>What happened?</p>
</div>

<div class="card">
<span class="tag fill">SUCCESS</span>
<p class="mt-3 ink">26 skill invocations across 10 sessions.<br/><strong>Something good is happening here.</strong></p>
</div>

</div>

<p class="mt-8 dim">Most of us aren't looking with a fine-tooth comb. We say <em>"prompt, no mistakes, clean code, go."</em> There's a lot to learn from that one prompt.</p>

<!--
Anomaly detection isn't just failures. Success is an anomaly too — 26 skill invocations out of 10
sessions means something's working. We're shooting from the hip. Even a simple prompt teaches you a
lot. That's why keeping trace sessions matters.
-->

---
layout: section
chapter: CHAPTER 07
---

# Tapes: dead tokens <span class="hl">→ skills.</span>

---

# Tapes <span class="hl">/ a Merkle DAG</span>

Like Git — every commit is a node with a hash.

```
session ─┬─ turn ─ turn ─ turn ─┐
         │                      ├─ tools · prompts · tokens · skills
         └─ turn ─ turn ────────┘
```

<div class="mt-4">

- Type `claude` / `codex` / `ollama` → a **session** begins
- `clear` or close → a **new session**
- Stored on your machine, **vectorized** for search

</div>

<!--
My co-founder designed this. A Merkle DAG, like Git — every commit sits in a DAG with a hash and
branches. Every session has turns. Type claude = session begins; clear or close = new session.
Everything's stored and vectorized.
-->

---
layout: two-cols
---

# Check the <span class="hl">Tapes</span>

A skill that reads back into the database.

```bash
"check the tapes: how did we
 solve the door cooldown?"
```

Go back **6 months**, not 30 days.

::right::

<div class="card mt-4">
<span class="tag red">REAL STORY</span>
<p class="mt-3 ink">Built a wireframe of our cloud product. Then:</p>
<blockquote class="mt-2">"Open a GitHub issue for every feature."</blockquote>
<p class="dim mt-2">Each issue carried the <strong class="ink">prompt, intent, and token usage</strong> — exactly what a designer needs.</p>
</div>

<!--
Check the Tapes goes deeper into the DB. A designer asked for my prompts — I don't save prompts, who
does? But the tapes had them. I said "open a GitHub issue per feature" and each issue had the prompt,
the intent, the token usage — everything a designer needs.
-->

---
layout: two-cols
---

# tape <span class="hl">search</span>

Natural-language, vectorized.

```bash
tape search "door cooldown window"
```

Pull the matching sessions → **generate a skill.**

::right::

<div class="mt-4">

- A **cheap model** is fine — I drive with year-old **4o** (I'm cheap)
- First pass is decent
- ⚠️ Have a **human** read the skill before trusting it
- Then use tapes to see how often it gets **invoked** — and whether it works

</div>

<!--
tape search is vectorized natural language. Pull the sessions, feed them to a small/cheap model — I
use 4o, a year old and cheap — and generate a skill. First pass is decent, but have a human clean it
up. Then the tapes tell you how often the skill actually gets invoked.
-->

---

# Tapes <span class="hl">Deck</span>

Open-source, command-line, **no sign-up.** Observability for your sessions.

<div class="grid grid-cols-3 gap-3 mt-8">
  <div class="card text-center"><div class="stat xl">⚙</div><p class="mt-2 text-sm ink">tool calls</p></div>
  <div class="card text-center"><div class="stat xl">✎</div><p class="mt-2 text-sm ink">skill invocations</p></div>
  <div class="card text-center"><div class="stat xl">⇅</div><p class="mt-2 text-sm ink">in / out tokens</p></div>
</div>

<p class="mt-8 dim text-sm">Tape = the most durable form of media. It's <strong class="ink">your</strong> raw data on <strong class="ink">your</strong> machine — so don't put secrets in your prompts. Sanitizing is on you.</p>

<!--
Tapes Deck — open source, command line, no sign-up. See every session, click in, see all the tokens
and prompts. Cogs = tool calls, pencils = skill invocations. Concern about sensitive prompts? It's
your data on your machine — just don't put secrets in your cloud code.
-->

---
layout: demo
---

# Tapes Deck, live

- Open the deck on the command line
- Click into a real session
- Show tool calls, tokens, and a skill invocation
- Run `tape search` and generate a skill from the hits

<!--
DEMO 3 (~5 min): open Tapes Deck, click into a session, show the token/tool/skill view, then run a
tape search and generate a skill live.
-->

---
layout: section
chapter: CHAPTER 08 · THE AUTOTUNE REPO
---

# What if I <span class="hl">train the model?</span>

<p>Everything from here is the <strong>autotune</strong> repo. Open source.</p>

<!--
We have skills. Now the side quest: what if I fine-tune a model on my own data? This whole last
section is the autotune repo.
-->

---

# I've done this <span class="hl">before</span>

<div class="grid grid-cols-2 gap-6 mt-6">

<div class="card">
<span class="tag">CONTINUE</span>
<h2 class="mt-2">Next Edit</h2>
<p class="dim mt-2">Worked there; we trained a tab-completion model with specialized fine-tuning.</p>
</div>

<div class="card">
<span class="tag">CURSOR</span>
<h2 class="mt-2">Composer 2</h2>
<p class="dim mt-2">Same pattern — fine-tuned on user data. Pro user? Congrats, your data's in the deal.</p>
</div>

</div>

<!--
I worked at Continue, where we trained a model called Next Edit — back when tab completion was cool.
Same technique Cursor used for Composer 2: fine-tuned on your data. Pro Cursor user? Your data's in it.
-->

---
layout: two-cols
---

# A model is a <span class="hl">book</span>

Lots of information, co-located. *(Why you can ask about Pokémon and restaurants in one session.)*

<div class="card mt-6">
<span class="tag">SFT</span>
<h2 class="mt-2">Notes in the margins</h2>
<p class="dim mt-1">Specialized fine-tuning. Write your skills <em>into</em> the book.</p>
</div>

::right::

<div class="card red mt-16">
<span class="tag red">DPO</span>
<h2 class="mt-2">CliffsNotes</h2>
<p class="dim mt-1">The book you read <em>instead</em> of the textbook. Always pick the better of two. Expensive.</p>
</div>

<!--
A model is a book — tons of info co-located. SFT = writing your skill in the margins, adding context
into the book. DPO = CliffsNotes, what you read instead of the textbook to pass the test — picks the
best of two choices every time. Reinforcement-style, expensive.
-->

---

# autotune <span class="hl">/ the whole loop</span>

```
        ┌──────────────── autotune loop ────────────────┐
 story  │  Try            Check + Reward         Nudge   │
 spec ─►│  rollout.py ──► verifier.py ──► ┌ nudge_sft.py │──► new genome /
        │  run pk agent   per-beat        └ nudge_steer  │    adapter
        │  (N tries)      pass=1/fail=0                   │
        └────────────────── loop back to Try ────────────┘
```

> **Try → Check → Reward (pass=1 / fail=0) → Nudge → loop.**
> *That arrow is the entire training process.*

<!--
autotune is that loop, locally. Try: run the pk agent N times. Check+Reward: verifier turns telemetry
into per-beat pass/fail. Nudge: do more of what passed. Loop. That arrow is the whole training process.
Same week I finished Pokémon, Karpathy's auto-research repo dropped; Qwen 3.6 ships this in the open
weights — Alibaba has self-healing in a model today.
-->

---

# autotune <span class="hl">/ Try</span>

```bash
uv run python -m autotune.rollout --n 3 --max-turns 500
```

<div class="mt-4">

- Runs **pokemon-kafka as the environment** — no edits to pk required
- Driven through pk's already-wired **`EVOLVE_PARAMS`** seam
- Produces **telemetry** for each of the N tries

</div>

<!--
Try = rollout.py. Runs the pokemon-kafka agent N times through its already-wired EVOLVE_PARAMS seam,
so I never touch pk. Each run produces telemetry.
-->

---

# autotune <span class="hl">/ Check + Reward</span>

`verifier.py` turns telemetry into a **per-beat** signal.

```
Pallet → Oak → Route 1 → ... → Viridian
  ✓       ✓       ✓        ✗        ✗     →  reward = 3
```

<div class="mt-3">

- The story is **pokemon-kafka's own** Route-1 progression (`MAP_PROGRESS` + `routes.json`)
- Per-beat **pass=1 / fail=0** — a story-shaped, *verifiable* signal
- Not a scalar "fitness blob"

</div>

<!--
Check+Reward = verifier.py. Scores each rollout against pokemon-kafka's OWN canonical Route-1 story —
chapter order from MAP_PROGRESS, waypoints from routes.json. Per-beat pass/fail, so the reward is
story-shaped and verifiable, not one mushy fitness number.
-->

---

# autotune <span class="hl">/ Nudge ×2</span>

```bash
uv run python -m autotune.loop --generations 1 --n 3 --nudge both
```

<div class="grid grid-cols-2 gap-5 mt-4">

<div class="card">
<span class="tag">--nudge sft</span>
<p class="mt-2 text-sm">Rejection-sample the rollouts that <strong>passed</strong> → LoRA SFT a local MLX model → it proposes the next genome.</p>
</div>

<div class="card">
<span class="tag">--nudge steer</span>
<p class="mt-2 text-sm">Mutate the genome from what passed + write a note to <code>notes.md</code>. <strong>No weight training.</strong></p>
</div>

</div>

<!--
Nudge has two backends. sft: rejection-sample the passing rollouts, LoRA-fine-tune a local MLX model,
use it to propose the next genome. steer: mutate the genome from what passed, write a nudge to
notes.md — no weights. `both` runs them in one loop.
-->

---

# The genome <span class="hl">/ 12 knobs</span>

```python
DEFAULT_PARAMS = {
    "stuck_threshold": 8,
    "door_cooldown": 8,          # ← remember the 7-second door?
    "waypoint_skip_distance": 3,
    "hp_run_threshold": 0.2,     # run from a wild battle below 20% HP
    "hp_heal_threshold": 0.25,   # use an item below 25% HP
    "unknown_move_score": 10.0,  # ... 12 params total, each bounded
}
```

The `EVOLVE_PARAMS` genome — the exact knobs the agent already exposes.

<!--
The thing being tuned is a 12-parameter genome — pokemon-kafka's EVOLVE_PARAMS. And look: door_cooldown
is right there. The 7-second door I found by hand earlier is now a tunable parameter. HP thresholds,
move scores, stuck thresholds — 12 knobs, each bounded.
-->

---
layout: two-cols
---

# Runs on your <span class="hl">Mac</span>

<div class="mt-2">

- **Apple Silicon** via **MLX**
- Default model: **`smollm3-3b-mlx`** (~5 GB) — fits 16 GB+
- Local **LoRA** training, no cloud GPU
- Swap via `AUTOTUNE_BASE_MODEL`

</div>

::right::

<div class="card red mt-10">
<span class="tag red">USE CASE</span>
<p class="mt-2 text-sm ink">Onboarding to a bespoke project. Weird languages. Manual memory management.</p>
<p class="dim mt-2 text-sm"><strong class="ink">Not</strong> your daily driver — it won't replace Sonnet.</p>
</div>

<!--
Runs on an M-series Mac via MLX. Default smollm3-3b, ~5GB, fits 16GB+. Local LoRA, no cloud GPU. Use
case is bespoke: onboarding, weird languages, memory management — not a Sonnet replacement. Modeled on
overdub, but swapping cloud GPUs for local training.
-->

---
layout: demo
---

# autotune loop, live

- `scripts/loop.sh 1 3 both` — one generation, 3 rollouts
- Show **Try** (rollouts) → **Check** (per-beat reward) → **Nudge**
- Watch SFT validation loss fall, and a new genome get proposed

```bash
uv run python -m autotune.loop --generations 1 --n 3 --nudge both
```

<!--
DEMO 4 (~5 min): run one generation. Narrate Try/Check/Nudge. Show SFT validation loss dropping and
the proposed genome. Have a cached run ready — real rollouts are slow.
-->

---
layout: section
chapter: CHAPTER 09
---

# The honest <span class="hl">results.</span>

<p>The harness works. The walls were <strong>experimental-design</strong> walls — not code bugs.</p>

<!--
The honest part — this is a journey, not a sales pitch. I ran three constrained experiments to see if
autotune could demonstrably LEARN by tuning the genome. Short answer: not yet, and the reasons are
instructive. The harness itself is built and tested end to end.
-->

---

# Three ways a learning loop <span class="hl">stalls</span>

<div class="space-y-3 mt-5">

<div class="card" style="padding:.7rem 1.1rem">
<span class="tag">1 · SATURATION</span>
<span class="ml-3 text-sm ink">Route 1 already solved by 80 hrs of baked-in waypoints. Reward flat at <strong>5.0</strong> every generation — nothing to improve.</span>
</div>

<div class="card" style="padding:.7rem 1.1rem">
<span class="tag">2 · DETERMINISM</span>
<span class="ml-3 text-sm ink">A save state replays identically (same RNG). Every rollout <code>won=1, turns=14</code>, bit-for-bit. <strong>Zero variance = no gradient.</strong></span>
</div>

<div class="card" style="padding:.7rem 1.1rem">
<span class="tag">3 · REWARD VALIDITY</span>
<span class="ml-3 text-sm ink"><code>maps_visited</code> rewarded wandering <em>backward</em>; <code>stuck_count</code> rewarded standing still. A reward must be <strong>directional.</strong></span>
</div>

</div>

<!--
Three traps, all real. (1) Saturation: Route 1 is already solved by 80 hours of hand-authored
waypoints, reward flat at 5.0 — no gap. (2) Determinism: a save state replays identically, every
rollout won=1, turns=14, bit-for-bit — zero variance, no gradient. (3) Reward validity: maps_visited
rewarded wandering the wrong way; stuck_count rewarded standing still. Reward must be directional.
-->

---
layout: two-cols
---

# Root <span class="hl">cause</span>

<p class="mt-4 text-xl ink">The 12-param genome has <strong>low leverage</strong> over outcomes.</p>

::right::

<div class="card soft mt-12">
<p class="ink">It's a set of fine-tuning <strong>knobs</strong> — not a decision-maker.</p>
<p class="dim mt-2">Real behavior lives in routes, scripts, and accumulated state. Tuning knobs rarely flips success vs failure.</p>
</div>

<!--
Root cause across all three: the genome has low leverage. The agent's real behavior is driven by
hardcoded routes, battle heuristics, accumulated internal state. The genome is fine-tuning knobs, not
the decision-maker. That's the honest finding — more useful than a fake win.
-->

---

# The hardware <span class="hl">rabbit hole</span>

| Setup | SFT | DPO |
|---|---|---|
| **4B params** | ✅ runs local | ❌ ~$1k/wk, broken |
| **7B params** | ✅ | ⚠️ works, but pricey |

<div class="mt-5 eyebrow">🎮 RTX 4070 (24GB) → SFT PERFECT &nbsp;·&nbsp; 💸 5090 (32GB) → MIDDLING &nbsp;·&nbsp; 🤝 BORROWED SOME H100s</div>

<!--
On hardware: SFT on 4B runs local, good. DPO on 4B — don't, ~$1k/week for something broken. DPO on 7B
works but expensive. I did SFT on a 4070 (24GB), perfect; DPO needed 32GB so I shamelessly got a 5090,
results middling, then borrowed some H100s from a friend at a certain GPU company.
-->

---
layout: section
chapter: THE ARC
---

# The whole thing <span class="hl">in 5 steps.</span>

---

# 1 / Capture the <span class="hl">sessions</span>

<p class="text-xl mt-4 ink">Leave this room and go look at <code>~/.claude/sessions</code> today.</p>
<p class="dim mt-3">Want them off your machine and durable? <strong class="ink">tapes.dev</strong> — open source.</p>

<div class="steps mt-10">▰ ▱ ▱ ▱ ▱</div>

<!--
Step 1: capture the sessions. Go look at .claude/sessions today. Pull them off your machine with
tapes.dev if you want them durable.
-->

---

# 2 / Transfer the <span class="hl">knowledge</span>

<p class="text-xl mt-4 ink">The best form of knowledge transfer is a <strong>skill.</strong></p>
<p class="dim mt-3">Solo-player coding becomes multiplayer. The learning moves between people and machines.</p>

<div class="steps mt-10">▰ ▰ ▱ ▱ ▱</div>

<!--
Step 2: knowledge transfer = skills. We all code solo on our own machines; skills are how that
learning becomes multiplayer and moves between people and boxes.
-->

---

# 3 / Harness <span class="hl">freedom</span>

<p class="text-xl mt-4 ink">Codex better this week? Move your skills over.</p>
<p class="dim mt-3">Claude Code, Codex, anything — you're not locked in.</p>

<div class="steps mt-10">▰ ▰ ▰ ▱ ▱</div>

<!--
Step 3: harness freedom. Once it's skills, move between Claude Code and Codex and anything else —
whatever's best this week.
-->

---

# 4 / Model <span class="hl">choice</span>

<p class="text-xl mt-4 ink">You don't need <strong>Opus</strong> all the time.</p>
<p class="dim mt-3">Haiku for the bespoke stuff. A small local model you fine-tuned. Some open-weight model.</p>

<div class="steps mt-10">▰ ▰ ▰ ▰ ▱</div>

<!--
Step 4: model choice. You don't need Opus always — Haiku for bespoke work, a small model you tuned, an
open-weight model. Optionality.
-->

---

# 5 / <span class="hl">Cost</span>

<p class="text-xl mt-4 ink">The other shoe drops.</p>
<p class="dim mt-3">Once you own the data and the skills, you can finally <strong class="ink">reason</strong> about cost.</p>

<div class="steps mt-10">▰ ▰ ▰ ▰ ▰</div>

<!--
Step 5: cost — the other shoe drops, and now you can reason about it. Trace the data, reinforce with
the learnings.
-->

---
layout: section
chapter: TRACE THE DATA · REINFORCE WITH LEARNINGS
---

# Thank <span class="hl">you.</span>

<p><strong>Brian "bdougie" Douglas</strong> · The Paper Compute Company · b.dougie.dev</p>

<div class="mt-6 flex gap-2">
  <span class="tag fill">TAPES.DEV</span>
  <span class="tag">STEREOS</span>
  <span class="tag red">SWEEPER</span>
  <span class="tag">POKEMON-KAFKA</span>
  <span class="tag">AUTOTUNE</span>
</div>

<p class="dim mt-6 text-sm">all open source · find me in the Expo Hall</p>

<!--
Trace the data, reinforce with learnings, find us in the Expo Hall. That's me, bdougie. Thank you.

FAQ — does tapes.dev work for Codex? Today: Claude Code, Conductor, Ollama. Not Codex yet — happy to
make it work if someone needs it.
-->
