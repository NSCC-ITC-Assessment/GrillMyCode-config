# GrillMyCode — Prompt Configuration

This repository holds the **live system prompt** used by [GrillMyCode](https://github.com/nscc-itc-assessment/grillmycode) to generate code-comprehension questions. It exists so the assessment prompt can be edited and tuned **without rebuilding or re-releasing the action**.

## How it works

At runtime, GrillMyCode fetches [`prompts/system.md`](prompts/system.md) from this repository's **default branch** and uses it as the core assessment rubric (Tier 1 of the prompt). Editing that file and committing to the default branch takes effect on the **next action run** — no rebuild, no release, no version bump.

If this repository is unreachable (network error, missing file, permissions), the action automatically falls back to the copy bundled inside its Docker image. Assessments keep working either way; they just use the last-shipped prompt until this repo is reachable again.

## Repository structure

Only one file is ever read. Everything else (including this README) is ignored.

```
.
└── prompts/
    └── system.md      ← the prompt template (required, exact path)
```

- **File path** must be exactly `prompts/system.md` at the repository root.
- **Branch** read is the repository's default branch (`main`).
- This repo must live under the **same owner/org** as the action, named `<action-repo>-config` (e.g. `grillmycode` → `grillmycode-config`). GrillMyCode derives this automatically; do not rename it.

## Editing the prompt

`prompts/system.md` is a plain Markdown template. It supports the following `{{placeholder}}` tokens, which GrillMyCode substitutes at runtime:

| Placeholder | Replaced with |
|---|---|
| `{{numQuestions}}` | Number of questions requested for the run |
| `{{numQuestionsPlus1}}` | `numQuestions + 1` (used by anti-over-generation rules) |
| `{{SHORT_ANSWER_MAX_CHARS}}` | Max character count for a "short" answer |
| `{{LONG_ANSWER_MAX_CHARS}}` | Max character count for a "long" correct answer |
| `{{untrustedOpen}}` | Opening marker of the untrusted-input boundary |
| `{{untrustedClose}}` | Closing marker of the untrusted-input boundary |

> ⚠️ Keep every placeholder intact. Removing or misspelling one (e.g. the untrusted-input markers) can break question formatting or weaken the prompt-injection trust boundary. Only the rubric **text** should be edited — leave the `{{...}}` tokens in place.

## Recommended workflow

1. Edit `prompts/system.md` on a branch and open a PR (so changes are reviewed before they go live).
2. Merge to the default branch.
3. The next GrillMyCode run picks up the change automatically.

Because every run reads the default branch, a change is **live the moment it merges** — consider protecting the default branch so prompt changes always go through review.
