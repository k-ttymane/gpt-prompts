# Salt audit module

Copy the block into a review that includes Salt.

````markdown
## Optional Salt module

<!-- BEGIN SALT MODULE -->

Apply these checks to Salt within the current review scope. Work read-only, cite the revision and file plus line or symbol, and check for a guard or platform setting that could disprove each claim. Preserve the main audit’s finding IDs and evidence labels when supplied. Give the smallest sound fix and one decisive check. Distinguish inspected source, observed output, and open questions.

### SA1. Determine which minion gets which state

Start with the Salt version, master/minion or masterless mode, real invocation, target matcher, minion ID, `saltenv`, `pillarenv`, file roots, and pillar roots. Trace top-file matching, included SLS files, imported maps, and pillar assignment for one representative minion. Compare a second minion that should receive a different result. Check whether externally controlled grains are being trusted to choose privileges or determine access to sensitive pillar data.

### SA2. Follow rendering into the actual state graph

Trace the configured renderer pipeline, commonly Jinja then YAML. Identify the effective pillar, grains, defaults, and execution functions used at render time. Inspect blank or missing values, implicit YAML conversions, generated IDs, duplicate IDs, and OS-specific mapping keys. Follow a rendered state ID and its `name` into the low-level state chunk for the real resource. Check that the correct file path, package, service, and state module survive compilation.

### SA3. Check ordering and change-triggered behavior

Reconstruct requisite edges from the compiled states, including reverse `*_in` declarations, includes, and extensions. `require` orders work and gates success. `onchanges` gates execution on changes. With `watch`, examine the watching state’s normal result and its module’s `mod_watch` behavior. With `listen`, check end-of-run timing. For a config-file/service pair, trace an unchanged file, a changed file, and a failed file update. Establish whether the service is left running, reloaded, restarted, or incorrectly touched.

### SA4. Test convergence, guards, and failure propagation

Inspect `cmd.run` states because a command that runs is normally reported as a change. Check command return codes, `onlyif`, `unless`, `creates`, `stateful`, and `onchanges` against the actual installed state. Trace ownership, permissions, sources and hashes, pillar-rendered secrets, and service ordering on a fresh minion and an already converged one. For orchestrations, follow a minion failure to the caller and inspect the target set and batch behavior before claiming success.

### SA5. Verify on an isolated minion configuration

Inspect templates, custom renderers, and execution functions before compiling. A template can execute functions during rendering. With a disposable masterless fixture, explicit config directory, and synthetic file/pillar roots, select `salt-call --local -c <fixture-config-dir> state.show_sls <sls>` to inspect high data or `state.show_low_sls` to inspect compiled chunks. Substitute the real fixture paths and SLS name. Where appropriate, inspect a fixture test run with `test=True`. A predicted change, often returned with `result: None`, still needs a convergence assertion. Keep minion-specific and secret values redacted. Record the state ID, target, environment, and whether evidence came from source, rendering, prediction, or a controlled state run.

<!-- END SALT MODULE -->
````

[State compilation](https://docs.saltproject.io/salt/user-guide/en/latest/topics/state-system.html) · [Requisites](https://docs.saltproject.io/en/latest/ref/states/requisites.html) · [Command state changes](https://docs.saltproject.io/en/latest/ref/states/all/salt.states.cmd.html) · [State inspection](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.state.html) · [State testing](https://docs.saltproject.io/en/latest/ref/states/testing.html) · [Grains](https://docs.saltproject.io/en/latest/topics/grains/index.html)
