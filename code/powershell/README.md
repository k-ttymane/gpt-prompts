# PowerShell audit module

Copy the block into a review that includes PowerShell.

````markdown
## Optional PowerShell module

<!-- BEGIN POWERSHELL MODULE -->

Apply these checks to PowerShell within the current review scope. Work read-only, cite the revision and file plus line or symbol, and check for a guard or platform setting that could disprove each claim. Preserve the main audit’s finding IDs and evidence labels when supplied. Give the smallest sound fix and one decisive check. Distinguish inspected source, observed output, and open questions.

### PS1. Identify the host and call contract

Follow the actual caller into the script, module, scheduled task, or remoting session. Record `$PSVersionTable`, edition, OS, architecture, module versions, invocation style, execution identity, and working directory. Test Windows PowerShell 5.1 and `pwsh` separately when both are supported. Inspect paths based on the caller’s directory, `$PSScriptRoot`, wildcard expansion versus `-LiteralPath`, and encoding or BOM behavior when another program consumes a file.

### PS2. Trace objects and arguments across the pipeline

Inspect parameter sets, pipeline binding by value or property name, and values supplied zero, one, or many times. Follow every object emitted on the success stream, including incidental output from helper functions. Look for a status string contaminating a stream of records, array enumeration changing the result shape, or `Format-Table` objects reaching export or serialization. For native programs, record the actual argument vector received for a space, an empty argument, and embedded quotes on the supported host. Check any applicable `$PSNativeCommandArgumentPassing` setting.

### PS3. Carry a failure through to the host

Distinguish terminating exceptions, non-terminating cmdlet errors, and native exit codes. Follow the actual `-ErrorAction` and `$ErrorActionPreference` settings into `try/catch/finally`. Check `$LASTEXITCODE` immediately after the native call, tool-specific successful nonzero codes, and the script’s final process status. Version-dependent native error preferences can change this path. Stub a native program to return a documented failure and assert both the caller’s status and whether a subsequent write occurs.

### PS4. Inspect mutations, remoting, and sensitive streams

For a cmdlet promising `-WhatIf`, trace every local and remote mutation to `$PSCmdlet.ShouldProcess(...)` or a verified cooperating command. Check that the guard precedes the side effect. Across remoting or out-of-process jobs, inspect which objects are serialized, which properties and methods reach the consumer, which scope supplies variables, and who receives job errors. Inspect transcripts, verbose output, exception messages, and child-process arguments for accidental credential disclosure. Check cleanup and location restoration on early return or cancellation.

### PS5. Verify the contract on the supported edition

For syntax, use `System.Management.Automation.Language.Parser.ParseFile` in the intended engine and inspect its returned parse errors. Use configured `Invoke-ScriptAnalyzer` rules and `Invoke-Pester` tests appropriate to that edition. Exercise a public command from a disposable module install or import it by its distributed manifest. Assert object types and counts, streams, process exit status, and side effects using synthetic paths and mocked remoting. Record which host and preferences produced the result.

<!-- END POWERSHELL MODULE -->
````

[Error handling](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_error_handling) · [Preference variables](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_preference_variables) · [Pipeline returns](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_return) · [ShouldProcess](https://learn.microsoft.com/en-us/powershell/scripting/learn/deep-dives/everything-about-shouldprocess) · [Remote output](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_remote_output) · [Pester](https://pester.dev/docs/commands/Invoke-Pester)
