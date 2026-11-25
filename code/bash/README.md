# POSIX shell audit module for macOS and Linux

Copy this block when a script must run under the system `/bin/sh` on macOS and Linux. Use POSIX.1-2017 as the portability baseline for mixed-version fleets. Record any stronger minimum versions the project promises.

````markdown
## Optional POSIX shell module for macOS and Linux

<!-- BEGIN POSIX SHELL MODULE -->

Review the shell code as portable POSIX `sh` for macOS and Linux. Use POSIX.1-2017 as the common baseline unless the project specifies newer minimum shells and utilities on both platforms. Work read-only, cite the revision and source location, and tie each finding to an input, observable effect, smallest portable fix, and test. If appended to a focused audit, stay within that run’s scope.

### SH1. Identify the actual interpreter and utilities

Trace direct execution, `sh script`, scripts loaded with the dot command, and the shell selected by CI, Make, containers, or a provisioning tool. Check the deployed interpreter on both operating systems. A Bash shebang or a developer’s interactive shell can hide a portability defect. Look for array syntax, double-bracket conditionals, process substitution, here-strings, `source`, and reliance on `local`. Check sourced helpers for changes to positional parameters, options, traps, and working directory.

Audit the external utilities as well as the shell grammar. Compare the exact options and output relied on for BSD/macOS and GNU/Linux tools. Common examples are in-place `sed` editing, `stat` format flags, `date` arithmetic, and `readlink` canonicalization. Check path assumptions such as a preinstalled GNU utility or an executable under a package-manager prefix. For portable output, check whether `printf '%s\n' "$value"` preserves the intended bytes.

### SH2. Preserve arguments, paths, and data

Follow quoted parameters and `"$@"` to the final process. Check unquoted expansion, field splitting, pathname expansion, leading dashes, and empty arguments. For dynamic argument lists, consider positional parameters built with `set -- "$@" "$next"`. Test filenames containing spaces, a literal wildcard, and a newline. Where arbitrary filenames are supported, POSIX `find ... -exec utility {} +` passes each name as a separate argument. For line-oriented records, check `IFS= read -r`, an empty line, and a final line without a terminating newline. Check command substitution where trailing newlines carry meaning.

### SH3. Follow failures, subprocesses, and cleanup

Trace a failing subprocess to the script’s actual exit status. Inspect how `set -e` behaves in conditional tests and boolean lists. With the baseline shell, an ordinary pipeline normally reports the last command’s status. Check how the script detects an earlier stage failing. Treat `pipefail` as an additional interpreter requirement wherever correctness depends on it. A portable fix can run and check each required stage explicitly, using a private intermediate file when the operation permits it.

Check assignments made in piped loops for reliance on parent-shell side effects. When consuming a file, redirect it into the loop if the caller needs the loop’s updated state. For background work, trace `$!`, `wait`, and failure reporting. Inspect temporary-file creation, `umask`, traps, signal exits, and cleanup after a partial run. Verify that replacing a target preserves required permissions and that failure leaves its previous contents usable.

### SH4. Verify on both operating systems

On a macOS runner and a Linux runner, use the native `/bin/sh -n ./script.sh` on the actual script and each sourced file. Run `shellcheck -s sh ./script.sh` where available. Parsing and linting cover different defects from execution. Invoke the real entry point under each native `/bin/sh` in a temporary directory. Use an argument-recording stub on a controlled `PATH` to capture argv and inject one nonzero exit. Keep the platform’s real utilities in the path for BSD/GNU compatibility tests. Assert stdout, stderr, exit status, and final files. Record the OS, interpreter, and utility version or provenance. Mark an unavailable platform as unverified and give its exact check.

<!-- END POSIX SHELL MODULE -->
````

[POSIX sh](https://man7.org/linux/man-pages/man1/sh.1p.html) · [POSIX find](https://man7.org/linux/man-pages/man1/find.1p.html) · [Shell option portability](https://www.shellcheck.net/wiki/SC3040)
