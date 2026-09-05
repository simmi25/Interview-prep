# Top 60 Shell / Bash Scripting Interview Questions
### For 7+ Years Experience — Senior/Lead Automation Engineer Level

---

## Section 1: Fundamentals & Shell Internals (Q1–Q10)

**Q1. What is the difference between `sh`, `bash`, `ksh`, and `zsh`?**
`sh` is the original POSIX Bourne shell (or a POSIX-compliant symlink to another shell like `dash` on Debian/Ubuntu). `bash` (Bourne Again Shell) extends `sh` with arrays, `[[ ]]`, process substitution, and more. `ksh` (Korn shell) influenced many bash features and is common on older Unix/AIX systems. `zsh` adds advanced completion, globbing, and plugin ecosystems (used by macOS default and Oh My Zsh). Scripts using `#!/bin/sh` should stick to POSIX syntax since `sh` may not actually be bash.

**Q2. What happens when you run a script vs. sourcing it (`./script.sh` vs `source script.sh`)?**
Running a script (`./script.sh`) spawns a new subshell/process; variable exports and `cd` changes inside it don't affect the parent shell. Sourcing (`source script.sh` or `. script.sh`) executes the script in the *current* shell context, so variable assignments, function definitions, and directory changes persist in the calling shell — commonly used for environment setup scripts.

**Q3. Explain the difference between a subshell and the current shell.**
A subshell is a child process forked to run a command group (e.g., in `(...)`, a pipeline, or `$(...)`), inheriting the parent's environment but unable to modify it. Changes to variables, `cd`, or `set` options inside a subshell don't propagate back to the parent once it exits.

**Q4. What is the difference between `$@` and `$*`?**
Both expand to all positional parameters, but `"$@"` expands to separate quoted words (each argument preserved individually, critical when arguments contain spaces), while `"$*"` expands to a single string with arguments joined by the first character of `IFS` (space by default). In scripts handling filenames with spaces, `"$@"` is almost always the correct choice.

**Q5. What is `$?`, `$$`, `$!`, and `$0`?**
`$?` is the exit status of the last executed command; `$$` is the PID of the current shell; `$!` is the PID of the most recently backgrounded process; `$0` is the name of the script/shell itself (used in usage messages or self-referential logic).

**Q6. Explain shell exit codes — what do 0, 1, 2, and 127 mean?**
`0` means success; any non-zero (1–255) means failure/error, though the specific meaning is command-defined; `2` conventionally indicates misuse of shell builtins (bad syntax/usage); `127` means "command not found" (often a `$PATH` issue); `126` means the command was found but not executable (permissions).

**Q7. What is the difference between `>`, `>>`, `2>`, `&>`, and `2>&1`?**
`>` redirects stdout, overwriting the target file; `>>` appends to it; `2>` redirects stderr; `&>` (bash-specific) redirects both stdout and stderr to the same target; `2>&1` redirects stderr to wherever stdout is *currently* pointing — order matters, e.g., `cmd > file 2>&1` sends both to `file`, but `cmd 2>&1 > file` does not (since stderr is duplicated to the terminal *before* stdout is redirected).

**Q8. What's the difference between `set -e`, `set -u`, and `set -x`? What does `set -o pipefail` do?**
`set -e` exits the script immediately if any command fails (non-zero exit); `set -u` treats use of an undefined variable as an error; `set -x` prints each command before executing it (debugging trace); `set -o pipefail` makes a pipeline's exit status reflect the last *failing* command in the pipe rather than just the last command overall — critical because without it, `false | true` reports success.

**Q9. Why is `set -e` alone often insufficient for robust error handling, and what are its known pitfalls?**
`set -e` doesn't trigger on failures inside conditionals (`if cmd; then`), inside `&&`/`||` chains, inside command substitutions in some contexts, or for the non-last command in a pipeline (without `pipefail`). Robust scripts combine `set -euo pipefail`, explicit error checks, and `trap` handlers rather than relying on `-e` alone.

**Q10. What is `trap`, and how is it used for cleanup?**
`trap` registers a command/function to run when the shell receives a signal (e.g., `EXIT`, `INT`, `TERM`, `ERR`). It's commonly used to guarantee cleanup of temp files or lock releases even if the script exits unexpectedly: `trap 'rm -f "$tmpfile"' EXIT`.

---

## Section 2: Variables, Quoting & Expansion (Q11–Q20)

**Q11. Difference between single quotes, double quotes, and no quotes?**
Single quotes (`'...'`) preserve everything literally, no expansion at all. Double quotes (`"..."`) allow variable and command substitution (`$var`, `$(cmd)`) but suppress globbing and word-splitting. No quotes leaves the value subject to word-splitting and pathname expansion (globbing) — a common source of bugs with filenames containing spaces.

**Q12. What is word-splitting, and how does `IFS` control it?**
Word-splitting is the shell breaking an unquoted expanded value into multiple words based on the characters in `IFS` (Internal Field Separator, default: space/tab/newline). Unquoted `$var` expansions are split on IFS characters, which is why unquoted variables are risky with filenames containing spaces — always quote (`"$var"`) unless splitting is explicitly intended (e.g., iterating a space-separated list).

**Q13. What's the difference between `${var}`, `${var:-default}`, `${var:=default}`, and `${var:?error}`?**
`${var}` is plain expansion; `${var:-default}` returns `default` if `var` is unset/empty without modifying `var`; `${var:=default}` does the same but also *assigns* `default` to `var`; `${var:?error}` prints `error` to stderr and exits the script if `var` is unset/empty — useful for mandatory-argument validation.

**Q14. Explain parameter expansion for substring/pattern operations: `${var#pattern}`, `${var##pattern}`, `${var%pattern}`, `${var%%pattern}`.**
`#`/`##` strip a matching prefix (shortest/longest match respectively); `%`/`%%` strip a matching suffix (shortest/longest). Commonly used for path/extension manipulation, e.g., `${file%.txt}` strips a `.txt` suffix, `${path##*/}` extracts the basename.

**Q15. How do you do string replacement in bash without calling `sed`?**
Using `${var/pattern/replacement}` (replaces first match) or `${var//pattern/replacement}` (replaces all matches) — native bash parameter expansion avoids spawning an external process, which matters for performance in tight loops.

**Q16. What is the difference between arithmetic expansion `$(( ))`, `let`, and `expr`?**
`$(( expr ))` is the modern, preferred way to do integer arithmetic inline; `let var=expr` is an older builtin doing the same but with quoting quirks around operators like `*`; `expr` is an external/legacy command (slow, POSIX-portable) mostly seen in older scripts — `$(( ))` should be used in new bash code.

**Q17. How do you declare and use arrays and associative arrays in bash?**
Indexed arrays: `arr=(a b c)`, accessed via `${arr[0]}`, all elements via `${arr[@]}`, length via `${#arr[@]}`. Associative arrays (bash 4+): `declare -A map; map[key]=value`, accessed via `${map[key]}`, all keys via `${!map[@]}`.

**Q18. What is the difference between `local` and a global variable inside a bash function?**
`local varname=value` scopes the variable to the function (and its callees), preventing it from leaking into or clobbering the global shell namespace — essential in larger scripts with many functions to avoid name collisions and accidental state mutation.

**Q19. What's the difference between `readonly` and `declare -r`, and why use them?**
Both mark a variable as immutable after assignment (`readonly VAR=value` / `declare -r VAR=value`), causing an error on any later attempt to modify it — useful for constants (e.g., config paths, magic numbers) to catch accidental reassignment bugs early.

**Q20. How do you safely pass a variable containing special characters into another command (avoiding injection/glitches)?**
Always double-quote expansions (`"$var"`), use arrays for building argument lists (`args=(--flag "$val"); cmd "${args[@]}"`) rather than string concatenation, and avoid `eval` on untrusted input — `eval` re-parses a string as shell code, which is a classic injection vector if the string includes user-controlled data.

---

## Section 3: Control Flow, Functions & Loops (Q21–Q30)

**Q21. Difference between `[ ]`, `[[ ]]`, and `(( ))`?**
`[ ]` is the POSIX `test` command (external or builtin), requiring careful quoting and using string operators like `-eq`/`-lt`. `[[ ]]` is a bash keyword offering safer word-splitting/globbing behavior, pattern matching (`==` with globs), and regex (`=~`) — preferred in bash-only scripts. `(( ))` is for arithmetic evaluation/comparison (e.g., `(( a > b ))`) and doesn't need `$` prefixes on variables inside it.

**Q22. How do you compare strings vs. integers correctly in bash?**
Strings: use `[[ "$a" == "$b" ]]` (or `=` in POSIX `[ ]`). Integers: use `[[ $a -eq $b ]]` or arithmetic context `(( a == b ))` — mixing them up (e.g., comparing numbers with `==` in `[ ]`) causes bugs, especially with leading zeros or non-numeric strings.

**Q23. Explain `case` statements and when you'd prefer them over `if/elif` chains.**
`case` matches a value against glob-style patterns, e.g., matching command-line flags or file extensions: `case "$1" in start) ...;; stop) ...;; *) ...;; esac`. It's cleaner and more efficient than long `if/elif` chains for multi-branch pattern matching, especially with wildcard patterns.

**Q24. How do you write a function in bash, and how do you return values from it?**
`function_name() { ... }` (or `function name { ... }`). Bash functions don't "return" arbitrary values like other languages — they return an integer *exit status* via `return N` (0–255), and typically communicate actual data via stdout (captured with `$(function_name)`) or by setting a global/`local -n` nameref variable.

**Q25. What is a `nameref` (`local -n`), and why is it useful?**
A nameref creates a variable that's a reference/alias to another variable by name, letting a function modify a caller's variable indirectly — useful for "pass by reference" patterns like populating an array passed into a function, avoiding fragile `eval`-based workarounds.

**Q26. Difference between `for i in $(seq 1 10)` and `for ((i=1; i<=10; i++))`?**
The former spawns an external `seq` process and word-splits its output (fragile with locale/IFS issues); the latter is a native bash C-style loop with no external process and no word-splitting concerns — generally preferred for performance and robustness in bash-only scripts.

**Q27. How do you loop over lines of a file safely (handling spaces/special characters)?**
`while IFS= read -r line; do ... done < file` — setting `IFS=` prevents leading/trailing whitespace trimming, and `-r` prevents backslash interpretation, together preserving each line exactly as written, which naive `for line in $(cat file)` does not.

**Q28. What's the danger of parsing `ls` output in scripts, and what's the safer alternative?**
`ls` output can be ambiguous with filenames containing spaces, newlines, or special characters, and its formatting can vary by locale/aliases. Safer alternatives are native globbing (`for f in *.txt`), `find ... -print0 | xargs -0`, or `find ... -exec` — avoiding a text-parsing layer entirely.

**Q29. How do you run tasks in parallel in a bash script, and how do you wait for them / cap concurrency?**
Background jobs with `&`, then `wait` (with no args waits for all, or `wait $pid` for a specific one) to block until they finish; for concurrency limits, use `wait -n` (bash 4.3+) to wait for the next job to finish before launching more, or use `xargs -P N` / GNU `parallel` for a more robust worker-pool pattern.

**Q30. What is the difference between `&&`, `||`, and `;` for chaining commands?**
`;` runs commands sequentially regardless of success/failure; `&&` runs the next command only if the previous succeeded (exit 0); `||` runs the next command only if the previous failed (non-zero exit) — commonly combined for inline error handling: `cmd || { echo "failed"; exit 1; }`.

---

## Section 4: Text Processing (grep/sed/awk/cut) (Q31–Q38)

**Q31. When would you use `grep` vs `awk` vs `sed`?**
`grep` filters/finds lines matching a pattern (searching). `sed` performs line-based stream *editing* (substitution, deletion, insertion). `awk` is a full pattern-scanning and field-processing language, best for column-based data extraction, calculations, and reformatting — as a rule of thumb: grep to find, sed to substitute, awk to extract/transform structured data.

**Q32. How do you extract the Nth column from a delimited file?**
`awk -F',' '{print $3}' file.csv` for the 3rd comma-delimited column, or `cut -d',' -f3 file.csv` for simple fixed-delimiter extraction — `awk` is preferable when you need conditional logic or multiple fields combined.

**Q33. Explain the difference between basic regex (BRE), extended regex (ERE), and PCRE, and where each applies in grep/sed.**
BRE (grep/sed default) requires escaping `+`, `?`, `|`, `()` with backslashes; ERE (`grep -E`/`egrep`, `sed -E`) treats those as special without escaping; PCRE (`grep -P`) supports Perl-compatible features like lookahead/lookbehind and non-greedy quantifiers not available in BRE/ERE.

**Q34. How do you do an in-place find-and-replace across multiple files?**
`sed -i 's/old/new/g' file1 file2 ...` (GNU sed); on macOS/BSD sed, `-i` requires an explicit (possibly empty) backup suffix argument: `sed -i '' 's/old/new/g' file` — a common cross-platform gotcha in shared scripts.

**Q35. How would you use `awk` to sum a column of numbers or compute an average?**
`awk '{sum+=$2} END {print sum}' file` sums column 2; for an average, `awk '{sum+=$2; n++} END {print sum/n}' file` — `awk`'s `BEGIN`/main/`END` block structure makes running aggregates straightforward without external tools.

**Q36. How do you find and delete files older than N days, or find files matching a pattern recursively?**
`find /path -type f -mtime +N -delete` (or pipe to `xargs rm` for more control/logging); pattern search: `find /path -type f -name "*.log"`. Always test with `-print` before adding `-delete` in production scripts to avoid destructive mistakes.

**Q37. How do you use `grep` to search recursively, show line numbers, and count matches?**
`grep -rn "pattern" /path` for recursive search with line numbers; `grep -c "pattern" file` counts matching lines per file; `grep -rl "pattern" /path` lists only filenames containing a match — useful for building quick audit/reporting scripts.

**Q38. How would you deduplicate lines in a file while preserving order (unlike plain `sort -u`)?**
`awk '!seen[$0]++' file` — prints each line only the first time it's seen, using an associative array to track duplicates, preserving original order (whereas `sort -u` reorders alphabetically).

---

## Section 5: Process Management & System Interaction (Q39–Q45)

**Q39. How do you check if a process is running and act on it (e.g., restart if down)?**
Using `pgrep -f "process_name"` (returns PID(s) if running, exit 1 if not) inside an `if`, or checking a PID file's contents against `/proc/<pid>` (Linux) / `ps -p <pid>` — commonly wrapped in a watchdog cron job or systemd unit with `Restart=on-failure` as the more robust modern alternative to hand-rolled polling scripts.

**Q40. What's the difference between `kill`, `kill -9`, and `kill -15`?**
`kill` (default) sends `SIGTERM` (15), asking the process to terminate gracefully (allowing cleanup handlers to run); `kill -9` sends `SIGKILL`, which the kernel enforces immediately and cannot be caught/ignored by the process — a last resort since it skips cleanup and can leave resources (locks, temp files) in an inconsistent state.

**Q41. How do you run a long-running script in the background and detach it from the terminal (survive logout)?**
`nohup ./script.sh > out.log 2>&1 &` prevents SIGHUP on logout and redirects output; alternatively `disown` after backgrounding a job removes it from the shell's job table; for production use, a proper `systemd` service or `screen`/`tmux` session is preferred over raw `nohup` for observability and restart behavior.

**Q42. How do you implement a simple file-based locking mechanism to prevent concurrent script runs (a common cron gotcha)?**
Using `flock`: `exec 200>/var/lock/myscript.lock; flock -n 200 || { echo "already running"; exit 1; }` — `flock` uses kernel-level advisory locking on the file descriptor, which is more reliable than checking for a PID file (which can go stale if a process crashes without cleanup).

**Q43. How do you capture both the exit code and output of a command in a variable?**
`output=$(cmd); rc=$?` — note `$?` must be captured *immediately* after the command substitution, since any intervening command (even an `if`) would overwrite it; for combined stdout+stderr, use `output=$(cmd 2>&1)`.

**Q44. How do you check available disk space or memory in a script and alert on thresholds?**
Disk: `df -h /path --output=pcent | tail -1` or parse `df -P` for portability; memory: `free -m` on Linux (parsing `awk` fields) or `vm_stat` on macOS — typically wrapped in a threshold check (`if [[ $used_pct -gt 90 ]]`) feeding into a monitoring/alert pipeline (email, Slack webhook, or a monitoring agent).

**Q45. How would you write a script to tail a log file and trigger an action when a specific pattern appears (e.g., an ERROR)?**
`tail -F /var/log/app.log | while IFS= read -r line; do [[ "$line" == *ERROR* ]] && notify "$line"; done` — `-F` (vs `-f`) follows the file even across log rotation (by filename, retrying if the file is recreated), which matters for long-running log-watching scripts.

---

## Section 6: Debugging, Robustness & Real-World Practices (Q46–Q50)

**Q46. What's your standard "header" for a production-grade bash script?**
Typically: `#!/usr/bin/env bash` (portable shebang), `set -euo pipefail`, `IFS=$'\n\t'` (safer default field splitting), a `trap` for cleanup on `EXIT`/`ERR`, and often a `readonly SCRIPT_DIR=$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)` to reliably locate the script's own directory regardless of the caller's working directory.

**Q47. How do you make a bash script's logging both readable in a terminal and useful for automation (e.g., parsed by monitoring)?**
Write structured log lines (timestamp, level, message) to stdout for humans, optionally with a `log()` function wrapping `echo`/`printf` consistently, and separate error/info streams (stdout vs stderr) so calling systems (cron, CI, AAP) can distinguish normal output from actual problems using exit codes and stderr content rather than string-matching stdout.

**Q48. How do you unit test bash scripts?**
Using frameworks like `bats` (Bash Automated Testing System) or `shunit2`, which let you write test cases asserting expected output/exit codes for functions or whole script invocations, often integrated into CI pipelines alongside `shellcheck` for static analysis.

**Q49. What is `shellcheck`, and why is it important in a mature scripting workflow?**
A static analysis linter that catches common bash pitfalls — unquoted variables, incorrect test operators, useless `cat`, subshell scoping mistakes, portability issues between `sh` and `bash` — before they cause production incidents; typically run as a pre-commit hook or CI gate on any shell script changes.

**Q50. Describe a real production incident you've debugged that stemmed from a shell scripting pitfall, and how you fixed it going forward.**
A strong answer names a concrete class of bug — e.g., an unquoted variable causing a script to silently process the wrong file when a path contained a space, or a missing `pipefail` masking a failed step in a pipeline so a deployment "succeeded" despite an upstream command failing — and describes the fix (quoting audit, `set -euo pipefail` adoption, adding `shellcheck` to CI) plus how it prevented recurrence. Interviewers at 7 years are listening for ownership and systemic fixes, not just a one-off patch.

---

## Section 7: File Management (Q51–Q60)

**Q51. How do you check and change file/directory permissions, and explain symbolic vs numeric (octal) modes?**
`ls -l` shows permissions as `rwxr-xr-x`; `chmod` changes them either symbolically (`chmod u+x,g-w file` — adjust specific bits) or numerically (`chmod 755 file`, where each digit is the sum of read=4, write=2, execute=1 for owner/group/others). Numeric mode is faster for setting an exact permission set; symbolic mode is better for relative adjustments without knowing the current mode.

**Q52. What's the difference between `chmod`, `chown`, and `chgrp`, and when do you need `sudo` for each?**
`chmod` changes permission bits; `chown` changes file ownership (user, and optionally group via `chown user:group file`); `chgrp` changes only the group. Changing ownership to another user typically requires root/`sudo` (a non-root user can't give a file away), while a file's owner can `chmod` their own files and change the group to any group they belong to without root.

**Q53. Explain the difference between a hard link and a symbolic (soft) link.**
A hard link (`ln file1 file2`) creates a second directory entry pointing to the *same inode* — both names are equally "real," the data persists until all hard links are removed, and hard links can't span filesystems or point to directories. A symbolic link (`ln -s target linkname`) is a special file containing a *path* to another file; it can span filesystems and link to directories, but breaks ("dangling link") if the target is moved or deleted.

**Q54. How do you find and safely remove duplicate or orphaned files in a directory tree?**
Compute checksums (`find /path -type f -exec md5sum {} \;` or `sha256sum`), then group by hash (e.g., with `sort` + `awk` or a script) to identify duplicates before removing extras — always dry-run with `-print`/logging first, since bulk deletes based on checksum collisions can be destructive if a script has a bug.

**Q55. How do you find the largest files/directories consuming disk space?**
`du -ah /path | sort -rh | head -20` lists the top 20 largest files/dirs human-readably; `du -sh /path/*` summarizes top-level directory sizes; `ncdu` (if available) gives an interactive, much faster alternative for large filesystems — a very common "clear disk space" troubleshooting task.

**Q56. How do you archive and compress files/directories, and what's the difference between `tar`, `gzip`, and `zip`?**
`tar` bundles multiple files/directories into a single archive (no compression by itself); it's commonly combined with compression: `tar -czvf archive.tar.gz dir/` (gzip) or `tar -cjvf archive.tar.bz2 dir/` (bzip2, better ratio, slower). `zip`/`unzip` bundles *and* compresses in one step and is more portable to Windows, but generally offers a worse compression ratio than `tar.gz`/`tar.xz` and doesn't preserve Unix permissions/symlinks as faithfully by default.

**Q57. How do you safely copy or sync a large directory tree, preserving permissions/ownership/symlinks, and only transferring changed files?**
`rsync -avz --delete source/ dest/` — `-a` (archive mode) preserves permissions, timestamps, symlinks, and ownership; `-z` compresses in transit; `--delete` removes files at the destination that no longer exist at the source (mirroring); `rsync`'s delta-transfer algorithm also means only changed portions of files move on subsequent syncs, making it far more efficient than `cp -r` for repeated large-scale syncs or backups.

**Q58. What is the sticky bit, SUID, and SGID, and where would you actually see them used?**
SUID (`chmod u+s`, shown as `s` in owner's execute bit) makes an executable run with the *file owner's* privileges regardless of who runs it (e.g., `/usr/bin/passwd` runs as root so any user can update `/etc/shadow`). SGID on a directory makes new files inherit the directory's group (useful for shared team directories). The sticky bit (`chmod +t`, seen on `/tmp`) restricts file deletion within a shared directory to the file's owner (or root), even if others have write access to the directory.

**Q59. How do you securely delete a file so its contents can't be recovered, and why doesn't plain `rm` guarantee that?**
Plain `rm` only removes the directory entry/inode reference — the underlying data blocks remain on disk until overwritten, so recovery tools can often reconstruct "deleted" files. `shred -u file` (Linux) overwrites the file's data multiple times before unlinking it; on SSDs, wear-leveling and TRIM can still leave residual data regardless of overwrite tools, so full-disk encryption is the more reliable guarantee for sensitive data.

**Q60. How do you monitor a directory for file changes in real time and trigger a script (e.g., a "drop folder" automation pattern)?**
Using `inotifywait` (from `inotify-tools` on Linux): `inotifywait -m -e create,modify,close_write /path | while read dir event file; do process "$dir$file"; done` runs a loop triggered on filesystem events without polling; on macOS, the equivalent is `fswatch`. This pattern is common for building lightweight file-drop automation (e.g., auto-processing uploaded files) without a full message-queue system.

---

*Tip: At this experience level, expect live-coding or whiteboard variants of these — e.g., "write a one-liner to find the top 5 largest files under /var/log" or "write a script that retries a failing command up to 3 times with backoff." Practice writing these from scratch, not just recognizing the answer.*
