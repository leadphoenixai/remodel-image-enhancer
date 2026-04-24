# Prompt retry rules

The enhancement prompt from Claude must satisfy two constraints before being
sent to fal:

1. It must start with `Enhance this` (case-insensitive).
2. It must be ≤ 170 words (whitespace-separated tokens).

When either fails, apply these rules.

## Retry policy

### Attempt 1 — same mode, fresh call

Re-run the prompt-generation call unchanged. Claude's output is
non-deterministic; a second call often lands under the limit.

### Attempt 2 — same mode, fresh call

Same as attempt 1. Do not change the template.

### Attempt 3 — same mode, fresh call

Last chance with the user's chosen selection mode.

### Fallback — downgrade selection mode

If three attempts all fail AND the selection mode was `both`, downgrade
to `quick-wins` (shorter prompts by design) and retry up to twice more.

If the mode was already `quick-wins` or `major-enhancements`, give up on
this image. Log the failure and continue the batch.

## Why not truncate?

Truncating risks cutting the prompt mid-instruction (e.g. chopping the
closing "Keep the roofline … unchanged." line), which degrades the fal
output. Better to regenerate than mangle.

## Logging

On failure, print one line per image:

```
[FAIL] project_05_kitchen.webp — prompt exceeded 170 words after 5 attempts
```

Do not abort the whole batch on a single failure.
