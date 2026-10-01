---
name: codestd
description: Coding conventions for writing, editing and reviewing code, with packs for Go and Python and language-agnostic rules for every other language. Use before writing or changing code, and when asked to review a diff, a branch, a pull request or a path against the conventions.
argument-hint: "[review [<diff | branch | path>]]"
---

# codestd

## Load the rules

1. List the files you are about to write or change. For a review, list the files in the target.
2. Read [core/conventions.md](core/conventions.md).
3. Read the pack for each language among those files:

   | Files | Pack |
   |---|---|
   | `*.go` | [languages/go.md](languages/go.md) |
   | `*.py`, `*.pyi` | [languages/python.md](languages/python.md) |

   A language without a pack follows the core alone.
4. Precedence: the project's own rules (`AGENTS.md`, `CLAUDE.md`, the linter configs in the
   repository), then the pack, then the core.

## Writing code

- Apply the rules to the code you add or change. Report a violation in code the change does not
  touch; do not fix it in the same change.
- Use a library only when the project already declares it as the pack describes. Ask before adding
  a dependency.
- Before finishing, check the change against core section 10 (agent-written code).

## Reviewing (`review`)

- Target: the diff, branch or path given as the argument. With no argument, review the working-tree
  changes against the default branch.
- Check each changed line against the core and the matching pack. Confirm every finding in the code
  before reporting it; drop what you cannot confirm.
- Report one table, most severe first:

  | Severity | File:line | Rule | Finding | Fix |
  |---|---|---|---|---|

  Severity is `CRITICAL` (correctness, security, broken contract), `WARNING` (should fix before
  merge) or `SUGGESTION`. Rule is the core or pack ID, such as `core 4.1` or `go: Errors`.
- A review only reads. Do not edit files, commit, or post comments unless asked.
