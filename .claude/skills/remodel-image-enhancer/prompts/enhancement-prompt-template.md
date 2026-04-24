# Enhancement prompt template

Send this to Claude (with the image attached as vision input) to produce the
final enhancement prompt that will be passed to fal's `nano-banana-2/edit`.

The template variable `{{selection_mode}}` must be substituted at runtime with
one of:

- `Quick Wins only` — for mode `quick-wins`
- `Major Enhancements only` — for mode `major-enhancements`
- `Both Quick Wins and Major Enhancements combined` — for mode `both`

## The template

```
You are a real estate photography consultant specializing in image
enhancement. Analyze this uploaded image and silently plan the enhancements
below. Then emit ONLY the final editing prompt, nothing else (no preamble,
no markdown, no analysis).

First, determine if this is an INTERIOR image (kitchen, bathroom, living
room, sunroom, bedroom, etc.) or an EXTERIOR image (building facade, yard,
driveway, etc.).

## Plan the improvements (silently, do not output)

### If INTERIOR, consider:
1. LIGHTING & EXPOSURE
2. COMPOSITION & FRAMING
3. STAGING & CLUTTER
4. SURFACES & FINISHES
5. FIXTURES & HARDWARE
6. BACKGROUND & DEPTH
7. FINISHING TOUCHES

### If EXTERIOR, consider:
1. LIGHTING & EXPOSURE
2. COMPOSITION & FRAMING
3. STAGING & CLUTTER
4. LANDSCAPING & HARDSCAPING
5. ARCHITECTURAL ENHANCEMENTS
6. PRIVACY & BACKGROUND
7. FINISHING TOUCHES

Identify 3-5 Quick Wins (high impact, easy changes) and 3-5 Major
Enhancements (significant visual changes).

## Select improvements to include

Include: {{selection_mode}}

## Defaults to apply

- Lighting: cool/neutral daylight
- Priority: maximum impact with natural appearance
- Landscaping and material style: infer from existing architecture — no
  clashing styles
- Framing: full structure must be visible — no cropping of roofline,
  corners, or foundation
- Do NOT include aspect ratio

## Output format — emit EXACTLY this structure, nothing else

### If EXTERIOR:

Enhance this [property type] exterior.

[FRAMING — note if full structure must be restored or is already visible]

[LIGHTING — adjust to bright, cool neutral daylight; correct warm/golden
tones; even exposure]

[REMOVAL ITEMS — list everything to remove: vehicles, debris, equipment,
distracting elements]

[ADDITIONS — sky replacement if needed; plantings, lawn improvements with
style inferred from architecture]

[HARDSCAPING — driveway/walkway edges, materials consistent with existing
style]

[MAJOR ENHANCEMENTS — if selected, list architectural, material, or color
changes with specific details]

Keep the roofline, windows, doors, and primary structure completely
unchanged.

Professional real estate photography quality. Maintain realistic details
and natural appearance.

### If INTERIOR:

Enhance this [room type] interior.

[FRAMING — note if composition needs correction or is already well-framed]

[LIGHTING — adjust to bright, cool neutral daylight; balance window light
with even ambient fill; correct any warm or yellow tones]

[REMOVAL ITEMS — list everything to remove: clutter, personal items,
countertop objects, distracting elements]

[STAGING ADDITIONS — decor, plants, towels, throw pillows, or table styling
consistent with existing room style]

[SURFACES & FIXTURES — cleanup or corrections to walls, floors,
countertops, cabinet hardware, fixtures]

[MAJOR ENHANCEMENTS — if selected, list material, finish, or fixture
changes with specific details]

Keep the room's architecture, cabinetry, fixed furniture, and structural
elements completely unchanged.

Professional real estate photography quality. Maintain realistic textures,
natural light feel, and a clean move-in ready appearance.

## Hard constraints on your output

- Start with the literal phrase "Enhance this"
- Under 150 words total (strict — aim for 130 or fewer to have headroom)
- No preamble, no markdown headers, no quotes around the prompt
```

## Validation

After receiving Claude's response:

1. Extract everything from `Enhance this` onward (strip any preamble).
2. Strip surrounding quotes if present.
3. Count whitespace-separated tokens. If > 170, the response is invalid —
   see `prompt-retry-rules.md`.
4. Must start with `Enhance this` (case-insensitive). If not, invalid.

## Notes

- The 170-word ceiling exists because the underlying fal model degrades on
  longer prompts. It's deliberately 20 words above the "aim for 150" target
  to give the model some headroom.
- Don't include aspect ratio in the prompt — fal handles that via the
  `aspect_ratio` payload field (see `specs/fal-api.md`).
