# Go

> Applies to Go 1.22+. Formatter: `gofmt` (or `gofumpt`) with `goimports`. Linter: `golangci-lint` v2.
> Read with the [core](../core/conventions.md); IDs like `2.2` refer to it.

## Thresholds

| Signal | Flag at | Core |
|---|---|---|
| Parameters | ≥ 5, not counting the receiver, `context.Context` or a trailing `...Option` | 2.2 |
| Adjacent same-type parameters | ≥ 3 | 2.2 |
| Results | ≥ 3 besides a trailing `error` | 2.4 |
| Nesting | depth > 4 | 3.2 |
| Cognitive complexity | > 15 | 2.6 |
| Function length | ~50 statements, soft | 2.6 |
| Clone size | 100 tokens | 7.2 |
| Naked returns | in functions over 30 lines | 2.1 |
| Line length | soft, ~100–120 | 9.1 |
| File length | no default; measure the repository | — |

## Names (core 1)

- MixedCaps everywhere, constants included: `maxRetries`, `secondsPerDay`, never `MAX_RETRIES`.
- Initialisms keep one case: `PolicyID`, `HTTPClient`, `xmlParser`.
- Getters drop `Get`: `Owner()`, not `GetOwner()`. Use `Fetch`, `Load` or `Compute` when the call is
  remote or expensive. Generated protobuf `GetX()` is exempt.
- Package names are short, lowercase, single words. No `util`, `common`, `misc`, `helper`, `types`
  or `models` packages; a suffix form such as `stringsutil` is acceptable.
- No stutter: `bufio.Reader`, `user.New`; do not repeat the package, receiver or parameter names in
  function names.
- One-method interfaces take an `-er` name (`Reader`); no `I` prefix.
- Receivers are one or two letters, the same across all methods of a type, never `this` or `self`.
- Errors: exported sentinels `ErrNotFound`, unexported `errNotFound`, types `NotFoundError`.
- Scope bands for 1.2: small 1–7 lines, medium 8–15, large 15–25.

## Functions (core 2)

- `ctx context.Context` is the first parameter, never stored in a struct or an options struct.
- Required dependencies past the limit go into an options struct; optional ones become functional
  options (`...Option`) on constructors and public APIs.
- Name 2–3 results of the same type before reaching for a result struct.
- Accept interfaces, return concrete types.

## Control flow (core 3)

- Indent the error flow: `if err != nil { return ... }`, and no `else` after a `return`.

## Errors (core 4)

- Check every error. `_ = f()` only with a comment saying why it is safe.
- Wrap with `%w` and the operation: `fmt.Errorf("load pool %s: %w", id, err)`. At RPC and storage
  boundaries, where callers must not depend on the cause, use `%v` or map to a status code.
- Error strings are lowercase, with no trailing punctuation and no "failed to" prefix.
- Match with `errors.Is` and `errors.As`, never by comparing `.Error()` strings.
- Sentinel `var ErrX = errors.New(...)` when callers match a static error; a custom type when they
  match a dynamic one; plain `fmt.Errorf` when nobody matches.
- No `panic` for expected failures. `MustX` helpers only for initialisation with constant input.
- No in-band errors: return `(v, ok)` or `(v, error)`, not `-1` or `""`.

## Comments (core 5)

- Every exported identifier has a doc comment: a full sentence that starts with the identifier's name.
  An echo doc on an exported name is rewritten (5.2); on unexported code it is deleted.
- Notes take the form `TODO(#123): ...` or `TODO(owner): ...`.
- Tool comments stay: `//go:build`, `//go:generate`, `//nolint:<linter> // reason`, and the
  `// Code generated ... DO NOT EDIT.` header.

## Tests (core 6)

- Table-driven tests with `t.Run(tc.name, ...)`. Case names are short and identifier-like
  (`rejects_negative_amount`).
- Write separate `TestX...` functions when cases need different setup or assertions. No
  `shouldError`, `setupMocks` or switch-dispatch fields inside the table.
- Failure messages: `Foo(%v) = %v, want %v`.
- One assertion style per repository: stdlib `testing` with `cmp.Diff`, or testify `require`. No
  `testify/suite`.
- `t.Helper()` in helpers; `t.Cleanup` over `defer` for fixtures.

## Reuse (core 7)

- `slices`, `maps`, `strings.Cut` and `errors.Join` before third-party helpers. A `map[K]struct{}` or
  `map[K]bool` is a fine set for membership checks.
- Ban superseded packages with `depguard`: `github.com/pkg/errors`, `io/ioutil`,
  `golang.org/x/exp/slices`.

## Preferred libraries (core 7)

Use a library only when the module's `go.mod` already requires it. Never add a dependency without
asking. The standard library comes first.

| Need | Standard library | Library, when already in `go.mod` |
|---|---|---|
| Search, sort, min/max of slices | `slices`, `cmp` | — |
| Map keys and values | `slices.Collect(maps.Keys(m))` | `lo.Keys`, `lo.Values` |
| Transform, filter, index, group, dedupe | plain loop | `samber/lo`: `Map`, `Filter`, `FilterMap`, `KeyBy`, `GroupBy`, `Uniq`, `Chunk` |
| Pointer to a value | — | `lo.ToPtr`, `lo.FromPtr` |
| Set membership | `map[K]struct{}` | a set library only for set algebra (union, difference) |
| Decimal arithmetic (money, rates) | — | `shopspring/decimal` |
| Concurrent tasks that return errors | — | `golang.org/x/sync/errgroup` |
| Diffs in tests | — | `google/go-cmp` (`cmp.Diff`) |
| Assertions in tests | `testing` | `stretchr/testify/require` |
| Logging | `log/slog` | — |
| Errors | `errors`, `fmt.Errorf` with `%w`, `errors.Join` | — |

With `samber/lo`:

- Use the standard library where it covers the call: `slices.Contains` over `lo.Contains`,
  `slices.Index` over `lo.IndexOf`, `slices.Max`/`slices.Min` over `lo.Max`/`lo.Min`.
- `lo.Ternary(cond, a, b)` evaluates both `a` and `b`. Use `if` or `lo.TernaryF` when either side
  has side effects or is costly.
- `lo.Must` panics: only in initialisation and tests.
- Write the loop when the callback would be longer than the loop itself.

## Boundaries (core 8)

- No mutable package-level state; `init()` does registration only.
- Log through `log/slog` or the project's logger. No `fmt.Print*` or `log.Print*` outside `main`.
- Use `internal/` for packages the module does not export.

## Formatting (core 9)

- `gofmt` and `goimports` are mandatory.
- Imports: standard library first, then third-party, then the module's own packages (`goimports
  -local` or `gci`), separated by blank lines. A project may add an organisation group.

## Linter settings

```yaml
version: "2"
linters:
  enable: [revive, gocognit, dupl, nakedret, errorlint]
  settings:
    errcheck:
      check-blank: true
    gocognit:
      min-complexity: 15
    dupl:
      threshold: 100
    nakedret:
      max-func-lines: 30
    revive:
      rules:
        - name: argument-limit        # counts ctx: exact when ctx is present, one late otherwise
          arguments: [5]
        - name: function-result-limit # counts error: exact when the function returns an error
          arguments: [3]
        - name: max-control-nesting
          arguments: [4]
        - name: early-return
        - name: indent-error-flow
        - name: superfluous-else
        - name: error-strings
        - name: context-as-argument
        - name: exported
```

## References

- [Effective Go](https://go.dev/doc/effective_go), [Go Code Review Comments](https://go.dev/wiki/CodeReviewComments), [Go Doc Comments](https://go.dev/doc/comment), [Table-driven tests](https://go.dev/wiki/TableDrivenTests), [Organizing a Go module](https://go.dev/doc/modules/layout)
- [Google Go Style Guide](https://google.github.io/styleguide/go/)
- [Uber Go Style Guide](https://github.com/uber-go/guide)
- [samber/lo](https://github.com/samber/lo), [shopspring/decimal](https://github.com/shopspring/decimal), [go-cmp](https://github.com/google/go-cmp), [testify](https://github.com/stretchr/testify), [x/sync](https://pkg.go.dev/golang.org/x/sync/errgroup)
- [golangci-lint](https://github.com/golangci/golangci-lint), [revive](https://github.com/mgechev/revive), [staticcheck](https://github.com/dominikh/go-tools), [funlen](https://github.com/ultraware/funlen), [dupl](https://github.com/mibk/dupl), [gocognit](https://github.com/uudashr/gocognit), [sonar-go](https://github.com/SonarSource/sonar-go)
- Project configs and style docs: [Kubernetes](https://github.com/kubernetes/community/blob/main/contributors/guide/coding-conventions.md), [Prometheus](https://github.com/prometheus/prometheus), [etcd](https://github.com/etcd-io/etcd), [Moby](https://github.com/moby/moby), [Grafana](https://github.com/grafana/grafana/blob/main/contribute/backend/style-guide.md), [CockroachDB](https://github.com/cockroachdb/cockroach/blob/master/docs/style.md)
