# Ansible audit module

Copy the block into a review that includes Ansible.

````markdown
## Optional Ansible module

<!-- BEGIN ANSIBLE MODULE -->

Apply these checks to Ansible within the current review scope. Work read-only, cite the revision and file plus line or symbol, and check for a guard or platform setting that could disprove each claim. Preserve the main audit’s finding IDs and evidence labels when supplied. Give the smallest sound fix and one decisive check. Distinguish inspected source, observed output, and open questions.

### AN1. Resolve the hosts, variables, and code that run

Start with the actual playbook, controller version, collections, configuration file, inventory sources, host pattern, limits, tags, and extra variables. For a representative host, trace a consequential value through group and host variables, role defaults and variables, registered results, and overrides. Name the winning source. Follow roles, imports, dynamic includes, and collection-qualified module names into the task that acts. Check controller-side lookups and filters separately from work on managed hosts.

### AN2. Trace delegation and batch behavior

For a delegated task, identify the machine supplying connection settings and privileges, the host whose data is being used, and the machine that changes. Trace `inventory_hostname`, `hostvars`, and delegated facts where applicable. Check `serial`, `strategy`, forks, and `run_once` against the expected operation count. Under serial batching, a `run_once` task can execute once per batch. For a cluster-wide migration, use a fixture with enough hosts for two batches and count actual invocations.

### AN3. Test convergence and handler timing

Follow a template, package, or file change through `changed_when`, `notify`, handler registration, and the point at which handlers run. Inspect `meta: flush_handlers`, validation of new configuration, and what happens if a later task fails on that host. Check modes, owners, quoting of file permissions, service state, and stale files after a rename. Inspect raw shell or command tasks for repeatable behavior and meaningful guards such as `creates` or `removes`. On disposable nodes, compare first convergence, a second unchanged run, and a run with one input changed. Assert observed service or file state as well as reported change counts.

### AN4. Follow failures and sensitive values

Read `failed_when` and `changed_when` conditions with their real return code and output shapes. Entries in a YAML list of these conditions are combined with AND. Check skipped registered results, retries and `until`, `ignore_errors`, rescue blocks, and unreachable hosts for false success or partial rollout. Trace `become` and credentials to the specific task. Look for secret values exposed by debug tasks, diffs, templates, callbacks, or controller logs. Use synthetic variables in examples.

### AN5. Separate syntax checks from convergence evidence

Inspect inventory plugins, lookups, and collection code before running local checks. Use the pinned toolchain and fixture inventory with `ansible-playbook --syntax-check` and the project’s `ansible-lint` configuration. Inspect dynamically included tasks separately. Check mode depends on module support, and a task with `check_mode: false` can execute normally under `--check`. Run behavioral or diff checks only against a disposable fixture inventory with sanitized variables. Record the exact host set, tags, overrides, and what the check actually exercised.

<!-- END ANSIBLE MODULE -->
````

[Check and diff modes](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_checkmode.html) · [Execution strategies](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_strategies.html) · [Delegation](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_delegation.html) · [Handlers](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_handlers.html) · [Error handling](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_error_handling.html)
