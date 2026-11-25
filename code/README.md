# Code audit prompts

This directory contains prompts you can paste into an LLM to review a pull request, a release candidate, or a whole codebase. Give the reviewer the software and its requirements. The prompts ask it to trace what real callers do, find where the implementation breaks a promise, and show the evidence for each finding.

## Choose a workflow

| Workflow | When to use it | What you get |
| --- | --- | --- |
| [Core audit prompt](code-clarity.md#core-audit-prompt) | You want one reviewer to follow a change across the system. Use the full prompt or name particular tasks. | One scoped report with findings, checks, coverage gaps, and fix order. |
| [Eight independent runs](8-runs.md) | You want independent reviews of the same revision from four complementary pairs of sessions. | Eight reports you can reconcile into one release decision. |

## Run your first audit

1. **Pin the target.** Name the revision or PR base and head. For a working tree, include the relevant uncommitted files. State a promise a caller can observe, the supported environment, and any decision the reviewer should treat as settled. [Project packet template](code-clarity.md#project-packet)
2. **Start a fresh chat or coding-agent session.** Copy the core block and add your project packet. Give a coding agent access to the checkout. In a chat, attach the tree, relevant entry points and callers, tests, manifests, configuration, and build or release files. A text-only reviewer can trace source and specify a check for you to run.
3. **Append a matching module when relevant.** A Python service managed by Ansible can use both blocks. For a large repository, send a journey together with its producer, consumer, storage contract, and tests. [Technology modules](#technology-modules)
4. **Read the verdict for the scope actually reviewed.** Each substantive finding should give you a location, trigger, expected and actual outcome, fix, and check. Prioritize the findings that can affect a real caller. The default mode is read-only.

### Copy-paste example

After copying the core prompt, you can add a project packet like this one. Replace the example details with the ones that apply to your project.

```text
Review the current PR against its merge base with origin/main.
Focus on the import worker, its storage transaction, and the notice consumer.
If a worker restarts after committing a record and the queue redelivers the
same message, the customer must receive one completion notice and the database
must contain one record. Old and new workers overlap during deployment.
Use the repository's local test configuration and synthetic records. Audit only.
```

## How the eight runs fit together

Run the eight numbered blocks **in order, each in a fresh session**, with the same project packet, source revision, and relevant technology modules. **Odd runs map system design and contracts. Even runs test concrete implementation and configuration.** Keep this cycle's reports aside until all eight reviews finish. For example, Run 3 can establish who owns duplicate suppression. Run 4 independently checks what actually survives a retry.

| Runs | Design question | Implementation question |
| --- | --- | --- |
| 1 and 2 | What does the public interface promise, and who owns it? | Does the shipped entry point deliver that outcome, and would its tests catch a disconnected implementation? |
| 3 and 4 | What survives failure, retry, and version overlap? | What state and side effects actually remain after interruption, migration, or rollback? |
| 5 and 6 | Who may act on which resource, and what should operators observe? | Do effective settings and resource-level controls enforce that policy and report the outcome? |
| 7 and 8 | What must be built, shipped, documented, and measured? | Does the installed artifact work as advertised at the relevant scale? |

After Run 8, supply the reports to the consolidation block. It groups findings by root cause, keeps the run IDs as aliases, and checks disagreements against evidence. [Consolidate reports](code-clarity.md#consolidate-reports)

## Technology modules

Each file has a fenced prompt you can copy into either workflow. Use the modules that match the components and the current review question.

| Module | What it examines | Example question |
| --- | --- | --- |
| [Go](go/golang.md) | Effective language version, API and error contracts, goroutines, I/O, build targets, and profiling | What unblocks a producer when its consumer exits early? |
| [Python](python/README.md) | Imports, shared state, iterators, async cancellation, test doubles, and distribution packaging | Can the installed CLI load its bundled template outside the checkout? |
| [Terraform](terraform/README.md) | Provider and backend selection, resource addresses, existing state, lifecycle, and plan evidence | Will inserting a list item change the identity of an existing resource? |
| [Packer](packer/README.md) | Image inputs, selected builders, host and guest provisioners, post-processing, and first boot | Did the provisioning step run inside the image that consumers actually receive? |
| [POSIX shell](bash/README.md) | Portable POSIX.1-2017 shell, BSD/GNU utility differences, exit status, and native `/bin/sh` on macOS and Linux | Does a failing producer reach the caller, and do quoted filenames survive on both platforms? |
| [PowerShell](powershell/README.md) | Editions, object streams, native exit codes, error preferences, remoting, and `ShouldProcess` | Can a native command fail while the surrounding script returns success? |
| [Ansible](ansible/README.md) | Inventory, variable precedence, delegation, serial batches, handlers, and convergence | Does a migration intended to run once across the fleet run once per batch? |
| [Salt](salt/README.md) | Minion targeting, pillar and grains, rendered states, requisites, and convergence | Does changing one managed file trigger the intended service action? |

## Work with the results

**Source-verified** means the cited code or configuration establishes the claim. **Observed** means a check exercised it. An assumption that could overturn a finding belongs in open questions with the cheapest way to settle it. A missing critical journey can justify a verdict of **Insufficient evidence to judge**.

For a P0 or P1, give a fresh reviewer the finding and the same revision to challenge its premise. If a review stops mid-journey, carry its handoff into the next session for that review. [Verify findings](code-clarity.md#independent-verification-prompt) · [Resume a review](code-clarity.md#resume-a-review)

