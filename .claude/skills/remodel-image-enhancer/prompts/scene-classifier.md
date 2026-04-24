# Scene classifier prompt

Send this to Claude alongside the image (as vision input). The response
should be a single snake_case tag, nothing else.

## System / user message

```
Classify this real-estate or construction photo into ONE short snake_case
scene tag. Prefer a tag from this list when one fits:

deck, backyard, exterior, front, front_porch, screened_porch,
kitchen, bathroom, bedroom, living_room, dining_room, basement, garage,
hallway, stairs, interior, roof, framing, foundation

If none of those fit, invent a short snake_case tag (1-2 words).

Output ONLY the tag. No punctuation, no explanation.
```

## Expected output

A single token like:

```
kitchen
```

or

```
front_porch
```

## Post-processing rules

- Lowercase the response.
- Replace anything that isn't `[a-z0-9_]` with `_`.
- Strip leading/trailing underscores.
- If the response is empty after normalization, fall back to `scene`.
