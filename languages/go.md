# Go

> Applies to Go 1.22+. Formatter: `gofmt` (or `gofumpt`) with `goimports`. Linter: `golangci-lint` v2.
> Read with the [core](../core/conventions.md); IDs like `2.2` refer to it.

## Thresholds

| Signal | Flag at | Core | Evidence |
|---|---|---|---|
| Parameters | ≥5, not counting the receiver, `context.Context` or a trailing `...Option` | 2.2 | revive `argument-limit` default 8 ([src][rv-arg]); Uber uses functional options for optional arguments, especially at ≥3 ([guide][uber-opts]); exported stdlib functions: ≥3 = 11.3 %, ≥5 = 1.7 % ([method](#measurement)) |
| Adjacent same-type parameters | ≥3 | 2.2 | Google best practices ([guide][g-sig]) |
| Results | ≥3 besides a trailing `error` | 2.4 | revive `function-result-limit` default 3, counting `error` ([src][rv-res]); exported stdlib functions: 0.61 % |
| Nesting | depth > 4 | 3.2 | sonar-go S134 default 4 ([src][sg-134]); revive `max-control-nesting` default 5 ([src][rv-nest]) |
| Cognitive complexity | > 15 | 2.6 | sonar-go S3776 default 15 ([src][sg-3776]); golangci-lint recommends 10–20 for gocognit ([ref][gcl-cog]) |
| Function length | ~50 statements, soft | 2.6 | revive `function-length` 50 statements ([src][rv-len]); funlen 40 statements / 60 lines ([src][funlen]); Code Review Comments sets no line count ([wiki][crc-len]) |
| Clone size | 100 tokens | 7.2 | dupl CLI default 100 ([src][dupl]); golangci-lint default 150 ([ref][gcl-dupl]) |
| Naked returns | in functions over 30 lines | 2.1 | nakedret default 30 ([ref][gcl-naked]); enabled in etcd, moby, grafana and golangci-lint |
| Line length | soft, ~100–120 | 9.1 | Code Review Comments and Google set no hard limit ([wiki][crc-line]); Uber soft limit 99 ([guide][uber-line]) |
| File length | no default; measure the repository | — | revive `file-length-limit` is off by default ([src][rv-file]); sonar-go S104 uses 750 |

No source sets the parameter limit at 3; flagging at 3 would hit 11–17 % of the standard library.

## Names (core 1)

- MixedCaps everywhere, constants included: `maxRetries`, `secondsPerDay`, never `MAX_RETRIES`
  ([CRC][crc-const]).
- Initialisms keep one case: `PolicyID`, `HTTPClient`, `xmlParser` ([CRC][crc-init], staticcheck
  ST1003).
- Getters drop `Get`: `Owner()`, not `GetOwner()`. Use `Fetch`, `Load` or `Compute` when the call is
  remote or expensive ([Effective Go][eg-get], [Google][g-get]). Generated protobuf `GetX()` is exempt.
- Package names are short, lowercase, single words. No `util`, `common`, `misc`, `helper`, `types`
  or `models` packages; a suffix form such as `stringsutil` is acceptable ([Google][g-pkg]).
- No stutter: `bufio.Reader`, `user.New`; do not repeat the package, receiver or parameter names in
  function names ([Google][g-stutter]).
- One-method interfaces take an `-er` name (`Reader`); no `I` prefix ([Effective Go][eg-iface]).
- Receivers are one or two letters, the same across all methods of a type, never `this` or `self`.
- Errors: exported sentinels `ErrNotFound`, unexported `errNotFound`, types `NotFoundError`
  ([Uber][uber-errname]).
- Scope bands for 1.2: small 1–7 lines, medium 8–15, large 15–25 ([Google][g-scope]).

## Functions (core 2)

- `ctx context.Context` is the first parameter, never stored in a struct or an options struct
  ([Google][g-ctx]).
- Required dependencies past the limit go into an options struct; optional ones become functional
  options (`...Option`) on constructors and public APIs ([Google][g-sig], [Uber][uber-opts]).
- Name 2–3 results of the same type before reaching for a result struct ([CRC][crc-named]).
- Accept interfaces, return concrete types.

## Control flow (core 3)

- Indent the error flow: `if err != nil { return ... }` and no `else` after a `return`
  ([CRC][crc-flow]). revive `early-return`, `indent-error-flow` and `superfluous-else` are enabled in
  prometheus and etcd.

## Errors (core 4)

- Check every error. `_ = f()` only with a comment saying why it is safe ([Google][g-discard]);
  errcheck needs `check-blank: true` to catch it.
- Wrap with `%w` and the operation: `fmt.Errorf("load pool %s: %w", id, err)`. At RPC and storage
  boundaries, where callers must not depend on the cause, use `%v` or map to a status code
  ([Google][g-wrap]).
- Error strings are lowercase, with no trailing punctuation and no "failed to" prefix
  ([CRC][crc-errstr], [Uber][uber-failed]).
- Match with `errors.Is` and `errors.As`, never by comparing `.Error()` strings.
- Sentinel `var ErrX = errors.New(...)` when callers match a static error; a custom type when they
  match a dynamic one; plain `fmt.Errorf` when nobody matches ([Uber][uber-errkind]).
- No `panic` for expected failures. `MustX` helpers only for initialisation with constant input
  ([Effective Go][eg-panic], [Uber][uber-panic]).
- No in-band errors: return `(v, ok)` or `(v, error)`, not `-1` or `""` ([CRC][crc-inband]).

## Comments (core 5)

- Every exported identifier has a doc comment: a full sentence that starts with the identifier's name
  ([Go Doc Comments][doc-req], [CRC][crc-doc]). An echo doc on an exported name is rewritten (5.2);
  on unexported code it is deleted.
- Notes take the form `TODO(#123): ...` or `TODO(owner): ...` ([Go Doc Comments][doc-notes]).
- Tool comments stay: `//go:build`, `//go:generate`, `//nolint:<linter> // reason`, and the
  `// Code generated ... DO NOT EDIT.` header.

## Tests (core 6)

- Table-driven tests with `t.Run(tc.name, ...)` ([wiki][wiki-tdt]). Case names are short and
  identifier-like (`rejects_negative_amount`): `go test -run` turns spaces into underscores and treats
  `/` as a separator ([Google][g-subtest]).
- Write separate `TestX...` functions when cases need different setup or assertions
  ([Google][g-split], [Uber][uber-split]). No `shouldError`, `setupMocks` or switch-dispatch fields
  inside the table ([Google][g-tablebad], [Uber][uber-tablebad]).
- Failure messages: `Foo(%v) = %v, want %v` ([CRC][crc-fail]).
- One assertion style per repository: stdlib `testing` with `cmp.Diff` ([Google][g-assert]) or
  testify `require`. Avoid `testify/suite`: moby bans it ([config][moby-suite]) and no style guide
  endorses it.
- `t.Helper()` in helpers; `t.Cleanup` over `defer` for fixtures.

## Reuse (core 7)

- `slices`, `maps`, `strings.Cut` and `errors.Join` before third-party helpers. A `map[K]struct{}` or
  `map[K]bool` is a fine set for membership checks ([Google least mechanism][g-least]).
- Ban superseded packages with `depguard`: `github.com/pkg/errors`, `io/ioutil`,
  `golang.org/x/exp/slices`.

## Boundaries (core 8)

- No mutable package-level state; `init()` does registration only ([Uber][uber-init],
  [Google][g-global]).
- Log through `log/slog` or the project's logger. No `fmt.Print*` or `log.Print*` outside `main`.
- Use `internal/` for packages the module does not export ([module layout][layout]).

## Formatting (core 9)

- `gofmt` and `goimports` are mandatory.
- Imports: standard library first, then third-party, then the module's own packages (`goimports
  -local` or `gci`), separated by blank lines. A project may add an organisation group
  ([CRC][crc-imports], [prometheus config][prom-gci]).

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

## Measurement

Standard-library percentages come from a `go/ast` walk over the `go1.24.0` source tree, skipping
`_test.go`, `internal/`, `cmd/`, `vendor/` and `testdata/`, and counting only functions with a body.
Parameters exclude the receiver and `context.Context`, with a variadic counted as one; results exclude
one trailing `error`.

[rv-arg]: https://github.com/mgechev/revive/blob/7ff27d13643f674bc27ef4343a76a3730994144c/rule/argument_limit.go#L16
[rv-res]: https://github.com/mgechev/revive/blob/7ff27d13643f674bc27ef4343a76a3730994144c/rule/function_result_limit.go#L25-L51
[rv-nest]: https://github.com/mgechev/revive/blob/7ff27d13643f674bc27ef4343a76a3730994144c/rule/max_control_nesting.go#L16
[rv-len]: https://github.com/mgechev/revive/blob/7ff27d13643f674bc27ef4343a76a3730994144c/rule/function_length.go#L81-L82
[rv-file]: https://github.com/mgechev/revive/blob/7ff27d13643f674bc27ef4343a76a3730994144c/rule/file_length_limit.go#L16-L29
[sg-134]: https://github.com/SonarSource/sonar-go/blob/7c7dea128a5fa2cfa0eeae93b268aae64181b501/sonar-go-checks/src/main/java/org/sonar/go/checks/TooDeeplyNestedStatementsCheck.java#L38
[sg-3776]: https://github.com/SonarSource/sonar-go/blob/7c7dea128a5fa2cfa0eeae93b268aae64181b501/sonar-go-checks/src/main/java/org/sonar/go/checks/FunctionCognitiveComplexityCheck.java#L31
[gcl-cog]: https://github.com/golangci/golangci-lint/blob/0e8087fdcf30179ea1233396310f1037a10f390b/.golangci.reference.yml#L771-L774
[gcl-dupl]: https://github.com/golangci/golangci-lint/blob/0e8087fdcf30179ea1233396310f1037a10f390b/.golangci.reference.yml#L406-L409
[gcl-naked]: https://github.com/golangci/golangci-lint/blob/0e8087fdcf30179ea1233396310f1037a10f390b/.golangci.reference.yml#L2401-L2404
[funlen]: https://github.com/ultraware/funlen/blob/955cef7e3e8d12c3f17141072ad57f7fec5a447a/funlen.go#L12-L13
[dupl]: https://github.com/mibk/dupl/blob/8836f5c0e8eacdc5233911754b94e70917cf0dba/main.go#L19
[uber-opts]: https://github.com/uber-go/guide/blob/1d60a91aa5e87d443002e23c21903c49489dbde5/style.md?plain=1#L3926-L3935
[uber-line]: https://github.com/uber-go/guide/blob/1d60a91aa5e87d443002e23c21903c49489dbde5/style.md?plain=1#L2360
[uber-errname]: https://github.com/uber-go/guide/blob/1d60a91aa5e87d443002e23c21903c49489dbde5/style.md?plain=1#L967-L1000
[uber-failed]: https://github.com/uber-go/guide/blob/1d60a91aa5e87d443002e23c21903c49489dbde5/style.md?plain=1#L920-L925
[uber-errkind]: https://github.com/uber-go/guide/blob/1d60a91aa5e87d443002e23c21903c49489dbde5/style.md?plain=1#L759-L780
[uber-panic]: https://github.com/uber-go/guide/blob/1d60a91aa5e87d443002e23c21903c49489dbde5/style.md?plain=1#L1163-L1166
[uber-split]: https://github.com/uber-go/guide/blob/1d60a91aa5e87d443002e23c21903c49489dbde5/style.md?plain=1#L3764-L3775
[uber-tablebad]: https://github.com/uber-go/guide/blob/1d60a91aa5e87d443002e23c21903c49489dbde5/style.md?plain=1#L3790-L3800
[uber-init]: https://github.com/uber-go/guide/blob/1d60a91aa5e87d443002e23c21903c49489dbde5/style.md?plain=1#L1603-L1615
[g-sig]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/go/best-practices.md?plain=1#L2099-L2120
[g-get]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/go/decisions.md?plain=1#L255-L262
[g-pkg]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/go/decisions.md?plain=1#L117
[g-stutter]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/go/best-practices.md?plain=1#L39-L100
[g-scope]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/go/decisions.md?plain=1#L272-L290
[g-ctx]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/go/decisions.md?plain=1#L2705
[g-discard]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/go/decisions.md?plain=1#L1080-L1095
[g-wrap]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/go/best-practices.md?plain=1#L924-L1030
[g-subtest]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/go/decisions.md?plain=1#L3358-L3380
[g-split]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/go/decisions.md?plain=1#L3488-L3493
[g-tablebad]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/go/decisions.md?plain=1#L3562-L3650
[g-assert]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/go/decisions.md?plain=1#L2867
[g-least]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/go/guide.md?plain=1#L186-L208
[g-global]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/go/best-practices.md?plain=1#L3319-L3330
[crc-len]: https://github.com/golang/wiki/blob/df4cf5cdd2e2e4bccc96c90dc7ad117df1c142d4/CodeReviewComments.md?plain=1#L384-L387
[crc-line]: https://github.com/golang/wiki/blob/df4cf5cdd2e2e4bccc96c90dc7ad117df1c142d4/CodeReviewComments.md?plain=1#L369-L372
[crc-const]: https://github.com/golang/wiki/blob/df4cf5cdd2e2e4bccc96c90dc7ad117df1c142d4/CodeReviewComments.md?plain=1#L389-L393
[crc-init]: https://github.com/golang/wiki/blob/df4cf5cdd2e2e4bccc96c90dc7ad117df1c142d4/CodeReviewComments.md?plain=1#L305-L311
[crc-named]: https://github.com/golang/wiki/blob/df4cf5cdd2e2e4bccc96c90dc7ad117df1c142d4/CodeReviewComments.md?plain=1#L411
[crc-flow]: https://github.com/golang/wiki/blob/df4cf5cdd2e2e4bccc96c90dc7ad117df1c142d4/CodeReviewComments.md?plain=1#L261-L304
[crc-errstr]: https://github.com/golang/wiki/blob/df4cf5cdd2e2e4bccc96c90dc7ad117df1c142d4/CodeReviewComments.md?plain=1#L135
[crc-inband]: https://github.com/golang/wiki/blob/df4cf5cdd2e2e4bccc96c90dc7ad117df1c142d4/CodeReviewComments.md?plain=1#L209
[crc-doc]: https://github.com/golang/wiki/blob/df4cf5cdd2e2e4bccc96c90dc7ad117df1c142d4/CodeReviewComments.md?plain=1#L127
[crc-fail]: https://github.com/golang/wiki/blob/df4cf5cdd2e2e4bccc96c90dc7ad117df1c142d4/CodeReviewComments.md?plain=1#L561-L571
[crc-imports]: https://github.com/golang/wiki/blob/df4cf5cdd2e2e4bccc96c90dc7ad117df1c142d4/CodeReviewComments.md?plain=1#L164-L186
[wiki-tdt]: https://github.com/golang/wiki/blob/df4cf5cdd2e2e4bccc96c90dc7ad117df1c142d4/TableDrivenTests.md?plain=1#L7
[eg-get]: https://go.dev/doc/effective_go#Getters
[eg-iface]: https://go.dev/doc/effective_go#interface-names
[eg-panic]: https://go.dev/doc/effective_go#panic
[doc-req]: https://github.com/golang/website/blob/af871f5d42e869aac0512d7fa773bc3cd7913f4d/_content/doc/comment.md?plain=1#L21
[doc-notes]: https://github.com/golang/website/blob/af871f5d42e869aac0512d7fa773bc3cd7913f4d/_content/doc/comment.md?plain=1#L526-L536
[layout]: https://github.com/golang/website/blob/af871f5d42e869aac0512d7fa773bc3cd7913f4d/_content/doc/modules/layout.md?plain=1#L99-L107
[moby-suite]: https://github.com/moby/moby/blob/43fbbcb58fd3b6433e0fc658ca06d2c44454b288/.golangci.yml#L62-L67
[prom-gci]: https://github.com/prometheus/prometheus/blob/c45b13bbdd96b47a5441000fb454dce42894349d/.golangci.yml#L7-L11
