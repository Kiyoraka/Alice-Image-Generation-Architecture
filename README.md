# Alice Image Generation — Architecture Diagram

A single-file, self-contained HTML diagram that maps **how a request becomes a rendered PNG** in Alice's image-generation pipeline: the **Prism** orchestrator (the skill that decides *who* & *how*) × the **gpt-image-2** capability layer (the only code that touches the OpenAI API).

> **Snapshot:** Prism Lv.12 · gpt-image-2 sole pipeline (Niji/Midjourney retired at Lv.9) · mapped May 25, 2026.

---

## What this is

`index.html` is a static, dependency-free architecture diagram. No build step, no server, no external assets — open it in any modern browser and it renders the full pipeline as a top-to-bottom flow of color-coded layers. It is documentation, not running code; the actual pipeline lives in the `alice-enchantments` plugin (Prism skill) and the `Invoke-AliceImageGen.ps1` helper.

## How to view

```
Open index.html in any browser — double-click it, or:
  • Windows:  start index.html
  • macOS:    open index.html
  • Linux:    xdg-open index.html
```

Fully responsive — three breakpoints (desktop / tablet ≤760px / phone ≤480px); wide tables scroll horizontally on small screens.

---

## The pipeline it documents

The diagram reads top-to-bottom through five layers, with a read-only knowledge base feeding the orchestration steps.

| # | Layer | What happens |
|---|-------|--------------|
| **1** | **Trigger** | Direct (Kiyo: *"prism this"*, *"make an image of…"*, *"render to [path]"*) or chained (Mirror, Fantasy-Gate, Shopping, Muse, Forge call Prism). |
| **2** | **Prism orchestration** | Decide WHO & HOW, then compose. Steps 1–6 below. |
| **3** | **Step 6.5 — Render gate** | `gpt-image` dialect → always render. `danbooru`/unrecognized → manual-paste fallback. |
| **4** | **Render capability** | `Invoke-AliceImageGen.ps1` — the only code that touches the API. |
| **5** | **Output routing** | Where the PNG lands, by scene type. |

### Layer 2 — Prism orchestration steps

- **Step 1 / 1B** — Fresh-read appearance from `current-session-memory.md` (never trust cached outfit); check Home-outfit-vs-public-scene compatibility.
- **Step 2** — Load outfit + hairstyle keywords from `Appearance/.../info.md`.
- **Step 3 (Lv.10 orthogonal decomposition)** — **Subject ⊥ Style**: who is rendered (Alice / Kiyo / Couple / External / scene) is decided independently of which template skeleton is used (solo-alice, magazine-cover, pokemon-trainer, wardrobe-ref, couple-scene, commemoration, fantasy-gate…).
  - **Step 3a.1 (side gate)** — External character: disambiguate via `WebSearch` + `AskUserQuestion`, fetch canonical features via `WebFetch`.
- **Step 3.5 / 3.6** — Commemoration detection (Forge Lv.up chains visualize the *actual* changes); resolve dialect + `target_folder` + canon identity block.
- **Step 4 (Lv.10 / Lv.12 compose)** — Substitute identity block × outfit/hair keywords × shot type × scene × mood into the chosen template. Appearance words live **only** in canon files, never inline.
- **Step 4B / 4C / 5** — Color isolation (purple aura fit/clash), anchor-variety audit (no repeated domestic props), moderator-safety calibration.
- **Step 6** — Present the final prompt in a dialect-formatted code block.

### Layer 4 — Render engine (`Invoke-AliceImageGen.ps1`)

`Resolve key` → `Build request` → `POST` OpenAI Images API → `Decode b64 → PNG`.

- **Security boundary (Mandatory Rule 16):** the API key is read inside the helper and never returned; `sk-...` patterns are scrubbed to `[REDACTED-KEY]` on any error before the exception is thrown. The key never appears in logs, diary, commits, or chat.
- **Returns** a `PSCustomObject`: `saved_path`, `cost_estimate_usd`, `elapsed_seconds`, `model`, `size`, `quality` — cost is reported every render (Rule 19; renders never feel "free").

### Layer 5 — Output routing by scene type

| Scene type | Destination | Auto-render |
|------------|-------------|-------------|
| commemoration | `daily-diary/current/{date}/` | yes (Forge chain) |
| couple / memory | `daily-diary/current/{date}/memories/` | yes |
| fantasy-gate | `fantasy-gate/{world}/scenes/` | yes |
| album-cover | `Appearance/scenes/album-covers/` | conditional (Muse) |
| draft | `D:\test-images\` | yes (throwaway) |
| external / other / wardrobe / hairstyle / appearance | `image-gen/` | conditional — curated to canonical library after-the-fact |

### Knowledge & data sources (read-only — everything is data-driven, not hardcoded)

- **Appearance:** `main/current-session-memory.md`, `Appearance/wardrobe/*/info.md`, `Appearance/hairstyles/*/info.md`
- **Identity canon:** `library/wisdom/alice-visual-canon.md`, `library/wisdom/kiyo-visual-canon.md`
- **Style skeletons:** `prism/templates/`, `library/prompt/images/chatgpt/{magazine-cover, pokemon-trainer-infographic}/prompt.md`
- **Moderation calibration:** `library/prompt/images/gpt-image-2-moderation-calibration.md`
- **API reference:** `library/prompt/images/openai-image-api-reference.md`
- **Credentials:** `Environment/.env` → `OPENAI_API_KEY` (git-ignored, read by helper only)

---

## Legend

| Color | Meaning |
|-------|---------|
| 🟪 Soft purple | Trigger |
| 🟣 Purple | Prism orchestration step |
| 🟡 Amber (dashed) | Knowledge / data source (read-only input) |
| 🟢 Green | Render capability (touches OpenAI API) |
| 🩷 Pink | Output routing |

---

## Design philosophy

> 💜 Prism Lv.12 thin-orchestrator × gpt-image-2 capability — **the skill decides, the helper renders.**
> Architecture-absorption pattern: Prism owns rendering directly (Lv.8+), the same shape as Seal→git and Mirror→file-IO.

## File structure

```
Alice Image Generation Architecture/
├── index.html    # the self-contained architecture diagram (HTML + inline CSS, no JS)
└── README.md     # this file
```
