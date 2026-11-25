# Python audit module

Copy the block into a review that includes Python.

````markdown
## Optional Python module

<!-- BEGIN PYTHON MODULE -->

Apply this module to the Python components in scope. Use the core audit prompt’s evidence rules, finding IDs, severity scale, and report format. For each finding, identify the supported interpreter, the caller that reaches the code, and the observable consequence. Run checks through the repository’s configured environment.

### PY1. Interpreter, imports, and production entry points

- Compare `requires-python`, dependency markers and extras, lock or constraints files, CI, and the interpreter in the released image or executable. Record the implementation and minor version for version-sensitive claims. Check new syntax, standard-library APIs, and native extensions on the oldest supported target and on the deployed target. [Python project metadata](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/)
- Start at the installed console script, `python -m` invocation, WSGI or ASGI application, registered task, or plugin that the caller uses. Check imports under its working directory and search path. Look for local modules shadowing dependencies, circular imports that expose partly initialized modules, and optional imports that catch a missing *transitive* dependency and silently disable a feature.
- Inspect import-time configuration, database access, and worker creation. For decorators used by a router or worker, follow the registered callable through its wrapper. Check its return value, async behavior, and the signature the framework reads. Before deleting an apparently unused symbol, check public re-exports, plugin metadata, dotted paths in configuration, and historical migrations that still import it.

### PY2. Values, shared state, and iteration

| Pattern to inspect | Check that can settle it |
| --- | --- |
| A mutable default or class attribute, a dataclass factory returning a shared object, or a shallow copy | Call the function twice or construct two instances. Mutate one nested value and observe the other. Verify whether sharing was intentional. A timestamp computed in a default argument is also fixed when the function is defined. |
| `value or default`, `mapping.get(key)`, or a truthiness check at an API boundary | Try an absent key, `None`, `0`, `False`, and an empty collection where those have distinct meanings. For a numeric validator, check whether accepting `bool` as an `int` matches the contract. |
| A callback or generator closes over a loop variable | Invoke the callbacks after the loop and compare the values each observes. Follow any helper that retains the callback. |
| An iterable is counted, inspected, then consumed again | Supply a one-shot iterator as well as a list. Check for lost elements after `any`, `next`, or a first traversal. Ordinary `zip` stops at the shorter input. If unequal lengths should raise an error, test that case and consider `strict=True` on Python 3.10+. |

For persisted or transmitted values, choose the boundary that matters to the caller. Examples include bytes decoded with the wrong encoding, monetary values constructed from binary floats, and a local wall-clock time that occurs twice during a daylight-saving transition. Check a round trip through the actual serializer and storage type. [Python programming FAQ](https://docs.python.org/3/faq/programming.html)

### PY3. Runtime types, exceptions, and resource ownership

- Trace an external value to its first runtime validation. `typing.cast` returns the original value, and `TypedDict` describes a static key contract. Locate the code that checks incoming mapping keys and values. Inspect `Any`, ignored type errors, coercion, and model constructors that bypass validation where they hide a reachable mismatch. If a framework reads annotations at runtime, exercise that registration path with forward references and `TYPE_CHECKING` imports. Python 3.14 introduced deferred annotation evaluation by default. [Typing](https://docs.python.org/3/library/typing.html#typing.cast) · [Python 3.14 changes](https://docs.python.org/3.14/whatsnew/3.14.html)
- Check production validation or side effects placed inside `assert` against supported optimized runs. Inspect broad `except` blocks, context managers that suppress exceptions, and `return` in `finally` for failures that become apparent successes. Where callers need the underlying cause, follow exception chaining through the public error boundary. [Assertion semantics](https://docs.python.org/3/reference/simple_stmts.html#the-assert-statement)
- Follow ownership of files, sockets, streaming responses, cursors, and generators through exceptions, partial iteration, and early exits. For example, a function can return a generator over a file whose `with` block ends before the caller reads the first item. Exercise the first read and early close at the caller.

### PY4. Async tasks, threads, and worker processes

- For each consequential coroutine, find who awaits it or owns the scheduled task, observes its exception, and drains it on shutdown. Inspect blocking calls such as `time.sleep` or synchronous HTTP requests on the event-loop thread. Bound fan-out against the connection pool or downstream service it consumes.
- Trace cancellation through `try/finally`, `async with`, and the operation doing the work. Check handlers that swallow `CancelledError`. With default `asyncio.gather`, one child raising an exception can leave siblings running. A `TaskGroup` on Python 3.11+ cancels remaining children when a child raises a non-cancellation exception. Establish which behavior the caller requires. [Asyncio task and cancellation semantics](https://docs.python.org/3/library/asyncio-task.html)
- A thread already executing work submitted through `asyncio.to_thread` or an executor can finish a write after its caller is cancelled. To check a promise about cancellation, pause the worker just before the write, request cancellation, and release the worker. Record when the caller acknowledges cancellation relative to the commit. Wait for the worker itself to finish before asserting the persisted state. [Future cancellation](https://docs.python.org/3/library/concurrent.futures.html#concurrent.futures.Future.cancel)
- Follow multi-step updates to shared state across threads. Check the synchronization protecting the whole invariant, including on free-threaded builds if supported. Across worker **processes**, establish which caches, locks, and connections are process-local. Check the production start method. For `spawn` and `forkserver`, verify that importing the main module is safe and the worker and its arguments are picklable. On Python 3.14+, applications that require `fork` must select it explicitly. [Multiprocessing start methods](https://docs.python.org/3/library/multiprocessing.html#contexts-and-start-methods)

### PY5. Database, network, and trust boundaries

- Follow a database write through commit, rollback, retry, and session closure. In `sqlite3`, the connection context manager handles the relevant transaction on exit and leaves the connection open. For an ORM, locate where a lazy query or relationship actually executes. Check whether it runs after the session closes or introduces a query per result. [SQLite connection context manager](https://docs.python.org/3/library/sqlite3.html#how-to-use-the-connection-context-manager)
- Read the timeout semantics of the HTTP client in use. For Requests, a read timeout bounds inactivity between bytes. A slowly streaming response can still exceed the caller’s total budget. Check response closure on early exit and whether a retried request repeats a committed side effect. [Requests timeouts](https://requests.readthedocs.io/en/latest/user/quickstart/#timeouts)
- Trace the provenance of data passed to `pickle`, object-constructing YAML loaders, `eval` or `exec`, and strings passed to `subprocess` with `shell=True`. Check database value binding and the destination of user-supplied file or archive paths. For a reachable trust-boundary error, cite the code, propose a constrained parser or API, and specify the intended rejection with benign local fixtures. [Pickle security guidance](https://docs.python.org/3/library/pickle.html)

### PY6. Tests that distinguish the broken behavior

- Check where `unittest.mock.patch` replaces a name. A module using `from client import send` looks up its own `send` binding. Follow that lookup in the production module. For async code, `AsyncMock.assert_called_once()` can pass when the returned coroutine was never awaited. Use an await assertion when awaiting is part of the contract, and verify the caller-visible result. [Where to patch](https://docs.python.org/3/library/unittest.mock.html#where-to-patch) · [AsyncMock](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.AsyncMock)
- Inspect fixtures that keep mutable objects, patch environment variables, change directories, or share event loops and database connections. Run a suspect test alone and alongside the test that changes its state. For concurrency, use events or barriers to reach the disputed ordering. For transaction behavior, check whether an enclosing test transaction hides a missing production commit.

### PY7. Prove the distribution works for its consumer

For a project that publishes or deploys a Python distribution, exercise this sequence in a disposable copy of the target revision.

1. Run the release build. With the PyPA `build` frontend, plain `python -m build` builds a source distribution and then builds the wheel from that archive. This can expose build inputs omitted from the source distribution. [PyPA build behavior](https://build.pypa.io/en/stable/explanation/how-it-works.html)
2. Install the exact resulting wheel into a clean virtual environment with the intended extras. Check that build requirements, runtime dependencies, and optional extras are declared in the appropriate metadata. From outside the checkout, record where the package was imported from and invoke its installed entry point.
3. Exercise a feature that loads a packaged resource, discovers a plugin, imports a native extension, or uses public type stubs, as applicable. For example, have the installed CLI render one bundled template. A source-checkout test can succeed while the wheel omits the template. [Testing an installed package](https://docs.pytest.org/en/stable/explanation/goodpractices.html)

### PY8. Verification and performance evidence

Choose the check that can decide the finding. In the report, replace example commands with the actual environment runner, paths, settings, and supported versions. Record the working directory and whether the code exercised came from the checkout or the installed artifact.

| Question | Evidence to collect |
| --- | --- |
| Which interpreter and dependency set ran? | Use `python -VV` and `python -c "import sys; print(sys.executable)"` in the target environment. `python -m pip check` checks installed dependency compatibility. For advisories, use the project’s configured tool, such as `pip-audit`, against the resolved production dependencies. [pip check](https://pip.pypa.io/en/stable/cli/pip_check/) |
| Does the regression test reach the caller? | Run `python -m pytest` with the actual test node ID, or `python -m unittest` with the dotted test name. Then run the neighboring suite. For a version-specific defect, repeat on the affected interpreter. Use configured tox or nox environments where available. |
| Does static analysis support the claim? | Use `ruff check` for configured lint rules, `ruff format --check` for a formatting check, or the project’s mypy or Pyright command for type checks. Inspect target versions, exclusions, and suppressions. Cite the diagnostic and trace its runtime or maintenance consequence. |
| What accounts for latency or memory growth? | Separate import cost, Python CPU time, external I/O, and retained memory. Use `python -X importtime` with the real entry point, `cProfile` for call costs, and `tracemalloc` snapshots for traced allocations. Inspect native allocations or process RSS when a Python allocation snapshot cannot explain growth. For `functools.cache` or `lru_cache`, check key cardinality, retained arguments and results, invalidation, and per-process lifetime. Compare repeated measurements on equivalent inputs and interpreters. [Allocation tracing](https://docs.python.org/3/library/tracemalloc.html) · [Cache retention](https://docs.python.org/3/library/functools.html#functools.lru_cache) |

<!-- END PYTHON MODULE -->
````
