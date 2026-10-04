# XTRAPP — the post/sentiment backend repo

XTRAPP is the dedicated GitHub repo that hosts the post-reader data and the
learned sentiment model, wiring into the TRAPP2 app the same way TRAPP2-1 hosts
FX/13F/macro data.

## What it holds

`data/xtrapp_data.json` — a single file with two parts:
```json
{
  "_schema": "valuatio-xtrapp-v1",
  "updatedAt": "...",
  "lexicon": { "bull": { "phrase": weight }, "bear": { ... }, "meta": {...} },
  "posts": [ { tracked call objects, entry price frozen } ]
}
```

- **lexicon** = the learnable sentiment model. Grows every time you confirm a
  call's direction in the app's Review tab. This is the "retraining" — confirmed
  phrases get recognized on future posts.
- **posts** = your tracked calls with frozen entry prices and performance.

## How it's wired (already built in the app)

- On load, the app fetches `xtrapp_data.json` from XTRAPP and merges the lexicon
  (repo wins on conflict, local-only learnings preserved) + posts. Console logs
  `[xtrapp] loaded lexicon ... + N posts`.
- The **Calls tab** (📣) reads posts, parses ticker/target/sentiment, tracks
  performance, overlays markers on the price chart.
- The **Review tab** (🧠) is the human-in-the-loop: confirm low-confidence
  guesses → retrains the lexicon → export back to XTRAPP.

## Setup (from your computer)

1. Create the **XTRAPP** repo under your GitHub account.
2. In the app's Review tab, click **Export for XTRAPP** → downloads
   `xtrapp_data.json`. Commit it to `XTRAPP/data/xtrapp_data.json`.
3. The app reads it back automatically on next load. The repo is now the shared
   source of truth, so the model + posts persist across devices.

If you later want **automatic** write-back (instead of export/commit), that's the
same pattern as the Editor: a small serverless function holding a GitHub token
commits the file. For now, export → commit keeps it databaseless and simple.

## The LLM "deep understanding" hook

`llmClassifyPost(text)` in app.js is the scaffold for true language comprehension.
It returns null (lexicon-only) until you set `valuatio.llm.key` in localStorage and
wire your provider's call inside it. The app falls back to the lexicon gracefully,
so it never silently depends on an API you haven't set up. The lexicon is the
honest, working core; the LLM is the upgrade path.

## Honest scope

The lexicon is lightweight phrase-frequency learning — transparent and editable,
it only knows what you've validated and stays conservative on the rest (right for
noisy X posts). It is NOT a neural net; that's what the LLM hook is for. Screenshot
reading still needs a vision API (the parser downstream of extraction already works).
