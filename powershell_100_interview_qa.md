# Top 100 PowerShell Interview Questions
### For 7+ Years Experience — Senior/Lead Automation Engineer Level

---

## Section 1: Fundamentals & Core Concepts (Q1–Q15)

**Q1. What is PowerShell, and how is it fundamentally different from traditional shells like cmd.exe or bash?**
PowerShell is an object-oriented shell and scripting language built on .NET, where commands (cmdlets) pass structured .NET objects through the pipeline rather than plain text. This means you can access properties/methods directly on output (e.g., `Get-Process | Where-Object {$_.CPU -gt 100}`) instead of parsing text with regex, which is the core distinction from cmd.exe/bash's text-stream model.

**Q2. What is the difference between Windows PowerShell and PowerShell 7 (PowerShell Core)?**
Windows PowerShell (5.1 and earlier) is built on the .NET Framework, Windows-only, and ships in-box with Windows. PowerShell 7+ (Core) is built on .NET Core/.NET 5+, is cross-platform (Windows/Linux/macOS), open-source, side-by-side installable with Windows PowerShell, and includes performance improvements, ternary operators, pipeline chain operators (`&&`/`||`), and parallel `ForEach-Object -Parallel`.

**Q3. What is a cmdlet, and what naming convention do cmdlets follow?**
A cmdlet is a lightweight, single-purpose .NET command implementing the `Cmdlet` base class, following a strict `Verb-Noun` naming convention (e.g., `Get-Process`, `Set-Item`, `New-Object`) using an approved verb list (`Get-Verb`) to ensure predictable, discoverable command names across the entire ecosystem.

**Q4. Difference between a cmdlet, a function, and a script?**
A cmdlet is compiled .NET code (or, in modern PowerShell, can also be an "advanced function" behaving like one). A function is PowerShell code defined with the `function` keyword, either in a session or a module. A script is a `.ps1` file containing one or more statements/functions executed as a unit — functionally similar to a function but invoked by file path.

**Q5. What is the PowerShell pipeline, and how does object-based piping differ from text-based piping?**
The pipeline (`|`) passes the *output object* of one command as input to the next, preserving full object structure (properties, methods, type) rather than flattening to text — so `Get-Service | Where-Object Status -eq 'Running'` filters on the actual `Status` property object, with no text parsing involved, unlike Unix pipes which pass raw text.

**Q6. Explain `$_` (or `$PSItem`) and where it's used.**
`$_` (aliased as `$PSItem` in PS 3.0+) represents the current object being processed in the pipeline within `Where-Object`, `ForEach-Object`, `Switch`, and process blocks — e.g., `Get-Process | ForEach-Object { $_.Name }`.

**Q7. What are the three types of profile scripts, and when does each load?**
`$PROFILE` variables cover: AllUsersAllHosts, AllUsersCurrentHost, CurrentUserAllHosts, and CurrentUserCurrentHost — loaded in that order at shell startup depending on scope (all users vs current user) and host (any host vs the specific host like ISE or Windows Terminal), letting you customize environment/aliases/functions per user or per host application.

**Q8. What is the difference between `Write-Host`, `Write-Output`, `Write-Verbose`, and `Write-Error`?**
`Write-Host` writes directly to the console host (bypassing the pipeline — can't be captured/redirected into a variable or another cmdlet). `Write-Output` sends objects down the success pipeline, capturable and consumable by downstream commands. `Write-Verbose` writes diagnostic messages visible only with `-Verbose`. `Write-Error` writes to the error stream, which can trigger `-ErrorAction` behavior and terminate execution under `$ErrorActionPreference = 'Stop'`.

**Q9. What are the different output/data streams in PowerShell (1–6)?**
1=Success (Output), 2=Error, 3=Warning, 4=Verbose, 5=Debug, 6=Information — each can be individually redirected (e.g., `2>errors.txt`) or merged (e.g., `*>&1` redirects all streams into the success stream), giving fine-grained control over what gets logged vs displayed vs captured.

**Q10. What is the difference between `-WhatIf` and `-Confirm`?**
`-WhatIf` shows what a cmdlet *would* do without actually executing the change (a dry run). `-Confirm` prompts the user for explicit confirmation before executing each change. Both require the cmdlet to support `SupportsShouldProcess` in its definition, common in built-in state-changing cmdlets and easily added to custom advanced functions.

**Q11. Explain execution policy and its purpose. Is it a security boundary?**
Execution policy (`Get-ExecutionPolicy`/`Set-ExecutionPolicy`) controls whether/how `.ps1` scripts can run (Restricted, AllSigned, RemoteSigned, Unrestricted, Bypass). It is explicitly *not* a security boundary — Microsoft documents it as a convenience/guardrail against accidental execution, not a defense against a determined attacker, since it can be bypassed (`-ExecutionPolicy Bypass`, encoded commands, etc.) — real security relies on AppLocker, Constrained Language Mode, or Just Enough Administration (JEA).

**Q12. What is the difference between a PowerShell "alias," "function," and "cmdlet" from a resolution/precedence standpoint?**
When you type a command name, PowerShell resolves it in this order: alias → function → cmdlet → external executable (native command) in the current session — which is why a custom function or alias with the same name as a cmdlet will shadow it.

**Q13. What is `Tab` completion / IntelliSense powered by in PowerShell, and how do you extend it?**
PowerShell's parameter/argument completion is powered by the type system and cmdlet metadata (parameter sets, `ValidateSet` attributes); it can be extended with custom argument completers via `Register-ArgumentCompleter`, letting you provide dynamic tab-completion for custom parameters (e.g., completing resource names fetched from an API).

**Q14. What is the difference between `Get-Member` and `Get-Command`?**
`Get-Member` inspects an *object's* properties, methods, and type (e.g., `Get-Process | Get-Member`) — essential for discovering what you can do with pipeline output. `Get-Command` discovers available *commands* (cmdlets, functions, aliases, applications) in the session, optionally filtered by module or verb/noun.

**Q15. What is `Get-Help`, and how do you write help documentation for your own functions?**
`Get-Help` retrieves documentation for cmdlets/functions, either from built-in XML help or comment-based help. You add comment-based help directly above/inside a function using `<# .SYNOPSIS .DESCRIPTION .PARAMETER .EXAMPLE #>` blocks, which `Get-Help MyFunction -Full` then parses and displays exactly like built-in cmdlet help.

---

## Section 2: Pipeline & Objects (Q16–Q25)

**Q16. Difference between `Where-Object` and `.Where()` method / `Select-Object` and `.ForEach()` method?**
`Where-Object`/`ForEach-Object` are pipeline cmdlets, more memory-friendly for large/streamed collections since they process one object at a time. The `.Where()`/`.ForEach()` *methods* (available on collections in PS 4.0+) operate on an already-in-memory array and can be faster for smaller in-memory collections since they avoid pipeline overhead, and `.Where()` supports modes like `'First'`, `'Last'`, `'SkipUntil'`.

**Q17. What is the difference between `Select-Object -Property` and `Select-Object -ExpandProperty`?**
`-Property` returns a new `PSCustomObject` wrapping the selected property/properties (preserving object structure, useful when selecting multiple properties). `-ExpandProperty` returns the *raw value* of a single property directly (unwrapped), useful when you want just the string/int value itself rather than an object containing it.

**Q18. How does `Sort-Object`, `Group-Object`, and `Measure-Object` work together for data analysis in a pipeline?**
`Sort-Object` orders objects by property; `Group-Object` buckets objects into groups sharing a property value (returning `Count` and `Group` per bucket); `Measure-Object` computes aggregate statistics (Sum, Average, Minimum, Maximum, Count) over a property — commonly chained, e.g., `Get-Process | Group-Object Company | Sort-Object Count -Descending`.

**Q19. What is `[PSCustomObject]`, and why is it preferred over `New-Object PSObject -Property @{}` in modern scripts?**
`[PSCustomObject]@{Name='x'; Value=1}` is a fast, ordered-property syntax (PS 3.0+) for creating custom objects, preserving the property order you define. The older `New-Object PSObject -Property @{}` approach used a hashtable internally, which doesn't guarantee property order (hashtables are unordered by default) — a common source of inconsistent `Format-Table`/CSV output in older scripts.

**Q20. How do you convert PowerShell objects to/from JSON, CSV, and XML?**
`ConvertTo-Json`/`ConvertFrom-Json` for JSON; `Export-Csv`/`Import-Csv` for CSV (note: `Export-Csv` stringifies all properties, losing type fidelity, and nested objects show as `System.Object[]` unless flattened first); `Export-Clixml`/`Import-Clixml` for full-fidelity XML serialization that *does* preserve .NET object type information, unlike CSV/JSON.

**Q21. What's the difference between `Export-Clixml` and `ConvertTo-Json` for persisting complex objects?**
`Export-Clixml` uses PowerShell's native XML serialization, preserving full object type fidelity (including nested/complex .NET objects and, notably, can securely serialize `SecureString` in a way tied to the user/machine via DPAPI). `ConvertTo-Json` produces a more universally portable, human-readable, cross-platform format but flattens objects to plain data (loses .NET type info and has a default depth limit of 2, requiring `-Depth` for nested structures).

**Q22. How do you flatten/unroll a nested object property before exporting to CSV?**
Manually reconstruct a flat `[PSCustomObject]` selecting/renaming nested properties (e.g., `Select-Object Name, @{N='IP';E={$_.NetworkSettings.IPAddress}}`), since `Export-Csv` doesn't recurse into nested objects/arrays automatically — this calculated-property pattern (`@{Name=...; Expression=...}`) is one of the most common real-world `Select-Object` techniques.

**Q23. What is the pipeline chain operator (`&&`/`||`), introduced in PowerShell 7?**
`command1 && command2` runs `command2` only if `command1` succeeds (exit code 0/no terminating error); `command1 || command2` runs `command2` only if `command1` fails — bringing bash-like conditional chaining natively into PowerShell 7+ without needing `if ($?) {...}`.

**Q24. How does `ForEach-Object -Parallel` work, and what are its limitations?**
Introduced in PowerShell 7.0, it runs pipeline iterations concurrently across separate runspaces (with a `-ThrottleLimit` controlling max concurrency), useful for I/O-bound tasks like parallel API calls. Limitations: each parallel iteration runs in an isolated runspace, so it doesn't share variables/functions from the parent scope by default (must pass via `$using:variable`), and debugging/error-aggregation is harder than sequential `ForEach-Object`.

**Q25. What is the difference between `begin`, `process`, and `end` blocks in a pipeline-aware function?**
`begin` runs once before any pipeline input is received (setup); `process` runs once *per pipeline object* received (the core per-item logic); `end` runs once after all pipeline input has been processed (cleanup/summary) — this structure is what makes a function properly "pipeline-aware" rather than only accepting a single upfront input.

---

## Section 3: Variables, Data Types & Operators (Q26–Q35)

**Q26. What is the difference between `$null`, `[string]::Empty`, and an empty array `@()`?**
`$null` represents "nothing"/absence of a value; `[string]::Empty` (or `""`) is an empty but *existing* string; `@()` is an empty array with zero elements. Comparisons behave subtly differently — e.g., `$null -eq @()` is `$false`, and counting an empty array (`(@()).Count`) is `0`, not an error, which matters when checking for "no results" from a command.

**Q27. Explain PowerShell's comparison operators: `-eq`, `-ne`, `-like`, `-match`, `-contains`, `-in`.**
`-eq`/`-ne` do equality/inequality comparison; `-like` does wildcard pattern matching (`*`, `?`); `-match` does regex matching (and populates `$Matches`); `-contains` checks if a *collection* contains a value (returns boolean, doesn't tell you where); `-in` is the reverse syntax of `-contains` (`$value -in $collection`).

**Q28. Why are PowerShell comparison operators case-insensitive by default, and how do you force case-sensitivity?**
By default, `-eq`, `-like`, `-match`, etc. are case-insensitive for usability. Prefixing with `c` (e.g., `-ceq`, `-clike`, `-cmatch`) forces case-sensitive comparison, and prefixing with `i` (e.g., `-ieq`) explicitly makes case-insensitivity explicit (same as default, but self-documenting).

**Q29. What is the difference between `=`, `-eq`, and `==`? Is `==` valid PowerShell?**
`=` is assignment; `-eq` is the equality comparison operator; `==` is *not* valid PowerShell syntax (it's a C-style operator) — a very common mistake for engineers coming from C#/Java/JavaScript, since PowerShell uses word-based operators (`-eq`, `-ne`, `-gt`, etc.) rather than symbolic ones for comparisons.

**Q30. What are `[ValidateSet]`, `[ValidateRange]`, `[ValidateNotNullOrEmpty]`, and other parameter validation attributes used for?**
They're declarative parameter validation attributes applied to advanced function parameters, enforcing constraints (allowed values, numeric range, non-null/non-empty) *before* the function body executes, producing a clear, immediate error instead of failing deep inside the function logic — a key building block of robust, reusable functions.

**Q31. Explain type casting/coercion in PowerShell, and how `[int]"5" + 3` differs from `"5" + 3`.**
`[int]"5" + 3` explicitly casts the string to an integer first, yielding `8` (integer addition). `"5" + 3` — because the *left* operand is a string — coerces the whole expression to string concatenation, yielding `"53"`; PowerShell's dynamic typing means operand order and type coercion rules matter and are a common source of subtle bugs.

**Q32. What is a hashtable, and how do you iterate over its keys/values in order?**
A hashtable (`@{Key='Value'}`) is an unordered key-value collection by default; use `[ordered]@{}` to create an *ordered* hashtable that preserves insertion order. Iterate with `foreach ($key in $hashtable.Keys) { $hashtable[$key] }` or `$hashtable.GetEnumerator()` for key/value pairs together.

**Q33. What's the difference between a PowerShell "splat" (`@params`) and passing parameters individually?**
Splatting (`$params = @{Name='x'; Path='y'}; Get-Item @params`) passes an entire hashtable (or array for positional params) as a set of named parameters, dramatically improving readability for commands with many parameters and enabling conditional parameter building (adding keys to the hashtable only when needed) rather than constructing long, branchy command-line strings.

**Q34. What are automatic variables like `$PSVersionTable`, `$Error`, `$Host`, and `$MyInvocation` used for?**
`$PSVersionTable` reports PowerShell/CLR/OS version info; `$Error` is an array of recent errors (most recent at index 0) accessible for post-mortem debugging; `$Host` provides info about the current host application (console, ISE, VS Code); `$MyInvocation` gives metadata about how the current script/function was invoked (useful for logging or self-referential path resolution).

**Q35. What is the difference between a script-scoped, global-scoped, and local-scoped variable, and how do you explicitly target scope?**
By default, variables are scoped to the current function/script block (local). `$script:var` explicitly targets script scope (persists across function calls within the same script), `$global:var` targets the session-wide global scope, and `$using:var` is used specifically to pull a caller's variable into a remote/parallel/job scope that otherwise wouldn't have access to it.

---

## Section 4: Functions, Scripts & Modules (Q36–Q47)

**Q36. What is an "advanced function," and what makes it different from a basic function?**
An advanced function uses `[CmdletBinding()]` and `param()` with attributes, giving it cmdlet-like behavior: common parameters (`-Verbose`, `-Debug`, `-ErrorAction`), pipeline input support (`ValueFromPipeline`), `-WhatIf`/`-Confirm` support (with `SupportsShouldProcess`), and proper parameter validation — essentially making a script function indistinguishable from a compiled cmdlet to the caller.

**Q37. How do you accept pipeline input in a custom function?**
Decorate a parameter with `[Parameter(ValueFromPipeline=$true)]` (for whole-object binding) or `[Parameter(ValueFromPipelineByPropertyName=$true)]` (binding by matching property name), and implement a `process {}` block so the function executes once per incoming pipeline object rather than only handling the first one.

**Q38. What is a PowerShell module, and what are the different types (script module, binary module, manifest module)?**
A module (`.psm1`) packages related functions/variables/aliases for reuse and distribution. A script module is pure PowerShell code; a binary module is a compiled .NET assembly (DLL) exposing cmdlets; a manifest module (`.psd1`) is metadata describing a module (version, dependencies, exported members, required PowerShell version) that can accompany either type, or represent a "manifest-only" module referencing multiple nested modules.

**Q39. What is a module manifest (`.psd1`), and what key fields does it typically define?**
A metadata file describing a module: `ModuleVersion`, `Author`, `RequiredModules`/`RequiredAssemblies` (dependencies), `FunctionsToExport`/`CmdletsToExport` (controls what's publicly visible vs internal), `PowerShellVersion` (minimum supported version), and `NestedModules` — critical for versioned distribution via the PowerShell Gallery.

**Q40. How do you control which functions in a module are exported/public vs internal/private?**
Use `Export-ModuleMember -Function PublicFunc1, PublicFunc2` at the end of the `.psm1` (or, more robustly, set `FunctionsToExport` explicitly in the `.psd1` manifest rather than using `'*'`), keeping helper functions private by simply not exporting them — this is both a cleanliness practice and, for large modules, a meaningful startup-performance optimization since wildcard exports force PowerShell to enumerate everything.

**Q41. How do you publish and install modules via the PowerShell Gallery?**
Publish with `Publish-Module -Path .\MyModule -NuGetApiKey $key` (requires a manifest and a PowerShell Gallery account/API key); install with `Install-Module -Name MyModule -Scope CurrentUser` (or `AllUsers` with elevation); `Find-Module` searches the Gallery before installing.

**Q42. What is the difference between `Import-Module`, `#Requires -Module`, and module auto-loading?**
`Import-Module` explicitly loads a module into the session. `#Requires -Module ModuleName` is a script-level directive that causes PowerShell to error out immediately if the required module isn't available, useful for fail-fast dependency checking. Module auto-loading (default since PS 3.0) automatically imports a module the first time one of its commands is invoked, without an explicit `Import-Module` call, as long as it's discoverable on `$env:PSModulePath`.

**Q43. What is `$PSModulePath`, and how does PowerShell decide which version of a module to load if multiple are installed?**
`$PSModulePath` is a semicolon/colon-separated list of directories PowerShell searches for modules (user-specific, system-wide, and program-specific paths). When multiple versions exist in different version-numbered subfolders, PowerShell loads the *highest* version by default unless `Import-Module -RequiredVersion` or `-MaximumVersion` pins a specific one.

**Q44. How do you write a function/module compatible with both Windows PowerShell 5.1 and PowerShell 7+?**
Avoid Windows-only .NET APIs and cmdlets (e.g., WMI-specific `Get-WmiObject` — use the cross-platform `Get-CimInstance` instead), test with `$PSVersionTable.PSEdition` (`Desktop` vs `Core`) for conditional logic where genuinely needed, declare `CompatiblePSEditions` in the manifest, and avoid assuming Windows-style path separators when the module might run on Linux/macOS under PS 7.

**Q45. What is the difference between dot-sourcing a script (`. .\script.ps1`) and calling it normally (`.\script.ps1`)?**
Dot-sourcing runs the script in the *current* scope, so any functions/variables it defines become available in your current session afterward — commonly used to load a library of functions. Calling it normally runs it in a new child scope, and anything it defines is discarded once it completes, isolated from the caller.

**Q46. How do you handle default parameter values that depend on other parameters or environment context?**
Use a parameter default value expression referencing `$env:` variables or another already-bound parameter (evaluated in parameter order), or more robustly, leave the parameter `$null`/unset and resolve the effective value inside `begin {}` using logic that can reference multiple inputs together — the latter avoids PowerShell's parameter-default evaluation-order pitfalls.

**Q47. What is `$PSDefaultParameterValues`, and how is it useful in enterprise scripting?**
A hashtable letting you set default parameter values globally across cmdlets in a session (e.g., `$PSDefaultParameterValues['*:Verbose'] = $true` or `$PSDefaultParameterValues['Connect-AzAccount:Tenant'] = $tenantId`), reducing repetition and enforcing consistent defaults (like always specifying a particular subscription or credential) across a whole automation script or profile.

---

## Section 5: Error Handling & Debugging (Q48–Q57)

**Q48. What is the difference between a terminating and non-terminating error in PowerShell?**
A non-terminating error is reported (added to `$Error`, written to the error stream) but execution continues to the next statement/pipeline item by default. A terminating error stops execution of the current command/script entirely (unless caught) — many cmdlets produce non-terminating errors by default *unless* `-ErrorAction Stop` is specified, which is a critical distinction for `try/catch` to actually catch them.

**Q49. Why doesn't `try/catch` always catch errors from cmdlets, and how do you fix it?**
`try/catch` only catches *terminating* errors. Since many cmdlets raise non-terminating errors by default, you must force them to terminate with `-ErrorAction Stop` on the specific cmdlet call (or set `$ErrorActionPreference = 'Stop'` more broadly) for `try/catch` to actually intercept the failure.

**Q50. What is `$ErrorActionPreference`, and what are its common values?**
A preference variable controlling default error-handling behavior across the session/script for non-terminating errors: `Continue` (default — show error, keep going), `Stop` (convert to terminating, usable with try/catch), `SilentlyContinue` (suppress display but still logs to `$Error`), and `Inquire` (prompt the user).

**Q51. What is the difference between `-ErrorAction SilentlyContinue` and `-ErrorVariable`?**
`-ErrorAction SilentlyContinue` suppresses an error from being displayed/thrown but the error is still recorded in `$Error`. `-ErrorVariable myErr` captures errors from that specific command into a named variable (without clearing pre-existing errors, unless prefixed with `+` to append) — useful for scoped error inspection without relying on the global `$Error` stack.

**Q52. How do you use `try/catch/finally` with specific exception types?**
`catch [System.IO.FileNotFoundException] { ... } catch [System.UnauthorizedAccessException] { ... } catch { # generic fallback } finally { # always runs, e.g., cleanup }` — ordering matters, as PowerShell evaluates `catch` blocks top-down and uses the first type-matching (or type-inheriting) block.

**Q53. What is `$_.Exception` inside a catch block, and what other useful properties does the caught error record expose?**
`$_` (or `$Error[0]`) inside `catch` is an `ErrorRecord`; `$_.Exception.Message` gives the underlying .NET exception message, `$_.Exception.GetType().FullName` gives the exception type, `$_.InvocationInfo.ScriptLineNumber`/`.Line` gives where the error occurred, and `$_.ScriptStackTrace` shows the call stack — all essential for meaningful logging in production automation.

**Q54. How do you re-throw an exception after partially handling it in a catch block?**
Use `throw` (with no arguments) inside the `catch` block to re-throw the *original* exception with its full stack trace intact, rather than `throw $_.Exception.Message` which creates a *new* exception and loses the original context/stack trace.

**Q55. What is `Set-StrictMode`, and why enable it in scripts?**
`Set-StrictMode -Version Latest` enforces stricter coding rules — e.g., referencing an uninitialized variable or a non-existent property throws an error instead of silently returning `$null` — catching typos and logic bugs early that would otherwise fail silently or produce confusing downstream errors.

**Q56. How do you debug a PowerShell script step-by-step?**
Use breakpoints (`Set-PSBreakpoint -Script script.ps1 -Line 10`, or `-Command`/`-Variable` breakpoints), then run the script and use the debugger commands (`s` step into, `v` step over, `c` continue) in the console, or use an IDE debugger (VS Code with the PowerShell extension) for a visual breakpoint/watch/call-stack experience — far more productive than sprinkling `Write-Host` for non-trivial scripts.

**Q57. What is the difference between `Write-Debug` and using the interactive debugger, and how does `$DebugPreference` affect it?**
`Write-Debug` emits messages to the debug stream, visible only when `-Debug` is used or `$DebugPreference` isn't `SilentlyContinue`; critically, `$DebugPreference = 'Inquire'` (the classic default behavior with `-Debug`) will actually *pause execution* at each `Write-Debug` call and prompt for action, effectively turning debug messages into ad-hoc breakpoints without a formal debugger session.

---

## Section 6: Remoting & Sessions (Q58–Q65)

**Q58. What is PowerShell Remoting, and what protocol does it use by default?**
PowerShell Remoting executes commands on remote machines using WS-Management (WinRM) by default on Windows (port 5985 HTTP / 5986 HTTPS), enabled via `Enable-PSRemoting`. PowerShell 7+ also supports SSH-based remoting (cross-platform, no WinRM dependency) as an alternative transport.

**Q59. Difference between `Invoke-Command`, `Enter-PSSession`, and a persistent `New-PSSession`?**
`Invoke-Command` runs a script block on one or more remote computers and returns, non-interactively (great for fan-out automation across many hosts). `Enter-PSSession` opens an *interactive* remote session against a single computer (like SSH-ing in). `New-PSSession` creates a *persistent, reusable* session object that retains state (variables, imported modules) across multiple subsequent `Invoke-Command -Session` calls, avoiding the overhead of establishing a new connection each time.

**Q60. What is `Implicit Remoting`, and when is it useful?**
Using `Import-PSSession`, you can import commands from a remote session into your local session as local proxy functions — calling them locally actually executes them remotely and returns the results, letting you use a remote server's specific module (e.g., Exchange Server management tools) without installing that module locally.

**Q61. How do you pass local variables into a remote `Invoke-Command` script block?**
Use the `$using:` scope modifier inside the remote script block (e.g., `Invoke-Command -ComputerName srv1 -ScriptBlock { Get-Service -Name $using:serviceName }`), which serializes the local variable's value and makes it available in the remote scope — without it, the remote session has no access to local session variables.

**Q62. What is CredSSP, and why/when would you need it in remoting scenarios?**
CredSSP (Credential Security Support Provider) is an authentication mechanism that allows credentials to be *delegated* through a remote session to a second hop (e.g., running a command on Server A that needs to access a file share on Server B using your credentials) — solving the classic PowerShell Remoting "double-hop" problem, though it has security trade-offs (credentials are passed to the intermediate server) so Kerberos delegation or CredSSP alternatives (like registering a `PSSessionConfiguration` with `-RunAs`) are often preferred where possible.

**Q63. What is Just Enough Administration (JEA), and what problem does it solve?**
JEA lets you create a constrained PowerShell endpoint where specific users/groups can connect and run only a pre-approved, role-scoped set of cmdlets/functions (via a Role Capability file and Session Configuration file) — running with a virtual account that has only the privileges the task needs, rather than granting full administrative remoting access, significantly reducing the attack surface for delegated admin tasks.

**Q64. How do you troubleshoot a failed `Invoke-Command`/remoting connection?**
Test basic connectivity/WinRM listener with `Test-WSMan -ComputerName target`, verify the WinRM service is running and firewall allows the port, check `TrustedHosts` (`winrm get winrm/config/client`) if not domain-joined/using Kerberos, and verify the target machine's `Get-PSSessionConfiguration` isn't restricting access — authentication (Kerberos vs NTLM vs CredSSP) mismatches are the most common root cause.

**Q65. How would you run the same script against 500 remote servers efficiently, and what should you consider for scale?**
`Invoke-Command -ComputerName $serverList -ScriptBlock {...} -ThrottleLimit 32` runs in parallel across servers up to the throttle limit (default 32) using fan-out remoting rather than looping serially; for very large fleets, consider batching, using `-AsJob` for asynchronous fire-and-collect patterns, robust per-server error capture (so one unreachable server doesn't abort the whole run), and centralized logging (e.g., writing structured results to a shared location) to aggregate results afterward.

---

## Section 7: WMI/CIM & System Management (Q66–Q72)

**Q66. What is the difference between `Get-WmiObject` and `Get-CimInstance`?**
`Get-WmiObject` (legacy, Windows PowerShell only, removed in PowerShell 7) uses DCOM for remote connections and doesn't support PowerShell Remoting's session model. `Get-CimInstance` (the modern replacement) uses WS-Management (or DCOM as a fallback) via `CimSession` objects, works cross-platform in PowerShell 7, and is generally more firewall-friendly/consistent with the rest of PowerShell's remoting infrastructure — `Get-CimInstance` is the recommended approach in all new scripts.

**Q67. How do you query specific WMI/CIM classes, and how do you discover available classes?**
`Get-CimInstance -ClassName Win32_OperatingSystem` (or `Win32_LogicalDisk`, `Win32_BIOS`, etc.) queries a class directly; `Get-CimClass -ClassName Win32_*` lists available classes matching a pattern for discovery when you're not sure of the exact class name for a given piece of system data.

**Q68. How do you invoke a WMI/CIM method (not just query properties) — e.g., forcing a process to terminate or a service to restart via CIM?**
`Invoke-CimMethod -ClassName Win32_Process -MethodName Create -Arguments @{CommandLine='notepad.exe'}` — `Invoke-CimMethod` calls a class or instance method with named arguments, used for actions WMI/CIM classes expose beyond simple property retrieval (starting processes, restarting services via `Win32_Service`, etc.).

**Q69. How do you register for and respond to WMI events (e.g., detect when a new process starts)?**
Using `Register-CimIndicationEvent` (modern) or the older `Register-WmiEvent`, subscribing to an event query (e.g., `SELECT * FROM __InstanceCreationEvent WITHIN 5 WHERE TargetInstance ISA 'Win32_Process'`), then handling matched events via an `-Action` script block or by polling `Get-Event` — used for lightweight real-time system monitoring without a full monitoring agent.

**Q70. How do you query remote systems' hardware/OS info at scale using CIM, and what's the performance consideration?**
Create reusable `CimSession` objects (`New-CimSession -ComputerName $list`) once and pass them to multiple `Get-CimInstance -CimSession $sessions` calls, rather than letting each call implicitly create/tear down a new connection — session reuse significantly reduces overhead when querying many classes across many machines.

**Q71. What is `Get-CimInstance -Filter` vs. piping to `Where-Object`, and why does it matter for remote/large queries?**
`-Filter` (using WQL syntax) pushes the filtering *down to the CIM provider/server side*, returning only matching instances over the wire. Piping unfiltered results to `Where-Object` retrieves *all* instances first, then filters client-side — for large datasets or slow network links, server-side filtering is significantly more efficient.

**Q72. How would you inventory installed software/patches across a domain using PowerShell?**
Query `Win32_Product` (slow, and its query has the side effect of triggering Windows Installer reconfiguration — generally discouraged) or better, `Get-CimInstance Win32_QuickFixEngineering` for hotfixes, and registry-based uninstall key enumeration (`HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall`) for a faster, non-invasive software inventory — a well-known gotcha at this experience level is knowing to *avoid* `Win32_Product` for that exact reason.

---

## Section 8: Security, Execution Policy & Credentials (Q73–Q80)

**Q73. How do you securely store and retrieve credentials for use in unattended/scheduled scripts?**
`Get-Credential` prompts interactively (not viable for unattended scripts); for automation, use `ConvertTo-SecureString`/`Export-Clixml` to save an encrypted credential (encrypted via Windows DPAPI, tied to the user+machine that created it, so it's not portable across machines/users) or, in an enterprise setting, integrate with a proper secrets manager (Azure Key Vault, CyberArk, HashiCorp Vault) rather than storing credentials on disk at all.

**Q74. What is a `SecureString`, and what are its real-world limitations?**
`SecureString` encrypts a string's contents in memory to reduce exposure to casual memory inspection, but it is *not* a strong security guarantee — it can still be decrypted by the same user/process context, doesn't protect against a privileged attacker with process memory access, and (notably) PowerShell 7 on non-Windows platforms doesn't actually encrypt `SecureString` in memory the same way Windows does, since there's no DPAPI equivalent.

**Q75. What is `PSCredential`, and how do you construct one programmatically (e.g., in CI/CD) without interactive prompting?**
`[PSCredential]::new($username, $securePassword)` constructs a credential object directly; `$securePassword` is typically built via `ConvertTo-SecureString $plainTextFromVault -AsPlainText -Force`, where the plaintext is pulled from a secure source (pipeline secret variable, Key Vault) at runtime rather than hardcoded in the script.

**Q76. What is Constrained Language Mode, and how does it relate to script security?**
A restricted PowerShell language mode (often enforced via AppLocker/WDAC policies) that disables access to .NET types/methods, COM objects, and other capabilities that could be used to bypass security controls, while still allowing core scripting — used to reduce the attack surface for scripts running in less-trusted contexts.

**Q77. How do you sign a PowerShell script, and why would `AllSigned` execution policy be used?**
`Set-AuthenticodeSignature -FilePath script.ps1 -Certificate $cert` signs a script using a code-signing certificate. Under `AllSigned` execution policy, only scripts signed by a trusted publisher can run at all (even locally created ones), providing tamper-evidence and provenance assurance in security-conscious environments — though as noted in Q11, this is a policy control, not an absolute security barrier.

**Q78. How would you audit what PowerShell commands were run on a system after a security incident?**
Enable and review PowerShell's transcription (`Start-Transcript` or Group Policy-enforced transcription) and Module/Script Block Logging (which logs full de-obfuscated script block content to the Windows Event Log, Event ID 4104), both of which are critical for forensic visibility since a bare command-line history alone won't capture in-memory or obfuscated script execution.

**Q79. What is the risk of using `Invoke-Expression` (`iex`), and when (if ever) is it acceptable?**
`Invoke-Expression` executes a string as PowerShell code, which is a major injection risk if any part of that string can be influenced by untrusted/external input (classic attack vector, e.g., used by malicious "one-liner" downloaders). It's best avoided entirely in production scripts in favor of proper cmdlets/parameters; if truly unavoidable (rare), the input must be rigorously validated/sanitized and never derived directly from user or network input.

**Q80. How do you restrict what commands a delegated admin can run remotely, short of full JEA implementation?**
Create a custom `PSSessionConfiguration` (`Register-PSSessionConfiguration`) with a restricted `-StartupScript` or `-RunAsCredential`, or use role-based access control at the target resource level (e.g., Azure RBAC for cloud resources) combined with logging — though JEA (Q63) remains the purpose-built, most maintainable solution for granular command-level restriction.

---

## Section 9: Desired State Configuration - DSC (Q81–Q85)

**Q81. What is PowerShell Desired State Configuration (DSC), and what problem does it solve?**
DSC is a declarative management platform where you define the *desired end state* of a system (e.g., "this Windows feature must be installed," "this file must have this content") in a `Configuration` block, and the DSC engine (Local Configuration Manager) applies and continuously enforces that state — conceptually similar to Ansible/Puppet/Chef but native to Windows/PowerShell.

**Q82. What is the difference between a DSC Configuration, a Resource, and a MOF file?**
A Configuration is the PowerShell script block defining desired state using one or more Resources. A Resource (e.g., `File`, `Service`, `WindowsFeature`, or custom resources) is the reusable unit implementing the actual get/set/test logic for a specific type of system state. Compiling a Configuration produces a MOF (Managed Object Format) file, the standardized, engine-consumable representation of that desired state that gets applied to target nodes.

**Q83. What are the three DSC configuration management approaches (Push, Pull, Azure Automation DSC)?**
Push mode manually sends the MOF to target nodes via `Start-DscConfiguration` (simple, but doesn't scale well and requires manual re-push for drift correction). Pull mode has nodes periodically check in with a central Pull Server to retrieve and apply their configuration automatically. Azure Automation State Configuration (Azure Automation DSC) is a managed, cloud-hosted pull server experience, removing the need to stand up/maintain your own pull server infrastructure.

**Q84. How do you check for and remediate configuration drift with DSC?**
`Test-DscConfiguration` compares the current system state against the last-applied MOF and reports compliance (True/False, with `-Detailed` showing which resources are out of compliance); combined with Pull mode's periodic consistency checks (or Azure Automation DSC's built-in reporting), drift is automatically detected and, depending on the `RefreshMode`/`ConfigurationMode` (`ApplyAndAutoCorrect`), automatically remediated.

**Q85. How do you write a custom DSC Resource?**
Implement a class-based DSC Resource (PS 5.0+) using `[DscResource()]` on a class with `Get()`, `Set()`, and `Test()` methods (returning current state, applying desired state, and checking compliance respectively), packaged inside a module with the appropriate `DscResourcesToExport` manifest entry — the modern replacement for the older, more verbose MOF-based resource authoring approach.

---

## Section 10: Azure PowerShell & Cloud Automation (Q86–Q93)

**Q86. What is the difference between the `Az` module and the older `AzureRM` module?**
`AzureRM` is Microsoft's original (now deprecated/retired) Azure Resource Manager module. `Az` is its modern, actively maintained successor — cross-platform (works on PowerShell 7/Core), with a more consistent cmdlet naming/parameter design and improved performance; `Az` was designed to eventually fully replace `AzureRM`, and Microsoft has since retired `AzureRM` entirely.

**Q87. How do you authenticate to Azure in a PowerShell script for unattended automation (e.g., a scheduled runbook)?**
Use a Service Principal with `Connect-AzAccount -ServicePrincipal -Credential $cred -Tenant $tenantId`, or preferably a Managed Identity when running from Azure infrastructure (`Connect-AzAccount -Identity`) which eliminates the need to manage/store a secret at all — interactive `Connect-AzAccount` (device code/browser login) isn't viable for unattended scenarios.

**Q88. How do you manage multiple Azure subscriptions in a single script?**
`Get-AzSubscription` lists accessible subscriptions; `Set-AzContext -SubscriptionId $id` (or `-Subscription $name`) switches the active context for subsequent cmdlets; for scripts operating across many subscriptions, loop over `Get-AzSubscription` and call `Set-AzContext` per iteration, or use `-DefaultProfile` on individual cmdlets to target a specific context without mutating the global one.

**Q89. What is an Azure Automation Runbook, and what PowerShell-specific considerations apply when writing one?**
A Runbook is a script (PowerShell, PowerShell Workflow, or Python) hosted and scheduled within Azure Automation, running in a sandboxed worker without needing your own infrastructure. Considerations include: using `Get-AutomationPSCredential`/`Get-AutomationVariable` to retrieve stored assets rather than hardcoding secrets, being mindful of the sandbox's module version constraints (Azure Automation-hosted modules can lag behind the latest `Az` releases), and understanding fair-share/job-slot limits that can throttle highly parallel runbook execution.

**Q90. How do you idempotently ensure an Azure resource exists (create-if-not-exists pattern) in PowerShell?**
`$rg = Get-AzResourceGroup -Name $name -ErrorAction SilentlyContinue; if (-not $rg) { New-AzResourceGroup -Name $name -Location $location }` — checking existence first (silencing the expected "not found" error) before creating avoids unnecessary errors/duplicate-resource conflicts on re-runs, mirroring the idempotency principle from configuration management tools.

**Q91. How would you use PowerShell to deploy an ARM template or Bicep file, and how do you pass parameters?**
`New-AzResourceGroupDeployment -ResourceGroupName $rg -TemplateFile ./main.bicep -TemplateParameterFile ./params.json` (Bicep is auto-compiled to ARM JSON at deployment time if the Az CLI/PowerShell tooling supports it locally), or pass parameters inline via `-TemplateParameterObject @{ vmSize = 'Standard_D2s_v3' }` for dynamic, script-computed parameter values rather than a static file.

**Q92. How do you handle Azure API throttling (429 errors) in a PowerShell automation script processing many resources?**
Implement retry logic with exponential backoff around cmdlet calls likely to be throttled (wrapping in a `try/catch` loop that inspects the exception for a 429/`TooManyRequests` status and sleeps before retrying), and where possible batch operations or reduce parallelism (`-ThrottleLimit`) to stay under subscription-level API rate limits in the first place.

**Q93. How do you write PowerShell automation that's portable between Azure PowerShell (`Az`) and Azure CLI, and when would you choose one over the other?**
For pure PowerShell object-pipeline workflows integrating with other PowerShell logic/modules, `Az` is more natural; Azure CLI (`az` commands, often called from PowerShell via native command invocation with JSON output parsed by `ConvertFrom-Json`) is sometimes preferred for cross-tooling consistency (same commands work identically in bash/CI pipelines) or when a specific feature is available in CLI before it lands in `Az`. Mature teams often standardize on one to avoid maintaining two credential/context models side by side.

---

## Section 11: Testing (Pester) & Advanced Best Practices (Q94–Q100)

**Q94. What is Pester, and what is it used for?**
Pester is PowerShell's native unit/integration testing framework, using a `Describe`/`Context`/`It` BDD-style syntax with `Should` assertions (e.g., `$result | Should -Be 5`), commonly integrated into CI pipelines to validate scripts/modules/DSC configurations before deployment.

**Q95. What is the difference between `Mock` in Pester and actually calling a real cmdlet during a test?**
`Mock` replaces a command's real implementation with a test double during test execution (e.g., `Mock Get-Service { return @{Status='Running'} }`), letting you test your function's logic in isolation without depending on real system state, external APIs, or side effects — essential for fast, deterministic, repeatable unit tests.

**Q96. How do you structure Pester tests for a module with multiple functions?**
Typically one `.Tests.ps1` file per function (or per logical grouping) alongside the module source, using `BeforeAll`/`BeforeEach` to import the module and set up fixtures, `Describe` blocks per function, and `Context` blocks for different scenarios (valid input, invalid input, edge cases) within each — run collectively via `Invoke-Pester` in CI with code coverage reporting (`-CodeCoverage`).

**Q97. What is `PSScriptAnalyzer`, and how is it used in a CI pipeline?**
A static analysis linter for PowerShell (analogous to `shellcheck` for bash) that flags style violations, potential bugs (e.g., unused variables, `Write-Host` misuse, missing `[CmdletBinding()]`), and security issues (e.g., use of `Invoke-Expression`, plaintext credentials) — typically run via `Invoke-ScriptAnalyzer` as a CI gate before merge, often alongside custom rule sets tailored to organizational standards.

**Q98. How do you optimize a PowerShell script that's slow when processing large datasets (e.g., thousands of AD users or CSV rows)?**
Avoid `+=` on arrays in a loop (each append recreates the entire array — O(n²) behavior); instead use a `[System.Collections.Generic.List[object]]` with `.Add()`, or output objects directly to the pipeline and collect once at the end; prefer server-side filtering (LDAP filters for AD, `-Filter` for CIM) over retrieving everything and filtering client-side; and avoid unnecessary `Format-*` cmdlets mid-pipeline, since they convert objects to display-only format objects that break further pipeline processing.

**Q99. How do you handle secrets/config differences across environments (dev/test/prod) in a shared PowerShell automation codebase?**
Externalize environment-specific values into separate configuration files/data (`.psd1` data files or environment variables) loaded based on a `-Environment` parameter or `$env:` variable, keep secrets out of source control entirely via a secrets manager (Azure Key Vault via `Get-AzKeyVaultSecret`), and structure scripts so environment context is passed explicitly through parameters rather than relying on ambient global state that's easy to mix up between environments.

**Q100. As a senior/lead engineer, how do you decide between PowerShell and another automation tool (Ansible, Python, Terraform) for a given task?**
PowerShell excels at deep Windows/AD/Exchange/Microsoft 365/Azure-native object manipulation where its tight .NET/Windows API integration gives it capabilities other tools reach for cmdlets or REST wrappers to approximate; Ansible/Terraform are typically better for cross-platform, declarative infrastructure state and orchestration across heterogeneous estates (Linux + Windows + cloud); Python often wins for complex data processing, third-party API integration, or long-lived services. In practice, mature environments run multiple tools together (e.g., Terraform provisions Azure infra, PowerShell DSC/Ansible configures Windows guests, and Azure Automation/AAP orchestrates the pipeline) rather than forcing one tool to do everything.

---

*Tip: At 7 years, expect scenario-based questions — "you have a script that works locally but fails when run as a scheduled task, why?" (classic culprits: execution context/profile not loaded, working directory assumptions, `-ErrorAction` defaults, or credentials/scope differences) — so be ready to reason through failure modes out loud, not just define terms.*
