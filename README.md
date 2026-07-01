# Training AI on Your Own Code — talk slides

[Slidev](https://sli.dev) deck for Brian "bdougie" Douglas. One ~60-minute talk covering the
arc: **pokemon-kafka → Sweeper → tapes → autotune (fine-tuning)**.

Styled to match the Paper Compute brand deck (warm paper bg, ink text, brick-red accent,
Archivo display + Space Mono eyebrows, `"Topic / subtitle"` headers).

## Run it

```bash
npm run dev          # http://localhost:3030  — hot-reloads slides.md
```

- **Presenter mode** (speaker notes from the transcript): press `p` or visit `/presenter`
- **Overview** of all slides: press `o`
- Navigate: arrow keys / space
- **Jump between chapters: `Shift`+`→` / `Shift`+`←`** (chapters = the section-divider
  slides; detected automatically, so it keeps working if you reorder them — see
  [`global-bottom.vue`](./global-bottom.vue))

## Pacing (60 min)

- **61 slides** at ~30s each ≈ 30 min of slides
- **4 LIVE DEMO bumpers** (Pokémon agent, Sweeper, Tapes Deck, autotune loop) ≈ 25–30 min
- Chapter dividers (dark slides) mark the 9 sections + the closing arc

## Edit

Everything is in [`slides.md`](./slides.md). `---` starts a new slide; speaker notes are the
`<!-- ... -->` blocks. Design system is [`style.css`](./style.css); custom layouts in
[`layouts/`](./layouts) (`section` = dark chapter break, `demo` = LIVE DEMO bumper).

Brand tokens (in `style.css`): `--paper #f4efe5` · `--ink #1c1813` · `--red #bf4231`.

## Export

```bash
npm i -D playwright-chromium
npm run export        # -> slides-export.pdf
npm run build         # -> static site in dist/
```

## TODO before presenting

- Drop real Pokémon `frames/` screenshots into "Pokémon Red / the logs"
- Swap placeholder boxes on the Tapes Deck slide for real screenshots
- Confirm the 4 demo commands match your live setup; pre-warm slow runs
- Add blog-post links (Pokémon, Sweeper, fine-tuning) where referenced
