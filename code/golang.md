## Optional Go module

<!-- BEGIN GO MODULE -->

Apply this module only to Go (Golang) components in scope. Use the core prompt’s evidence rules, finding IDs, and report. Check version-sensitive advice against the supported release targets. Group minor modernization items into one P3 entry when they have a concrete benefit.

### G1. Establish the effective build

Read each relevant `go.mod` and `go.sum`, any `go.work`, the `go` and `toolchain` directives, build tags, file suffixes, and CI and release settings. Record the selected compiler, the applicable language version for the file, `GOOS`/`GOARCH`, cgo, and embedded assets. Version build constraints can change a file’s language version. Check pinned generators and what `//go:embed` includes in the artifact. [Go toolchains](https://go.dev/doc/toolchain)

Test supported workspace builds as shipped. For an independently published module, also build a small external consumer with `GOWORK=off` and the intended module version or release candidate. A build within the module can still honor its own `replace` directives with the workspace disabled. Check whether a local replacement, untracked file, or warm module cache supplied something downstream users will lack. [Go modules reference](https://go.dev/ref/mod)

### G2. Packages, APIs, and error contracts

- Read an exported API from a caller’s perspective. Check useful zero values, ownership, method sets, visibility, package-name stutter, and initialisms such as `URL` and `ID`. Consider early returns where nesting hides the happy path and `internal/` for private implementation. Inspect catch-all packages, import-cycle workarounds, mutable globals, and hidden `init` side effects.
- Tie every interface to a consumer or other concrete need. Small consumer-defined interfaces and concrete return types often simplify a caller. Inspect pointer parameters such as `*string`, `*[]T`, or `*io.Reader` for meaningful optionality or mutation. Check pointer and value receivers for unintended copies, lost mutation, interface breakage, or exposed lock methods. Look for copied `sync.Mutex`, `sync.RWMutex`, `sync.WaitGroup`, or atomic values after first use. Use `Example` tests when an exported usage example should stay executable.
- Trace every consequential error to its caller. Inspect shadowing with `:=`, overwritten errors, ignored `_ =` results, and success reported before a write, flush, close, or commit completes. When a function promises a nil error on success, check for an interface containing a typed nil pointer. Verify intended inspection with `errors.Is` or `errors.As` and wrapping with `%w` when the underlying error is part of the contract.
- Check who should log or return an error, duplicate logging, and conventions for composable error messages such as lowercase initial words and omitted trailing punctuation. Inspect `panic` at recovery boundaries. `log.Fatal` and `os.Exit` bypass deferred cleanup, which matters if an unfinished write or caller can observe the result.

### G3. Goroutines, cancellation, and synchronization

- For each consequential goroutine, identify its owner, what unblocks it, how it stops, and who waits or observes its result. Trace an early consumer exit through sends, receives, and fan-out. Check channel-close ownership across all senders, bounded concurrency, lock ordering, locks held across I/O or channel operations, and work registration before a `WaitGroup` wait can see zero. Inspect done channels and `errgroup` lifetimes where used.
- Follow `context.Context` from caller to the actual blocking operation. Check cancellation paths through `select`, HTTP, database calls, and worker loops. Prefer an explicit first parameter for per-call contexts. Check the lifetime if a struct stores one. Ensure the owner calls or transfers each `CancelFunc` from `WithCancel`, `WithTimeout`, or `WithDeadline` on all paths. Consider `signal.NotifyContext` for shutdown and `WithCancelCause` when the reason matters. With `context.AfterFunc`, check who needs to stop the callback and how callers wait if it already started. [Context package](https://pkg.go.dev/context)
- Trace ownership of shared maps, slices, and buffers across goroutines. Include reads performed **while evaluating arguments** to a `go` statement. For example, if one goroutine appends under a lock, another can race while reading the slice header to pass it as an argument, before its new goroutine begins. [Go statements](https://go.dev/ref/spec#Go_statements)
- For loop captures, inspect the actual declaration and what the closure **or a callee** retains. Under Go 1.22 language semantics, variables declared by the loop have per-iteration identities. A range loop assigning into an existing variable with `=` still reuses it. Check aliasing through pointers and shared backing data before recommending or removing a `v := v` copy. [Loop semantics](https://go.dev/blog/loopvar-preview)
- Inspect `time.After` in hot loops, ticker shutdown, and timer stop/reset logic under the actual runtime semantics. For Go 1.23–1.26, check the main module’s version and the effective `asynctimerchan` setting. Go 1.27 removed that switch. Separate the need to stop live periodic work from collection of an unreachable timer or ticker. [Timer channel behavior](https://go.dev/wiki/Go123Timer) · [GODEBUG history](https://go.dev/doc/godebug)

### G4. Resources, HTTP, and databases

- Check file, listener, body, row, timer, transaction, and cancel-function lifetimes through error exits and long loops. A `defer` runs when the enclosing function returns. Look for accumulated handles or locks and scope cleanup to the iteration when needed. For `io.Reader` callers, process the `n > 0` bytes even when the same call returns `io.EOF` or another error.
- Inspect `http.DefaultClient`, `http.Get`/`Post`, and `http.ListenAndServe` in the context of actual request deadlines, server settings, proxies, and streaming. Check transport reuse, client timeouts, and `ReadHeaderTimeout`, `ReadTimeout`, `WriteTimeout`, and `IdleTimeout` where relevant. A total client timeout includes response-body consumption, which matters for long-lived streams. [HTTP client contract](https://pkg.go.dev/net/http#Client)
- Close response bodies after a successful request on every path. For HTTP/1.x connection reuse, consider reading a bounded remainder to EOF when the time and byte budgets allow it. Bound untrusted incoming bodies with `http.MaxBytesReader` or a checked limit. With `io.LimitReader`, verify that a truncated prefix cannot be mistaken for a complete acceptable body.
- In `database/sql`, inspect `rows.Close()` and `rows.Err()`, rollback on early exits after a successful `BeginTx`, and errors from `Commit`. Trace every transactional operation to the intended `*sql.Tx`. A call through `*sql.DB` can escape that transaction. Check context-aware methods, binding of query values, and pool sizing and lifetimes against real concurrency and database limits (`SetMaxOpenConns`, `SetMaxIdleConns`, `SetConnMaxLifetime`, `SetConnMaxIdleTime`). [Transaction guidance](https://go.dev/doc/database/execute-transactions)

### G5. Memory, I/O representations, and portability

- Follow slice capacity as well as length. Appending to a subslice can overwrite a caller’s elements, and a small view can retain a large allocation. A full-slice expression such as `s[a:b:b]` limits growth into the original tail. Existing elements still alias. Consider `slices.Clone` or an explicit copy when independent backing storage is needed. Inspect reused buffers and copies of structs containing slices or a nonempty `bytes.Buffer`.
- Check the serialized contract with the actual JSON package, options, and field tags. For example, default `encoding/json` marshaling of `[]string(nil)` and `[]string{}` produces `null` and `[]` respectively. Check absent, null, empty, and zero fields, `omitempty`, and version support for `omitzero` (Go 1.24+). Large integers decoded into `map[string]any` can lose precision through `float64`. Consider typed fields or `Decoder.UseNumber`. Use `DisallowUnknownFields` only for contracts that require rejection. [JSON package](https://pkg.go.dev/encoding/json) · [Go 1.24 changes](https://go.dev/doc/go1.24)
- Check assumptions about map iteration order, interface equality when the dynamic type can be uncomparable, `path` versus `path/filepath`, and `int` size or narrowing conversions on supported targets. In measured hot paths, evaluate bounded slice/map preallocation, `strings.Builder` or `bytes.Buffer`, redundant `[]byte`/`string` conversions, and padding in frequently allocated structs.

### G6. Standard library and version compatibility

Check a replacement’s semantics, minimum supported toolchain, and effects on callers before proposing it. An established logging, routing, or reproducibility contract can justify the existing implementation.

| Existing pattern | Candidate and compatibility check |
| --- | --- |
| `io/ioutil` or `interface{}` | Consider `io`/`os` equivalents (Go 1.16+) and `any` (1.18+) when they improve the code under review. |
| Repeated slice, map, or comparison utilities | Check the exact API in `slices`, `maps`, or `cmp` and built-ins `min`, `max`, or `clear` (packages and built-ins introduced in 1.21). Later additions have their own minimum versions. |
| Ad hoc structured logging or request routing | Consider `log/slog` (1.21+) or `http.ServeMux` method and path patterns (1.22+) when they preserve fields, matching behavior, and supported routes. |
| Package-level random seeding | `math/rand` added default automatic seeding in 1.20. `math/rand/v2` is a distinct API from 1.22. Preserve deterministic sequences where required using an appropriate local generator. Use `crypto/rand` for security-sensitive values. |

[Go 1.20 changes](https://go.dev/doc/go1.20) · [Go 1.21 changes](https://go.dev/doc/go1.21) · [Go 1.22 changes](https://go.dev/doc/go1.22)

### G7. Checks and diagnostic tools

Start from the repository’s own scripts. Select commands that answer a question for the reviewed module, package, tags, platform, and toolchain. Run applicable module-wide checks from **each relevant module root**. Use disposable paths for any generated binary, profile, benchmark output, or rewritten manifest.

| Question | Check and interpretation |
| --- | --- |
| Is the reviewed source formatted and buildable? | Run `gofmt -l` with the actual first-party filenames as arguments. Use `go vet ./...`, `go test -count=1 ./...`, and `go build ./...` with shipped settings. `-count=1` requests a fresh test execution even when build artifacts are cached. |
| Do exercised concurrent paths and test ordering hold up? | Where supported with cgo, run `go test -race -count=1 ./...`. Name the paths exercised. For order dependence, use `go test -shuffle=on -count=1 ./...` and retain the reported seed. Use a bounded existing fuzz target when it tests a parser contract. [Race detector](https://go.dev/doc/articles/race_detector) |
| Are dependencies and manifests consistent? | Use the project’s `govulncheck ./...` setup for applicable advisories. Use `go mod tidy -diff` with Go 1.23+ tooling. On an older toolchain, run `go mod tidy` in a disposable copy and inspect the manifest and checksum diff. [Go 1.23 changes](https://go.dev/doc/go1.23) |
| Which static or leak checks can decide a finding? | Use configured `staticcheck` or `golangci-lint` as applicable. Target checks such as `errcheck`, `errorlint`, `bodyclose`, `sqlclosecheck`, `nilerr`, `noctx`, `exhaustive`, `gosec`, `gocritic`, `prealloc`, `unconvert`, or `unused` where the installed version supports them. For a goroutine lifetime issue, consider a focused `go.uber.org/goleak` test or the `goroutineleak` runtime profile where available. [Goroutine leak profiles](https://go.dev/blog/goroutine-leak-profiles) |
| Is the change worth its cost? | From the package containing a representative benchmark, run `go test -run '^$' -bench . -benchmem -count=10` on comparable revisions. Save separate outputs and compare with `benchstat before.txt after.txt`. Use `pprof` CPU, heap, mutex, block, or goroutine profiles to localize a cost. For an allocation hypothesis, inspect `go build -gcflags=-m` for the affected package. [benchstat](https://pkg.go.dev/golang.org/x/perf/cmd/benchstat) |

### G8. Release binaries and runtime tuning

Evaluate `-trimpath` for reproducible paths, `-ldflags="-s -w"` for size, and linker `-X` for version metadata against the project’s release and debugging needs. Preserve usable diagnostic artifacts. Before recommending `CGO_ENABLED=0`, verify native dependencies, runtime behavior, and actual linkage on shipped targets. Use representative profiles for profile-guided optimization (`default.pgo`). Measure latency, throughput, and memory headroom before proposing `GOGC` or the soft runtime limit `GOMEMLIMIT` in a constrained deployment. [Profile-guided optimization](https://go.dev/doc/pgo) · [GC guide](https://go.dev/doc/gc-guide)

<!-- END GO MODULE -->
