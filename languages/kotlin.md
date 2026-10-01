# Kotlin

> Applies to Kotlin 1.9+ on the JVM and Multiplatform. Formatter: ktlint or ktfmt (through spotless or
> detekt-formatting). Linter: detekt 1.23. Read with the [core](../core/conventions.md); IDs like `2.2`
> refer to it.

## Thresholds

| Signal | Flag at | Core |
|---|---|---|
| Parameters | functions ≥ 6, constructors ≥ 7; not counting default-valued parameters, overrides, data-class constructors or DI-annotated constructors | 2.2 |
| Tuple returns | any `Pair` or `Triple` in a non-private signature | 2.4 |
| Nesting | depth ≥ 4 (detekt counts `when` and scope-function lambdas) | 3.2 |
| Cognitive complexity | ≥ 15 | 2.6 |
| Cyclomatic complexity | ≥ 15 | 2.6 |
| Compound conditions | ≥ 4 conditions | 3.3 |
| Function length | 60 lines, soft | 2.6 |
| Class length | 600 lines, soft | — |
| Line length | 120, owned by the formatter | 9.1 |

## Names (core 1)

- Two-letter acronyms are all caps (`IOStream`); longer ones capitalise only the first letter
  (`XmlFormatter`, `HttpClient`, `userId`).
- `const val` and deeply immutable top-level values use `SCREAMING_SNAKE_CASE`.
- A private backing property is `_name` behind a public `name`: the one allowed member prefix.
- Cheap, non-throwing state is a property, not a `getX()` function.
- Booleans start with `is`, `has`, `are`, `can` or `should`.
- Test functions are exempt from function-naming rules.

## Functions (core 2)

- Default parameters over overloads; count only parameters without a default.
- Name Boolean and same-type arguments at the call site: `load(path, strict = true)`.
- A Boolean that selects behaviour becomes a separate function or an enum.
- Prefer the expression form: `return when (state) { ... }`.
- Keep extension functions private or internal until another module needs them (7.3).

## Control flow (core 3)

- Configure detekt `ReturnCount` with `excludeGuardClauses: true` so early returns are not counted.

## Errors (core 4)

- Validate arguments with `require(...)`, state with `check(...)`, and unreachable states with
  `error(...)`.
- Domain failures are sealed classes or nullable returns, not `Result<T>`.
- Keep detekt's exception rules on: `SwallowedException`, `EmptyCatchBlock` (comment or an `_`,
  `ignored` or `expected` name), `TooGenericExceptionCaught`, `ThrowingExceptionsWithoutMessageOrCause`,
  `PrintStackTrace`. Catch `Throwable` only in infrastructure code that rethrows
  `CancellationException`.
- Never swallow `CancellationException`: no `runCatching` around suspend calls. Inject dispatchers
  instead of hard-coding `Dispatchers.IO`.

## Comments (core 5)

- Published libraries document every public declaration in KDoc: behaviour, edge cases, exceptions.
  Enable `explicitApi()` there.
- In application and service code KDoc is optional. Where it exists it never restates the signature
  and skips `@param`/`@return` lines that add nothing.
- Write TODOs as `TODO(#123): ...`; detekt's default `ForbiddenComment` rejects bare `TODO:`.

## Tests (core 6)

- One test class per unit (`PoolServiceTest`), one `@Test` per behaviour; `@Nested` groups related
  cases and `@ParameterizedTest` covers identical bodies.
- Pick one name format per repository: backtick sentences (`` `rejects negative amount` ``),
  `unit_condition_result`, or `testXxx` camelCase.
- One assertion library per repository (AssertJ, `kotlin.test` or Kotest).

## Reuse (core 7)

- Kotlin collection operations and sequences over `java.util.stream`.

## Boundaries (core 8)

- Typed config: Spring Boot `@ConfigurationProperties` with constructor binding instead of scattered
  `@Value`; in Ktor, deserialise config sections into data classes.
- Constructor injection. `val` and read-only collections in public APIs.
- No `println` or `printStackTrace` in production code.

## Formatting (core 9)

- ktlint or ktfmt; 4-space indent.
- Imports form one lexicographic block with no blank lines, using ktlint's default layout
  `*,java.**,javax.**,kotlin.**,^`.
- No wildcard imports. A project may allow them for its own DSL packages.

## Linter settings

```yaml
# detekt.yml — overrides on top of the detekt 1.23 default config
complexity:
  LongParameterList:
    functionThreshold: 6
    constructorThreshold: 7
    ignoreDefaultParameters: true
    ignoreDataClasses: true
  NestedBlockDepth:
    threshold: 4
  CognitiveComplexMethod:
    active: true
    threshold: 15
  CyclomaticComplexMethod:
    threshold: 15
  ComplexCondition:
    threshold: 4
  LongMethod:
    threshold: 60
  LargeClass:
    threshold: 600
naming:
  BooleanPropertyNaming:
    active: true
    allowedPattern: '^(is|has|are|can|should)'
style:
  ReturnCount:
    excludeGuardClauses: true
  MaxLineLength:
    maxLineLength: 120
```

## References

- [Kotlin coding conventions](https://kotlinlang.org/docs/coding-conventions.html), [Kotlin API guidelines](https://kotlinlang.org/docs/api-guidelines-introduction.html), [KEEP: Result](https://github.com/Kotlin/KEEP/blob/main/proposals/stdlib/KEEP-0127-result.md)
- [Android Kotlin style guide](https://developer.android.com/kotlin/style-guide), [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)
- [detekt](https://github.com/detekt/detekt), [ktlint](https://github.com/pinterest/ktlint), [ktfmt](https://github.com/facebook/ktfmt), [SonarKotlin](https://github.com/SonarSource/sonar-kotlin), [SonarJava](https://github.com/SonarSource/sonar-java), [Checkstyle](https://github.com/checkstyle/checkstyle), [PMD](https://github.com/pmd/pmd)
- [Spring Boot externalized configuration](https://docs.spring.io/spring-boot/reference/features/external-config.html), [Spring dependency injection](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html), [Ktor configuration](https://ktor.io/docs/server-configuration-file.html)
- Project configs and contributor guides: [Kotlin](https://github.com/JetBrains/kotlin), [Ktor](https://github.com/ktorio/ktor), [kotlinx.coroutines](https://github.com/Kotlin/kotlinx.coroutines), [OkHttp](https://github.com/square/okhttp), [Retrofit](https://github.com/square/retrofit), [Moshi](https://github.com/square/moshi), [Exposed](https://github.com/JetBrains/Exposed), [Now in Android](https://github.com/android/nowinandroid), [Spring Framework](https://github.com/spring-projects/spring-framework)
