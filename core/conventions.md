# Core conventions

Language-agnostic rules for writing and reviewing code. They define what good code looks like. The
language packs in [`../languages/`](../languages/) supply the syntax, the tools, the exemptions and
every number.

- **Precedence.** Project rules, then the language pack, then this file. A language's official style
  guide, or an idiom a codebase has established, also overrides this file: Go writes `PolicyID`,
  Kotlin writes `XmlParser`, and a DI framework may require `IFooService`.
- **Scope.** Code you add or change. Report a violation in untouched code; do not fix it in the same
  change.
- **Numbers live in packs.** Parameter count, nesting depth, complexity, function length, file length
  and line length are set per language from linter defaults and measured distributions. A number is
  a review signal; it is never a reason to split cohesive code.
- **Rule IDs** (`2.3`) are stable. Packs and review comments cite them.

## 1. Names

Test: hide the body. Does the name alone say what the thing is or does?

- **1.1 No filler words** in names that outlive a few lines: `Manager`, `Helper`, `Util`, `Data`,
  `Info`, `Object`, `Common`, `Base`, `process`, `handle`. Name the domain concept. A name that is
  hard to choose usually means the unit does more than one thing.
- **1.2 Name length scales with scope.** `i`, `err`, `ctx`, `tmp`, `ok` and receivers are fine in a
  scope of a few lines; the further a use is from its declaration, the more the name must say.
- **1.3 A verb needs its object.** `process()` processes what? Commands are verb phrases
  (`settleLoan`). Queries and accessors follow the language: a noun (`Owner()`) or a property where
  the language prefers it.
- **1.4 Booleans read as predicates:** `isSpent`, `hasReward`, `canWithdraw`, not `flag` or `status`.
- **1.5 One word per concept, and different words for different costs.** Pick `get`, `fetch` or
  `retrieve` for one behaviour and keep it. A cheap accessor and a remote or expensive call are
  different concepts and get different words (`Owner()` vs `fetchOwner()`).
- **1.6 Types are noun phrases.** No type named `Process` or `Validate`.
- **1.7 No encodings.** No Hungarian notation (`strName`, `iCount`) and no member prefixes (`m_`),
  except where the language idiom requires one (Kotlin's `_name` backing property). No `I` prefix on
  interfaces unless the language or the codebase's idiom requires it. Avoid type words in names; a
  `List`/`Map`/`Set` suffix, when used, must match the real type.
- **1.8 No look-alike names:** no lone `l`, `O` or `I`, which read as `1` and `0`.
- **1.9 No stutter** with the enclosing package, module or class, or with the receiver and parameter
  names: `pool.New`, not `pool.NewPool`.
- **1.10 Searchable constants.** A literal with business meaning, or one that repeats, becomes a named
  constant in the language's constant case. `-1`, `0`, `1` and `""` stay inline. Never `ZERO = 0`.

## 2. Functions

- **2.1 One thing, at one level of abstraction.** The name states the whole effect, with no hidden
  side effect. A function either answers a question or changes state, not both.
- **2.2 Parameter count.** Past the pack's limit, group related parameters into an object, or use
  named or keyword arguments where the language has them. Not counted: the receiver or `self`, a
  context or cancellation handle, an options object (counts as one), and signatures you do not own
  (overrides, interface implementations, framework callbacks, DI-wired constructors). Adjacent
  parameters of the same type risk being swapped: use named arguments or a struct.
- **2.3 No flag arguments.** A boolean that switches behaviour becomes two functions, an enum, or an
  argument the caller must name (keyword-only, named argument). If the signature cannot change, name
  the literal at the call site (`/* dryRun= */ true`). A boolean that is plain two-state data, not a
  behaviour switch, is fine.
- **2.4 Multiple return values.** Past the pack's limit, return a named result type. A pair is usually
  fine; three or more values usually deserve a name.
- **2.5 No output parameters.** Return a value instead of mutating an argument.
- **2.6 Length is a symptom.** Extract a block when it has a purpose you can name and understand
  without reading its caller. Do not extract when the pieces stay conjoined (understanding one needs
  the other) or the new function only passes its arguments through. Cognitive complexity and nesting
  are the limits; line counts are a soft pack signal.
- **2.7 Law of Demeter.** No call chains `a.b().c().d()` that reach across types. Splitting the chain
  into locals does not remove the coupling. Generated accessors and fluent builders are exempt.
- **2.8 Minimal surface.** Export only what another component calls; use the narrowest visibility.
- **2.9 No speculative generality.** No parameters, options, layers or interfaces added for a caller
  that does not exist yet.

## 3. Control flow

- **3.1 Happy path at the left margin.** Handle errors and edge cases first and return or continue
  early. Never leave the success branch inside `if` with the error in `else`.
- **3.2 Bounded nesting.** Past the pack's depth, invert the condition, return early, or extract a
  function. A `switch`/`match` inside a loop and a genuine state machine count as one level.
- **3.3 Named conditions.** A compound condition past the pack's operator limit becomes a named
  predicate or variable. State conditions positively (`isValid`, not `!isInvalid`).

```
// nested                                 // guards first, happy path flat
load(id):                                 load(id):
  if id != "":                              if id == "": return ErrEmptyID
    pool, err = repo.get(id)                pool, err = repo.get(id)
    if err == nil:                          if err != nil: return err
      if pool.active: return pool           if !pool.active: return ErrInactive
      return ErrInactive                    return pool
    else: return err
  return ErrEmptyID
```

## 4. Errors

- **4.1 Never swallow an error.** No empty `catch`, no discarded error result, and no asynchronous work
  whose failure nobody observes (an unawaited promise, an unjoined task). Ignoring an error is allowed
  only through the language's explicit form (such as suppressing one named exception type) or with a
  comment saying why it is safe.
- **4.2 Catch narrowly.** The `try` block covers only the call that can fail. A catch-all only
  re-raises, logs the full trace, or sits at an isolation boundary such as a request handler or a
  worker loop.
- **4.3 Wrap with context and keep the cause:** which operation, on which id (`load pool <id>:
  <cause>`). No "failed" or "failed to" filler, and no repeating what the cause already says.
- **4.4 Handle each error once.** Log it or return it, not both.
- **4.5 Raise real error objects** that carry a stack and a cause, never strings or literals.
- **4.6 Crash only on programmer error or unrecoverable startup.** Expected failures are values or
  errors the caller handles.
- **4.7 No in-band errors.** Return `(value, ok)`, an optional or an error instead of a sentinel such
  as `-1`, `""` or `null`.
- **4.8 Assertions are not runtime validation.** Validate input at trust boundaries with real checks.
- **4.9 One catalogue of domain errors per service**, holding the cases callers need to tell apart,
  mapped to transport codes (gRPC status, HTTP status) at the boundary.
- **4.10 No secrets in errors or logs:** keys, seeds, tokens, passwords, full personal data.

## 5. Comments

- **5.1 Implementation comments say why:** a reason, an invariant, a workaround, an external quirk
  (`// API returns newest first; reverse before paging`). Delete comments that restate the line
  below, name an obvious field, or draw banners (`// ==== helpers ====`; split the file or the
  function instead).
- **5.2 Doc comments on public API state the contract:** what it does or returns, preconditions,
  errors, side effects, units, ownership, concurrency. A doc comment that only echoes the signature is
  rewritten to say what the signature cannot. Delete it only when the symbol is not public API and
  neither the language nor the project requires a doc comment.
- **5.3 New code meets the same bar as old code.** A field, parameter or function does not get a
  comment because it is new: if its siblings need none, it needs none. If a name needs a comment to be
  understood, rename it first. Comments describe the code as it is, never the change that produced it
  (`// added for feature X`, `// new field for …`); history belongs in version control.
  - **Check before pushing:** for each new comment in the diff, ask whether the existing siblings
    carry this kind of comment. If not, delete it, unless 5.2 requires a doc comment there.
- **5.4 TODO and FIXME carry what, why and a tracking reference.** Prefer an issue link to a person's
  name. A bare `TODO` gets that context or becomes an issue.
- **5.5 No commented-out code.** Version control is the archive.
- **5.6 No social content:** author bylines, `@mentions`, review-thread citations, "as the reviewer
  asked".
- **5.7 Comments that tools read are code.** Keep directives, build tags and lint suppressions; an
  unexplained suppression is a finding. Call-site parameter names (`/* dryRun= */ true`) stay.
- **5.8 Exempt:** generated files and executable doc examples.

## 6. Tests

- **6.1 One test group per unit under test, one case per behaviour.** Merge cases into a table or a
  parametrized test only when their bodies are identical. When cases need different setup or
  assertions, write separate tests.
- **6.2 No conditional logic inside a table loop.** No `if tc.shouldError` branch and no switch over
  mock setup; split those cases into their own tests.
- **6.3 Case names state behaviour:** the expected result and the condition. The pack sets the
  format.
- **6.4 F.I.R.S.T.:** Fast · Independent (no shared mutable state, no ordering) · Repeatable (inject
  the clock, randomness and schedulers) · Self-validating (assert, never eyeball output) · Timely
  (written with the change).
- **6.5 Assert every field the change touched**, not only the fields older tests already assert.
  Failure messages identify the input, the actual value and the expected value.
- **6.6 A test must fail when the code breaks.** Never weaken, skip or delete a failing test to make a
  change pass. Never commit focused or disabled tests.
- **6.7 A regression test references the issue it reproduces.**
- **6.8 One assertion library per repository.** The pack names the default.

## 7. Reuse

- **7.1 Least mechanism.** Language construct, then the standard library, then the project's approved
  libraries and shared packages, then new code. Do not hand-roll what the standard library provides;
  a plain loop is fine when it is the clearest form.
- **7.2 Rule of three.** A second copy: consider extracting. A third: extract. Only merge copies that
  must change together. A clone detector is a review aid, not a CI gate. Test setup repeated for
  readability is exempt.
- **7.3 Hoist generic helpers when a second module needs them.** A helper that touches only the
  standard library, the shared library and primitives moves to the shared library once a second
  module calls it. Until then it stays private, next to its caller.

## 8. Boundaries

- **8.1 Generated and transport types stop at the adapter.** Protobuf messages, OpenAPI models, ORM
  rows and SDK responses become domain types in the adapter layer; domain logic never imports them.
- **8.2 Interfaces belong to the consumer** and exist only when a real consumer needs one. Do not
  declare an interface next to its implementation only to mock it.
- **8.3 Config is loaded in one place.** Env vars and files are read into a typed object; types and
  required values are validated at the first load, which fails fast. Nothing is read at import or
  static-initialisation time, and tests can override every value. How components obtain the config
  (constructor injection, a cached getter, a framework settings object) follows the language and
  framework idiom.
- **8.4 No mutable global state** and no side effects at import or initialisation time.
- **8.5 Logging goes through the project's logger** with levels and request context. No print or
  console output in production code (CLIs and scripts are exempt). Pass values as arguments instead
  of pre-formatting the message.
- **8.6 Validate input at trust boundaries.**
- **8.7 Prefer immutable values** and read-only collections in public signatures.
- **8.8 Enforce layer and import boundaries with a tool** where one exists.

## 9. Formatting

- **9.1 The formatter owns layout and line length.** Never format by hand.
- **9.2 Import order and grouping follow the language pack.** Where the formatter does not sort
  imports, a lint rule does.
- **9.3 Run auto-fixers on the files you touched**, never repo-wide, and keep formatting changes out
  of behavioural changes.

## 10. Agent-written code

Failure modes common in AI-generated changes. Review for them explicitly.

- **10.1 Invented APIs.** Every function, option, flag and config key exists in the codebase or in the
  installed dependency version.
- **10.2 Duplicate implementations.** Search for an existing helper before writing one. No `_v2`,
  `_new` or `_copy` siblings.
- **10.3 Change narration in code**: comments explaining the edit, why a field was added, or what used
  to be there (5.3).
- **10.4 Scope creep**: unrelated refactors, renames or reformatting in the same change (9.3).
- **10.5 Unverified success**: weakened tests (6.6), or a claim of success without saying what ran and
  what did not.
- **10.6 Speculative abstraction** (2.9).

## Sources

- Google engineering practices, [what to look for in a code review](https://github.com/google/eng-practices/blob/3bb3ec25b3b0199f4940b1aa75f0ac5c5753301c/review/reviewer/looking-for.md)
- Linux kernel [coding style](https://docs.kernel.org/process/coding-style.html)
- R. Martin, *Clean Code*, via [Clean Code notes](https://github.com/JuanCrg90/Clean-Code-Notes/blob/50a8cb934de955cf746964f5eb6f4e25de408cdf/README.md)
- J. Ousterhout and R. Martin, [A Philosophy of Software Design vs Clean Code](https://github.com/johnousterhout/aposd-vs-clean-code/blob/2ce0742228fbf850e15101f83308de2cd72144b2/README.md)
- S. McConnell, *Code Complete 2*, via the [book's checklists](https://github.com/oceord/cc2e-checklists/tree/ad81fa502aa03189166aec5399bf3d1d0e942fc9/checklists)
- M. Fowler, [Flag Argument](https://martinfowler.com/bliki/FlagArgument.html) and [Data Clump](https://martinfowler.com/bliki/DataClump.html)
- G. A. Campbell, [Cognitive Complexity](https://www.sonarsource.com/docs/CognitiveComplexity.pdf) (SonarSource white paper)
- SonarSource analyzers, e.g. [S107 parameter exclusions in sonar-java](https://github.com/SonarSource/sonar-java/blob/e40c86614add14321803df7a0bda710759d1f547/java-checks/src/main/java/org/sonar/java/checks/TooManyParametersCheck.java#L54-L75)
- Microsoft .NET [parameter design guidelines](https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/parameter-design)
- btseee/clean-code-skills, [agent smells](https://github.com/btseee/clean-code-skills/blob/49b1354be3ec9d0158c99b1c1c9bc1f833133640/skills/clean-code/references/review-checklist.md#L132-L148)
