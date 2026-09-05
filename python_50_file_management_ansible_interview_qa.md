# Top 50 Python Interview Questions — File Management & Ansible Focus
### For 3 Years Experience — Mid-Level Automation/DevOps Engineer

---

## Section 1: Python Fundamentals for Automation (Q1–Q10)

**Q1. Why is Python a common choice for writing custom automation tooling around Ansible (custom modules, dynamic inventory, filter plugins)?**
Ansible itself is written in Python, and its module/plugin API is Python-native, so writing custom modules, dynamic inventory scripts, or filter/lookup plugins means working directly with `AnsibleModule` and other `ansible.module_utils` classes without any language bridge — plus Python's rich standard library (file I/O, JSON/YAML, subprocess, networking) makes it well-suited to the kind of glue code automation work typically needs.

**Q2. What is the difference between a list, a tuple, and a dictionary, and where would each be useful in an automation script?**
A list (`[]`) is an ordered, mutable collection — useful for a list of hostnames or file paths to iterate over. A tuple (`()`) is ordered and immutable — useful for fixed data that shouldn't change, like a (host, port) pair. A dictionary (`{}`) is an unordered (insertion-ordered since 3.7) key-value mapping — the natural structure for representing parsed JSON/YAML config, or Ansible facts/inventory data.

**Q3. What is the difference between `==` and `is` in Python?**
`==` compares *value* equality (do two objects contain the same data), while `is` compares *identity* (are two variables literally the same object in memory) — a common bug is using `is` to compare strings or numbers, which can appear to work due to interning but isn't reliable, so `==` should be used for value comparisons.

**Q4. What are Python list comprehensions, and why are they preferred over manual loops for building lists?**
A list comprehension (`[x for x in items if condition]`) builds a new list in a single, readable expression rather than a multi-line `for` loop with `.append()` calls — generally more concise and often faster, since the looping is implemented in C internally rather than interpreted Python bytecode for each append call.

**Q5. What is the difference between `*args` and `**kwargs` in a function definition?**
`*args` collects any number of extra *positional* arguments into a tuple; `**kwargs` collects any number of extra *keyword* arguments into a dictionary — commonly used in wrapper functions or when building flexible automation helper functions that need to pass arguments through to another function without knowing its exact signature upfront.

**Q6. What is a virtual environment, and why is it important when writing Python automation tooling?**
A virtual environment (`venv`, `virtualenv`) is an isolated Python installation with its own site-packages, letting different projects use different (potentially conflicting) package versions — e.g., one Ansible project pinned to a specific `ansible-core` and collection version, without affecting the system Python or other projects on the same machine.

**Q7. What is the difference between a Python module and a package?**
A module is a single `.py` file containing definitions/statements that can be imported. A package is a directory containing multiple modules plus an `__init__.py` file (making it importable as a namespace) — Ansible collections and custom `module_utils` libraries are typically organized as packages.

**Q8. What is a decorator in Python, and can you give a practical automation use case?**
A decorator (`@decorator_name`) wraps a function to modify or extend its behavior without changing its code — practical examples include a `@retry` decorator that automatically retries a flaky API call (common when calling cloud provider SDKs) or a `@timing` decorator that logs how long a long-running automation task took.

**Q9. What is the difference between `deepcopy` and a shallow copy in Python, and why does it matter when manipulating nested config dictionaries (e.g., parsed YAML)?**
A shallow copy (`copy.copy()` or `dict.copy()`) copies the top-level structure but nested objects (like a list or dict inside a dict) are still shared references with the original — modifying a nested value affects both copies. `copy.deepcopy()` recursively copies everything, giving a fully independent structure — important when you need to modify a parsed YAML/JSON config (e.g., an Ansible inventory dict) without mutating the original loaded data.

**Q10. What is the Global Interpreter Lock (GIL), and how does it affect writing parallel file-processing or API-calling scripts?**
The GIL ensures only one thread executes Python bytecode at a time in the standard CPython interpreter, meaning threading doesn't give true CPU parallelism for pure-Python computation. However, for I/O-bound automation tasks (file reads, network/API calls to cloud providers), threads still provide effective concurrency because the GIL is released during I/O waits — for CPU-bound work, `multiprocessing` (separate processes, each with its own interpreter/GIL) is the correct choice instead.

---

## Section 2: Core File Handling (Q11–Q25)

**Q11. What is the recommended way to open and read a file in Python, and why use a `with` statement?**
`with open('file.txt', 'r') as f: content = f.read()` — the `with` statement (context manager) guarantees the file is properly closed automatically when the block exits, even if an exception occurs inside it, avoiding resource leaks that manual `f.open()`/`f.close()` calls risk if an error happens before `close()` is reached.

**Q12. What are the common file open modes (`'r'`, `'w'`, `'a'`, `'r+'`, `'rb'`), and how do they differ?**
`'r'` opens for reading (error if file doesn't exist); `'w'` opens for writing, truncating/overwriting the file if it exists (or creating it if not); `'a'` opens for appending, writing at the end without truncating; `'r+'` opens for both reading and writing without truncating; adding `'b'` (e.g., `'rb'`, `'wb'`) opens in binary mode for non-text data (images, serialized objects) instead of text mode.

**Q13. What is the difference between `.read()`, `.readline()`, and `.readlines()`?**
`.read()` reads the entire file content into a single string. `.readline()` reads one line at a time (useful for processing very large files without loading everything into memory at once). `.readlines()` reads all lines into a list of strings — convenient for small-to-medium files, but memory-inefficient for very large ones compared to iterating the file object directly.

**Q14. What is the most memory-efficient way to process a very large file line by line?**
Iterate the file object directly: `with open('big.log') as f: for line in f: process(line)` — this reads one line at a time lazily under the hood, rather than loading the whole file into memory like `.readlines()` would, which matters significantly for multi-gigabyte log files.

**Q15. How do you write to a file, and what's the difference between `.write()` and `.writelines()`?**
`.write(string)` writes a single string (no automatic newline added — you must include `\n` yourself). `.writelines(list_of_strings)` writes a list of strings in sequence, again without automatically adding newlines between them unless each string already includes one — a common beginner mistake is assuming `writelines` adds line breaks like `print()` does.

**Q16. How do you check if a file or directory exists before operating on it, and what's the more "Pythonic" modern approach?**
The modern, preferred approach uses `pathlib`: `from pathlib import Path; Path('file.txt').exists()` (or `.is_file()`/`.is_dir()` for more specificity); the older `os.path.exists('file.txt')` still works and is common in existing/legacy codebases, but `pathlib`'s object-oriented API is generally recommended for new code.

**Q17. What is the difference between `os.path` and `pathlib`, and why has `pathlib` become preferred?**
`os.path` provides path manipulation as string-based functions (`os.path.join()`, `os.path.exists()`). `pathlib` (Python 3.4+) represents paths as objects with methods and operators (`Path('a') / 'b'` for joining, `.exists()`, `.read_text()`), offering a more readable, chainable, and cross-platform-consistent API — most modern Python style guides and codebases now prefer `pathlib` for new code.

**Q18. How do you handle exceptions specifically related to file operations (e.g., file not found, permission denied)?**
Wrap file operations in `try/except`, catching specific exceptions: `FileNotFoundError` (file doesn't exist), `PermissionError` (insufficient access rights), `IsADirectoryError` (tried to open a directory as a file), and a general `OSError` as a catch-all fallback for other filesystem-related errors — catching specific exceptions (rather than a bare `except:`) lets you respond appropriately to each failure mode.

**Q19. How do you safely handle a scenario where multiple parts of your script might try to write to the same file (avoiding a race condition)?**
Use a file locking mechanism — on Unix, the `fcntl` module (`fcntl.flock()`), or cross-platform, the third-party `filelock` package — to ensure only one process/thread writes at a time, similar in concept to shell scripting's `flock` for preventing concurrent cron job execution.

**Q20. How do you read and write JSON files in Python?**
`import json; data = json.load(open('file.json'))` (or `json.loads(string)` for a string already in memory) to read; `json.dump(data, open('file.json', 'w'), indent=2)` (or `json.dumps(data)` for a string) to write — `indent` produces human-readable pretty-printed output, useful for config files a human might also read.

**Q21. How do you read and write YAML files in Python, and why does this matter specifically for Ansible-adjacent tooling?**
Using the third-party `PyYAML` library: `import yaml; data = yaml.safe_load(open('playbook.yml'))` to parse, and `yaml.dump(data, open('file.yml', 'w'))` to write. This matters directly for Ansible tooling since playbooks, inventory files, and `group_vars`/`host_vars` are all YAML — custom scripts that generate or validate Ansible content need to parse/emit YAML that Ansible itself can consume correctly.

**Q22. Why should you use `yaml.safe_load()` instead of `yaml.load()` when parsing YAML files?**
`yaml.load()` (without an explicit `Loader`) can, depending on PyYAML version, execute arbitrary Python object construction embedded in the YAML content — a security risk if parsing YAML from an untrusted source. `yaml.safe_load()` restricts parsing to basic Python types (strings, numbers, lists, dicts) only, which is the safe default for essentially all real-world use cases, including parsing Ansible content.

**Q23. How do you read and write CSV files in Python, and what module handles quoting/escaping edge cases correctly?**
The built-in `csv` module: `csv.reader()`/`csv.writer()` for basic row-by-row access, or `csv.DictReader()`/`csv.DictWriter()` to read/write rows as dictionaries keyed by header column names — using the `csv` module (rather than naive `line.split(',')`) correctly handles edge cases like commas or newlines embedded within quoted fields.

**Q24. What is the difference between text mode and binary mode when working with files, and when do you need binary mode?**
Text mode (`'r'`/`'w'`) automatically handles encoding/decoding (e.g., UTF-8) and universal newline translation. Binary mode (`'rb'`/`'wb'`) reads/writes raw bytes with no such translation — required for non-text data (images, compressed archives, serialized pickle files) or when you need exact byte-for-byte control, such as computing a file's checksum.

**Q25. How do you compute a file's checksum (e.g., for verifying integrity or detecting duplicates), and why must you read it in binary/chunks for large files?**
`import hashlib; h = hashlib.sha256(); with open(path, 'rb') as f: [h.update(chunk) for chunk in iter(lambda: f.read(8192), b'')]; h.hexdigest()` — reading in fixed-size chunks (rather than `f.read()` all at once) keeps memory usage constant regardless of file size, which matters for hashing very large files (e.g., verifying a downloaded ISO or backup archive) without loading the entire thing into RAM.

---

## Section 3: Advanced File & Directory Management (Q26–Q35)

**Q26. How do you list all files in a directory, and how do you do it recursively?**
Non-recursive: `os.listdir(path)` (names only) or `Path(path).iterdir()` (Path objects, distinguishes files/dirs more easily). Recursive: `os.walk(path)` (yields a tuple of `(dirpath, dirnames, filenames)` for every subdirectory) or the more modern `Path(path).rglob('*')` (or `.rglob('*.log')` for pattern-matched recursive search).

**Q27. What is the difference between `glob.glob()` and `pathlib.Path.glob()`?**
`glob.glob('*.txt')` (from the older `glob` module) returns a list of matching filename *strings*. `Path('.').glob('*.txt')` returns an iterator of `Path` *objects*, which can then be chained with other `pathlib` methods (`.stat()`, `.unlink()`, `.read_text()`) directly — generally preferred in modern code for the same reasons `pathlib` is preferred over `os.path` overall.

**Q28. How do you create directories in Python, including nested/parent directories that don't yet exist?**
`os.makedirs(path, exist_ok=True)` creates all necessary parent directories in one call without raising an error if they already exist; the `pathlib` equivalent is `Path(path).mkdir(parents=True, exist_ok=True)` — omitting `exist_ok=True`/`parents=True` is a common source of `FileExistsError`/`FileNotFoundError` bugs in scripts that re-run against partially-existing directory structures.

**Q29. How do you copy, move, and delete files/directories in Python (beyond basic `open()`)?**
The `shutil` module: `shutil.copy(src, dst)` (copies file content + permissions), `shutil.copytree(src, dst)` (recursively copies an entire directory tree), `shutil.move(src, dst)` (moves/renames a file or directory, even across filesystems), and `shutil.rmtree(path)` (recursively deletes a directory tree) — `os.remove()`/`Path.unlink()` handle single-file deletion, while `os.rmdir()` only removes empty directories.

**Q30. What is the danger of using `shutil.rmtree()` in an automation script, and how do you make it safer?**
`shutil.rmtree()` deletes a directory tree permanently and recursively with no confirmation or recycle bin — a path bug (e.g., an empty variable resulting in deleting from the filesystem root, or a wrong variable pointing somewhere unintended) can be catastrophic. Safer practices include validating the target path is within an expected base directory before deleting, logging the exact path being removed, and testing destructive logic with `--dry-run`/print statements before enabling actual deletion in production runs.

**Q31. How do you get file metadata (size, modification time, permissions) in Python?**
`os.stat(path)` (or `Path(path).stat()`) returns a `stat_result` object with attributes like `.st_size` (bytes), `.st_mtime` (last modified, as a Unix timestamp — convert with `datetime.fromtimestamp()`), and `.st_mode` (permission bits, often inspected via the `stat` module's helper functions like `stat.S_ISDIR()`).

**Q32. How do you change file permissions in Python (equivalent to `chmod`)?**
`os.chmod(path, 0o755)` sets permissions using an octal literal, mirroring Unix `chmod` numeric mode semantics directly — useful in deployment scripts that need to ensure a script or key file has the correct restrictive permissions (e.g., `0o600` for a private key) after being written out.

**Q33. How do you create a temporary file or directory safely in Python (that's automatically cleaned up)?**
The `tempfile` module: `tempfile.NamedTemporaryFile()` creates a temp file (optionally auto-deleted on close), and `tempfile.TemporaryDirectory()` creates a temp directory that's automatically recursively removed when the `with` block exits — both handle unique naming and OS-appropriate temp locations (`/tmp` on Linux) automatically, avoiding manual, collision-prone naming schemes.

**Q34. How do you monitor a directory for new/changed files in Python (similar to `inotifywait` in bash)?**
The third-party `watchdog` library provides a cross-platform file-system event observer (`Observer()` + a custom `FileSystemEventHandler` subclass reacting to `on_created`/`on_modified`/`on_deleted` events) — the Python equivalent of bash's `inotifywait`, useful for building a "watch this drop folder and process new files" automation pattern.

**Q35. How would you safely write to a file such that a crash mid-write never leaves a corrupted/partial file (atomic write pattern)?**
Write the new content to a temporary file in the same directory, then use `os.replace(tmp_path, final_path)` to atomically rename it into place — `os.replace()`'s rename operation is atomic at the filesystem level on POSIX systems, so any process reading the target file either sees the complete old version or the complete new version, never a partially-written file, unlike writing directly to the final path.

---

## Section 4: Python & Ansible Integration (Q36–Q50)

**Q36. What is the structure of a custom Ansible module written in Python?**
It's a Python script importing `AnsibleModule` from `ansible.module_utils.basic`, defining an `argument_spec` dictionary describing expected parameters (types, required/optional, defaults), instantiating `module = AnsibleModule(argument_spec=argument_spec, supports_check_mode=True)`, performing the actual logic, and finally calling `module.exit_json(changed=bool, **result_data)` on success or `module.fail_json(msg=str)` on failure — Ansible calls the script, passing parameters as JSON via stdin, and expects JSON output on stdout.

**Q37. How does a custom Ansible module support check mode (`--check`) in its Python implementation?**
Set `supports_check_mode=True` when instantiating `AnsibleModule`, then inside the module's logic check `module.check_mode` before performing any actual state-changing action — if `True`, compute and report what *would* change (setting `changed=True/False` appropriately) without actually executing the change, letting the module behave correctly under `ansible-playbook --check`.

**Q38. How do you write a dynamic inventory script for Ansible in Python, and what output format does Ansible expect?**
A dynamic inventory script must support being called with `--list` (returning a JSON structure of all groups, their hosts, and `_meta.hostvars` for per-host variables) and `--host <hostname>` (returning that host's variables as JSON, though modern Ansible mostly relies on `_meta` in the `--list` output for performance) — the script queries whatever external source (CMDB, cloud API, database) and formats the result as this specific JSON structure for Ansible to consume.

**Q39. What is `ansible-runner`, and why would you use it instead of shelling out to `ansible-playbook` directly from Python?**
`ansible-runner` is a Python library/CLI providing a stable, supported interface for programmatically executing Ansible playbooks/modules/roles from other applications, returning structured results (event data, return codes, artifacts) rather than requiring you to parse raw subprocess stdout text — it's the officially recommended way to embed Ansible execution inside a custom Python application (e.g., a web UI or orchestration service) rather than fragile `subprocess.run(['ansible-playbook', ...])` calls with manual output parsing.

**Q40. If you do need to call `ansible-playbook` via Python's `subprocess` module directly, what should you watch out for?**
Use `subprocess.run([...], capture_output=True, text=True, check=False)` with the command as a *list* (not a shell string, to avoid shell injection risks), explicitly check the returncode rather than assuming success, capture both stdout and stderr for debugging failures, and consider using `-v`/JSON callback output (`ANSIBLE_STDOUT_CALLBACK=json`) if you need to parse results programmatically, since human-readable default output isn't reliably parseable.

**Q41. How would you write a Python script to validate that a set of Ansible playbook YAML files are syntactically valid before committing them (a pre-commit style check)?**
Parse each file with `yaml.safe_load()` to catch basic YAML syntax errors, and/or shell out to `ansible-playbook --syntax-check playbook.yml` (or `ansible-lint`) for Ansible-specific structural validation beyond plain YAML correctness — wrapping this in a script that exits non-zero on any failure makes it usable directly as a git pre-commit hook or CI gate.

**Q42. How do you parse an Ansible inventory file programmatically from Python (outside of Ansible itself)?**
For simple INI-style static inventories, the `configparser` module can handle basic cases, but Ansible's inventory format has quirks (group vars, children, ranges) that a naive parser will mishandle — the more robust approach is using Ansible's own `ansible.inventory.manager.InventoryManager` and `DataLoader` classes (from `ansible-core`'s Python API) to parse inventory exactly the same way `ansible-playbook` itself would.

**Q43. How would you write a Python script that reads a CSV of server names/IPs and generates a corresponding Ansible static inventory YAML file?**
Read the CSV with `csv.DictReader`, build a nested Python dictionary matching Ansible's inventory YAML structure (`{'all': {'children': {'webservers': {'hosts': {...}}}}}`), and write it out with `yaml.dump(data, default_flow_style=False)` — a common real-world integration task connecting a CMDB export to Ansible-consumable inventory.

**Q44. What is the role of `module_utils` in Ansible, and how does it relate to standard Python code organization?**
`module_utils` is a shared library location (`ansible/module_utils/` in core, or a collection's `plugins/module_utils/`) for Python code reused across multiple custom modules — avoiding duplicating helper functions (e.g., a common REST API client) in every module file, following the same "shared package" principle as organizing reusable code into a Python package in any other project.

**Q45. How would you write a Python script to compare the actual state of files on disk against what an Ansible playbook expects (a drift-detection helper outside of `--check` mode)?**
Read the playbook's expected file states (paths, expected content or checksums, e.g., pulled from `template`/`copy` tasks' source/dest), compute the actual current SHA256 checksums of the deployed files on the target (via a lightweight local script, or SSH/`ansible_facts`), and diff the two sets — reporting any mismatches; this is essentially reimplementing a narrow slice of what `ansible-playbook --check --diff` already gives you, so it's more commonly built when you need drift data in a custom format/dashboard rather than raw Ansible output.

**Q46. How would you use Python's `jinja2` library directly (outside of an actual Ansible run) to test a Jinja2 template used in an Ansible `template` task?**
`from jinja2 import Environment, FileSystemLoader; env = Environment(loader=FileSystemLoader('templates')); template = env.get_template('config.j2'); print(template.render(my_var='value'))` — rendering the template directly with representative sample variables lets you validate template logic/output quickly without running a full playbook, useful for unit-testing complex Jinja2 templates in isolation.

**Q47. How do you securely pass secrets from a Python script into an Ansible playbook run (e.g., when orchestrating Ansible from a larger Python application)?**
Avoid passing secrets as plain `--extra-vars` on the command line (visible in process listings/shell history); instead pass them via environment variables the playbook reads, write them to a temporary Ansible Vault-encrypted file consumed via `--extra-vars @secrets.yml` with a vault password supplied through `--vault-password-file` (itself sourced securely, e.g., from a secrets manager at runtime), or use `ansible-runner`'s structured secrets-handling support rather than raw subprocess argument passing.

**Q48. How would you write a Python unit test for a custom Ansible module's core logic, separate from actually running it through Ansible?**
Refactor the module's core logic into plain, testable functions (independent of `AnsibleModule` where possible), then use `pytest` to unit test those functions directly with various inputs/mocked dependencies; for testing the full module interface itself, mock `AnsibleModule`'s `params` and `exit_json`/`fail_json` methods (a common pattern shown in Ansible's own module development documentation) to simulate how Ansible would invoke it without needing a real playbook run.

**Q49. How do you handle logging in a Python automation script that's meant to be run both interactively and as an unattended job (e.g., triggered by AAP/cron)?**
Use the standard `logging` module rather than `print()` — configure handlers so output goes to both console (for interactive runs) and a log file (for unattended runs) via `logging.StreamHandler()` and `logging.FileHandler()`, set an appropriate log level (`INFO` for normal operation, `DEBUG` for troubleshooting), and use structured, leveled messages (`logger.info(...)`, `logger.error(...)`) so downstream systems (like AAP's job output or a monitoring pipeline) can parse severity consistently instead of relying on ad-hoc print statement text.

**Q50. Describe a real script you've written that bridges Python and Ansible (e.g., a custom module, dynamic inventory, or orchestration wrapper) — what problem did it solve?**
A strong answer names a specific, concrete integration — e.g., a dynamic inventory script pulling VM data from Azure and tagging groups by resource group/environment, a custom module wrapping an internal REST API not covered by any existing collection, or a Python wrapper using `ansible-runner` to trigger playbooks from a self-service internal tool — and explains the actual problem it solved (removing manual inventory upkeep, filling a gap no existing module covered, giving non-engineers safe self-service access) plus how file I/O (reading config, writing structured output/logs) played into it, since interviewers at this level are listening for real, applied experience rather than textbook module syntax.

---

*Tip: At 3 years, expect a mix of "write this small function on the spot" (e.g., recursively find all `.yml` files over 1MB, or safely read a large log file) and "explain how you'd wire Python into an existing Ansible workflow" scenario questions — comfort with `pathlib`, `try/except` around file I/O, and reading/writing YAML correctly (especially `safe_load`) tends to come up more than deep Python language trivia.*
