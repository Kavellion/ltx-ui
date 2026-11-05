# LTX UI

**Status: CODE VERIFIED ONLY.** This build passed its full
sandbox bench suite (real tokenizer loads, the app's own
loaders against real model files, an end-to-end generation
through the app with real pipeline classes, and a real-shape
decode bench). It has **not** yet run on the target PC, so no
speed, memory, or first-run claim here is PC-proven yet.

A simple standalone Gradio app for **LTX-Video 0.9.8 dev
(13B)** — text-to-video and image-to-video in one window,
built for a 12 GB graphics card with CPU offload.

- One prompt → a short MP4 clip (silent; this model makes
  no audio).
- Optional input picture → the clip animates the picture
  (image-to-video).
- Prompt queue (one prompt per line), Clips per prompt,
  session history, Stop, Unload, saved setups, Star batch.
- Native size 1216x704, 121 frames at 24 fps ≈ a 5-second
  clip. Sizes snap to the model's 32-pixel grid; frame
  counts to 8n+1.

## The model

- **LTX-Video 0.9.8 dev**, the full 13B dev model — real
  classifier-free guidance and a working negative prompt.
  (The 0.9.8 weights are byte-identical to the 0.9.7 dev
  weights; Lightricks never published a 0.9.8 diffusers
  repo, so components load from
  `Lightricks/LTX-Video-0.9.7-dev` while the transformer
  GGUF comes from `QuantStack/LTXV-13B-0.9.8-dev-GGUF`.)
- **Not gated** — no license click, no Hugging Face token.
- **License: LTXV Open Weights License** (not Apache).
  Fine for personal use; read the license in the model repo
  before using clips commercially.
- LTX-2 / LTX-2.3 (the newer 19B/22B audio-video models)
  were assessed and deliberately **not** chosen: even their
  FP8 checkpoints plus the Gemma text encoder do not
  honestly fit a 12 GB card with 48 GB of RAM, and their
  license is more restrictive. See BUILD-NOTES.txt.

## How it fits

- The 13B transformer loads from a **GGUF** file.
  Default **Q3_K_S** (5.86 GB) is the quant that leaves
  headroom on a 16 GB RAM machine; **Q4_K_S** is the
  recommended pick once RAM is 32 GB or more. Q4_K_M,
  Q5_K_S, Q5_K_M, Q6_K and Q8_0 are also offered.
- The T5-XXL text encoder loads in **FP8** by default
  (torchao, quantized at load — parks ~4.8 GB); BF16 is
  offered for 32 GB+ machines (parks ~9.5 GB). The encoder
  download is ~19 GB either way.
- The VAE stays BF16; decoding is **tiled and chunked in
  the app** (the pipeline returns latents; the app decodes
  a few latent frames at a time), so a full clip's frames
  are never decoded in one giant tensor — the failure shape
  that killed early Wan runs on 12 GB cards.
- CPU offload is always on. Parked weights at the
  defaults: ~13.1 GB of system RAM.
- First Generate downloads **~27 GB** at the default
  quant (GGUF 5.9 + encoder 19.0 + VAE 2.5 GB).

## Prompting

LTX was trained on long descriptive prompts. Write a short
paragraph: what moves, where, what the camera does, how
the light looks. The model hears up to ~128 tokens. The
app's Random prompt button composes examples in that
shape.

The card's optional multi-scale workflow (a second spatial
upscaler model and extra passes) is **not** in this build —
one model, one pass, by design.

## Install & run (Windows)

1. Extract the zip anywhere (e.g. `C:\AI\ltx`).
2. Run `install.bat` once (bundled Python; ~2.6 GB
   PyTorch download plus requirements).
3. Run `run.bat`. The page opens at 127.0.0.1 in your
   browser.

Requires an NVIDIA card (built and benched against an
RTX 4070 12 GB target) and ~35 GB of free disk.

## Honest notes

- At the defaults (121 frames, 1216x704, 30 steps) expect
  **minutes per clip**, not seconds, on a 12 GB card with
  offload. The 480p presets are the fast lane. Per-step
  progress prints in the terminal window.
- Changing the transformer quant or encoder precision in
  the UI reloads the model on the next Generate.
- Saved setups live in `setups.json` next to `app.py`;
  clips land in `videos_out\<batch name>\` with a
  `settings.txt` per run; Star batch copies the last batch
  into `videos_out\favorites\`.

License for this app code: MIT (see LICENSE).
Model license: LTXV Open Weights License — see the model
repositories.
