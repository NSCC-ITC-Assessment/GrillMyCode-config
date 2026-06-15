# Bundled prompt fallbacks

These Markdown files are the **bundled fallback copies** of GrillMyCode's prompts, shipped inside the action's Docker image.

At runtime the action prefers the **live** versions of these files from the prompt config repo (`<action-repo>-config`, e.g. `grillmycode-config`), fetched from `prompts/` on its default branch. Each fragment falls back independently: a file here is used only when its live counterpart can't be fetched (config repo unreachable, file missing, or a network/permissions error). See [`../prompt.js`](../prompt.js) (`loadPromptPart` / `loadPromptParts`).

**To change the prompts in normal operation, edit the config repo — not these files.** Editing here only changes the fallback and requires rebuilding/releasing the action to take effect. Keep these in sync with the config repo so a fallback never silently serves a stale prompt.

## Files

| File                    | Role                                                             |
| ----------------------- | ---------------------------------------------------------------- |
| `system.md`             | Tier 1 — core rubric & formatting rules (always used)            |
| `user.md`               | The per-run user message (always used)                           |
| `assignment-context.md` | Tier 2 wrapper — used only when `assignment_context` is set      |
| `instructor-context.md` | Tier 3 wrapper — used only when `instructor_context` is set      |
| `context-summary.md`    | Summary instruction — used only when `instructor_context` is set |

The directory layout here mirrors the config repo's `prompts/` exactly, so this folder can be copied over as the starting point for that repo.

## Placeholders

Each file uses `{{placeholder}}` tokens that the action substitutes at runtime (see `applySubstitutions` in [`../prompt.js`](../prompt.js)). Leave the tokens intact when editing — the untrusted-input boundary markers (`{{untrustedOpen}}`/`{{untrustedClose}}`/`{{refOpen}}`/`{{refClose}}`) are a security boundary around student-controlled content, not just formatting.
