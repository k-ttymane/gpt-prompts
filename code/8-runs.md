# Eight independent code-audit runs

Use eight fresh sessions on the same revision, in numerical order. **Runs 1, 3, 5, and 7 map system design and contracts. Runs 2, 4, 6, and 8 test concrete implementation and configuration.** A map can show that storage and notification delivery have different owners. A concrete check can show what happens to both after a committed message is redelivered.

1. **Prepare one project packet.** Give each session the repository or relevant files, target revision, comparison base for a change review, scope, current requirements and corrections, accepted tradeoffs, supported deployments, critical journeys, workload, and permitted local checks. Supply a tree, manifests and lockfiles, build/CI/release configuration, README, and representative tests. Include redacted fixtures or command output when useful.
2. **Run one block in each fresh session.** Copy everything inside that run’s Markdown block and attach the same packet. Append the relevant technology module for the components in scope. Keep the other seven reports aside during each review. Each block contains its own assignment, evidence rules, and report format.
3. **Keep the target stable.** Record working-tree changes or use a reproducible snapshot. For a large system, start with the highest-consequence journeys and batch by connected callers, implementations, consumers, and tests. Carry a coverage ledger within a run. Record the next unreviewed path if the available context runs out. A reviewer with source-only access can still establish source findings and give the exact missing check.
4. **Reconcile after Run 8.** Keep all eight reports, including clean results and coverage gaps. Merge duplicates by root cause, retain the run IDs as aliases, and assign stable consolidated IDs such as `F-001`. Resolve disagreements using cited contracts, source paths, or observed checks. Compare with earlier *historical* findings and mark fixed, open, regressed, refuted, superseded, or unverified. Attach related docs and tests to each fix, order fixes by impact and dependencies, and give a verdict for the revision and coverage actually reviewed. An uncovered critical path belongs in the final verdict.

| Pair | Odd run maps design and contracts | Even run tests implementation and configuration |
| --- | --- | --- |
| 1–2 | Public promises, system boundaries, ownership, and change costs | Shipped entry points, observable results, regressions, and test sensitivity |
| 3–4 | Durable state, recovery, concurrency, and compatibility guarantees | Partial failure, races, retry effects, legacy data, migrations, and rollback |
| 5–6 | Trust boundaries, configuration authority, and operational promises | Resource access, effective settings, failure signals, and safe recovery |
| 7–8 | Release inputs, supported artifacts, workload metrics, and public docs | Clean build, installed artifact, quickstart, release tests, and measured costs |

## Run 1 — Map system design and public behavior

````markdown
# Run 1 — Map system design and public behavior

Perform an independent, read-only review of system design and contracts. Reconstruct the promises and ownership from the artifacts in this session. Distinguish a documented promise, an implemented convention, and an assumption that needs an owner. Reconstruct what the product promises, which components own those promises, and where a real caller enters the system. Find design contradictions and missing or ambiguous ownership.

**Inputs.** Use the supplied repository or files, revision and comparison base, scope, current requirements, accepted tradeoffs, supported runtime and deployment, workload, and local check permissions. Use available evidence when a field is missing. For a working-tree review, include relevant staged, unstaged, and untracked files. For snippets, record versions and missing links.

**Review tasks**

1. Recover intent from the latest explicit requirements, public documentation, accepted decisions, and consumer contracts. Identify which earlier requirement a correction superseded. When sources disagree, cite both and state the product decision that would resolve the conflict.
2. Map the available first-party system from its shipped entry points. Include UI, CLI, API, library, scheduled work, and message consumers as applicable. For each critical journey, identify the production registration, component boundaries, stored or transmitted data, key side effects, and observable outcome. Work backward from one promised outcome to its registration as a check on the map.
3. Trace a representative maintenance change through its owners. Look for a default or business rule that would need edits in several places, hidden ownership, circular dependencies, and helpers whose mode flags encode independent policies. Name the rule and the callers that would drift. Respect deliberate separation where the contracts differ.
4. Identify compatibility surfaces before proposing simplification or deletion. Include public re-exports, configuration keys, plugins, dynamic registration, generated code, feature selections, and external consumers. Document unknown consumers and what evidence could establish safe removal.
5. Prioritize the mapped journeys by consequence, reach, recent changes, and recovery cost. Distinguish a demonstrated contradiction from an edge whose other side was unavailable.

**Evidence and report**

- Give a compact map with columns for journey, promised result, public entry point, owning component, downstream consumer, and evidence location. Include one concrete change scenario where ownership helps or hinders maintenance.
- Give one ID per demonstrated root cause, starting with `R1-001`. State the consequence, severity, real `path:line` or file and symbol, expected and actual behavior or concrete cost, strongest contrary evidence checked, smallest sound fix, and decisive verification. Use an exact excerpt for anonymous snippets. Verify the base before claiming a regression. Group affected sites under the same ID. Zero findings is valid.
- Rank P0 for a release blocker, P1 for a serious reachable failure, P2 for a bounded defect or recurring cost, and P3 for a minor actionable correction. Separate source-verified claims, observed runs, and inferences. Put unresolved hypotheses under open questions with their missing fact and cheapest check.
- Record the target, paths reviewed, material coverage gaps, and exact directory, command, and outcome for checks. Distinguish fresh, cached, supplied, blocked, and unrun checks. Close with `Ready to ship`, `Fix before shipping`, or `Insufficient evidence to judge` for this run’s stated scope. If interrupted, leave the next exact file or check.
````

## Run 2 — Test caller behavior and production wiring

````markdown
# Run 2 — Test caller behavior and production wiring

Perform an independent, read-only review of concrete implementation and configuration. Derive expectations from a requirement, public contract, or independent consumer in this session. Run focused local checks when available and preserve the reviewed working tree and deployed systems. Establish whether the actual public entry points execute the intended implementation and deliver the documented result. Find regressions that isolated helper tests can miss.

**Inputs.** Use the supplied repository or files, revision and comparison base, scope, current requirements, accepted tradeoffs, supported runtime and deployment, workload, and local check permissions. Use available evidence when a field is missing. For a working-tree review, include relevant staged, unstaged, and untracked files. For snippets, record versions and missing links.

**Review tasks**

1. Select the highest-consequence supported journeys directly from the supplied requirements and repository. Derive an expected result from a consumer or independent example. Invoke or trace the real entry point through defaults, validation, state changes, errors, and output. Check one main success and one consequential failure for each selected journey.
2. Challenge dispatch and iteration drift. Check a renamed option, an old caller or alternate path, a missing registration, a placeholder returning success, and a fallback that substitutes empty or dummy data where applicable. Inspect pinned APIs and the actual precedence of flags, environment, and config files. Compare base behavior before labeling a regression.
3. Use small boundary fixtures for missing, empty, null, zero, malformed, oversized, duplicate, or paginated inputs as applicable. Include character encoding, numeric limits, units, time zones, and ordering when they affect a consumer. Check exit status, stdout and stderr, stored data, and external side effects as relevant.
4. Test the tests. Name a wrong implementation the suite would still accept. Inspect mocks of the behavior under test, vacuous assertions, missing awaits or callbacks, removed tests, new goldens, and time- or order-dependent flakes. Specify a setup, action, and assertion that would distinguish the working and broken implementations.
5. For confirmed implementation drift or dead paths, check public and dynamically selected uses before proposing a deletion. Show the smallest fix and the behavior a neighboring caller must retain.

**Evidence and report**

- Give a matrix of entry point, input or state, expected result, observed or source-traced result, and test or code anchor. Include at least one disconnection or false-success case where the available evidence permits it.
- Give one ID per demonstrated root cause, starting with `R2-001`. State the consequence, severity, real `path:line` or file and symbol, expected and actual behavior or concrete cost, strongest contrary evidence checked, smallest sound fix, and decisive verification. Use an exact excerpt for anonymous snippets. Verify the base before claiming a regression. Group affected sites under the same ID. Zero findings is valid.
- Rank P0 for a release blocker, P1 for a serious reachable failure, P2 for a bounded defect or recurring cost, and P3 for a minor actionable correction. Separate source-verified claims, observed runs, and inferences. Put unresolved hypotheses under open questions with their missing fact and cheapest check.
- Record the target, paths reviewed, material coverage gaps, and exact directory, command, and outcome for checks. Distinguish fresh, cached, supplied, blocked, and unrun checks. Close with `Ready to ship`, `Fix before shipping`, or `Insufficient evidence to judge` for this run’s stated scope. If interrupted, leave the next exact file or check.
````

## Run 3 — Map state, recovery, and compatibility contracts

````markdown
# Run 3 — Map state, recovery, and compatibility contracts

Perform an independent, read-only review of system design and contracts. Reconstruct the promises and ownership from the artifacts in this session. Distinguish a documented promise, an implemented convention, and an assumption that needs an owner. Reconstruct how durable state and side effects are supposed to behave across boundaries, failures, concurrent calls, and mixed-version deployments. Identify guarantees for which two components assume different owners.

**Inputs.** Use the supplied repository or files, revision and comparison base, scope, current requirements, accepted tradeoffs, supported runtime and deployment, workload, and local check permissions. Use available evidence when a field is missing. For a working-tree review, include relevant staged, unstaged, and untracked files. For snippets, record versions and missing links.

**Review tasks**

1. Identify the authoritative copy of each consequential record, its identifiers and units, who may mutate it, which reads can be stale, and which effects live outside the transaction. Map the producer, broker or transport, worker, storage system, and downstream consumer where present.
2. For a critical mutation, map the promised ordering of validation, commit, acknowledgement, notification, and cleanup. Identify the durability boundary, retry policy, deduplication scope and retention, and recovery owner after each interruption. Compare any exactly-once promise with what is promised separately for stored rows and for external effects.
3. Describe ownership of shared state and work. Include locks or leases, worker completion, deadlines, cancellation, shutdown and drain, and resources that must be released. Record what callers may assume after cancellation is acknowledged and what unblocks a waiting owner.
4. Map serialized forms, persisted data, migration phases, public protocol versions, and rollout ordering. Determine which old readers and new writers can coexist, what a rollback can still read, and which compatibility promises downstream consumers rely on.
5. Identify any gap that requires a product or operator decision, such as whether duplication or delayed completion is acceptable. Search for contract provisions and compensating guarantees before escalating it.

**Evidence and report**

- Return a state-transition table with owner, transition, durable result, externally visible effect, failure window, and recovery or compatibility promise. Cite the source of each material guarantee.
- Give one ID per demonstrated root cause, starting with `R3-001`. State the consequence, severity, real `path:line` or file and symbol, expected and actual behavior or concrete cost, strongest contrary evidence checked, smallest sound fix, and decisive verification. Use an exact excerpt for anonymous snippets. Verify the base before claiming a regression. Group affected sites under the same ID. Zero findings is valid.
- Rank P0 for a release blocker, P1 for a serious reachable failure, P2 for a bounded defect or recurring cost, and P3 for a minor actionable correction. Separate source-verified claims, observed runs, and inferences. Put unresolved hypotheses under open questions with their missing fact and cheapest check.
- Record the target, paths reviewed, material coverage gaps, and exact directory, command, and outcome for checks. Distinguish fresh, cached, supplied, blocked, and unrun checks. Close with `Ready to ship`, `Fix before shipping`, or `Insufficient evidence to judge` for this run’s stated scope. If interrupted, leave the next exact file or check.
````

## Run 4 — Test failure, concurrency, migration, and rollback

````markdown
# Run 4 — Test failure, concurrency, migration, and rollback

Perform an independent, read-only review of concrete implementation and configuration. Derive expectations from a requirement, public contract, or independent consumer in this session. Run focused local checks when available and preserve the reviewed working tree and deployed systems. Test whether the implementation preserves state and external effects during interruptions, concurrent requests, retries, cancellation, and version overlap.

**Inputs.** Use the supplied repository or files, revision and comparison base, scope, current requirements, accepted tradeoffs, supported runtime and deployment, workload, and local check permissions. Use available evidence when a field is missing. For a working-tree review, include relevant staged, unstaged, and untracked files. For snippets, record versions and missing links.

**Review tasks**

1. Choose a real operation that crosses two writes, a durable write and an acknowledgement, or an internal commit and an external effect. Derive the expected invariant from a requirement or consumer. In a local fixture, make the second step fail or withhold the acknowledgement after commit, then retry. Assert final stored state and side-effect counts separately.
2. Inspect all paths around the claimed protection. Check transaction membership, unique keys and their scope, retry eligibility, concurrent check-then-act, ordering, and consumer deduplication. Where concurrency matters, force the relevant ordering with a barrier or event. Record which interleaving a test actually exercised.
3. Trace cancellation, timeout, shutdown, and restart through a blocking operation. Check whether orphaned work can still commit, who releases resources, and how callers distinguish partial completion from failure. Verify cleanup even when the consumer exits early.
4. Check mixed-version readers and writers with representative legacy fixtures and supported schema versions. Follow migration and rollback paths from an existing installation. Verify converted values, defaults, order, and compatibility with background workers still on the older version.
5. Check retry and rollback expectations in the tests. A test that only asserts success can miss duplicate notices or a partially committed write. Use an isolated fixture for stateful work and record the remaining gap when only static inspection is possible.

**Evidence and report**

- Return a table with the interruption or ordering, precondition, action or fixture, expected invariant, actual remaining state and side effects, and the guard or evidence checked.
- Give one ID per demonstrated root cause, starting with `R4-001`. State the consequence, severity, real `path:line` or file and symbol, expected and actual behavior or concrete cost, strongest contrary evidence checked, smallest sound fix, and decisive verification. Use an exact excerpt for anonymous snippets. Verify the base before claiming a regression. Group affected sites under the same ID. Zero findings is valid.
- Rank P0 for a release blocker, P1 for a serious reachable failure, P2 for a bounded defect or recurring cost, and P3 for a minor actionable correction. Separate source-verified claims, observed runs, and inferences. Put unresolved hypotheses under open questions with their missing fact and cheapest check.
- Record the target, paths reviewed, material coverage gaps, and exact directory, command, and outcome for checks. Distinguish fresh, cached, supplied, blocked, and unrun checks. Close with `Ready to ship`, `Fix before shipping`, or `Insufficient evidence to judge` for this run’s stated scope. If interrupted, leave the next exact file or check.
````

## Run 5 — Map trust, configuration authority, and operations

````markdown
# Run 5 — Map trust, configuration authority, and operations

Perform an independent, read-only review of system design and contracts. Reconstruct the promises and ownership from the artifacts in this session. Distinguish a documented promise, an implemented convention, and an assumption that needs an owner. Determine who may do what to which resource, which configuration controls that decision, and which signals allow operators to tell what happened. Look for discrepancies between the contracts at adjacent boundaries.

**Inputs.** Use the supplied repository or files, revision and comparison base, scope, current requirements, accepted tradeoffs, supported runtime and deployment, workload, and local check permissions. Use available evidence when a field is missing. For a working-tree review, include relevant staged, unstaged, and untracked files. For snippets, record versions and missing links.

**Review tasks**

1. Map actors, service identities, tenants, resources, trust boundaries, and delegated privileges. Cover alternate entry points such as jobs, bulk operations, plugins, and administrative paths. Identify the owner of authentication and resource-level authorization, including the assumed identity after a handoff.
2. Follow sensitive data and untrusted inputs through components, storage, logs, metrics, error messages, URLs, build artifacts, and outbound destinations. Map required validation, retention or redaction, rate and size bounds, and the owners of those controls. Use redacted examples.
3. Map precedence and authority for flags, files, environment, secrets, feature settings, deployment overrides, and infrastructure configuration. Identify the intended safe default and how invalid, missing, empty, or conflicting values are supposed to surface. Check whether different components duplicate the same policy.
4. Define the operational contract for startup, readiness, timeouts, retries, backpressure, drain, cancellation, and partial completion. Which signal should tell an operator that a job is delayed, rejected, committed, or safe to retry? Trace the documented response to the signal that enables it.
5. Include build and dependency trust where it crosses a real boundary. Identify who can change pins or release inputs and which component verifies or consumes them. Treat a missing piece of deployment evidence as a specific open question.

**Evidence and report**

- Give a boundary table with actor or input, protected resource, promised control, policy owner, configuration source, and observable outcome. Mark any deployment assumption that could reverse a conclusion.
- Give one ID per demonstrated root cause, starting with `R5-001`. State the consequence, severity, real `path:line` or file and symbol, expected and actual behavior or concrete cost, strongest contrary evidence checked, smallest sound fix, and decisive verification. Use an exact excerpt for anonymous snippets. Verify the base before claiming a regression. Group affected sites under the same ID. Zero findings is valid.
- Rank P0 for a release blocker, P1 for a serious reachable failure, P2 for a bounded defect or recurring cost, and P3 for a minor actionable correction. Separate source-verified claims, observed runs, and inferences. Put unresolved hypotheses under open questions with their missing fact and cheapest check.
- Record the target, paths reviewed, material coverage gaps, and exact directory, command, and outcome for checks. Distinguish fresh, cached, supplied, blocked, and unrun checks. Close with `Ready to ship`, `Fix before shipping`, or `Insufficient evidence to judge` for this run’s stated scope. If interrupted, leave the next exact file or check.
````

## Run 6 — Test controls, configuration, and operator signals

````markdown
# Run 6 — Test controls, configuration, and operator signals

Perform an independent, read-only review of concrete implementation and configuration. Derive expectations from a requirement, public contract, or independent consumer in this session. Run focused local checks when available and preserve the reviewed working tree and deployed systems. Verify that shipped code and configuration enforce intended controls and expose accurate results to callers and operators. Work from reviewed source, locally controlled fixtures, and redacted evidence.

**Inputs.** Use the supplied repository or files, revision and comparison base, scope, current requirements, accepted tradeoffs, supported runtime and deployment, workload, and local check permissions. Use available evidence when a field is missing. For a working-tree review, include relevant staged, unstaged, and untracked files. For snippets, record versions and missing links.

**Review tasks**

1. Trace a permitted and a rejected action through the same resource-level control using synthetic identities and records. Check alternate handlers, background operations, tenant ownership, and side effects on rejection. For file, query, command, deserialization, redirect, or outbound-request boundaries, inspect the actual validating or binding operation and propose a specific defensive correction if it is missing.
2. Exercise the effective configuration loader or renderer with absent, empty, conflicting, and invalid values. Verify precedence, target environment selection, defaults, feature toggles, credentials scope, and behavior after reload or restart. Inspect rendered or packaged configuration with synthetic secrets. Check permissions, transport settings, exposed admin/debug surfaces, and dependencies against the pinned versions and applicable advisories.
3. In local fixtures, trace rejection, dependency timeout, cancellation, and partial completion through the status returned, logs, metrics, and readiness signal. Look for swallowed exceptions, false success, duplicate logging, hot-loop logging, sensitive fields, or unbounded metric labels. Name the operational decision affected.
4. Inspect retry budgets, queue and connection limits, backoff, startup order, and shutdown. Check what happens if a dependency never becomes ready or a worker exits early. Verify a failed configuration does not report successful readiness where that promise applies.
5. Before reporting a failed control, challenge it with the strongest actual guard, middleware, configuration layer, deployment restriction, or test you can find. Use source review and ordinary local control assertions within the named scope.

**Evidence and report**

- Return a control-check table with expected policy, effective code or setting, benign fixture or source trace, caller-visible result, operator signal, and decisive evidence.
- Give one ID per demonstrated root cause, starting with `R6-001`. State the consequence, severity, real `path:line` or file and symbol, expected and actual behavior or concrete cost, strongest contrary evidence checked, smallest sound fix, and decisive verification. Use an exact excerpt for anonymous snippets. Verify the base before claiming a regression. Group affected sites under the same ID. Zero findings is valid.
- Rank P0 for a release blocker, P1 for a serious reachable failure, P2 for a bounded defect or recurring cost, and P3 for a minor actionable correction. Separate source-verified claims, observed runs, and inferences. Put unresolved hypotheses under open questions with their missing fact and cheapest check.
- Record the target, paths reviewed, material coverage gaps, and exact directory, command, and outcome for checks. Distinguish fresh, cached, supplied, blocked, and unrun checks. Close with `Ready to ship`, `Fix before shipping`, or `Insufficient evidence to judge` for this run’s stated scope. If interrupted, leave the next exact file or check.
````

## Run 7 — Map delivery, workload, and documentation contracts

````markdown
# Run 7 — Map delivery, workload, and documentation contracts

Perform an independent, read-only review of system design and contracts. Reconstruct the promises and ownership from the artifacts in this session. Distinguish a documented promise, an implemented convention, and an assumption that needs an owner. Reconstruct how a revision becomes a release artifact, who consumes it, the workloads it must handle, and the contracts the documentation and CI claim to establish.

**Inputs.** Use the supplied repository or files, revision and comparison base, scope, current requirements, accepted tradeoffs, supported runtime and deployment, workload, and local check permissions. Use available evidence when a field is missing. For a working-tree review, include relevant staged, unstaged, and untracked files. For snippets, record versions and missing links.

**Review tasks**

1. Map source and generated inputs through dependency resolution, generators, build roots, CI, tests, packaging, publishing, and the installed entry point. Identify the owner and supported platform for each step. Include assets, migrations, native dependencies, default configuration, version metadata, and consumer-visible diagnostics.
2. Define the clean-build and release promises. Identify what may legitimately come from a cache and what must be reconstructible from the stated inputs. Map cache invalidation and platform-specific builds, and distinguish published-package consumers from users inside a development workspace.
3. For each critical runtime or build workload, map the relevant metric, unit, plausible scale, and owner. Separate latency, throughput, memory, and cost from dependency download time, cold build, incremental rebuild, tests, linking, and artifact size. Identify where a representative measurement can be taken and which configuration affects it.
4. Walk the README, CLI help, public API docs, examples, and comments as contracts. Identify promised prerequisites, supported versions, flags, defaults, output, error semantics, ownership, lifecycle, and concurrency. Map critical claims to production entry points and checks designed to detect a broken promise.
5. Find duplicated release rules, build targets, dependency declarations, docs, or generated output. Identify the canonical owner, the behavior a simplification must preserve, and which downstream public or dynamic consumers still require it.

**Evidence and report**

- Give a release-and-consumer map from input through build gate and artifact to the promised outcome. Include a workload/metric table and at least one documentation promise whose verification is material to shipping.
- Give one ID per demonstrated root cause, starting with `R7-001`. State the consequence, severity, real `path:line` or file and symbol, expected and actual behavior or concrete cost, strongest contrary evidence checked, smallest sound fix, and decisive verification. Use an exact excerpt for anonymous snippets. Verify the base before claiming a regression. Group affected sites under the same ID. Zero findings is valid.
- Rank P0 for a release blocker, P1 for a serious reachable failure, P2 for a bounded defect or recurring cost, and P3 for a minor actionable correction. Separate source-verified claims, observed runs, and inferences. Put unresolved hypotheses under open questions with their missing fact and cheapest check.
- Record the target, paths reviewed, material coverage gaps, and exact directory, command, and outcome for checks. Distinguish fresh, cached, supplied, blocked, and unrun checks. Close with `Ready to ship`, `Fix before shipping`, or `Insufficient evidence to judge` for this run’s stated scope. If interrupted, leave the next exact file or check.
````

## Run 8 — Test the release artifact, costs, and public instructions

````markdown
# Run 8 — Test the release artifact, costs, and public instructions

Perform an independent, read-only review of concrete implementation and configuration. Derive expectations from a requirement, public contract, or independent consumer in this session. Run focused local checks when available and preserve the reviewed working tree and deployed systems. Verify the release candidate as an actual consumer, check consequential runtime or build costs, and independently judge the available release evidence for this scope.

**Inputs.** Use the supplied repository or files, revision and comparison base, scope, current requirements, accepted tradeoffs, supported runtime and deployment, workload, and local check permissions. Use available evidence when a field is missing. For a working-tree review, include relevant staged, unstaged, and untracked files. For snippets, record versions and missing links.

**Review tasks**

1. Read the real CI, build, and release commands. In a disposable copy representing the target, exercise the clean build and applicable supported targets. Check whether local overrides, untracked files, a warm cache, missing generator inputs, or omitted runtime assets change the result. Compare cold and incremental stages when relevant.
2. Install or extract the candidate outside the source checkout. Follow the published quickstart, import or invoke the shipped program, and exercise a feature that uses an asset, plugin, config file, or dependency. Check prerequisites, paths, versions, flags, defaults, exit status, and output. Supply the exact corrected sentence or command when documentation is wrong. Report an implementation defect when the established public promise is valid.
3. Inspect release tests, skipped jobs, weakened assertions, changed snapshots, and linter or type-check suppressions. Name a broken build, missing registration, or omitted side effect that could still pass. Choose the smallest external-consumer check that would catch it.
4. For a material cost, trace a representative request, job, or build stage to repeated I/O, N+1 queries, quadratic work, contention, unnecessary copies, unbounded data, repeated generation, bad cache keys, or oversized images. Quantify the input size and measurement method. Compare repeated equivalent runs before claiming a numerical improvement. List unmeasured opportunities as hypotheses with their tradeoffs.
5. Inspect any proposed removal of a release target, dependency, wrapper, or duplicated document against shipped consumers, generators, configuration, and supported builds. Give an atomic fix order for findings from this run, with the regression test and contract update attached to each fix.

**Evidence and report**

- Return the artifact and consumer check results, exact documentation replacements where applicable, and a focused measurement table giving workload, baseline, candidate if measured, settings, and uncertainty. Give a release verdict bounded to the evidence you gathered in this session.
- Give one ID per demonstrated root cause, starting with `R8-001`. State the consequence, severity, real `path:line` or file and symbol, expected and actual behavior or concrete cost, strongest contrary evidence checked, smallest sound fix, and decisive verification. Use an exact excerpt for anonymous snippets. Verify the base before claiming a regression. Group affected sites under the same ID. Zero findings is valid.
- Rank P0 for a release blocker, P1 for a serious reachable failure, P2 for a bounded defect or recurring cost, and P3 for a minor actionable correction. Separate source-verified claims, observed runs, and inferences. Put unresolved hypotheses under open questions with their missing fact and cheapest check.
- Record the target, paths reviewed, material coverage gaps, and exact directory, command, and outcome for checks. Distinguish fresh, cached, supplied, blocked, and unrun checks. Close with `Ready to ship`, `Fix before shipping`, or `Insufficient evidence to judge` for this run’s stated scope. If interrupted, leave the next exact file or check.
````
