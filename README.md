# ICU ICD-10 Coder (DCCM-Mode-3)

A single-file, offline, browser-only tool that helps adult critical-care clinicians
turn free-text clinical notes into an ICD-10-CM problem list — and, optionally, a
structured clinical summary. Everything runs on the device: **no server, no backend,
no external API, and no patient data ever leaves the machine.**

**Live tool:** https://kna85.github.io/DCCM-Mode-3/

> ⚠️ **Coding aid, not a source of truth.** Every code and summary must be validated
> against the institution's current ICD-10-CM/AM master and the original note before
> any clinical, billing, or documentation use. Do not paste patient identifiers
> (name, MRN, DOB) when testing.

---

## What it does

The tool has three tabs.

**Coder** — Paste the history, senior input, and any extra information into the three
panes and press *Generate ICD-10 list*. A curated, weighted keyword matcher scans the
text against a reviewed critical-care ICD-10-CM seed list, applies clause-level
negation (so "no GI bleed" is held back rather than coded), and returns:

- suggested codes (high confidence) and possible codes to review;
- codes that were mentioned but negated, listed separately and not selected;
- an educational, non-validated SOI/ROM acuity estimate;
- a deterministic, offline structured summary that already includes the selected codes.

Select the codes you want, then *Copy selected* or *Download CSV*.

**Reference list** — The full seed code set, searchable and filterable by specialty
group, with one-click copy of any code.

**AI Summary** — Paste a clinical note and generate a multi-paragraph narrative
summary of it. The suggested ICD-10 codes from the coder engine are appended
automatically beneath the summary (see *AI Summary details* below).

---

## AI Summary details

The AI Summary tab runs a small language model **entirely inside your browser** via
WebGPU (using [WebLLM](https://github.com/mlc-ai/web-llm)). The model weights
(~1–2 GB) download once from a CDN and are then cached, so subsequent use works
offline. The pasted text is processed locally and is never transmitted to OpenAI,
Anthropic, Google, or any server.

Available models:

| Model | Notes |
|---|---|
| Llama 3.2 · 3B | Balanced — default |
| Llama 3.2 · 1B | Fastest, smallest |
| Qwen 2.5 · 3B | Alternative |

How it works:

1. The model writes a faithful, multi-paragraph narrative summary — background and
   reason for admission, active problems and their lab trajectory, examination
   findings by system, and the ongoing plan — using only what is stated in the note.
2. The **suggested ICD-10 codes are produced by the deterministic coder engine, not
   by the language model.** This keeps every code grounded in the reviewed seed list
   and preserves negation handling, so the model cannot invent or misformat codes.

**Capability note:** the in-browser model is small (1–3 B parameters) — the price of
keeping everything on-device. It produces a usefully structured draft but is less
polished than a large cloud model and can miss or misread values in dense lab tables.
Treat the summary as a draft and verify it against the source note.

Requires a recent desktop **Chrome** or **Edge** with WebGPU. The Coder and Reference
tabs work in any browser; only the AI Summary tab needs WebGPU.

---

## Privacy

- Fully client-side. No backend, no analytics, no network calls except the one-time
  model-weights download in the AI Summary tab.
- The clinical text you paste stays in the browser tab and is discarded when you
  close or reload it.
- Do not enter real patient identifiers during testing.

---

## Deployment

The tool is a single `index.html` with no build step and no dependencies to install.

**GitHub Pages (current setup)**

1. Keep `index.html` and the empty `_nojekyll` file in the repository root
   (`_nojekyll` stops GitHub from running Jekyll on the page).
2. Enable GitHub Pages for the repository.
3. Pushing an updated `index.html` redeploys automatically; every device sees the new
   version at the same URL within a minute or two.

**Hospital PC with no internet**

Serve `index.html` from an internal web server or a shared drive. The Coder and
Reference tabs work with no network at all. The AI Summary tab needs its model weights
cached beforehand (open it once on a machine with internet, or host the WebLLM model
files internally).

---

## Maintaining the code list

- The code set is the `SEED` array in `index.html`. Each row is
  `["CODE", "Description", "Specialty · Group"]` — keep that exact shape and grouping.
- Clinician shorthand lives in the `SYN` map (e.g. AKI, CRRT, DKA, NSTEMI). Add new
  abbreviations there, mapped to the canonical term used in a description.
- The matcher builds a weighted keyword index and applies clause-level negation.
  Preserve negation behaviour when editing.
- Never add a backend, external script, or network call to the Coder or Reference
  logic — the "no PHI leaves the machine" guarantee depends on it. (The AI Summary
  tab's one-time model download from a CDN is the only permitted network access.)
- After edits, keep it a single self-contained `index.html`.

---

## Disclaimer

This tool is provided for educational and documentation-support purposes only. It is
not a medical device and does not provide medical advice. The SOI/ROM acuity estimate
is a transparent heuristic, **not** the validated 3M APR-DRG grouper, and must not be
used for prognostication, benchmarking, or billing. A qualified clinician is
responsible for validating all output before any clinical use.
