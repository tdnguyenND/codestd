# TypeScript

> Applies to TypeScript 5+ (and JavaScript where noted). Formatter: Prettier, dprint or Biome. Linter:
> ESLint 9+ with typescript-eslint. Read with the [core](../core/conventions.md); IDs like `2.2` refer
> to it.

## Thresholds

| Signal | Flag at | Core | Evidence |
|---|---|---|---|
| Parameters | ≥ 4; an options object counts as one; `this: void` and constructors made only of DI parameter properties are not counted | 2.2 | ESLint `max-params` default 3 ([src][esl-params]); typescript-eslint 3 ([src][tse-params]); Deno caps public APIs at 3 with an options object ([guide][deno-args]); Biome 4; SonarJS 7 |
| Tuple returns | ≥ 3 elements; a pair is fine | 2.4 | Airbnb returns an object for multiple values ([guide][air-obj]); Google accepts a 2-tuple but prefers named properties ([guide][gts-tuple]) |
| Nesting | depth > 4 | 3.2 | ESLint `max-depth` default 4 ([src][esl-depth]); SonarJS S134 3 |
| Cognitive complexity | > 15 | 2.6 | SonarJS S3776 15, in its recommended set ([src][sjs-cog]); Biome 15 |
| Function length | ~50 lines, soft | 2.6 | ESLint `max-lines-per-function` 50 ([src][esl-fnlen]); Biome 50 |
| File length | ~300 lines, soft | — | ESLint `max-lines` 300 ([src][esl-lines]); Biome 300; SonarJS 1000 |
| Clones | 50 tokens, review aid | 7.2 | jscpd default 50 tokens ([src][jscpd]); SonarJS `no-identical-functions` |
| Line length | owned by the formatter (Prettier default 80) | 9.1 | Prettier `printWidth` 80 ([src][prettier-width]); Angular 100; ESLint `max-len` is deprecated |

No TypeScript source sets the parameter limit at 3 (flagging at ≥3). None of eight large projects
sampled (VS Code, TypeScript, React, Angular, Next.js, Nest, Playwright, Vue) enables a size or
complexity rule, so treat every row as a review signal unless the project opts in.

## Names (core 1)

- `camelCase` for values and functions, `PascalCase` for types, `CONSTANT_CASE` for module-level
  immutable constants ([Google][gts-const]).
- Acronyms are words: `customerId`, `loadHttpUrl` ([Google][gts-camel], [Deno][deno-acr]).
- No `I` prefix on interfaces ([Google][gts-naming], [TypeScript wiki][ts-wiki], Angular). A codebase
  whose DI framework pairs an interface with a same-name service token (VS Code's
  `IFileService`) keeps its idiom (core precedence).
- Booleans start with `is`, `has`, `can` or `should` ([Angular][ng-bool]).
- Name handlers after what they do (`activateRipple`), not the event (`handleClick`)
  ([Angular][ng-names]).

## Functions (core 2)

- Optional parameters go into an options object; at most one trailing optional positional parameter
  ([Deno][deno-args], [Google][gts-opts]).
- Cancellation (`AbortSignal`, `CancellationToken`) is excluded from the count and goes last.
- Prefer one signature with optional parameters over overloads ([handbook][ts-dos]).
- When a literal argument is unclear and the signature cannot change, name it at the call site:
  `load(path, /* strict= */ true)` ([Google][gts-callsite], TypeScript's `argument-trivia` lint).

## Control flow (core 3)

- No `else` after `return` (`no-else-return`, Airbnb). No nested ternaries (`no-nested-ternary`,
  Airbnb, unicorn).

## Errors (core 4)

- Throw only `Error` objects: `throw new PoolNotFoundError(id)`, never a string
  (`@typescript-eslint/only-throw-error`, [Google][gts-throw]).
- An unawaited promise is a swallowed error (`@typescript-eslint/no-floating-promises`, in
  `recommended-type-checked` ([src][tse-float])).
- Rethrow with the cause: ``throw new Error(`load pool ${id}`, { cause: err })`` (ESLint
  `preserve-caught-error`, recommended since v10 ([src][esl-cause])).
- An empty `catch` needs a comment saying why (`no-empty`, [Google][gts-catch]).
- Catch variables are `unknown`; narrow to `Error` before use ([Google][gts-rethrow]). `strict`
  enables `useUnknownInCatchVariables` for `try/catch`; `use-unknown-in-catch-callback-variable`
  covers `.catch()` callbacks.

## Comments (core 5)

- Exported API carries JSDoc that states the contract ([Google][gts-doc], Angular, Deno). An echo doc
  is rewritten (5.2); `jsdoc/informative-docs` detects docs that only restate the name
  ([src][jsd-informative]).
- No types in JSDoc; the TypeScript signature carries them (`jsdoc/no-types`, [Google][gts-jsdoctype]).
- Call-site parameter comments (`/* strict= */ true`) are kept.
- TODOs carry an issue or owner: `// TODO(#123): ...` ([Deno][deno-todo]).

## Tests (core 6)

- `describe(<unit>)` with `it('should <expected> when <condition>')` ([Angular][ng-tests], Deno).
- `it.each` / `test.each` only when bodies are identical.
- No focused, skipped or commented-out tests committed (`no-focused-tests`, `no-disabled-tests`,
  `no-commented-out-tests` in eslint-plugin-jest and vitest).

## Reuse (core 7)

- Built-ins before libraries: `Array` methods, `Object.groupBy`, `structuredClone`, `URL`.
- A `for…of` loop is preferred over a hard-to-read `reduce` (unicorn `no-array-reduce`, Angular).

## Boundaries (core 8)

- No `console.log` in production code; `console.warn` and `console.error` only where the project's
  logger is not available (Angular, Vue, Playwright).
- Services are injected through the constructor and declared there, not resolved later
  ([VS Code][vsc-di]).
- Export only what another module imports.
- Enforce layer boundaries with an import allow-list (`no-restricted-imports`, Playwright's
  `DEPS.list`).

## Formatting (core 9)

- The formatter owns layout. Prettier does not sort imports ([rationale][prettier-imports]); use
  `import/order` or the formatter's organise-imports feature.
- Import groups: `node:` built-ins, packages, internal aliases, relative paths.

## Linter settings

```js
// eslint.config.js — ESLint ≥ 9.35, typescript-eslint, eslint-plugin-sonarjs
import tseslint from 'typescript-eslint';
import sonarjs from 'eslint-plugin-sonarjs';

export default [
  ...tseslint.configs.recommendedTypeChecked,
  {
    languageOptions: { parserOptions: { projectService: true } },
    plugins: { sonarjs },
    rules: {
      'max-params': 'off',
      '@typescript-eslint/max-params': ['error', { max: 3 }],
      'max-depth': ['error', 4],
      'max-lines': ['warn', { max: 300, skipBlankLines: true, skipComments: true }],
      'sonarjs/cognitive-complexity': ['warn', 15],
      'no-else-return': ['error', { allowElseIf: false }],
      'no-nested-ternary': 'error',
      'no-console': ['error', { allow: ['warn', 'error'] }],
      'preserve-caught-error': 'error',
      '@typescript-eslint/only-throw-error': 'error',
      '@typescript-eslint/no-floating-promises': 'error',
      '@typescript-eslint/use-unknown-in-catch-callback-variable': 'error',
      '@typescript-eslint/naming-convention': [
        'error',
        { selector: 'interface', format: ['PascalCase'], custom: { regex: '^I[A-Z]', match: false } },
      ],
    },
  },
];
```

[esl-params]: https://github.com/eslint/eslint/blob/bfaea12ddc30b458fe4cfbb6c306a62b5299c177/lib/rules/max-params.js#L68
[esl-depth]: https://github.com/eslint/eslint/blob/bfaea12ddc30b458fe4cfbb6c306a62b5299c177/lib/rules/max-depth.js#L48
[esl-fnlen]: https://github.com/eslint/eslint/blob/bfaea12ddc30b458fe4cfbb6c306a62b5299c177/lib/rules/max-lines-per-function.js#L82
[esl-lines]: https://github.com/eslint/eslint/blob/bfaea12ddc30b458fe4cfbb6c306a62b5299c177/lib/rules/max-lines.js#L69
[esl-cause]: https://github.com/eslint/eslint/blob/bfaea12ddc30b458fe4cfbb6c306a62b5299c177/packages/js/src/configs/eslint-recommended.js#L70
[tse-params]: https://github.com/typescript-eslint/typescript-eslint/blob/338fad98047cf80b88e2090c85dab9def3e2e782/packages/eslint-plugin/src/rules/max-params.ts#L64
[tse-float]: https://github.com/typescript-eslint/typescript-eslint/blob/338fad98047cf80b88e2090c85dab9def3e2e782/packages/eslint-plugin/src/configs/flat/recommended-type-checked.ts#L37
[sjs-cog]: https://github.com/SonarSource/SonarJS/blob/f5a24da97cfb88b2bcf0cc39ab06dacd9c3bc62f/packages/analysis/src/jsts/rules/S3776/config.ts#L23
[jscpd]: https://github.com/kucherenko/jscpd/blob/f72c3dcf7610041a2995bddbd23272ed455cd21f/rust/crates/cpd/src/options.rs#L158-L162
[jsd-informative]: https://github.com/gajus/eslint-plugin-jsdoc/blob/33203d2f5792e16e4dfa875004fb88da59f4ab1e/.README/rules/informative-docs.md
[prettier-width]: https://github.com/prettier/prettier/blob/2006ae8397a35c9d1c49761d67776142848604a3/src/main/core-options.evaluate.js#L154-L158
[prettier-imports]: https://github.com/prettier/prettier/blob/2006ae8397a35c9d1c49761d67776142848604a3/docs/rationale.md#L359
[deno-args]: https://github.com/denoland/docs/blob/24105cb7b28a1e2ae35587183566d1d76ac1a242/runtime/contributing/style_guide.md#L81-L88
[deno-acr]: https://github.com/denoland/docs/blob/24105cb7b28a1e2ae35587183566d1d76ac1a242/runtime/contributing/style_guide.md#L549-L551
[deno-todo]: https://github.com/denoland/docs/blob/24105cb7b28a1e2ae35587183566d1d76ac1a242/runtime/contributing/style_guide.md#L38-L45
[air-obj]: https://github.com/airbnb/javascript#destructuring--object-over-array
[gts-tuple]: https://google.github.io/styleguide/tsguide.html#tuple-types
[gts-const]: https://google.github.io/styleguide/tsguide.html#identifiers-constants
[gts-camel]: https://google.github.io/styleguide/tsguide.html#camel-case
[gts-naming]: https://google.github.io/styleguide/tsguide.html#naming-style
[gts-opts]: https://google.github.io/styleguide/tsguide.html#parameter-initializers
[gts-callsite]: https://google.github.io/styleguide/tsguide.html#comments-when-calling-a-function
[gts-throw]: https://google.github.io/styleguide/tsguide.html#only-throw-errors
[gts-catch]: https://google.github.io/styleguide/tsguide.html#empty-catch-blocks
[gts-rethrow]: https://google.github.io/styleguide/tsguide.html#catching-and-rethrowing
[gts-doc]: https://google.github.io/styleguide/tsguide.html#document-all-top-level-exports-of-modules
[gts-jsdoctype]: https://google.github.io/styleguide/tsguide.html#jsdoc-type-annotations
[ts-wiki]: https://github.com/microsoft/TypeScript/wiki/Coding-guidelines
[ts-dos]: https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html
[ng-bool]: https://github.com/angular/angular/blob/3f9191166936166085320652eef1a562504aa3e4/contributing-docs/coding-standards.md#L175
[ng-names]: https://github.com/angular/angular/blob/3f9191166936166085320652eef1a562504aa3e4/contributing-docs/coding-standards.md#L176-L218
[ng-tests]: https://github.com/angular/angular/blob/3f9191166936166085320652eef1a562504aa3e4/contributing-docs/coding-standards.md#L255-L280
[vsc-di]: https://github.com/microsoft/vscode/blob/038fc2b33ba70491390813f5c2facf13c33c4ab9/.github/copilot-instructions.md#L152
