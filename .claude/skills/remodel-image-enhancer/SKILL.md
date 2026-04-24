---
name: remodel-image-enhancer
description: Enhance real-estate / remodel / construction photos using Claude + fal.ai nano-banana-2/edit. Use this skill when the user asks to enhance, clean up, retouch, or improve images in the `input/` folder of this repo.
---

# Remodel Image Enhancer

You enhance raw construction / remodel / real-estate photos using **Claude**
(for classification and prompt generation) and **fal.ai's
`fal-ai/nano-banana-2/edit`** model (for the actual enhancement).

All the logic is in markdown (this file + `prompts/` + `specs/`). You write and
execute a small Python script at runtime to do the work. Do not create a
persistent script in the repo — write it to a temp location, run it, and
discard it.

## When to invoke

Any time the user asks you to enhance, clean up, retouch, or improve photos in
this repo — typically images in `input/`. Also use this skill when the user
says things like "run the enhancer", "enhance these", or "process the images".

## Inputs

- **Source images**: everything in `input/` (JPG, JPEG, PNG, WEBP).
- **API keys**: loaded from `.env` at the repo root
  (`ANTHROPIC_API_KEY`, `FAL_API_KEY`). If `.env` is missing, stop and tell
  the user to create it from `.env.example`.
- **Slug** (optional): a short project identifier the user can specify
  (e.g. `addition_2024`). If the user doesn't specify one, default to
  `project`.
- **Selection mode** (optional): `quick-wins`, `major-enhancements`, or
  `both`. Default to `both`.

## Workflow

Run these steps, sequentially (no concurrency — one image at a time):

### 1. Verify environment

- Confirm `.env` exists at the repo root and contains both keys.
- Confirm `input/` has at least one image file (extensions: `.jpg`, `.jpeg`,
  `.png`, `.webp`). Ignore `.DS_Store` and any other non-image files.
- Confirm Python dependencies are installed: `anthropic`, `fal-client`,
  `python-dotenv`. If any are missing, tell the user to run
  `pip install -r requirements.txt`.

### 2. Classify each image

For every image, call Claude (model: `claude-haiku-4-5`) with the prompt in
[`prompts/scene-classifier.md`](prompts/scene-classifier.md) and the image as
vision input. Collect a short snake_case tag like `kitchen`, `bathroom`,
`deck`, `exterior`.

### 3. Rename files

Sort images alphabetically (deterministic ordering). Assign sequential
two-digit numbers starting at `01`. Rename following
[`specs/naming-convention.md`](specs/naming-convention.md):

```
{slug}_{NN}_{scene}.{ext}
```

Example: `project_01_deck.jpg`, `project_02_kitchen.webp`.

Keep the original file extension, lowercased.

### 4. Generate the enhancement prompt

For each renamed image, call Claude (model: `claude-sonnet-4-5` — vision is
needed and this template benefits from a stronger model) using the template
in [`prompts/enhancement-prompt-template.md`](prompts/enhancement-prompt-template.md).
Substitute `{{selection_mode}}` with the user's chosen mode (default `both`).

The response must start with `Enhance this` and be under 170 words. If it
isn't, retry up to 3 times; if still failing and mode is `both`, fall back to
`quick-wins`. See [`prompts/prompt-retry-rules.md`](prompts/prompt-retry-rules.md).

### 5. Upload to fal storage

Use `fal_client.upload_file(path)` to get a public URL for the renamed image.
This is the only image-hosting step — no Cloudinary, no base64.

### 6. Call fal's nano-banana-2/edit

Follow [`specs/fal-api.md`](specs/fal-api.md). Send the enhancement prompt
and the uploaded image URL. Expect an enhanced image URL back.

If fal returns a `no_media_generated` error, retry up to 3 times with
`auto_fix: true`. If it still fails, log and move on — don't abort the
whole batch.

### 7. Download and save

Create an `enhanced/` folder at the repo root (alongside `input/`) if it
doesn't exist. Download the fal result and save it as:

```
enhanced/{slug}_{NN}_{scene}_enhanced.png
```

Regardless of input format, the enhanced file is always `.png`.

### 8. Report

After the batch finishes, print a short summary: total processed, failures
(if any, with reasons), and where outputs live.

## How to structure the runtime script

Write a single Python script to a temp path (e.g. `/tmp/enhance_run.py`) that:

1. Loads `.env` via `python-dotenv`.
2. Uses `anthropic.Anthropic()` for classification + prompt generation.
3. Uses `fal_client.upload_file()` and `fal_client.run()` (or the HTTP
   equivalent documented in `specs/fal-api.md`) for enhancement.
4. Uses `urllib.request` or `subprocess curl` to download the final PNG —
   prefer curl if Python SSL complains about certs on macOS.

Don't split this into multiple files. Don't save it inside the repo. When
the batch is done, the script can be deleted.

## What NOT to do

- Don't run enhancements in parallel (no concurrency).
- Don't do before/after compositing — that's out of scope for this skill.
- Don't commit anything to git on behalf of the user.
- Don't upload to any service other than fal (no Cloudinary, no S3).
- Don't cache results to disk — each run is a fresh batch.
- Don't skip the `Enhance this` + 170-word validation; it's what keeps fal
  from rejecting the prompt.
