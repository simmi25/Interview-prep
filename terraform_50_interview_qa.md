# Top 50 Terraform Interview Questions
### For 3 Years Experience — Mid-Level Infrastructure/DevOps Engineer

---

## Section 1: Fundamentals & IaC Concepts (Q1–Q10)

**Q1. What is Terraform, and what problem does it solve?**

Terraform is an open-source Infrastructure as Code (IaC) tool by HashiCorp that lets you define cloud/on-prem infrastructure declaratively in configuration files (HCL) and then provisions/manages it consistently and repeatably — solving the problem of manual, error-prone, undocumented infrastructure changes by making infrastructure changes version-controlled, reviewable, and reproducible.

**Q2. What is the difference between declarative and imperative infrastructure provisioning?**

Declarative (Terraform's approach) means you describe the *desired end state* and the tool figures out how to get there. 
Imperative (e.g., a shell script calling `aws ec2 run-instances`) means you specify the exact *sequence of steps* to perform — declarative tools handle ordering, dependency resolution, and diffing automatically, which is why Terraform can tell you exactly what will change before applying it.

**Q3. What is the difference between Terraform and Ansible/Chef/Puppet?**

Terraform is primarily an *infrastructure provisioning* tool (creating VMs, networks, load balancers, managed databases) working at the API/resource level. Ansible/Chef/Puppet are primarily *configuration management* tools (installing packages, managing files, configuring services) working *inside* already-provisioned machines. In practice, they're often used together: Terraform provisions the infrastructure, then Ansible configures it.

**Q4. What is the typical Terraform workflow (the core commands)?**

`terraform init` (initialize working directory, download providers/modules), `terraform plan` (preview changes), `terraform apply` (execute changes), and `terraform destroy` (tear down managed infrastructure) — this init → plan → apply cycle is the backbone of nearly every Terraform operation.

**Q5. What does `terraform init` actually do?**

It downloads and installs the required provider plugins (per `required_providers` blocks), initializes the backend (where state will be stored), downloads any referenced modules, and sets up the `.terraform` working directory — it must be re-run whenever providers, backend config, or modules change.

**Q6. What is the difference between `terraform plan` and `terraform apply`?**

`terraform plan` computes and displays an execution plan — what resources will be created, changed, or destroyed — without making any actual changes, letting you review before committing. `terraform apply` executes that plan against real infrastructure (and by default re-computes the plan itself unless you pass a previously saved plan file).

**Q7. How do you save a plan and apply exactly that plan later (rather than re-computing at apply time)?**

`terraform plan -out=tfplan` saves the plan to a file, then `terraform apply tfplan` applies precisely that saved plan — ensuring what gets approved in a review/CI gate is exactly what gets executed, with no risk of drift between the reviewed plan and the applied changes.

**Q8. What is HCL, and is Terraform configuration allowed in JSON instead?**

HCL (HashiCorp Configuration Language) is Terraform's native, human-friendly configuration syntax. Terraform also accepts an equivalent JSON representation (`.tf.json` files), primarily intended for machine-generated configuration rather than hand-authoring, since HCL is far more readable for humans.

**Q9. What file extensions does Terraform use, and does file naming matter?**

`.tf` for configuration files and `.tfvars`/`.tfvars.json` for variable definition files. Within a directory, Terraform loads and merges *all* `.tf` files together regardless of name (there's no strict naming requirement), though conventions like `main.tf`, `variables.tf`, `outputs.tf`, and `providers.tf` are widely followed for readability.

**Q10. What is idempotency in the context of Terraform, and how does it achieve it?**

Idempotency means running `terraform apply` repeatedly against unchanged configuration produces no further changes. Terraform achieves this by comparing the desired configuration against its recorded state (and the real infrastructure) each run, only acting on the *diff* — if nothing has changed, the plan shows zero actions.

---

## Section 2: HCL Syntax, Variables & Expressions (Q11–Q20)

**Q11. What are the main block types in a Terraform configuration?**
`terraform` (settings — required providers, backend), `provider` (configures a specific provider like AWS/Azure), `resource` (defines an infrastructure object to manage), `data` (reads existing/external information), `variable` (input parameters), `output` (exposes values), `locals` (computed internal values), and `module` (calls a reusable module).

**Q12. What is the difference between `variable`, `output`, and `locals`?**
`variable` blocks define *inputs* to a configuration/module (values supplied by the caller). `output` blocks expose *values* from a module/configuration for use by the caller or displayed after apply. `locals` define *internal, computed* named values used to avoid repeating an expression multiple times within the same configuration — they're not inputs or outputs, just internal convenience.

**Q13. How do you set variable values, and what is the precedence order?**
Ways to set variables: `-var` CLI flag, `-var-file`, `.tfvars`/`.tfvars.json` files (auto-loaded if named `terraform.tfvars` or `*.auto.tfvars`), `TF_VAR_name` environment variables, or interactive prompt if unset. Precedence (highest to lowest, roughly): CLI `-var`/`-var-file` flags → `*.auto.tfvars` (alphabetical) → `terraform.tfvars` → environment variables → variable block defaults.

**Q14. What is a `variable` block's `type`, `default`, `description`, and `validation` used for?**
`type` constrains what kind of value is accepted (string, number, bool, list, map, object, etc.), catching type errors early; `default` provides a fallback if no value is supplied; `description` documents intent (shown in `terraform plan`/docs); `validation` blocks let you enforce custom rules (e.g., a string must match a regex or be one of an allowed set) with a custom error message.

**Q15. Explain Terraform's basic data types: string, number, bool, list, map, and object.**
`string`/`number`/`bool` are primitives; `list(type)` is an ordered collection of a single type; `map(type)` is a key-value collection of a single value type; `object({...})` defines a structured type with named attributes of potentially different types — useful for passing structured configuration (e.g., a "server config" object) as a single variable.

**Q16. What is the difference between `count` and `for_each` for creating multiple resource instances?**
`count` creates N instances indexed numerically (`resource[0]`, `resource[1]`); `for_each` creates instances indexed by a map key or set value (`resource["web"]`, `resource["db"]`). `for_each` is generally preferred when instances are meaningfully distinct (removing one from the middle doesn't force recreation of the others, unlike `count`'s index-shifting behavior).

**Q17. Give an example of when `count` causes problems that `for_each` avoids.**
If you have `count = 3` creating three VMs from a list and you remove the *middle* item from that list, Terraform re-indexes and sees VM[1] and VM[2] as changed, potentially destroying and recreating VMs unnecessarily — with `for_each` keyed by a stable identifier (like a name), removing one item only affects that specific instance, leaving the others untouched.

**Q18. What are Terraform expressions and functions? Give a few commonly used built-in functions.**
Expressions combine values, references, and operators to compute a result (e.g., `"${var.env}-vpc"`). Terraform ships built-in functions (not user-definable) like `length()`, `lookup()`, `merge()`, `concat()`, `join()`, `split()`, `coalesce()`, `cidrsubnet()`, and `jsonencode()`/`jsondecode()` for manipulating strings, collections, and structured data within configuration.

**Q19. What is a `for` expression, and how does it differ from `for_each`?**
A `for` expression (e.g., `[for s in var.subnets : s.cidr_block]`) transforms a list/map into another list/map *within an expression* (used to build a value), whereas `for_each` is a *meta-argument* on a resource/module block controlling how many instances of that resource/module get created — related concept (iteration), different purpose.

**Q20. What is the difference between `count.index`, `each.key`, and `each.value`?**
`count.index` is the numeric index of the current instance when using `count`. `each.key` and `each.value` are available when using `for_each` — for a map, `each.key` is the map key and `each.value` is the corresponding value; for a set of strings, `each.key` and `each.value` are the same (the string itself).

---

## Section 3: State Management (Q21–Q28)

**Q21. What is Terraform state, and why is it necessary?**
State (`terraform.tfstate`) is a JSON file recording the mapping between your configuration's resources and the real-world infrastructure objects they represent (including metadata not visible in config), letting Terraform know what it's already created, detect drift, and compute accurate diffs on the next plan — without it, Terraform would have no way to know what already exists.

**Q22. Why should you avoid using local state files for team/production work?**
Local state is a single file on one person's machine — it isn't shared, isn't locked (risking concurrent-edit corruption if two people run `apply` at once), and often contains sensitive data in plaintext, so most real-world usage requires a *remote backend* instead for both collaboration safety and security.

**Q23. What is a remote backend, and name a few common examples.**
A remote backend stores state outside the local filesystem, in a shared, durable location — common examples: AWS S3 (paired with DynamoDB for locking), Azure Storage Account (blob container), Google Cloud Storage, Terraform Cloud/HCP Terraform (which also provides locking, versioning, and a UI natively).

**Q24. What is state locking, and why does it matter?**
State locking prevents two people/processes from running `apply` against the same state simultaneously, which could corrupt the state file or cause conflicting changes — S3 backends implement this via a DynamoDB table, while Terraform Cloud and Azure/GCS backends have locking built in natively.

**Q25. What is the difference between `terraform state list`, `terraform state show`, and `terraform show`?**
`terraform state list` lists all resources tracked in state; `terraform state show <resource>` shows detailed attributes for one specific tracked resource; `terraform show` displays the full current state (or a saved plan file, if given one as an argument) in human-readable form.

**Q26. How do you import an existing, manually created resource into Terraform state?**
`terraform import <resource_address> <resource_id>` (e.g., `terraform import aws_instance.web i-0123456789`) links an already-existing real resource to a resource block you've written in configuration, so Terraform starts managing it going forward — you must still manually write matching configuration for that resource, since `import` only updates state, not the `.tf` files (Terraform 1.5+'s `import` blocks improve this by letting you generate config too).

**Q27. What happens if your Terraform state gets out of sync with real infrastructure (drift), and how do you detect/fix it?**
Drift occurs when something changes infrastructure outside of Terraform (manual console change, another tool). `terraform plan` will detect and show the drift as a diff on the next run; you fix it either by applying Terraform's plan to revert the manual change back to the declared config, or by updating the configuration to match the new reality (and potentially `terraform apply` to reconcile state) if the manual change should be kept.

**Q28. What is the difference between `terraform state rm` and `terraform destroy`?**
`terraform state rm` removes a resource from Terraform's *state tracking only* — the real infrastructure object is left untouched, just no longer managed by Terraform. `terraform destroy` (or targeted destroy of a specific resource) actually *deletes* the real infrastructure object. Confusing these is a common, costly mistake — `state rm` is for "stop managing this," not "delete this."

---

## Section 4: Providers, Resources & Data Sources (Q29–Q37)

**Q29. What is a Terraform provider?**
A plugin that translates Terraform configuration into API calls for a specific platform (AWS, Azure, GCP, Kubernetes, GitHub, etc.), implementing the actual create/read/update/delete logic for each resource type that platform offers — Terraform Core itself knows nothing about any specific cloud; all platform knowledge lives in providers.

**Q30. How do you pin/constrain provider versions, and why is this important?**
Using a `required_providers` block with a version constraint (e.g., `version = "~> 5.0"`), which prevents `terraform init` from silently pulling a newer major version that could introduce breaking changes to resource schemas — version pinning is essential for reproducible builds across team members and CI pipelines.

**Q31. What is the difference between a `resource` block and a `data` block?**
A `resource` block declares something Terraform should *create and manage* (and eventually can destroy). A `data` block *reads* information about an existing object (managed by Terraform or not) — e.g., looking up an existing VPC ID or the latest AMI — without creating or managing it; data sources are read-only.

**Q32. What is a resource's implicit dependency vs. an explicit dependency (`depends_on`)?**
Implicit dependency occurs automatically when one resource's argument references another resource's attribute (e.g., `subnet_id = aws_subnet.main.id`) — Terraform infers the correct creation order from this reference graph. `depends_on` is used to declare a dependency explicitly when there's no direct attribute reference but an ordering requirement still exists (e.g., IAM policy propagation before a resource that needs it).

**Q33. What is the Terraform dependency graph, and how do you visualize it?**
Terraform builds a directed acyclic graph (DAG) of all resources based on their references/dependencies, which determines both plan-time ordering and apply-time parallelism (independent resources can be created concurrently). `terraform graph` outputs this graph in DOT format, which can be rendered visually (e.g., piped into Graphviz) for troubleshooting complex dependency chains.

**Q34. What is a provisioner in Terraform (e.g., `local-exec`, `remote-exec`), and why is it generally discouraged as a first choice?**
Provisioners run scripts/commands during resource creation or destruction (`local-exec` on the machine running Terraform, `remote-exec` on the newly created resource via SSH/WinRM). They're discouraged as a primary tool because Terraform can't track/manage what they do (no state awareness of their effects, no idempotency guarantee) — Hashicorp's official guidance is to prefer native provider features, cloud-init/user-data, or a dedicated configuration management tool (Ansible) for post-provisioning setup, reserving provisioners for last-resort cases.

**Q35. What is `lifecycle` meta-argument, and what do `create_before_destroy`, `prevent_destroy`, and `ignore_changes` do?**
`lifecycle` customizes how Terraform manages a specific resource's replacement/deletion behavior. `create_before_destroy = true` creates the replacement resource before destroying the old one (avoiding downtime during forced replacement). `prevent_destroy = true` blocks any plan that would destroy that resource (safety guard for critical infra like production databases). `ignore_changes` tells Terraform to ignore drift on specific attributes (e.g., ones modified outside Terraform, like auto-scaling group desired count).

**Q36. What is a "tainted" resource, and how do you force recreation of a resource?**
A tainted resource is one Terraform has marked for destruction and recreation on the next apply (historically via `terraform taint`, now more commonly achieved with `terraform apply -replace=<resource_address>` in modern Terraform versions) — used when a resource is in a bad/corrupted state that Terraform's normal diff wouldn't otherwise detect as needing replacement.

**Q37. How does Terraform determine whether changing a resource attribute requires an in-place update vs. full replacement (destroy + recreate)?**
This is defined by the *provider's resource schema* — each attribute is marked internally as updatable in-place or as "ForceNew" (requiring replacement) by the provider's authors, based on what the underlying cloud API actually supports; `terraform plan` shows a `-/+` (replace) vs `~` (update in-place) indicator so you can see which behavior a given change will trigger before applying.

---

## Section 5: Modules (Q38–Q44)

**Q38. What is a Terraform module, and why use one?**
A module is a reusable, self-contained package of Terraform configuration (a directory with its own resources, variables, and outputs) that can be called multiple times with different inputs — promoting DRY infrastructure code, consistency across environments, and easier maintenance versus copy-pasting the same resource blocks repeatedly.

**Q39. What is the difference between a root module and a child module?**
The root module is the main working directory where you run `terraform apply` (the entry point). A child module is any module called (via a `module` block) from the root module or from another module — modules can be nested multiple levels deep.

**Q40. How do you call a module, and where can module source come from?**
`module "name" { source = "..."; var1 = "value" }` — source can be a local relative path (`./modules/vpc`), the public Terraform Registry (`terraform-aws-modules/vpc/aws`), a Git repository URL, or an internal private registry — Terraform downloads/references the module code accordingly during `terraform init`.

**Q41. How do you pass outputs from one module as inputs to another?**
Reference the source module's declared output via `module.<module_name>.<output_name>` (e.g., `subnet_id = module.network.subnet_id`) as the value for another module's input variable or a resource argument — this is how modules compose together in larger configurations.

**Q42. What is module versioning, and why does it matter for the public Terraform Registry?**
When sourcing a module from the Registry (or a Git tag), you can (and should) pin a specific `version` constraint (e.g., `version = "5.1.2"`), preventing an unreviewed/breaking module update from silently changing your infrastructure behavior on the next `terraform init -upgrade` — the same reasoning as provider version pinning (Q30).

**Q43. What are some best practices for designing a reusable, well-structured module?**
Keep modules focused on a single logical concern (e.g., "networking" vs. one giant module doing everything), expose sensible variables with good defaults and validation, output the values downstream consumers are likely to need, avoid hardcoding environment-specific values inside the module itself (pass them in as variables instead), and document usage with a README and examples.

**Q44. How do you test a Terraform module before publishing/relying on it broadly?**
Use `terraform plan`/`apply` against example configurations in an `examples/` directory within the module repo, validate with `terraform validate` and `terraform fmt -check`, and for more rigorous testing use a dedicated framework like Terratest (Go-based) or the native `terraform test` command (introduced in Terraform 1.6+) to write assertions against real or planned resource output.

---

## Section 6: Workspaces, Backends & Collaboration (Q45–Q50)

**Q45. What are Terraform workspaces, and what are they good (and not good) for?**
Workspaces let a single configuration directory manage multiple, separate state files (e.g., `terraform workspace new dev`) referenced via `terraform.workspace` in config. They're good for small variations of the same infrastructure (e.g., quick feature-branch sandboxes); they're generally *not* recommended as the sole mechanism for separating major environments like dev/staging/prod, since all workspaces share the same backend config and code path, making it easy to accidentally apply against the wrong one — separate state files/directories per environment (or a tool like Terragrunt) are usually safer for that purpose.

**Q46. What is the difference between Terraform workspaces and having entirely separate directories/state files per environment?**
Workspaces share the exact same `.tf` configuration files, differing only in state and the `terraform.workspace` variable value — useful for near-identical environments. Separate directories (each with their own backend config, `.tfvars`, and potentially slightly different configuration) give full isolation and are typically preferred for environments with meaningfully different topology, sizing, or compliance requirements (e.g., prod having stricter settings than dev).

**Q47. How do you structure a Terraform project for multiple environments (dev/staging/prod) in a team setting?**
Common patterns: a shared `modules/` directory containing reusable building blocks, with separate environment directories (`environments/dev`, `environments/staging`, `environments/prod`) each with their own backend config and `.tfvars` calling those shared modules — this keeps environments isolated while avoiding code duplication, and is often managed with a wrapper tool like Terragrunt for very large multi-environment estates.

**Q48. How do you integrate Terraform into a CI/CD pipeline safely?**
Typical pattern: on a pull request, run `terraform fmt -check`, `terraform validate`, and `terraform plan` (posting the plan output as a PR comment for review); on merge to main, run `terraform apply` (often with a manual approval gate for production) using a saved plan artifact from the PR stage to guarantee what's approved is exactly what's applied, with credentials injected via CI secrets rather than hardcoded.

**Q49. What is `terraform fmt` and `terraform validate`, and how do they differ?**
`terraform fmt` automatically rewrites configuration files to a canonical formatting style (indentation, alignment) — purely cosmetic, no logic checking. `terraform validate` checks configuration for internal syntax/consistency errors (missing required arguments, type mismatches, invalid references) without needing provider credentials or contacting any API — both are typically run as an early, fast CI check before the more expensive `plan` step.

**Q50. How do you handle secrets (like database passwords or API keys) in Terraform configuration safely?**
Never hardcode secrets in `.tf`/`.tfvars` files committed to version control; instead, mark sensitive variables with `sensitive = true` (which redacts them from CLI/plan output), source actual secret values from a secrets manager (AWS Secrets Manager, Azure Key Vault via a `data` source, HashiCorp Vault provider) or CI-injected environment variables (`TF_VAR_*`) at runtime, and be aware that even with `sensitive = true`, the value is still stored in plaintext within the state file itself — so state file access control and encryption at rest (e.g., encrypted S3 bucket) matter just as much as the config-level protection.

---

*Tip: At 3 years, expect a mix of concept questions and small scenario/troubleshooting ones — e.g., "your `terraform apply` shows a resource will be destroyed and recreated that you didn't expect, how do you investigate?" (check for a `ForceNew` attribute change, review the plan's reasoning, consider `lifecycle.ignore_changes` if it's expected drift). Being able to read and reason about `terraform plan` output confidently is one of the most practically tested skills at this level.*
