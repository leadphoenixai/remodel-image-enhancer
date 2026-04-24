# Naming convention

Every input image gets renamed to a consistent, sortable, human-readable
filename before enhancement. The enhanced output mirrors the input name
with `_enhanced.png` appended.

## Pattern

```
{slug}_{NN}_{scene}.{ext}
```

| Part    | Meaning                                             | Example       |
|---------|-----------------------------------------------------|---------------|
| `slug`  | Short project identifier (user-specified or default) | `project`    |
| `NN`    | Two-digit sequence number, starting at `01`          | `01`, `02`…  |
| `scene` | snake_case scene tag from the classifier             | `kitchen`     |
| `ext`   | Original file extension, lowercased                  | `jpg`         |

### Examples

```
project_01_deck.jpg
project_02_kitchen.webp
addition_2024_03_front_porch.jpeg
```

## Output name

The enhanced image uses the same core stem with `_enhanced` and a forced
`.png` extension:

```
project_01_deck.jpg        →  project_01_deck_enhanced.png
project_02_kitchen.webp    →  project_02_kitchen_enhanced.png
```

## Rules

- **Default slug** is `project`. If the user specifies one (e.g.
  "use slug `addition_2024`"), use theirs verbatim — don't re-slugify.
- **Ordering**: sort inputs alphabetically (case-insensitive) before
  assigning numbers. Deterministic = repeatable runs produce the same
  names.
- **Scene tag**: comes from the classifier. Already snake_case. Use as-is.
- **Extension**: keep the original's extension for the input rename, but
  lowercase it. `.JPG` → `.jpg`, `.JPEG` → `.jpeg`.
- **Collisions**: if a target name already exists (from a prior run),
  overwrite. This is a re-runnable skill, not a vault.

## What to do if the user wants to re-slug later

They can just rename the folder prefix with a shell command:

```bash
cd input
for f in project_*; do mv "$f" "${f/project_/addition_2024_}"; done
```

No need to re-run the skill — the slug is cosmetic, not load-bearing.

## What to skip

- `.DS_Store`, `Thumbs.db`, hidden dotfiles
- Anything that isn't a supported image extension (`.jpg`, `.jpeg`, `.png`,
  `.webp`)
- Files already matching `{slug}_NN_scene.{ext}` — treat them as already
  renamed and don't rename twice
