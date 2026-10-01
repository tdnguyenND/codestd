# Kotlin

> Applies to Kotlin 1.9+ on the JVM and Multiplatform. Formatter: ktlint or ktfmt (through spotless or
> detekt-formatting). Linter: detekt 1.23. Read with the [core](../core/conventions.md); IDs like `2.2`
> refer to it.

## Thresholds

detekt 1.23 reports at `threshold` or above.

| Signal | Flag at | Core | Evidence |
|---|---|---|---|
| Parameters | functions ≥ 6, constructors ≥ 7; not counting default-valued parameters, overrides, data-class constructors or DI-annotated constructors | 2.2 | detekt `LongParameterList` 6 / 7 ([config][dk-lpl]); SonarKotlin S107 > 7, skipping overrides, Spring mappings and `@Composable` ([src][sk-107]); SonarJava skips `@Inject`/`@Autowired` constructors ([src][sj-107]); in a 150-file sample of each of six projects, ≥ 3 parameters hits a median 9.8 % of functions and ≥ 6 hits 0.65 % |
| Tuple returns | any `Pair` or `Triple` in a non-private signature | 2.4 | Kotlin docs prefer a named data class, even for two values ([docs][kd-data], [docs][kd-destr]); detekt `DestructuringDeclarationWithTooManyEntries` 3 ([config][dk-destr]); `Triple` appears 0 times as a return type in 900 sampled files |
| Nesting | depth ≥ 4 | 3.2 | detekt `NestedBlockDepth` 4, which counts `when` and scope-function lambdas ([config][dk-nest]); SonarKotlin S134 3 |
| Cognitive complexity | ≥ 15 | 2.6 | SonarKotlin S3776 15 ([src][sk-3776]); detekt `CognitiveComplexMethod` 15, off by default ([config][dk-cog]) |
| Cyclomatic complexity | ≥ 15 | 2.6 | detekt `CyclomaticComplexMethod` 15 ([config][dk-cyc]) |
| Compound conditions | ≥ 4 conditions | 3.3 | detekt `ComplexCondition` 4 ([config][dk-cond]); Checkstyle `BooleanExpressionComplexity` 3 |
| Function length | 60 lines, soft | 2.6 | detekt `LongMethod` 60 ([config][dk-lm]); PMD `NcssCount` 60 |
| Class length | 600 lines, soft | — | detekt `LargeClass` 600 ([config][dk-lc]); Kotlin conventions keep files to a few hundred lines ([docs][kc-file]) |
| Line length | 120, owned by the formatter | 9.1 | detekt `MaxLineLength` 120 ([config][dk-mll]); median of seven project configs is 120 ([ktor][ktor-ec]) |

No Kotlin or Java linter sets the parameter limit at 3; the lowest default is detekt's 6.

## Names (core 1)

- Two-letter acronyms are all caps (`IOStream`); longer ones capitalise only the first letter
  (`XmlFormatter`, `HttpClient`, `userId`) ([conventions][kc-acronym]).
- `const val` and deeply immutable top-level values use `SCREAMING_SNAKE_CASE` ([conventions][kc-const]).
- A private backing property is `_name` behind a public `name`: the one allowed member prefix
  ([conventions][kc-backing]).
- Cheap, non-throwing state is a property, not a `getX()` function ([conventions][kc-funprop]).
- Booleans start with `is`, `has`, `are`, `can` or `should`. detekt's `BooleanPropertyNaming` default
  pattern `^(is|has|are)` needs widening ([config][dk-bool]).
- Test functions are exempt from function-naming rules ([conventions][kc-test]).

## Functions (core 2)

- Default parameters over overloads ([conventions][kc-default]). Count only parameters without a
  default: in a 150-file sample of ktor, excluding defaults drops the ≥ 6 share from 4.2 % to 0.9 %.
- Name Boolean and same-type arguments at the call site: `load(path, strict = true)`
  ([conventions][kc-named]).
- A Boolean that selects behaviour becomes a separate function or an enum ([API guidelines][kapi-bool]).
- Prefer the expression form: `return when (state) { ... }` ([conventions][kc-cond]).
- Keep extension functions private or internal until another module needs them (core 7.3,
  [conventions][kc-ext]).

## Control flow (core 3)

- detekt's `ReturnCount` (max 2) counts guard clauses unless `excludeGuardClauses: true`
  ([config][dk-rc]); set it, as detekt's own config does ([config][dko-rc]).

## Errors (core 4)

- Validate arguments with `require(...)`, state with `check(...)`, and unreachable states with
  `error(...)` ([API guidelines][kapi-validate]).
- Domain failures are sealed classes or nullable returns, not `Result<T>`, which is not designed for
  domain errors ([KEEP][keep-result]).
- Keep detekt's exception rules on: `SwallowedException`, `EmptyCatchBlock` (comment or an `_`,
  `ignored` or `expected` name), `TooGenericExceptionCaught`, `ThrowingExceptionsWithoutMessageOrCause`,
  `PrintStackTrace`. Catching `Throwable` is acceptable only in infrastructure code that rethrows
  `CancellationException`.
- Never swallow `CancellationException`: no `runCatching` around suspend calls
  (`SuspendFunSwallowedCancellation`). Inject dispatchers instead of hard-coding `Dispatchers.IO`
  (`InjectDispatcher`).

## Comments (core 5)

- Published libraries document every public declaration in KDoc: behaviour, edge cases, exceptions
  ([conventions][kc-lib], [API guidelines][kapi-doc]). Enable `explicitApi()` there
  ([ktor][ktor-explicit], [moshi][moshi]).
- In application and service code KDoc is optional. Where it exists it never restates the signature,
  and it skips `@param`/`@return` boilerplate that adds nothing ([conventions][kc-doc]).
- detekt's default `ForbiddenComment` rejects `TODO:`; `TODO(#123): ...` passes and follows core 5.4
  ([config][dk-fc]).

## Tests (core 6)

- One test class per unit (`PoolServiceTest`), one `@Test` per behaviour; `@Nested` groups related
  cases and `@ParameterizedTest` covers identical bodies.
- Pick one name format per repository: backtick sentences (`` `rejects negative amount` ``, as in
  ktor), `unit_condition_result` (Google Java, Android, [style guide][and-fn]), or `testXxx` camelCase
  (kotlinx.coroutines and Exposed, which ban backticks because KDoc cannot link them,
  [Exposed][ex-test]).
- One assertion library per repository (AssertJ, `kotlin.test` or Kotest).

## Reuse (core 7)

- Kotlin collection operations and sequences over `java.util.stream`; detekt's `UnnecessaryFilter`
  and `RedundantHigherOrderMapUsage` catch hand-rolled forms.

## Boundaries (core 8)

- Typed config: Spring Boot `@ConfigurationProperties` with constructor binding instead of scattered
  `@Value` ([Spring Boot][sb-cp]); Ktor deserialises config sections into data classes
  ([Ktor][ktor-cfg]).
- Constructor injection ([Spring][sp-di]). `val` and read-only collections in public APIs
  ([conventions][kc-immut], [API guidelines][kapi-mutable]).
- No `println` or `printStackTrace` in production code (detekt `ForbiddenMethodCall`, `PrintStackTrace`).

## Formatting (core 9)

- ktlint or ktfmt; 4-space indent.
- Imports form one lexicographic block with no blank lines; ktlint's default layout is
  `*,java.**,javax.**,kotlin.**,^`, so the standard library comes last ([ktlint][kl-imp],
  [Android][and-imports]).
- No wildcard imports ([ktlint][kl-wild], [Android][and-imports]). A project may allow them for its
  own DSL packages, as ktor does for `io.ktor.*`.

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

[dk-lpl]: https://github.com/detekt/detekt/blob/v1.23.8/detekt-core/src/main/resources/default-detekt-config.yml#L133-L139
[dk-nest]: https://github.com/detekt/detekt/blob/v1.23.8/detekt-core/src/main/resources/default-detekt-config.yml#L147-L149
[dk-cog]: https://github.com/detekt/detekt/blob/v1.23.8/detekt-core/src/main/resources/default-detekt-config.yml#L96-L98
[dk-cond]: https://github.com/detekt/detekt/blob/v1.23.8/detekt-core/src/main/resources/default-detekt-config.yml#L99-L101
[dk-cyc]: https://github.com/detekt/detekt/blob/v1.23.8/detekt-core/src/main/resources/default-detekt-config.yml#L108-L110
[dk-lc]: https://github.com/detekt/detekt/blob/v1.23.8/detekt-core/src/main/resources/default-detekt-config.yml#L127-L129
[dk-lm]: https://github.com/detekt/detekt/blob/v1.23.8/detekt-core/src/main/resources/default-detekt-config.yml#L130-L132
[dk-bool]: https://github.com/detekt/detekt/blob/v1.23.8/detekt-core/src/main/resources/default-detekt-config.yml#L310-L312
[dk-destr]: https://github.com/detekt/detekt/blob/v1.23.8/detekt-core/src/main/resources/default-detekt-config.yml#L545-L547
[dk-fc]: https://github.com/detekt/detekt/blob/v1.23.8/detekt-core/src/main/resources/default-detekt-config.yml#L586-L595
[dk-mll]: https://github.com/detekt/detekt/blob/v1.23.8/detekt-core/src/main/resources/default-detekt-config.yml#L645-L647
[dk-rc]: https://github.com/detekt/detekt/blob/v1.23.8/detekt-core/src/main/resources/default-detekt-config.yml#L688-L695
[dko-rc]: https://github.com/detekt/detekt/blob/ee1c04f2d7d2a4b5a181273a4d467e7a5a28c01a/config/detekt/detekt.yml#L263-L265
[sk-107]: https://github.com/SonarSource/sonar-kotlin/blob/12a870ce5a97014c9a3cd53fe6897ad384a725fc/sonar-kotlin-checks/src/main/java/org/sonarsource/kotlin/checks/TooManyParametersCheck.kt#L30-L59
[sk-3776]: https://github.com/SonarSource/sonar-kotlin/blob/12a870ce5a97014c9a3cd53fe6897ad384a725fc/sonar-kotlin-checks/src/main/java/org/sonarsource/kotlin/checks/FunctionCognitiveComplexityCheck.kt#L31-L32
[sj-107]: https://github.com/SonarSource/sonar-java/blob/e40c86614add14321803df7a0bda710759d1f547/java-checks/src/main/java/org/sonar/java/checks/TooManyParametersCheck.java#L40-L111
[kc-file]: https://github.com/JetBrains/kotlin-web-site/blob/3b022f0af1cbab7e9f4b63455a7fae73e6a2bcf8/docs/topics/coding-conventions.md#L100-L104
[kc-test]: https://github.com/JetBrains/kotlin-web-site/blob/3b022f0af1cbab7e9f4b63455a7fae73e6a2bcf8/docs/topics/coding-conventions.md#L182-L194
[kc-const]: https://github.com/JetBrains/kotlin-web-site/blob/3b022f0af1cbab7e9f4b63455a7fae73e6a2bcf8/docs/topics/coding-conventions.md#L196-L205
[kc-backing]: https://github.com/JetBrains/kotlin-web-site/blob/3b022f0af1cbab7e9f4b63455a7fae73e6a2bcf8/docs/topics/coding-conventions.md#L222-L234
[kc-acronym]: https://github.com/JetBrains/kotlin-web-site/blob/3b022f0af1cbab7e9f4b63455a7fae73e6a2bcf8/docs/topics/coding-conventions.md#L247-L250
[kc-doc]: https://github.com/JetBrains/kotlin-web-site/blob/3b022f0af1cbab7e9f4b63455a7fae73e6a2bcf8/docs/topics/coding-conventions.md#L839-L859
[kc-immut]: https://github.com/JetBrains/kotlin-web-site/blob/3b022f0af1cbab7e9f4b63455a7fae73e6a2bcf8/docs/topics/coding-conventions.md#L907-L915
[kc-default]: https://github.com/JetBrains/kotlin-web-site/blob/3b022f0af1cbab7e9f4b63455a7fae73e6a2bcf8/docs/topics/coding-conventions.md#L930-L939
[kc-named]: https://github.com/JetBrains/kotlin-web-site/blob/3b022f0af1cbab7e9f4b63455a7fae73e6a2bcf8/docs/topics/coding-conventions.md#L967-L974
[kc-cond]: https://github.com/JetBrains/kotlin-web-site/blob/3b022f0af1cbab7e9f4b63455a7fae73e6a2bcf8/docs/topics/coding-conventions.md#L976-L1006
[kc-funprop]: https://github.com/JetBrains/kotlin-web-site/blob/3b022f0af1cbab7e9f4b63455a7fae73e6a2bcf8/docs/topics/coding-conventions.md#L1106-L1116
[kc-ext]: https://github.com/JetBrains/kotlin-web-site/blob/3b022f0af1cbab7e9f4b63455a7fae73e6a2bcf8/docs/topics/coding-conventions.md#L1117-L1122
[kc-lib]: https://github.com/JetBrains/kotlin-web-site/blob/3b022f0af1cbab7e9f4b63455a7fae73e6a2bcf8/docs/topics/coding-conventions.md#L1179-L1189
[kd-data]: https://github.com/JetBrains/kotlin-web-site/blob/3b022f0af1cbab7e9f4b63455a7fae73e6a2bcf8/docs/topics/data-classes.md#L138-L141
[kd-destr]: https://github.com/JetBrains/kotlin-web-site/blob/3b022f0af1cbab7e9f4b63455a7fae73e6a2bcf8/docs/topics/destructuring-declarations.md#L42-L64
[kapi-bool]: https://kotlinlang.org/docs/api-guidelines-readability.html#avoid-using-the-boolean-type-as-an-argument
[kapi-doc]: https://kotlinlang.org/docs/api-guidelines-informative-documentation.html#thoroughly-document-your-api
[kapi-validate]: https://kotlinlang.org/docs/api-guidelines-predictability.html#validate-inputs-and-state
[kapi-mutable]: https://kotlinlang.org/docs/api-guidelines-predictability.html#avoid-exposing-mutable-state
[keep-result]: https://github.com/Kotlin/KEEP/blob/54993607f09e92444ccf64d985c37885c8ee6e22/proposals/stdlib/result.md#L395-L434
[and-imports]: https://developer.android.com/kotlin/style-guide#import_statements
[and-fn]: https://developer.android.com/kotlin/style-guide#function_names
[ex-test]: https://github.com/JetBrains/Exposed/blob/0703402bda3819e36031268d01f8f83b1f36ae34/documentation-website/Writerside/topics/Contributing.md#L76-L80
[ktor-ec]: https://github.com/ktorio/ktor/blob/1d177d32df5de4df0dfee67cee159e01e226419f/.editorconfig#L9-L33
[ktor-explicit]: https://github.com/ktorio/ktor/blob/1d177d32df5de4df0dfee67cee159e01e226419f/build-logic/src/main/kotlin/ktorbuild.kmp.gradle.kts#L31
[ktor-cfg]: https://ktor.io/docs/server-configuration-file.html#deserialize-config
[moshi]: https://github.com/square/moshi/blob/889013ec2edb8d8034902662a1dc8c4f3b3f8111/build.gradle.kts#L49-L84
[kl-imp]: https://github.com/pinterest/ktlint/blob/76575a60036d857ed3f27f0b9551400e1df92522/documentation/release-latest/docs/rules/standard.md#L1249-L1271
[kl-wild]: https://github.com/pinterest/ktlint/blob/76575a60036d857ed3f27f0b9551400e1df92522/documentation/release-latest/docs/rules/standard.md#L2814-L2832
[sb-cp]: https://docs.spring.io/spring-boot/reference/features/external-config.html#features.external-config.typesafe-configuration-properties
[sp-di]: https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html#beans-constructor-vs-setter-injection
