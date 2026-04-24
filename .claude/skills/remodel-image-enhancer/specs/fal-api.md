# fal.ai API spec — nano-banana-2/edit

This is the spec for calling fal's `fal-ai/nano-banana-2/edit` model, which
does the actual image enhancement.

## Auth

The fal client reads `FAL_KEY` from the environment. Our `.env` uses
`FAL_API_KEY` for consistency with other fal docs, so set both:

```python
os.environ["FAL_KEY"] = os.environ["FAL_API_KEY"]
```

...or map the one you load from `.env` to `FAL_KEY` before importing
`fal_client`.

## Upload flow

```python
import fal_client
url = fal_client.upload_file(str(local_path))
```

`url` is a signed fal-storage URL valid for the duration of the run. Pass
it in `image_urls`.

## Enhancement call

### Endpoint

`fal-ai/nano-banana-2/edit` — use `fal_client.subscribe()` to block until
completion, or `fal_client.submit()` + poll if you prefer.

### Payload

```python
{
    "prompt": <the enhancement prompt from Claude, validated>,
    "image_urls": [<fal_client.upload_file() result>],
    "num_images": 1,
    "aspect_ratio": "4:3",
    "resolution": "2K",
    "output_format": "png",
    "safety_tolerance": "4",
    "limit_generations": True,
    "auto_fix": False,
}
```

### Recommended call

```python
result = fal_client.subscribe(
    "fal-ai/nano-banana-2/edit",
    arguments=payload,
    with_logs=False,
)
enhanced_url = result["images"][0]["url"]
```

### Response shape

```json
{
  "images": [
    {"url": "https://v3b.fal.media/files/.../...png", "width": 2000, "height": 1500}
  ],
  "seed": 123456
}
```

Take the first image's `url`.

## Retry rules

fal occasionally returns a `no_media_generated` error — the model refused
to produce an output (usually a soft safety or prompt-compatibility issue,
not a hard block).

1. Retry up to 3 times with `auto_fix: True` (lets fal internally adjust
   the prompt slightly).
2. If still failing, log the failure and move on. Do not abort the batch.

## Payload knobs you should NOT change

- `aspect_ratio`: always `"4:3"`. The prompt template is tuned for this.
- `resolution`: always `"2K"`. Lower resolutions produce visible artifacts;
  higher resolutions cost more without meaningfully better results.
- `output_format`: always `"png"`. The skill's output contract says enhanced
  files end in `.png`.
- `num_images`: always `1`. The user picks the best by eye; we don't do a
  selection step.

## Payload knobs a user might legitimately override

- `safety_tolerance`: bump to `"5"` or `"6"` if fal keeps refusing valid
  real-estate content. Default `"4"` is fine for almost all cases.

## Source of truth

fal's model page: https://fal.ai/models/fal-ai/nano-banana-2/edit. If the
schema on that page diverges from this file, the fal page wins — update
this spec.
