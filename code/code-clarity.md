# Code audit prompt for large repositories

## Before you run it

1. **Start with a revision and a promise.** Name the release, commit, or pull request base and head. Describe what one real caller must be able to do and what must survive a failure. Include any condition that changes the outcome, such as old and new workers running together during a rollout.
2. **Give the reviewer the shipping context.** Supply repository access or a tree, README, manifests and lockfiles, entry points, build and CI scripts, deployment configuration, migrations, generators, and relevant tests. Include supported platforms, workload, and redacted sample data. Carry over the latest requirements, corrections, accepted tradeoffs, and earlier finding IDs.
3. **Start a fresh review conversation.** Give it the decision history and let it form its own view of the code. For a consequential release, a second reviewer or different model can challenge the first report. Resolve disagreements and agreements alike against the code and checks.
4. **Batch by a user journey.** Map the system first. When a journey and its dependencies no longer fit comfortably in context, review one at a time. Send the producer, worker, storage contract, notification consumer, and relevant tests together. In a new session, carry forward the map, revision, and last handoff. Ask for one consolidated report after the last batch.
5. **Choose what to copy.** Attach this file and name the sections to use, or copy between the matching `BEGIN` and `END` comments in GitHub’s **Raw** view. Use the core for every project. Append a language module only for the components it covers. Fill the context fields you know or give them in your message. Tell the reviewer whether it can run local checks. With pasted code, bring back the output of the most important proposed check.

[Core audit prompt](#core-audit-prompt) · [Independent verification prompt](#independent-verification-prompt) · [Template for other languages](#template-for-other-languages) · [Optional Go module](#optional-go-module)

**Example request for a pull request**

> Use the Core audit prompt in `CODE_AUDIT_PROMPT.md`. Compare this PR with its merge base on `origin/main`. Prioritize import retries. After a worker restart and redelivery, I expect one stored record and one completion notice. The older worker stays deployed during the rollout. Run the repository’s local checks where available. Audit only.

For a narrower follow-up, specify the journey and passes, for example “Run passes 3 and 7 on the import retry path.” Before acting on a high-impact finding, use the independent verification prompt with the same revision.

## Core audit prompt

<!-- BEGIN CORE AUDIT PROMPT -->

Review this software as the maintainer responsible for its next release. Identify which promises hold, where a real caller can encounter a failure, what can safely be simplified or removed, which runtime or build costs merit action, and which decisions need an owner. Ground each recommendation in the reviewed revision.

### Project context

Use my accompanying message and the repository to fill these fields. An empty field is an unknown. State an assumption when it could change a conclusion, and continue with the evidence available.

| Input | Your context (replace the guidance below) |
| --- | --- |
| Target and scope | Repository and revision, PR base and head, working tree, or supplied snippets. Whole project or named paths, exclusions, sweep, and pass numbers. |
| Product and promises | Users, entry points, critical journeys, expected outcomes, public contracts, and acceptance criteria. |
| Runtime and deployment | Languages and pinned versions, supported platforms, deployment model, persisted data, overlapping versions, workload, and resource limits. |
| History and decisions | Latest corrections and what they supersede, accepted tradeoffs and their premises, areas repeatedly changed, suspected regressions, prior finding IDs and revisions. |
| Evidence and access | Attached source, tests, release artifacts, configuration, logs, and available local checks. Identify any supplied CI results by revision. |
| Mode | Audit only by default. If requesting edits, name the findings or scope to fix. |

### How to conduct the review

- **Respect scope and mode.** Default to the available first-party project and all ten passes. Honor any selected passes, paths, and exclusions. Use read-only inspection and local checks that preserve the working tree. When I request edits, make the smallest coherent changes within that request, preserve unrelated work, and validate the affected callers.
- **Map first, then trace.** For a full audit, use pass 1 in Sweep A to map the system. Sample entry points, build and release definitions, operational configuration, and docs to prioritize Sweep B. In Sweep B, follow each journey through the applicable passes. Start with failures that have serious consequences, wide reach, recent changes, or difficult recovery, then cover the remaining in-scope paths. For a selected-pass review, collect just the context needed to apply those passes.
- **Test each claim against contrary evidence.** Look for a caller guarantee, alternate entry point, upstream guard, transaction, consumer deduplication, build constraint, or test that defeats the proposed trigger. Read how it works and where it applies. Treat “fixed,” “tested,” and “all callers updated” as leads. Reopen a settled decision when evidence changes its premise, and identify the change.
- **Make findings actionable.** Cite a real `path:line` at the target revision, or a file and symbol. For snippets, quote a short exact excerpt. Connect the caller, trigger, and consequence. Verify dependency APIs, flags, and platform behavior against the pinned versions. Group affected sites under one stable ID per root cause. Keep clean areas in the coverage ledger. Zero findings is a valid outcome.
- **Keep the audit resumable.** With repository access, continue across batches yourself. Carry the handoff below into each new batch. Resolve open links, keep a canonical ID and any aliases for merged findings, and recheck affected evidence if the revision changes. If space runs out, finish the current finding and end with `RESUME` plus the handoff. Consolidate once the available batches are complete.

| Batch handoff | What the next reviewer needs |
| --- | --- |
| Target and coverage | Revision and base. Journeys, passes, and paths reviewed in depth, skimmed, unseen, or excluded. |
| Findings and checks | Stable IDs and evidence locations, commands already run and their outcomes, and material assumptions. |
| Open boundary | The promise still to trace across components, which side was inspected, and the missing caller, consumer, or contract. |
| Resume at | The next exact file, symbol, or check and the question it should settle. |

### Audit passes

#### 1. Intent, scope, and coverage

Translate the latest explicit requirements and compatibility promises into observable outcomes. Identify which earlier requirement each correction superseded. If tests, docs, and code conflict, record the sources of disagreement and the precise decision needed from the owner.

For a PR, record the actual comparison base and head and follow changed behavior into unchanged callers and consumers. For local work, account for relevant staged, unstaged, and untracked files. For snippets, establish the latest identifiable version of each piece and mark missing or incompatible pieces. Map entry points, component boundaries, persistent and transmitted data, tests, feature flags, generated code, build inputs, and release artifacts. Read the project’s conventions before judging a design choice.

#### 2. End-to-end behavior and iteration drift

Start at a real UI action, CLI invocation, API or library call, scheduled job, or message delivery. Follow inputs, defaults, validation, state changes, errors, and side effects through the main success path and the most consequential failure. Work backward from each critical promise to the production registration or call site and the test that would fail if that connection disappeared. Include an external caller for a published library.

Look for incomplete renames, stale options or defaults, competing implementations, missed registrations, fixes applied to only one caller, and conflicting config loaders, loggers, or error policies. Trace placeholders, catch-and-log paths, dummy results, empty boilerplate, and fallbacks that report success. Check unfamiliar flags and methods against the installed dependency. Before calling anything a regression, inspect the same path and guarantee at the base revision.

#### 3. Correctness, recovery, and compatibility

- Exercise reachable boundaries. Distinguish missing, null, empty, and zero values. Consider malformed, partial, oversized, negative, and overflowing inputs, Unicode and encoding, duplicates, ordering, pagination, units, time zones, daylight-saving transitions, and clock or randomness assumptions.
- Interrupt a multi-step operation at its most consequential boundary. What survives a partial write, lost acknowledgement, retry, concurrent call, cancellation, restart, or rollback? Trace transactions, idempotency keys, lost updates, check-then-act races, and externally visible side effects. A uniqueness constraint can protect an imported row while redelivery still sends a second notice.
- Identify ownership and cleanup on every relevant exit. Follow files, sockets, connections, transactions, locks, timers, temporary files, subprocesses, and background work. Check who can unblock a wait and who observes completion.
- Read the public contract through its consumers. Check API shapes, CLI flags and exit codes, stdout and stderr, wire formats, stored records, migrations, and old defaults. During upgrades and rollback, trace an old reader consuming new data and a new reader consuming old data wherever the versions can overlap.

#### 4. Security, privacy, and trust boundaries

- In the supplied code and configuration, trace identity, role, tenant, and resource ownership to each relevant read or mutation, including alternate handlers, batch jobs, and workers. Identify the exact control that enforces the decision.
- Follow untrusted data to file paths, queries, commands, templates, and deserializers. Check controls for path traversal, injection, and unsafe decoding. For redirects and outbound requests, inspect destination restrictions against open redirects and server-side request forgery. Check size, rate, and time limits at the boundary that consumes the value.
- Review secrets and personal data in logs, errors, metrics, URLs, artifacts, and CI output. Check file permissions, TLS verification, cross-origin policy, admin and debug exposure, and cryptographic randomness where the contract requires it. Check dependency and CI action pinning, applicable advisories, and the affected paths the product actually uses. Consider retiring unmaintained dependencies when existing facilities cover the required behavior.
- Tie each finding to a reachable boundary and a specific defensive fix. Cite redacted source evidence and validate the intended control with ordinary local fixtures, such as two synthetic owners and an assertion about which records each may read.

#### 5. Operations and observability

Check startup validation, configuration precedence across flags, environment, and files, hidden working-directory assumptions, dependency readiness, and shutdown on supported signals. Trace what happens to in-flight work when a deployment drains or restarts a process.

Can callers and operators distinguish rejection, timeout, cancellation, partial completion, and success? Follow error reporting to the point someone can act. Look for swallowed failures, repeated logging of one error, hot-loop log spam, and unbounded metric labels. Evaluate timeout budgets, queue bounds, backpressure, retry eligibility and limits, backoff with jitter, and circuit breaking against an actual dependency failure. Propose telemetry for a specific operational question.

#### 6. Architecture, duplication, and deletion

Read the main path in execution order. Identify concrete changes made risky by mixed responsibilities, hidden initialization, surprising mutation, unclear state ownership, deep branching, circular dependencies, or domain policy split across transport and persistence. Look for external-client coupling that blocks an important test, mode flags that obscure callers, unexplained literals, and names that hide units, ownership, or side effects. Show how a simpler path preserves behavior.

Before sharing code, name the business rule, default, validation, mapping, or compatibility policy that must change together, its current locations, and what would drift. Search for its existing owner. Keep independently evolving policies separate. Consider splitting or inlining a helper when each caller needs a different flag, callback, or exception.

Look for obsolete adapters, pass-through layers, speculative interfaces and factories, abandoned flags, stale fallbacks, unused dependencies, redundant conversions and build targets, or commented-out code. Check public consumers, interface satisfaction, reflection, plugins, dynamic registration, feature-specific builds, generators, configuration, and scripts before declaring a deletion safe. If external usage is unknown, describe the compatibility check or deprecation path. State what to keep, move, merge, inline, or delete and which behavior must survive.

#### 7. Tests and their ability to detect failure

Derive expectations from a requirement, consumer contract, or independent example. For each critical guarantee, name a wrong implementation the suite would accept. Would the test still pass if a registration disappeared, the output were empty, or a side effect never occurred?

Inspect skipped tests, assertions that check only for “no error,” tests that duplicate the implementation, mocks that replace the behavior under test, and snapshots updated alongside a regression. In a diff, read removed tests and weakened assertions. Check for timing sleeps, wall clocks, random order, shared state, parallelism, network dependencies, and machine-specific paths. Pick the smallest decisive check with a setup, action, and observable assertion. Useful choices include a boundary table, legacy-data fixture, packaged CLI run, bounded property or fuzz test for a parser, or deterministic cancellation test.

#### 8. Runtime performance and scale

Start with the workload, input size or concurrency, and governing metric such as latency, throughput, peak memory, or cost. Inspect repeated I/O, N+1 queries, quadratic work, repeated parsing or compilation, chatty or unbuffered operations, missing batching or pagination, contention, unnecessary copies, whole-stream buffering, and unbounded caches or queues. Explain how the cost grows and whether the deployment reaches that scale.

For each candidate, give the smallest change, its tradeoff, and a measurement that would justify keeping it. Use representative profiles and repeated before-and-after samples with equivalent work, inputs, toolchain, and settings. Distinguish a cost visible in source from its measured effect. Label unmeasured improvements as hypotheses. Report numerical speedups only from actual measurements.

#### 9. Build, packaging, CI, and delivery

Trace the commands that produce what users install. Check clean builds, platform and architecture targets, feature settings, native dependencies, pinned tools and generators, lockfiles, embedded assets, and runtime files. Look for local overrides, warm caches, or untracked files masking release failures. For independently published packages, build an external consumer outside the workspace. Smoke-test the packaged artifact from a clean location when feasible.

Identify which stage a proposed improvement affects — dependency download, cold compilation, incremental rebuild, tests, linking, or artifact/image size. Look for repeated generation, forced rebuilds, ineffective cache keys, invalidated container layers, and redundant scripts. Inspect build context (including `.dockerignore`), multi-stage packaging, runtime user privileges, and version/commit metadata where relevant. Preserve diagnostic information and reproducibility. Trace stale generated output to its inputs or generator and fix it there when edits are authorized.

#### 10. Comments, public docs, and examples

Walk the README quickstart from a first-time user’s clean checkout through recognizable success. Check prerequisites, required services, versions, paths, flags, environment variables, defaults, commands, and sample output against the shipped interface, help text, and build. Check whether users can find architecture, configuration, development, testing, and troubleshooting information when they need it.

Read public API docs and comments beside their callers. Check preconditions, default values, ownership, concurrency, lifecycle, and errors. Preserve explanations of invariants, lock ordering, compatibility, and external quirks. Flag misleading comments, narration that obscures the code, stale TODOs, and suppressions whose reasons have expired. Where the implementation violates an established documented promise, report the implementation defect. Where the docs are wrong, supply the exact corrected sentence, command, or small example. Use executable examples for important usage and link duplicated documentation to its authoritative home.

### Verification and reporting

Inspect the project’s own CI definitions and scripts before choosing commands. Run relevant local format checks, linters, type checks, tests, dependency checks, clean builds, and smoke checks using the project’s settings. When commands write files, use a disposable copy that represents the actual target, including in-scope uncommitted work. Record the command, directory or module, revision, settings, outcome, and relevant skips. Distinguish fresh runs, cached results, user-supplied results, and checks blocked or unrun. With source-only access, finish the static review and supply exact project-specific checks for material gaps.

Use `Source-verified` when the cited code or configuration establishes the claim and `Observed` when a run demonstrated it. Label an inference about reach or impact `Inferred` and state its premise. If a missing fact could overturn the finding, put it under open questions as `Unverified`. Use `Hypothesis` for an unmeasured performance opportunity. A passing check establishes only the behavior it exercised.

Set severity by consequence, reach, likelihood, and recoverability. Keep effort and confidence separate from priority.

| Severity | Use when |
| --- | --- |
| P0 | A reachable failure blocks release, for example widespread data loss, material exposure, or an unusable primary path. |
| P1 | A serious reachable failure affects a supported workflow, such as incorrect results, a leak, a race, or sustained unavailability. |
| P2 | A bounded defect or recurring maintenance, performance, or operational cost has a concrete consequence. |
| P3 | A minor, actionable clarity, documentation, or idiom issue is worth a brief correction. |

Deliver one report in the following order. Use finding IDs to cross-reference details. If an area has no findings, distinguish reviewed coverage from an area left unreviewed.

1. **Verdict and scope.** Choose `Ready to ship`, `Fix before shipping`, or `Insufficient evidence to judge` for the named revision and scope. In up to five sentences, state what drives the decision, any material gap, and the largest systemic risk or complexity cost established. A known blocker warrants a fix verdict even if coverage is incomplete. Use insufficient evidence when a gap prevents a release judgment. Account separately for prior IDs as `fixed`, `still open`, `regressed`, `refuted`, `superseded`, or `unable to verify`, with the evidence for each status.
2. **Findings index.** Return a Markdown table with columns for ID (e.g. `F-001`), severity, primary pass, location, consequence, evidence, origin, and rough effort (S/M/L or unknown). Distinguish `regression` (the base met the promise), `introduced` (a defect in newly added behavior), `pre-existing`, and `unknown`. Use the same IDs throughout this and subsequent reviews.
3. **Findings in priority order.** Title each ID with the observable consequence. Cite the requirement or consumer, trigger, source path, expected and actual outcome or specific maintenance cost, and the strongest contrary evidence checked. Give the smallest sound fix and the test, measurement, or inspection that would establish it. Use a short patch or snippet where it helps. Check the proposed fix for changed contracts, races, cleanup failures, or new complexity. Keep minor findings to a line or two.
4. **Deletion and simplification.** For each candidate, cite its ID, the action, the complexity or measurable size it removes, the usage checks performed, and the contract to preserve. Identify misleading similarities worth keeping separate when consolidation was a real option.
5. **Documentation corrections.** Cite each affected ID and location. Give ready-to-paste replacements for broken commands, examples, or contract statements.
6. **Checks, coverage, and open questions.** Give the command ledger and coverage by journey, subsystem, and pass. For each material unknown, list the missing fact, who can decide it if needed, and the cheapest decisive check. Put unmeasured opportunities here. Tie suggested tooling, build flags, CI changes, and linter rules to a finding or documented gap.
7. **Fix order and edits.** Group IDs into small, dependency-aware changes. Start with release and data-integrity risks, then safe simplifications, then measured optimizations and delivery improvements. Keep regression checks and necessary contract or documentation updates with each fix. In audit-only mode, suggest the first group and leave the choice of implementation phase to me. When edits were requested, list the IDs actually fixed, the affected entry points, and exact validation. Where feasible, observe the regression test fail for the intended reason before the fix and pass afterward. Check neighboring behavior and state any remaining verification limit.

<!-- END CORE AUDIT PROMPT -->

## Independent verification prompt

<!-- BEGIN VERIFICATION PROMPT -->

Review the supplied audit report against the specified revision, comparison base, and accepted decisions. Independently challenge every P0 and P1 and any other finding I name. Keep the review read-only.

For each ID, restate the claim in one sentence. Trace the actual caller, triggering state, and consequence. Seek the strongest contrary evidence in the caller, guards, transactions, consumer guarantees, build settings, or tests. For a claimed regression, inspect how the base behaved. Check that the suggested fix preserves the contract for other callers.

Return a table with the original ID, verdict (`Confirmed`, `Refuted`, or `Needs evidence`), decisive source location or observed result, and next action. For a confirmed defect, give a short regression test using the project’s own interfaces, or a precise setup, action, and observable assertion if the harness is unavailable. It should fail for the identified reason before the fix and pass afterward. For a refuted claim, show exactly which protection covers the trigger. For an unresolved one, give the missing fact and cheapest check. Keep to the supplied findings unless you independently establish a new release blocker, which should be identified separately.

<!-- END VERIFICATION PROMPT -->

## Template for other languages

<!-- BEGIN LANGUAGE MODULE TEMPLATE -->

Apply these checks to the named language, runtime version, and components. Fill in project-specific examples or select the relevant row below. Map conclusions back to the core passes and finding IDs.

- Check language and toolchain versions, build targets, package boundaries, public interfaces, and conventions. Verify that a proposed replacement exists in the supported versions.
- Trace error propagation, resource release, cancellation, blocking work, and shared mutable state using the language’s actual semantics. Check nullability, numeric precision, encoding, serialization defaults, and copy or reference behavior at public boundaries.
- Inspect the framework’s request lifecycle, HTTP and database clients, query binding, pooling, and timeouts. Select the repository’s formatter, static analysis, test runner, sanitizer, dependency checker, or profiler to settle specific claims.

| Ecosystem | Examples to adapt |
| --- | --- |
| Python | Mutable default arguments, overly broad exception handling, blocking I/O in async code, context-managed resources. Consider Ruff, mypy or Pyright, pytest, and pip-audit where configured. |
| JavaScript / TypeScript | Unhandled promises, unvalidated data crossing typed boundaries, loss of type checking via `any`, or unexpected compiler strictness settings. Consider `tsc --noEmit`, ESLint, the configured test runner, and `npm audit` for npm projects. |
| Rust | Reachable `unwrap` or `expect` panics, invariants around `unsafe`, and locks held across `.await`. Consider Clippy, cargo-audit or cargo-deny, and Miri for supported unsafe-code tests. |
| Java / Kotlin | Resource closure, blocking work on shared executors, and unchecked casts crossing API boundaries. Consider the project’s compiler checks, SpotBugs, Error Prone, and dependency analysis where applicable. |

<!-- END LANGUAGE MODULE TEMPLATE -->
