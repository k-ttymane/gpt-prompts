# Terraform audit module

Copy the block into a review that includes Terraform.

````markdown
## Optional Terraform module

<!-- BEGIN TERRAFORM MODULE -->

Apply these checks to Terraform within the current review scope. Work read-only, cite the revision and file plus line or symbol, and check for a guard or platform setting that could disprove each claim. Preserve the main audit’s finding IDs and evidence labels when supplied. Give the smallest sound fix and one decisive check. Distinguish inspected source, observed output, and open questions.

### T1. Establish the deployment boundary

Start at the root module and command that CI actually invokes. Identify the Terraform version, backend, workspace or state key, provider aliases, account or subscription, region, variable sources, and module path. Follow provider configurations through child modules. Show which environment a resource would change if a default provider or variable wins unexpectedly. Check provider selections and checksums in `.terraform.lock.hcl` separately from remote module versions and source refs.

### T2. Trace addresses and existing state

Pick an important resource already in use. Follow its address and instance key from the base revision to the review target. Inspect `count` index shifts, unstable `for_each` keys, module or resource renames, provider changes, imports, and `moved` blocks. Adding an item in the middle of a list can reassign indexed resources. Test the existing two-instance case with a third item inserted and check which original objects keep their identity. Cite the state mapping, provider schema, or redacted plan evidence before claiming a replacement.

### T3. Evaluate values, ordering, and lifecycle

Trace missing, null, empty, and unknown-at-plan-time values through expressions and module outputs. Check whether `try`, defaults, or conditionals silently select the wrong account, network, or size. Inspect graph edges and data-source reads. An ordering dependency can leave an external service unready. Check the actual readiness contract, quota, and unique-name constraints before recommending `depends_on` or `create_before_destroy`. Identify drift that `ignore_changes` would conceal and the limits of `prevent_destroy` if a block is removed.

### T4. Follow state, privilege, and recovery

Check state isolation, backend access and locking where supported, CI credentials, provider permissions, and public exposure against the intended environment. Ordinary sensitive values can persist in state and saved plans, and JSON output can expose them. Use redacted evidence. Trace one partial operation or interrupted migration into the next plan. Identify what is already recorded, what may exist only at the provider, and which consumer or recovery procedure depends on it.

### T5. Use checks with a known scope

Run `terraform fmt -check -recursive` for formatting. In a disposable copy with trusted dependencies, `terraform init -backend=false -input=false` followed by `terraform validate` checks configuration consistency for the root in scope. Preserve the committed lockfile when testing its resolution. Use an existing redacted plan or a reviewed mock-provider test to assess a specific resource transition. A plan can read remote objects, and `terraform test` can create real resources. Inspect test run modes, providers, and data sources before choosing an isolated fixture. Record the root, workspace assumptions, variable set, addresses, and what the check established.

<!-- END TERRAFORM MODULE -->
````

[Validation scope](https://developer.hashicorp.com/terraform/cli/commands/validate) · [Module refactoring](https://developer.hashicorp.com/terraform/language/modules/develop/refactoring) · [Dependency locks](https://developer.hashicorp.com/terraform/language/files/dependency-lock) · [Plan and state output](https://developer.hashicorp.com/terraform/cli/commands/show) · [Terraform tests](https://developer.hashicorp.com/terraform/cli/commands/test)
