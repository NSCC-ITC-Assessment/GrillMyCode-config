# GrillMyCode — Prompt Configuration

This repository holds the **live prompts** used by [GrillMyCode](https://github.com/nscc-itc-assessment/grillmycode) to generate code-comprehension questions. It exists so the assessment prompts can be edited and tuned **without rebuilding or re-releasing the action**.

## How it works

At runtime, GrillMyCode fetches each prompt fragment from the [`prompts/`](prompts/) directory of this repository's **default branch** and assembles them into the messages sent to the AI model. Editing any of these files and committing to the default branch takes effect on the **next action run** — no rebuild, no release, no version bump.

Each fragment falls back **independently**: if a single file is unreachable or missing, the action uses the copy bundled inside its Docker image for *that file only* and still uses your live versions of the rest. If this whole repository is unavailable, every fragment falls back to its bundled copy. Either way, assessments keep working; the action log records exactly which fragments fell back.

## Repository structure

Only the files under `prompts/` are read. Everything else (including this README) is ignored.

```
.
└── prompts/
    ├── system.md               ← Tier 1: core rubric & formatting rules (always used)
    ├── user.md                 ← the per-run user message (always used)
    ├── assignment-context.md   ← Tier 2 wrapper, used only when assignment_context is set
    ├── instructor-context.md   ← Tier 3 wrapper, used only when instructor_context is set
    └── context-summary.md      ← summary instruction, used only when instructor_context is set
```

- File paths must be exactly as shown, under `prompts/` at the repository root.
- The **default branch** (`main`) is what gets read.
- This repo must live under the **same owner/org** as the action, named `<action-repo>-config` (e.g. `grillmycode` → `grillmycode-config`). GrillMyCode derives this automatically; do not rename it.

## How the fragments fit together

The **system message** is assembled as:

```
system.md
  └─ (+ assignment-context.md if assignment_context is provided)
  └─ (+ instructor-context.md  if instructor_context is provided)
  └─ (+ context-summary.md     if instructor_context is provided)
```

The **user message** is `user.md` on its own. Higher-priority instructions are placed later in the prompt on purpose (models weight later content more heavily), so `instructor-context.md` is the genuine override channel.

## Editing the prompts

Each file is plain Markdown with `{{placeholder}}` tokens that GrillMyCode fills in at runtime. Keep every placeholder intact — removing or misspelling one can break formatting or, for the trust-boundary markers, weaken the protection against prompt injection from student code. Only edit the prose; leave the `{{...}}` tokens in place.

### `system.md`
| Placeholder | Replaced with |
|---|---|
| `{{numQuestions}}` | Number of questions requested for the run |
| `{{numQuestionsPlus1}}` | `numQuestions + 1` (used by anti-over-generation rules) |
| `{{SHORT_ANSWER_MAX_CHARS}}` | Max character count for a "short" answer |
| `{{LONG_ANSWER_MAX_CHARS}}` | Max character count for a "long" correct answer |
| `{{untrustedOpen}}` / `{{untrustedClose}}` | The untrusted-input boundary markers (carry a random per-run token) |

### `user.md`
| Placeholder | Replaced with |
|---|---|
| `{{numQuestions}}` / `{{numQuestionsPlus1}}` | Question count and count + 1 |
| `{{truncatedNote}}` | A warning line when the code was truncated (empty otherwise) |
| `{{filesList}}` | Comma-separated list of the changed files |
| `{{codeContent}}` | The student's submitted code (untrusted) |
| `{{untrustedOpen}}` / `{{untrustedClose}}` | The untrusted-input boundary markers |

### `assignment-context.md`
| Placeholder | Replaced with |
|---|---|
| `{{assignmentContext}}` | The instructor-configured assignment material (untrusted reference data) |
| `{{refOpen}}` / `{{refClose}}` | The reference-data boundary markers (carry a random per-run token) |

### `instructor-context.md`
| Placeholder | Replaced with |
|---|---|
| `{{instructorContext}}` | The free-text instructor instructions for the run |

### `context-summary.md`
| Placeholder | Replaced with |
|---|---|
| `{{numQuestions}}` | Number of questions requested for the run |

> ⚠️ **Security note:** `{{codeContent}}` and `{{assignmentContext}}` contain content that students may control. The boundary markers (`{{untrustedOpen}}`/`{{untrustedClose}}`/`{{refOpen}}`/`{{refClose}}`) are what keep that content from being treated as instructions. Do not remove them, and do not move student-controlled placeholders outside their marker pairs.

## Recommended workflow

1. Edit the relevant file(s) under `prompts/` on a branch and open a PR (so changes are reviewed before they go live).
2. Merge to the default branch.
3. The next GrillMyCode run picks up the change automatically.

Because every run reads the default branch, a change is **live the moment it merges** — protect the default branch so prompt changes always go through review.
