# Training AI on Your Own Code — Speaker Notes

Brian "bdougie" Douglas · The Paper Compute Company · b.dougie.dev

A written form of the talk. Pacing target: ~60 min (~30s per content slide + 4 live demos, ~5 min each).

---

## Intro

This is Brian Douglas — bdougie on the internet. Ex-GitHub (~5 years). Now building The Paper Compute Company. Nothing to sell — this is all open source from the last couple of months. The booth's upstairs.

The talk in one line: what happens when you stop throwing away agent session data — from speedrunning Pokémon Red to fine-tuning a model on your own work.

Three tools recur throughout: **tapes**, **stereOS**, and **sweeper**.

## Who's talking

GitHub for ~5 years, content & community. Now Paper Compute. Background is finance — guilty pleasure is SEC filings.

The disclaimer: I have **nothing** to sell you. Everything here is open source. The only way to pay us is a ⭐⭐⭐⭐⭐ review. (And water at the end.)

## How this goes

This is a speed run, not a workshop. No tidy step 1 → 2 → 3. I'm sharing how I actually learned this. There *is* a blog post with the numbered steps and the full setup — I'll share it later. Also: this got me banned from Claude. Use at your own discretion.

> Blog: **What I learned running 10 Pokémon bots in 36 seconds** — https://briandouglas.me/posts/2026/03/10/what-i-learned-running-10-pokemon-bots-in-36-seconds/ (full setup, screenshots, and the numbered logs). Code: `papercomputeco/pokemon`.

---

## Chapter 01 — It starts with Pokémon

Who knows Pokémon? Good. This all connects — bear with me.

### The game

Pokémon Red, Game Boy, 1996. I'm an elder millennial — I spent *a lot* of time here. It was a big deal.

I read a paper (Karpathy-style auto-research — more on that later) and wanted to validate what we're building at a serious amount of scale. An intense amount of scale.

### The goal

Speedrun Pokémon, **1,000 turns per session**. Each session is an **epoch**. Every run captured, then I learn from it and feed it back. That's the **self-healing loop**.

- 1,000 turns per session
- 1 epoch = 1 session, captured
- ∞ loop — learn, feed back, repeat

### PyBoy — Game Boy, headless

PyBoy runs Game Boy emulators in Python. ROMs are the flashed games. `window="null"` means headless — I don't watch it. No rendering, no 60fps cap, ~100× real-time.

The loop each turn: read memory, decide, press a button, tick.

```python
from pyboy import PyBoy

pyboy = PyBoy("rom/pokemon_red.gb", window="null")  # no display
while pyboy.tick():            # ~100× real-time, no 60fps cap
    state = read_memory(pyboy) # battle / HP / map / coords from RAM
    action = decide(state)
    press(pyboy, action)       # up / down / left / right / A / B
```

### The unlock — Claude Code is just a harness

Then it clicked: use Claude Code (or Codex) to drive PyBoy. It says "code," but it's a **general-purpose harness**. You can drive almost anything with it. Even a Game Boy.

### The setup

Each turn: tick PyBoy, read memory, decide, send buttons. `paperd` proxies the LLM calls and records every session. Concretely, the memory reader (`read_overworld_state()`) pulls `map_id`, `x`, `y`, `badges`, and `party_count`; a fitness pass also tracks `turn_count`, `battles_won`, `maps_visited`, and `stuck_count`. Reaching Oak's Lab = **map 40**. *(See the Appendix for the full mechanics.)*

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

### Two open-source pieces

Two things at Paper Compute, both open source:

- **tapes.dev** — capture sessions. Turn your dead tokens into skills. A log for agents.
- **stereOS** — a runtime for agents. Like a sandbox, but batteries included.

Most folks run agents on their MacBook. The skilled ones use a cloud sandbox. We're all early. If you have that problem, come find us.

---

## Chapter 02 — The agent wasn't learning

Now the part where it all goes wrong — and that's the interesting part.

### The logs

A screenshot every **10 turns**, so the agent could always go back and see what happened — especially when it got blocked crossing a bridge.

### Goal Zero

The first goal was deliberately trivial: you start as a kid, wake up late, walk to Professor Oak's lab, pick a starter. No battling. No conflict. Just get there.

### It ran for hours

And it would **not** get anywhere.

### The failure mode — politely hallucinating progress

The agent celebrated fake wins: "We finished 1,000 turns. We did it!" — that "absolutely right!" energy. (Context: this was February — roughly Codex 5.4 and an early Opus.)

### The bug was me

I forgot to tell the agent to **talk to people**.

The first context clue in the game: *talk to your mom* → she tells you exactly where to go. An agent has no idea to do that. That was the whole blocker.

### The one rule — no internet

My one rule: it can't use the internet to learn how to play. Everything self-learned inside the world I gave it. That constraint is the whole point.

### 🎬 Demo 1 — Pokémon agent, live (~5 min)

The point to land: **there's no separate Python runner — Claude Code *is* the harness.** It reads memory, decides, and presses buttons directly. "The beauty of training an agent is the harness."

- Drive the pokemon-kafka agent straight from Claude Code (no `uv`, no external script)
- Watch it read memory → decide → press buttons; narrate a few turns
- Show a screenshot from `frames/` mid-run

```bash
# in Claude Code, from the pokemon-kafka repo:
> speedrun Pokémon Red headless — read memory, decide, press buttons, to the first Pokémon
```

Have a pre-warmed run ready in case boot is slow.

---

## Chapter 03 — Serving context back

How do you fix "the agent has no awareness of the world"? You serve context back.

### Tapes — the database

I don't review sessions on Fridays — good for you if you do. I dump context into a **tapes** DB. SQLite first, then Postgres because SQLite is single-writer and this was **10 agents × 1,000 turns**, all writing at once, constantly learning and feeding back.

```
sessions
  └─ turns
       ├─ prompts
       ├─ tool calls
       ├─ tokens (in / out)
       └─ skill invocations
```

### Observational memory

A `memory/` folder of human-readable observations, written by the agent in Markdown — like a journal entry at the end of the day. (Concept borrowed from Mastra.) They're readable Markdown I can actually act on.

> `memory/world.md`: NPCs exist. We've talked to **none** of them. We have no awareness of the world. We should build that into the loop.

### observer_state.json — the physics of the world

The agent learned the physics of the world. The discovery: **doors have a cooldown** — you can't spam back and forth through doors/caves to farm. Found it reading `observer.md` and Socratic-chatting with the agent. **Remember this — it comes back in the autotune section as the `door_cooldown` knob.**

*Accuracy note (from the blog):* in code the door cooldown is measured in **frames/turns, default 8** ("eight frames seems like enough" — Log 5), and the fix also needed **oscillation detection** because the stuck counter reset when the agent bounced between two tiles (Log 6). On stage I round it to the "7-second door" for story flow — just don't let the 7 vs 8 trip you up if someone asks.

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

### 3 seconds to Pokémon

A vicious self-healing loop got it to **3 seconds to Pokémon**. Game start → spam `A` (name your character, name your Pokémon) → run to Oak. But you *don't* hit `A` to pick your starter — you hit `B`, because Oak asks yes/no and `A` loops you back. Tiny nuance, huge difference. The observer caught the last trap.

*Accuracy note (from the blog):* the measured figures are ~**5.4s** for a single variant to reach Pokémon selection, **11.1s** for all 10 in parallel, headline "**10 variants in 36s**." The "3 seconds" on stage is the punchy best-case story number — fine to say, just know the logged numbers if pressed.

```diff
- spam A to pick your starter
+ hit B — Oak asks yes/no, and A loops you back
```

### pokemon-kafka

Runs live on **pokemon-kafka**: 10 agents streaming gameplay events through **Kafka**, **Flink** for real-time anomaly detection.

```
agent ×10  ──►  Kafka topics  ──►  Flink  ──►  anomalies
                                              ├─ stuck at a door
                                              └─ HP low → eat a berry / switch
```

Anomalies aren't just bugs — they're *learnings*. "Below X HP, you should switch."

---

## Chapter 04 — The value is being lost

Zoom out from Pokémon to everyone in this room.

### 30 days, then deleted

Claude Code and Codex store your sessions on your machine for **30 days** — then they're gone. We get a 5-hour window and a weekly allowance, pay ~$200/mo, and extract **no lasting value** back. We rent the tokens.

### Brute-force discovery is *good*, actually

Brute-force discovery is a great way to understand a codebase:

- Search & grep to understand a codebase
- Identify the files you should know
- I like writing blog posts about codebases I don't know — I won't write the code, but I need to navigate it and discuss it with engineers

The problem isn't the exploration. It's that the exploration **evaporates**.

---

## Chapter 05 — Sweeper Agent

All the Pokémon work turned into this: **Sweeper Agent**. Funny to say out loud.

### What it does

Sweeps a codebase and massively fixes things.

- Old vibe-coded repo I never touched again
- In **under an hour**: fixed every lint error I'd avoided for years
- ~100 users on it; brought 6-month-old code to a modern standard

**How:** each agent gets its own **stereOS VM** + a **tape session** → 10 tape sessions of data to generate skills, docs, and dev context.

*Aside worth landing (from the live talk):* this is a massive amount of data, and if you're at an enterprise paying for this stuff, you should be **extracting the value**. Some companies have a deal where their data trains Anthropic in exchange for a discount — that's *really* good, expensive data. Cursor is reportedly in a deal worth **$10B minimum with SpaceX**. If you're a Cursor Pro user, congrats — your data is in that deal. People are selling your data because it's valuable. So collect it yourself and do something with it. That's the whole pitch.

### ⚠ Incident report — banned from Claude

Running **10 parallel Sweeper agents** got my Claude account banned. Turns out the fix is: write a blog post, email an apology → unblocked in ~12 hours. All robots answered the email — a blog post was apparently good enough.

### Cheaper models, at scale

Agents don't always need a better model. We just got Opus 4.8 — great, but not for *this*.

- Sweeper is bespoke & rudimentary — up/down/left/right works. Lint fixes. Docs.
- So use **Haiku ×10**. Maybe slower, but it's a background agent running overnight. None the wiser.

### 🎬 Demo 2 — Sweeper, live (~5 min)

- Point Sweeper at a crusty repo with known lint errors
- Fan out N agents in parallel VMs
- Watch lint errors drop to zero — each run recorded as a tape (mention this)

```bash
sweeper run --target "fix all lint" --agents 6 --model haiku
```

---

## Chapter 06 — Anomalies cut both ways

Anomaly detection isn't just failures. **Success is an anomaly too.**

- **Failure:** 10 tool calls failed. What happened?
- **Success:** 26 skill invocations across 10 sessions. Something good is happening here.

Most of us aren't looking with a fine-tooth comb. We say "prompt, no mistakes, clean code, go." There's a lot to learn from that one prompt — which is why keeping trace sessions matters.

---

## Chapter 07 — Tapes: dead tokens → skills

### A Merkle DAG

My co-founder designed this. It's a **Merkle DAG**, like Git — every commit is a node with a hash and branches.

```
session ─┬─ turn ─ turn ─ turn ─┐
         │                      ├─ tools · prompts · tokens · skills
         └─ turn ─ turn ────────┘
```

- Type `claude` / `codex` / `ollama` → a **session** begins
- `clear` or close → a **new session**
- Stored on your machine, **vectorized** for search

### Check the Tapes

A skill that reads back into the database. Go back **6 months**, not 30 days.

```bash
"check the tapes: how did we solve the door cooldown?"
```

This is the thing that replaced `git blame` for me — who's actually using `git blame`? I built a tool on top of **LanceDB** called **GitBlamer** that was basically this. It was bad; I wrote a blog post about it and don't use it anymore. But "check the tapes" does the job — go pull every session where we solved the door-cooldown window and turn it into a story. *(Aside: in the SQLite era the tapes were labeled — front-end vs design vs infra — which made this even sharper. We're not labeling right now, but it's coming.)*

**Real story:** a designer asked for my prompts. I don't save prompts — who does? But the tapes had them. I'd built a wireframe of our cloud product, then said "open a GitHub issue for every feature." Each issue carried the **prompt, intent, and token usage** — exactly what a designer needs.

### tape search

Natural-language, vectorized. Pull the matching sessions → generate a skill.

```bash
tape search "door cooldown window"
```

- A cheap model is fine — I drive with year-old **4o** (I'm cheap)
- First pass is decent
- ⚠️ Have a **human** read the skill before trusting it
- Then use tapes to see how often it gets **invoked** — and whether it works

### Tapes Deck

Open-source, command-line, **no sign-up**. Observability for your sessions: tool calls, skill invocations, in/out tokens.

Tape = the most durable form of media. It's **your** raw data on **your** machine — so don't put secrets in your prompts. Sanitizing is on you.

### 🎬 Demo 3 — Tapes Deck, live (~5 min)

- Open the deck on the command line
- Click into a real session
- Show tool calls, tokens, and a skill invocation
- Run `tape search` and generate a skill from the hits

---

## Chapter 08 — The autotune repo

Everything from here is the **autotune** repo. Open source.

### What if I train the model?

We have skills. Now the side quest: what if I fine-tune a model on my own data?

*Set the scale first (from the live talk):* I've been recording since January — **77,000 tape sessions**, about **4 GB** of data, and I'm generating roughly **1 GB a week**. Push that out to a year and you're looking at **50–100 GB** of your own agent data sitting on your machine that you've never looked at. That's the raw material.

*London callback if the room's right:* this is the home of Pink Floyd, the Rolling Stones, the Beatles. What I love about the Beatles around this era is they intentionally used **fewer tracks** — everyone in the room, played live. Sometimes you do more with less. Right now I have a *lot* — so the question is what to do with it.

### I've done this before

- **Continue → Next Edit:** we trained a tab-completion model with specialized fine-tuning (back when tab completion was cool).
- **Cursor → Composer 2:** same pattern, fine-tuned on user data. Pro Cursor user? Congrats, your data's in the deal.

### A model is a book

A model is a book — lots of information, co-located. (It's why you can ask about Pokémon and restaurants in one session.)

- **SFT = notes in the margins.** Specialized fine-tuning. Write your skills *into* the book. (I actually kind of hate buying a used book full of some stranger's notes — but that's the mechanism.)
- **DPO = CliffsNotes.** The book you read *instead* of the textbook — you get the CliffsNotes, then take the test. It's reinforcement-style: given two choices, always pick the better one. Expensive. *(Great "US university costs so much because we cheat" laugh line — and the Matthew McConaughey "all right, all right, all right": it just picks the right selection every time.)*

**Tooling & the real experiment:** the tool most people reach for is **QLoRA** with **PyTorch** or **Unsloth**. I fine-tuned **Qwen 4B** — a small model — embedding *my* skills (actually me + three teammates' skills) into it. Results were genuinely good for a small local model.

You'll ask: why embed a skill in the model instead of letting the harness invoke it? Honest answer: **why not** — this was a nerd-snipe, and I wanted to test it. It works decently on a MacBook / Linux box for the *bespoke* case (Pokémon, onboarding, weird languages, manual memory management). It is **not** your daily driver and won't replace Sonnet.

### The whole loop

```
        ┌──────────────── autotune loop ────────────────┐
 story  │  Try            Check + Reward         Nudge   │
 spec ─►│  rollout.py ──► verifier.py ──► ┌ nudge_sft.py │──► new genome /
        │  run pk agent   per-beat        └ nudge_steer  │    adapter
        │  (N tries)      pass=1/fail=0                   │
        └────────────────── loop back to Try ────────────┘
```

**Try → Check → Reward (pass=1 / fail=0) → Nudge → loop.** That arrow is the entire training process.

(Timing note: the same week I finished Pokémon, Karpathy's auto-research repo dropped. Qwen 3.6 ships this in the open weights — Alibaba has self-healing in a model today. Lineage also traces to DeepMind's **AlphaEvolve** — source code as a genome, an LLM proposing mutations against a fitness metric.)

*Note to self:* the visible `uv run python -m autotune…` commands are the **evolution harness** from the blog (`run_10_agents.py` / `evolve.py` lineage) — a fitness loop where the *numbers* decide. That's distinct from the **live Pokémon demo**, which is driven straight from Claude Code. Both are "the harness"; keep the two framings straight if asked.

### Try

```bash
uv run python -m autotune.rollout --n 3 --max-turns 500
```

- Runs **pokemon-kafka as the environment** — no edits to pk required
- Driven through pk's already-wired **`EVOLVE_PARAMS`** seam
- Produces **telemetry** for each of the N tries

### Check + Reward

`verifier.py` turns telemetry into a **per-beat** signal, scored against pokemon-kafka's *own* canonical Route-1 story (chapter order from `MAP_PROGRESS`, waypoints from `routes.json`).

```
Pallet → Oak → Route 1 → ... → Viridian
  ✓       ✓       ✓        ✗        ✗     →  reward = 3
```

- Per-beat **pass=1 / fail=0** — a story-shaped, *verifiable* signal
- Not a scalar "fitness blob"

### Nudge ×2

```bash
uv run python -m autotune.loop --generations 1 --n 3 --nudge both
```

- **`--nudge sft`** — rejection-sample the rollouts that **passed** → LoRA SFT a local MLX model → it proposes the next genome.
- **`--nudge steer`** — mutate the genome from what passed + write a note to `notes.md`. **No weight training.**

`both` runs them in one loop.

### The genome — 12 knobs

The thing being tuned is a 12-parameter genome — pokemon-kafka's `EVOLVE_PARAMS`. Note `door_cooldown` is right there: the 7-second door I found by hand earlier is now a tunable parameter.

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

### Runs on your Mac

- **Apple Silicon** via **MLX**
- Default model: **`smollm3-3b-mlx`** (~5 GB) — fits 16 GB+
- Local **LoRA** training, no cloud GPU
- Swap via `AUTOTUNE_BASE_MODEL`

**Use case:** onboarding to a bespoke project, weird languages, manual memory management. **Not** your daily driver — it won't replace Sonnet. (Modeled on overdub, but swapping cloud GPUs for local training.)

### 🎬 Demo 4 — autotune loop, live (~5 min)

- `scripts/loop.sh 1 3 both` — one generation, 3 rollouts
- Show **Try** (rollouts) → **Check** (per-beat reward) → **Nudge**
- Watch SFT validation loss fall, and a new genome get proposed

```bash
uv run python -m autotune.loop --generations 1 --n 3 --nudge both
```

Have a cached run ready — real rollouts are slow.

---

## Chapter 09 — The honest results

This is a journey, not a sales pitch. I ran three constrained experiments to see if autotune could demonstrably **learn** by tuning the genome. Short answer: not yet — and the reasons are instructive. The harness works. The walls were **experimental-design** walls, not code bugs.

### Three ways a learning loop stalls

1. **Saturation** — Route 1 was already solved by 80 hrs of baked-in waypoints. Reward flat at **5.0** every generation — nothing to improve.
2. **Determinism** — a save state replays identically (same RNG). Every rollout `won=1, turns=14`, bit-for-bit. **Zero variance = no gradient.**
3. **Reward validity** — `maps_visited` rewarded wandering *backward*; `stuck_count` rewarded standing still. A reward must be **directional**.

### Root cause

The 12-param genome has **low leverage** over outcomes. It's a set of fine-tuning knobs, not a decision-maker. Real behavior lives in routes, scripts, and accumulated state — tuning knobs rarely flips success vs failure. That's the honest finding, and it's more useful than a fake win.

### The hardware rabbit hole

| Setup | SFT | DPO |
|---|---|---|
| **4B params** | ✅ runs local | ❌ ~$1k/wk, broken |
| **7B params** | ✅ | ⚠️ works, but pricey |

SFT on a 4070 (24 GB) → perfect. DPO wouldn't even run — it needed **32 GB** of VRAM and I had 24 — so I shamelessly upgraded to a 5090 (32 GB) → results middling. Then I borrowed some **H100s** from a friend at a certain GPU company (NVIDIA — name withheld on stage). Bottom line: **DPO on 4B isn't worth it** (~$1k/wk for something broken); DPO on 7B works but is pricey. This is a side quest — I'm meeting with people who actually do research to figure it out properly.

---

## The Arc — the whole thing in 5 steps

1. **Capture the sessions.** Leave this room and go look at `~/.claude/sessions` today. Want them off your machine and durable? **tapes.dev** — open source.
2. **Transfer the knowledge.** The best form of knowledge transfer is a **skill**. Today we're all *solo-player* on our own machines — even your second machine isn't sharing data with your first. Skills are how coding becomes **multiplayer**: the learning moves between people and boxes.
3. **Harness freedom.** Codex better this week? Move your skills over. Claude Code, Codex, anything — you're not locked in.
4. **Model choice.** You don't need Opus all the time. Haiku for the bespoke stuff, a small local model you fine-tuned, some open-weight model. *(Aside if it lands: Anthropic just filed to IPO — my finance background means reading that SEC filing is genuinely my flight-home plan. Past performance doesn't predict future growth, including for models — 4.8 over 4.7, "apparently.")*
5. **Cost.** The other shoe drops. Once you own the data and the skills, you can finally **reason** about cost.

---

## Thank you

Trace the data, reinforce with the learnings. Find me in the Expo Hall.

**Brian "bdougie" Douglas** · The Paper Compute Company · b.dougie.dev

Open source: **tapes.dev · stereOS · sweeper · pokemon-kafka · autotune**

**FAQ — does tapes.dev work for Codex?** Today: Claude Code, Conductor, Ollama. Not Codex yet — happy to make it work if someone needs it.

---

## Appendix — how the game is actually played (from the blog)

Grounding detail from *[What I learned running 10 Pokémon bots in 36 seconds](https://briandouglas.me/posts/2026/03/10/what-i-learned-running-10-pokemon-bots-in-36-seconds/)*. Use this when someone at the booth wants the real mechanics.

**Reading state.** `read_overworld_state()` returns `map_id`, `x`, `y`, `badges`, `party_count`. A fitness pass also tracks `turn_count`, `battles_won`, `maps_visited` (a set), and `stuck_count` (incremented on events containing `"STUCK"`). One early bug: text-box detection keyed off a background *tile* address instead of a game-state flag.

**Pressing buttons.** Built on PyBoy. The gotcha (Log 5): `pyboy.button_press()` doesn't fire reliably in **headless** mode — a real time-sink. Log 6 added **oscillation detection** because the stuck counter kept resetting when the agent bounced between two tiles.

**The loop.** `choose_action()` picks the next input; a waypoint navigator skips a waypoint when stuck. Debugging was manual: *watch it run → notice something wrong → form a hypothesis → write a fix → run again.*

**10 in parallel.** `run_10_agents.py` launches 10 parameter variants at once via subprocess isolation; each variant's genome is passed in through the `EVOLVE_PARAMS` env var. Headless runs at ~100× real-time — **10 runs to Pokémon selection in 11.1s**, individual variants ~5.4s, headline "10 variants in 36s."

**The genome (EVOLVE_PARAMS).** `stuck_threshold` (default 8), `door_cooldown` (default 8), `waypoint_skip_distance` (3), `axis_preference_map_0` ("y"), and more. Variants raced: `baseline`, `low_stuck`(4), `high_stuck`(12), `short_door`(4), `long_door`(12). **`short_door` won** (score 40375, 9 stuck, 5.4s); `long_door`/conservative did worst (40340, 16 stuck) — walking *away* from a door faster beat lingering.

**Fitness.** `score()` = `final_map_id×1000 + badges×5000 + party_size×500 + battles_won×10 − stuck_count×5 − turns×0.1`. Weighted toward map progress, penalizes getting stuck, mildly prefers fewer turns. Warning that maps to Chapter 09's *reward validity*: reward `maps_visited` naively and you incentivize teleport/back-and-forth exploits.

**Lineage.** Inspired by DeepMind's **AlphaEvolve** — treat source code as a genome, LLM proposes mutations against a fitness metric. (On stage I also tie this to Karpathy's auto-research repo and Qwen's open-weights self-healing — same idea, converging from three directions.)

**Blog images (candidates for the `frames/` placeholders in Ch. 02):** "Pokémon Red agent running headless in the terminal" (hero) · "Single agent navigating Pallet Town" · "Multiple agents in a split-screen view" · "Ten agents racing simultaneously."
