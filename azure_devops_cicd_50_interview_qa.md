# Top 50 Azure DevOps CI/CD Interview Questions
### For 3 Years Experience — Mid-Level DevOps/Automation Engineer

---

## Section 1: Azure DevOps Fundamentals (Q1–Q8)

**Q1. What is Azure DevOps, and what are its core services?**

Azure DevOps is Microsoft's suite of DevOps tools covering the full software delivery lifecycle: Azure Boards (work item tracking/Agile planning), Azure Repos (Git/TFVC source control), Azure Pipelines (CI/CD build and release automation), Azure Test Plans (manual/exploratory testing), and Azure Artifacts (package management for NuGet/npm/Maven/Python feeds).

**Q2. What is the difference between Azure DevOps Services and Azure DevOps Server?**

Azure DevOps Services is the Microsoft-hosted, cloud/SaaS version (formerly VSTS/Visual Studio Online). Azure DevOps Server is the on-premises, self-hosted version (formerly Team Foundation Server/TFS) that organizations install and manage on their own infrastructure — functionally similar but with different update cadences and hosting responsibilities.

**Q3. What is a Project in Azure DevOps, and what is an Organization?**

An Organization is the top-level container (tied to a specific Azure AD tenant/billing) holding one or more Projects. A Project is a workspace within that organization containing its own Boards, Repos, Pipelines, and Artifacts feeds — used to logically separate different products/teams within the same organization.

**Q4. What is the difference between CI (Continuous Integration) and CD (Continuous Delivery/Deployment)?**

CI is the practice of automatically building and testing code every time changes are pushed, catching integration issues early. Continuous Delivery automates the release process up to a manual approval gate before production. Continuous Deployment goes further, automatically deploying every passing change all the way to production with no manual gate — the distinction between the latter two is entirely about whether a human approval step exists before the final production release.

**Q5. What is the difference between a Build pipeline and a Release pipeline in Azure DevOps' classic model?**

A Build pipeline compiles/tests code and produces artifacts (the CI side). A Release pipeline takes those artifacts and deploys them through a sequence of stages/environments (the CD side) — this was the original two-pipeline classic UI model; modern YAML pipelines can express both build and multi-stage deployment in a single pipeline definition instead.

**Q6. What is the difference between classic (UI-based) pipelines and YAML pipelines in Azure Pipelines?**

Classic pipelines are configured through a visual designer UI, with definitions stored in Azure DevOps' database (not natively version-controlled alongside code). YAML pipelines are defined in a `.yml` file checked into the same repository as the code, giving version history, code review via PRs, and "pipeline as code" — YAML is now Microsoft's recommended approach for new pipelines.

**Q7. What is a Service Connection in Azure DevOps, and why is it needed?**

A Service Connection stores the authentication details (service principal, personal access token, SSH key, etc.) needed for a pipeline to securely connect to an external service — Azure subscriptions, Docker registries, Kubernetes clusters, GitHub, or on-prem servers — without embedding credentials directly in pipeline YAML.

**Q8. What is the difference between a Personal Access Token (PAT) and a Service Principal for authenticating pipelines to Azure resources?**

A PAT is tied to an individual user account, inherits that user's permissions, and expires/needs manual renewal — risky for automation since it's linked to a person who might leave or have it revoked. A Service Principal is an Azure AD identity created specifically for an application/automation to use, with its own scoped permissions independent of any individual user, and is the recommended approach for pipeline-to-Azure authentication (often used via a Service Connection with Managed Identity or federated credentials as the most secure current option).

---

## Section 2: Azure Pipelines — YAML & Structure (Q9–Q18)

**Q9. What is the basic structure/hierarchy of a YAML pipeline (stages, jobs, steps)?**

A Pipeline contains one or more Stages (logical phases, e.g., Build, Test, Deploy), each Stage contains one or more Jobs (units of work that run on an agent), and each Job contains a sequence of Steps (individual tasks or scripts) — this hierarchy lets you control grouping, dependencies, and parallelism at the appropriate level.

**Q10. What is the difference between a `task` and a `script` step in a YAML pipeline?**
A `task` invokes a pre-built, reusable Azure DevOps task (e.g., `AzureWebApp@1`, `PublishBuildArtifacts@1`) with structured inputs, maintained/versioned by Microsoft or a marketplace publisher. A `script`/`bash`/`pwsh`/`powershell` step runs raw shell commands directly — tasks are preferred when a well-supported one exists (better input validation, cross-platform abstraction), while scripts offer maximum flexibility for anything a task doesn't cover.

**Q11. What is a trigger in Azure Pipelines, and what's the difference between a CI trigger and a PR trigger?**
A `trigger` (CI trigger) defines which branch pushes automatically kick off the pipeline (e.g., `trigger: branches: include: [main]`). A `pr` trigger runs the pipeline against pull requests targeting specified branches (typically for validation before merge) — both can be scoped with path filters to avoid triggering on irrelevant changes (e.g., docs-only commits).

**Q12. What is a scheduled trigger, and how do you configure a pipeline to run nightly?**
A `schedules:` block using cron syntax defines time-based triggers independent of code pushes, e.g., `schedules: - cron: "0 2 * * *" branches: include: [main] always: true` runs the pipeline daily at 2 AM regardless of whether there were new commits (`always: true`) or only if there were changes (`always: false`, the default).

**Q13. What is the difference between `dependsOn` and default stage/job execution order?**
By default, stages/jobs within the same pipeline run sequentially in the order defined unless you explicitly enable parallelism. `dependsOn` lets you explicitly control ordering and enable *parallel* execution of independent stages/jobs (since without any dependency, Azure Pipelines can run them concurrently), and also supports conditional logic based on a dependency's outcome.

**Q14. How do you run stages/jobs conditionally in a YAML pipeline?**
Using the `condition:` property with pipeline expressions, e.g., `condition: succeeded()` (default — only run if prior steps succeeded), `condition: failed()`, `condition: always()` (run regardless of prior outcome, useful for cleanup steps), or custom logic like `condition: eq(variables['Build.SourceBranch'], 'refs/heads/main')` to only deploy from the main branch.

**Q15. What is a pipeline template in Azure DevOps, and why use one?**
A template (`.yml` file referenced via `extends:`, `template:` for steps/jobs/stages) lets you define reusable pipeline logic once and reference it from multiple pipelines with parameters — critical for enforcing consistent build/deploy patterns across many microservices/repos without copy-pasting the same YAML everywhere, and for centrally updating shared logic in one place.

**Q16. What is the difference between a variable, a variable group, and a pipeline parameter?**
A variable is a simple key-value pair defined at pipeline/stage/job scope, resolved at *runtime*. A variable group is a named collection of variables (optionally linked to Azure Key Vault) shared across multiple pipelines, managed centrally in Library. A parameter is defined with a type and resolved at *compile time* (before the pipeline even starts running), enabling more advanced logic like conditionally including entire steps/stages based on the parameter's value — something runtime variables can't do.

**Q17. What is the difference between a multi-stage YAML pipeline and using separate Release pipelines (classic) for deployment?**
A multi-stage YAML pipeline expresses build *and* deployment through multiple environments entirely as code in one file, version-controlled with the app. Classic separates these into a distinct Build pipeline (producing artifacts) and a separately configured Release pipeline (UI-based, not code-reviewed the same way) — Microsoft's current guidance favors multi-stage YAML for the "everything as code" and review-ability benefits.

**Q18. How do you pass data (like a computed value) from one job/step to another within a pipeline?**
Using logging commands to set an output variable: `echo "##vso[task.setvariable variable=myVar;isOutput=true]someValue"` inside a step with a `name:` set on that step, then reference it elsewhere as `$[ dependencies.JobName.outputs['StepName.myVar'] ]` (across jobs) or `$(myVar)` within the same job — this is how, e.g., a build job can pass a computed version number to a later deploy job.

---

## Section 3: Build & Release Concepts, Artifacts (Q19–Q28)

**Q19. What is a build artifact, and how do you publish and consume one across pipeline stages?**
An artifact is the packaged output of a build (e.g., a compiled binary, container image reference, or zipped web app) that needs to persist beyond the job that created it. `PublishBuildArtifacts@1` (or the newer `publish:` shorthand) uploads it to Azure Pipelines' artifact storage, and `DownloadBuildArtifacts@1` (or `download:`) retrieves it in a later stage/job — necessary because each job/stage may run on a different, ephemeral agent with no shared filesystem.

**Q20. What is the difference between Azure Pipelines artifacts and Azure Artifacts (the package feed service)?**
Pipeline artifacts are transient build outputs scoped to a single pipeline run, used to pass files between stages of that run. Azure Artifacts is a persistent package management feed (like a private NuGet/npm/PyPI/Maven registry) for versioned packages meant to be *reused across multiple, unrelated pipelines/projects* over time — different lifecycles and purposes.

**Q21. What is an Environment in Azure Pipelines, and what does it provide beyond just being a deployment target label?**
An Environment (Pipelines → Environments) represents a deployment target (e.g., "Production") and provides deployment history/tracking, the ability to attach approval checks and gates specific to that environment, and (for Kubernetes/VM resource types) resource-level health/status visibility — it's more than a label; it's the object approval and audit policies attach to.

**Q22. What are approvals and gates in a release/deployment, and what's the difference between them?**
Approvals require an explicit human sign-off (a named user/group must approve) before deployment proceeds to that environment. Gates are automated checks (e.g., querying a monitoring system for no active incidents, checking a work item query, calling a REST API/Azure Function for a custom validation) that must pass automatically without human intervention — both can be combined, and both are commonly used to protect production deployments.

**Q23. What is a deployment strategy, and what are the built-in strategies Azure Pipelines supports (`runOnce`, `rolling`, `canary`)?**
`runOnce` deploys once, straightforwardly, to all targets (default, simplest). `rolling` deploys to a subset of targets at a time (useful for VM-based deployments to avoid taking all instances down simultaneously). `canary` deploys to a small subset first, validates, then progressively increases — these strategies are primarily used with Kubernetes and VM deployment groups where partial/rollout control matters.

**Q24. What is a Deployment Group in Azure Pipelines, and when would you use it instead of standard agent pools?**
A Deployment Group is a set of target machines (VMs) with the Azure Pipelines deployment agent installed on each, registered together for deployment purposes — used specifically for deploying directly to a fleet of VMs (rather than containers/PaaS), enabling rolling deployment strategies across that group of machines.

**Q25. How do you version and track which specific build/commit is running in each environment?**
Each pipeline run has a unique `Build.BuildNumber`/`Build.BuildId`, and the Environment view in Azure DevOps shows deployment history per environment including which build/commit was deployed and when — combined with tagging container images or artifacts with the build number/git SHA, this gives full traceability from a running environment back to the exact source commit.

**Q26. What is the difference between a pipeline "run" and a pipeline "definition"?**
A pipeline definition is the YAML/configuration describing *what* the pipeline does (the template). A run is one specific execution instance of that definition (triggered by a commit, PR, schedule, or manual trigger), with its own logs, artifacts, and status — one definition produces many runs over time.

**Q27. How do you roll back a failed or problematic deployment in Azure Pipelines?**
Options include re-running a previous successful pipeline/release (redeploying the last known-good artifact/build), using the deployment strategy's built-in rollback behavior if configured (some strategies support automatic rollback on failed health checks), or triggering a new deployment of a previous Git tag/commit — Azure Pipelines doesn't have a single "undo" button, so rollback strategy needs to be planned as part of the pipeline design (e.g., blue-green or keeping N previous artifact versions available).

**Q28. What is the difference between a "stage" being skipped versus "canceled" versus "failed" in a pipeline run?**
Skipped means the stage's `condition:` evaluated to false, so it was intentionally never run. Canceled means it was manually stopped mid-execution (or the pipeline overall was canceled) before completing. Failed means it ran but a step returned a non-zero/error result — these distinctions matter when interpreting pipeline run history and building conditions for downstream stages that check on a specific dependency's outcome.

---

## Section 4: Agents, Pools & Execution Environment (Q29–Q35)

**Q29. What is the difference between a Microsoft-hosted agent and a self-hosted agent?**
A Microsoft-hosted agent is a fresh, ephemeral VM provisioned by Microsoft for each pipeline run (pre-installed with common tooling, automatically cleaned up afterward, no maintenance needed, but limited job run-time and less control over the environment). A self-hosted agent runs on infrastructure you manage (on-prem or your own cloud VM), persists between runs, can be pre-configured with exactly the tools/versions/network access your pipeline needs, and has no Microsoft-imposed job time limit.

**Q30. When would you choose a self-hosted agent over a Microsoft-hosted one?**
When you need access to internal/private network resources not reachable from Microsoft's hosted pool (e.g., deploying to on-prem servers, internal databases), need specific pre-installed software/licenses that are expensive or slow to install fresh every run, need faster pipeline runs by avoiding repeated tool installation/caching, or need to exceed Microsoft-hosted agents' job execution time limits.

**Q31. What is an agent pool, and how does it relate to parallel jobs?**
An agent pool is a logical grouping of one or more agents that pipelines are assigned to run on; the number of pipelines/jobs that can run *simultaneously* is bounded by the number of parallel jobs licensed/available for that pool (for Microsoft-hosted, a fixed free or paid parallelism count; for self-hosted, roughly the number of registered agents) — understanding this explains why pipeline runs sometimes queue rather than starting immediately.

**Q32. How do you target a pipeline job to run on a specific agent (e.g., one with a required tool or OS)?**
Using `pool: name: 'PoolName' demands: - myCapability` to require a specific self-hosted pool and a capability tag registered on qualifying agents, or simply `pool: vmImage: 'ubuntu-latest'` to select a specific Microsoft-hosted image — demands let you route jobs to agents meeting specific prerequisites within a larger, heterogeneous self-hosted pool.

**Q33. What is a container job in Azure Pipelines, and why use it instead of a plain agent VM?**
A container job runs the pipeline's steps inside a specified Docker container (`container: image: myregistry/mytools:latest`) rather than directly on the agent's host OS, ensuring a fully consistent, reproducible tool environment regardless of which underlying agent VM picks up the job — similar in spirit to Ansible's Execution Environments solving the same "consistent runtime" problem.

**Q34. How do you cache dependencies (e.g., npm packages, pip packages) between pipeline runs to speed up builds?**
The `Cache@2` task (or the `- task: Cache@2` shorthand `- cache:`) stores/restores a specified path keyed on a hash of a lock file (e.g., `package-lock.json`, `requirements.txt`), so if dependencies haven't changed, subsequent runs restore the cache instead of re-downloading everything from scratch — particularly valuable on Microsoft-hosted agents where every run starts from a completely clean VM.

**Q35. How do you troubleshoot a pipeline job that fails only on the hosted agent but works locally?**
Check for environment differences: tool/SDK version mismatches (pin versions explicitly with a `UseDotNet@2`/`UsePythonVersion@0`-style task rather than relying on whatever happens to be pre-installed), missing environment variables/secrets that exist locally but weren't configured in the pipeline, OS differences if using a different `vmImage`, and network/firewall restrictions the hosted agent doesn't have access to (like an internal resource only your local machine can reach) — enabling `system.debug: true` for verbose diagnostic logging is often the fastest way to narrow it down.

---

## Section 5: Variables, Secrets & Security (Q36–Q43)

**Q36. What is the difference between a regular pipeline variable and a "secret" variable?**
Secret variables (marked via the lock icon in the UI, or defined in a variable group linked to Key Vault) are encrypted at rest and automatically masked (replaced with `***`) in pipeline logs to prevent accidental exposure — regular variables are stored/displayed in plain text and shouldn't be used for anything sensitive.

**Q37. How do you integrate Azure Key Vault secrets into a pipeline?**
Link a Variable Group to an Azure Key Vault (via a Service Connection with appropriate access policy/RBAC permissions) in Library settings, or use the `AzureKeyVault@2` task directly within a pipeline step to pull specific secrets at runtime — both approaches keep the actual secret values out of the pipeline YAML/variable definitions entirely, sourcing them live from Key Vault on each run.

**Q38. Why are secret variables still visible as `$(secretName)` in YAML, and is that a security risk?**
The variable *reference* (its name) is visible in YAML since that's just pointing to where the value comes from — the actual *value* is what's protected (encrypted at rest, masked in logs); this is not a risk by itself, though secrets can still leak if a script explicitly echoes/logs the variable's resolved value or writes it to an unprotected output file, so masking isn't a substitute for careful script design.

**Q39. What is the difference between pipeline-level, stage-level, and variable-group-level variable scope, and what's the precedence if the same name is defined at multiple levels?**
Variables can be defined at the whole pipeline, a specific stage, or via a linked variable group; more specific scope generally overrides broader scope for the same name (e.g., a stage-level variable overrides a pipeline-level one of the same name within that stage) — being explicit about scope avoids confusing "why did my variable not apply here" bugs in larger multi-stage pipelines.

**Q40. What are pipeline permissions and branch policies, and how do they work together to protect production deployments?**
Branch policies (in Azure Repos) enforce rules before code can be merged (required reviewers, passing build validation, linked work items). Pipeline permissions control who can edit a pipeline definition, approve specific environments, or manage its service connections/secrets — together they ensure both the code merging into a protected branch *and* the pipeline that deploys it are governed, rather than just one half of that chain.

**Q41. What is the principle of least privilege as applied to Service Connections, and how do you implement it in Azure DevOps?**
Rather than granting a single Service Connection's Service Principal broad Owner/Contributor rights across an entire Azure subscription, scope its RBAC role assignment to only the specific resource group(s)/resources the pipeline actually needs to touch — limiting the blast radius if the Service Connection's credentials were ever compromised or a pipeline misconfigured to touch the wrong resource.

**Q42. How do you prevent secrets from being exposed if a pipeline script uses `echo`/`Write-Host` for debugging?**
Azure Pipelines automatically scans step output for known secret variable values and masks them as `***` even in ad-hoc `echo`/print output — but this masking is value-based (it can't catch a secret that's been transformed, e.g., base64-encoded, before being printed), so scripts should still avoid deliberately printing secret values even when relying on automatic masking as a backstop, not a guarantee.

**Q43. What is Workload Identity Federation (federated credentials) for Azure Service Connections, and why is it now preferred over storing a Service Principal secret?**
Workload Identity Federation lets Azure Pipelines authenticate to Azure AD using a short-lived, automatically issued token trust relationship rather than a long-lived Service Principal client secret stored in the Service Connection — eliminating the risk and operational overhead of secret rotation/expiry entirely, and is Microsoft's currently recommended approach for new Azure Service Connections.

---

## Section 6: Integrations — Ansible, Testing & Real-World Pipelines (Q44–Q50)

**Q44. How would you run an Ansible playbook as part of an Azure Pipeline?**
Use a self-hosted Linux agent (Microsoft-hosted agents can run Ansible too, but a self-hosted agent is more common when the playbook needs network access to internal infrastructure), install Ansible as a pipeline step (or use a container job with a pre-built image containing Ansible), then run `ansible-playbook -i inventory site.yml` via a `script`/`bash` step, passing any needed variables/vault password via pipeline secret variables rather than hardcoding them.

**Q45. How do you securely pass an Ansible Vault password into a pipeline-triggered playbook run?**
Store the vault password as a secret pipeline variable (or in Key Vault via a linked variable group), write it to a temporary file at runtime within the pipeline step (`echo "$(vaultPassword)" > /tmp/vault_pass.txt`), reference it with `--vault-password-file /tmp/vault_pass.txt`, and clean up the temp file afterward — never commit the vault password itself to the repository or print it in logs.

**Q46. How would you structure a pipeline that provisions Azure infrastructure with Terraform and then configures it with Ansible?**
A multi-stage pipeline: a "Provision" stage runs `terraform init/plan/apply` producing infrastructure and outputting connection details (IPs, resource IDs) as pipeline output variables; a subsequent "Configure" stage consumes those outputs to dynamically generate/update an Ansible inventory (or use a dynamic inventory plugin against the newly created Azure resources) and runs the appropriate playbook — with `dependsOn` ensuring Configure only runs after Provision succeeds.

**Q47. What is a self-hosted agent's role when your Ansible playbook needs to reach on-prem/internal-only servers, and how do you set it up?**
Since Microsoft-hosted agents run in Microsoft's cloud with no route into your private network, a self-hosted agent installed *inside* that network (or with VPN/ExpressRoute connectivity to it) is required so `ansible-playbook`'s SSH/WinRM connections can actually reach the target hosts — registered to an agent pool the relevant pipeline/job is configured to use.

**Q48. How do you incorporate automated testing (unit tests, `ansible-lint`, `PSScriptAnalyzer`, etc.) into a CI pipeline as quality gates?**
Add dedicated steps/stages early in the pipeline running the relevant linter/test framework (`ansible-lint playbooks/`, `pytest`, `Invoke-Pester`, `Invoke-ScriptAnalyzer`), publish results using `PublishTestResults@2` (for a unified Tests tab in the run summary), and set the stage/pipeline to fail (non-zero exit code propagates naturally) if any check fails — placing these checks *before* deployment stages ensures broken or non-compliant code never reaches later environments.

**Q49. How would you design a pipeline to deploy the same Ansible-managed configuration safely across dev → staging → production, minimizing risk to production?**
Use separate Environments (Q21) per target with production requiring manual approval, run the identical playbook/role against each environment's own inventory/variables (never hand-edited per-environment logic, just different `group_vars`/inventory data) so what's tested in staging is exactly what runs in prod, gate production with both an approval and a smoke-test/health-check gate, and keep the Git branch/tag being deployed clearly visible in the pipeline run so it's always traceable which version reached each environment.

**Q50. Describe a real Azure DevOps pipeline you've built or maintained — what did it automate, and what was the trickiest part to get right?**
A strong answer names something concrete — e.g., a multi-stage YAML pipeline provisioning Azure infrastructure and then running Ansible for OS configuration, or a pipeline gating production deploys behind manual approval plus a health-check gate — and identifies a genuine pain point solved along the way (a flaky self-hosted agent needing a container job for consistency, secret handling for a vault password, or getting variable-passing between stages working correctly) since interviewers at this level are listening for hands-on experience with the messy realities of pipeline design, not just knowing the terminology.

---

*Tip: At 3 years, expect follow-up questions probing *why* you made specific pipeline design choices — e.g., "why a variable group instead of inline variables," or "why did you gate that stage with an approval instead of an automated check" — showing you understand the trade-offs (security, maintainability, speed) behind common Azure DevOps patterns tends to matter more than reciting task names.*
