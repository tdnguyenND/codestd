# TypeScript

> Applies to TypeScript 5+ (and JavaScript where noted). Formatter: Prettier, dprint or Biome. Linter:
> ESLint 9+ with typescript-eslint. Read with the [core](../core/conventions.md); IDs like `2.2` refer
> to it.

## Thresholds

| Signal | Flag at | Core |
|---|---|---|
| Parameters | ≥ 4; an options object counts as one; `this: void` and constructors made only of DI parameter properties are not counted | 2.2 |
| Tuple returns | ≥ 3 elements; a pair is fine | 2.4 |
| Nesting | depth > 4 | 3.2 |
| Cognitive complexity | > 15 | 2.6 |
| Function length | ~50 lines, soft | 2.6 |
| File length | ~300 lines, soft | — |
| Clones | 50 tokens, review aid | 7.2 |
| Line length | owned by the formatter (Prettier default 80) | 9.1 |

## Names (core 1)

- `camelCase` for values and functions, `PascalCase` for types, `CONSTANT_CASE` for module-level
  immutable constants.
- Acronyms are words: `customerId`, `loadHttpUrl`.
- No `I` prefix on interfaces. A codebase whose DI framework pairs an interface with a same-name
  service token (`IFileService`) keeps its idiom.
- Booleans start with `is`, `has`, `can` or `should`.
- Name handlers after what they do (`activateRipple`), not the event (`handleClick`).

## Functions (core 2)

- Optional parameters go into an options object; at most one trailing optional positional parameter.
- Cancellation (`AbortSignal`, `CancellationToken`) is not counted and goes last.
- Prefer one signature with optional parameters over overloads.
- When a literal argument is unclear and the signature cannot change, name it at the call site:
  `load(path, /* strict= */ true)`.

## Control flow (core 3)

- No `else` after `return`. No nested ternaries.

## Errors (core 4)

- Throw only `Error` objects: `throw new PoolNotFoundError(id)`, never a string.
- An unawaited promise is a swallowed error: await it, return it, or handle its rejection.
- Rethrow with the cause: ``throw new Error(`load pool ${id}`, { cause: err })``.
- An empty `catch` needs a comment saying why.
- Catch variables are `unknown`; narrow to `Error` before use. `strict` enables
  `useUnknownInCatchVariables` for `try/catch`; `use-unknown-in-catch-callback-variable` covers
  `.catch()` callbacks.

## Comments (core 5)

- Exported API carries JSDoc that states the contract. An echo doc is rewritten (5.2);
  `jsdoc/informative-docs` detects docs that only restate the name.
- No types in JSDoc; the TypeScript signature carries them.
- Call-site parameter comments (`/* strict= */ true`) are kept.
- TODOs carry an issue or owner: `// TODO(#123): ...`.

## Tests (core 6)

- `describe(<unit>)` with `it('should <expected> when <condition>')`.
- `it.each` / `test.each` only when bodies are identical.
- No focused, skipped or commented-out tests committed.

## Reuse (core 7)

- Built-ins before libraries: `Array` methods, `Object.groupBy`, `structuredClone`, `URL`.
- A `for…of` loop over a hard-to-read `reduce`.

## Boundaries (core 8)

- No `console.log` in production code; `console.warn` and `console.error` only where the project's
  logger is not available.
- Services are injected through the constructor and declared there, not resolved later.
- Export only what another module imports.
- Enforce layer boundaries with an import allow-list (`no-restricted-imports` or a dependency list
  checked in lint).

## Formatting (core 9)

- The formatter owns layout. Prettier does not sort imports; use `import/order` or the formatter's
  organise-imports feature.
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

Test files add `no-focused-tests`, `no-disabled-tests` and `no-commented-out-tests` from
eslint-plugin-jest or the vitest plugin.

## References

- [Google TypeScript Style Guide](https://google.github.io/styleguide/tsguide.html), [Google JavaScript Style Guide](https://google.github.io/styleguide/jsguide.html)
- [TypeScript coding guidelines](https://github.com/microsoft/TypeScript/wiki/Coding-guidelines), [TypeScript handbook: Do's and Don'ts](https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html)
- [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript), [Deno style guide](https://docs.deno.com/runtime/contributing/style_guide/)
- [ESLint](https://github.com/eslint/eslint), [typescript-eslint](https://github.com/typescript-eslint/typescript-eslint), [SonarJS](https://github.com/SonarSource/SonarJS), [Biome](https://github.com/biomejs/biome), [eslint-plugin-unicorn](https://github.com/sindresorhus/eslint-plugin-unicorn), [eslint-plugin-jest](https://github.com/jest-community/eslint-plugin-jest), [eslint-plugin-jsdoc](https://github.com/gajus/eslint-plugin-jsdoc), [Prettier](https://github.com/prettier/prettier), [jscpd](https://github.com/kucherenko/jscpd)
- Project configs and contributor guides: [VS Code](https://github.com/microsoft/vscode), [TypeScript](https://github.com/microsoft/TypeScript), [React](https://github.com/facebook/react), [Angular](https://github.com/angular/angular/blob/main/contributing-docs/coding-standards.md), [Next.js](https://github.com/vercel/next.js), [Nest](https://github.com/nestjs/nest), [Playwright](https://github.com/microsoft/playwright), [Vue](https://github.com/vuejs/core)
