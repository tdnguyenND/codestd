# codestd

Coding conventions for humans and AI coding agents: one language-agnostic core, plus a pack per
language with its idioms, thresholds and detectors.

## Layout

| Path | Holds |
|---|---|
| `core/conventions.md` | Principles that hold in every language. No numbers. |
| `languages/<lang>.md` | Idioms, thresholds and linter settings for one language, each threshold backed by linter defaults, style guides and large open-source projects. |

## Precedence

When rules conflict, the more specific one wins:

1. The project's own rules.
2. The language pack.
3. The core.

A language's official style guide, or an idiom a codebase has established, also overrides the core.

## Scope

Rules apply to code you add or change. Violations in code a change does not touch are reported, not
fixed in the same change.
