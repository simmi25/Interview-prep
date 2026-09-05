# Top 50 Git Interview Questions
### For 3 Years Experience — Mid-Level Developer/DevOps Engineer

---

## Section 1: Fundamentals & Core Concepts (Q1–Q10)

**Q1. What is Git, and how does it differ from centralized version control systems like SVN?**
Git is a distributed version control system (DVCS) — every clone is a full repository with the complete history, so most operations (commit, diff, log, branch) happen locally without needing network access to a central server. Centralized systems like SVN keep a single authoritative repository, and most operations require contacting that server, making Git generally faster and more resilient (no single point of failure for history).

**Q2. What is the difference between the working directory, the staging area (index), and the repository?**
The working directory is your actual files on disk that you edit. The staging area (index) is where you assemble the *next* commit by selecting specific changes (`git add`). The repository (`.git` directory) stores the committed history permanently — this three-stage model is what lets you commit only part of your changes at a time.

**Q3. What is a commit, and what does a commit's SHA-1/SHA hash represent?**
A commit is a snapshot of the entire repository's tracked files at a point in time, along with metadata (author, timestamp, message, parent commit(s)). Its hash is a checksum computed over the commit's content and metadata, uniquely identifying it — if anything in the commit or its history changes, the hash changes, which is also what makes Git's history tamper-evident.

**Q4. What is the difference between `git init` and `git clone`?**
`git init` creates a brand-new, empty Git repository in the current directory. `git clone <url>` copies an *existing* remote repository (full history and all branches/tags metadata) to your local machine, automatically setting up the remote (`origin`) connection.

**Q5. What is `HEAD` in Git?**
`HEAD` is a pointer to the current commit/branch tip you're working from — normally it points to a branch name (e.g., `refs/heads/main`), which in turn points to a commit; when you check out a specific commit directly (rather than a branch), you get a "detached HEAD" state.

**Q6. What is the difference between `git add`, `git commit`, and `git push`?**
`git add` stages changes (moves them from working directory into the index, preparing them for the next commit). `git commit` permanently records the staged changes as a new commit in the local repository history. `git push` uploads local commits to a remote repository so others can access them.

**Q7. What is a `.gitignore` file used for?**
It specifies file/directory patterns that Git should never track or show as untracked (e.g., build artifacts, `node_modules/`, `.env` files with secrets, IDE config folders) — keeping the repository clean and preventing accidental commits of generated or sensitive files.

**Q8. What's the difference between `git diff`, `git diff --staged`, and `git status`?**
`git diff` shows changes in the working directory that are *not yet staged*. `git diff --staged` (or `--cached`) shows changes that *are staged* but not yet committed. `git status` gives a summary overview of which files are staged, unstaged, or untracked, without showing the actual line-level content differences.

**Q9. What is the difference between a blob, a tree, and a commit object in Git's internal data model?**
A blob stores raw file content (no filename/metadata). A tree represents a directory, mapping filenames to blobs (files) or other trees (subdirectories). A commit points to a single root tree (the snapshot) plus parent commit(s) and metadata — together these three object types are how Git represents an entire repository's history as a content-addressed graph.

**Q10. What is the difference between `git log` and `git reflog`?**
`git log` shows the commit history reachable from the current branch/HEAD — the "official" project history. `git reflog` shows a local, time-ordered record of everywhere `HEAD` has pointed (including commits from branches that were since deleted, or before a reset/rebase) — invaluable for recovering "lost" commits that are no longer reachable from any branch.

---

## Section 2: Branching & Merging (Q11–Q20)

**Q11. What is a branch in Git, technically speaking?**
A branch is simply a lightweight, movable pointer (a 40-character reference stored in `.git/refs/heads/`) to a specific commit — creating a branch is nearly instantaneous because it doesn't copy any files, unlike some older VCS tools where branching was an expensive operation.

**Q12. What is the difference between `git branch <name>` and `git checkout -b <name>` (or `git switch -c <name>`)?**
`git branch <name>` creates a new branch pointer but leaves you on your current branch. `git checkout -b <name>` (or the newer, clearer `git switch -c <name>`) creates the branch *and* switches your working directory to it in one step.

**Q13. What is the difference between `git checkout` and `git switch`/`git restore`?**
`git checkout` historically did multiple unrelated things (switching branches, restoring files, checking out specific commits), which was confusing. Git 2.23+ introduced `git switch` (specifically for changing branches) and `git restore` (specifically for restoring working directory/staged file contents) to split those responsibilities into clearer, more purpose-specific commands.

**Q14. Explain the difference between a fast-forward merge and a three-way (true) merge.**
A fast-forward merge happens when the target branch has no new commits since the feature branch diverged — Git simply moves the branch pointer forward, no new merge commit is created. A three-way merge happens when both branches have diverged with separate new commits — Git creates a new merge commit with two parents, combining both histories.

**Q15. What is a merge conflict, and when does it occur?**
A merge conflict occurs when Git can't automatically reconcile changes because the same lines (or file) were modified differently on both branches being merged — Git marks the conflicting sections in the file with `<<<<<<<`, `=======`, `>>>>>>>` markers and requires manual resolution before the merge can be completed.

**Q16. How do you resolve a merge conflict, step by step?**
Open the conflicted file(s), manually edit the content to keep the correct/combined result (removing the conflict markers), stage the resolved file with `git add <file>`, then complete the merge with `git commit` (or `git merge --continue`) — you can also abort entirely with `git merge --abort` if you want to back out and reconsider.

**Q17. What is the difference between `git merge` and `git rebase`?**
`git merge` combines two branches by creating a new merge commit that ties both histories together, preserving the full (sometimes messy) branching history exactly as it happened. `git rebase` replays your branch's commits one by one on top of another branch's tip, producing a linear history as if you'd branched off the latest point — rewriting commit history rather than merging it.

**Q18. What are the trade-offs of rebasing vs. merging in a team workflow?**
Rebasing gives a cleaner, linear history (easier to read/bisect) but rewrites commit hashes, which is dangerous on shared/public branches since anyone who already pulled the old commits will have a diverging history. Merging preserves true history and is always safe on shared branches, but can produce a more tangled commit graph with many merge commits over time — the common rule is "never rebase commits that have been pushed and shared with others."

**Q19. What does "never rebase a public/shared branch" mean in practice, and what happens if you violate it?**
If you rebase a branch that others have already pulled and continued working from, their local history diverges from the new rewritten history — when they try to push/pull afterward, they'll hit confusing conflicts or duplicate commits, since Git sees the rebased commits as entirely different objects (new hashes) even though the content is similar.

**Q20. What is a fast-forward-only merge (`git merge --ff-only`), and why would a team enforce it?**
It only allows the merge to proceed if a fast-forward is possible (i.e., no divergent commits), failing otherwise — teams enforce this on certain branches to guarantee no accidental merge commits are created, often requiring rebasing feature branches onto the target first to keep history strictly linear.

---

## Section 3: Remote Repositories & Collaboration (Q21–Q29)

**Q21. What is the difference between `git fetch` and `git pull`?**
`git fetch` downloads new commits/branches from the remote into your local repository's remote-tracking branches (e.g., `origin/main`) *without* touching your current working branch. `git pull` does a `fetch` followed immediately by a `merge` (or rebase, if configured) into your current branch — `fetch` is the safer, "look before you leap" option.

**Q22. What is the difference between `git pull` and `git pull --rebase`?**
Plain `git pull` merges the fetched remote changes into your local branch (creating a merge commit if your branch has diverged). `git pull --rebase` instead replays your local unpushed commits on top of the newly fetched remote commits, avoiding an extra merge commit and keeping history linear — many teams set this as the default (`git config pull.rebase true`) to avoid noisy "merge branch into branch" commits from routine pulls.

**Q23. What is a remote-tracking branch (e.g., `origin/main`), and how does it differ from your local `main`?**
A remote-tracking branch is a local, read-only bookmark reflecting the last-known state of a branch on the remote as of your last fetch — it's *not* automatically kept in sync in real time; you update it by fetching. Your local `main` is the branch you actually commit to and work on, which may be ahead of, behind, or diverged from `origin/main` until you push/pull.

**Q24. What is the difference between `git push` and `git push --force` (and `--force-with-lease`)?**
A normal `git push` is rejected if it would overwrite remote history that your local branch doesn't already contain (protecting against accidentally clobbering others' work). `git push --force` overwrites the remote branch unconditionally with your local history, discarding any remote commits it doesn't include — dangerous on shared branches. `git push --force-with-lease` is a safer variant that fails if the remote has been updated by someone else since your last fetch, preventing you from accidentally destroying a teammate's just-pushed work.

**Q25. What is a pull request (PR) / merge request (MR), and why is it central to modern Git workflows?**
A PR/MR is a platform feature (GitHub, GitLab, Bitbucket — not a native Git concept) proposing that changes from one branch be merged into another, providing a space for code review, automated CI checks, and discussion before the merge happens — it's the standard mechanism enforcing quality gates and collaboration in team-based Git workflows.

**Q26. How do you configure and use multiple remotes (e.g., `origin` and `upstream`) in a fork-based workflow?**
`git remote add upstream <url>` adds a second remote (commonly the original repository you forked from), letting you `git fetch upstream` and merge/rebase its changes into your fork's branches to stay in sync with the original project, while `origin` remains your own fork where you push your work and open PRs from.

**Q27. What is `git clone --depth 1` (a shallow clone), and when would you use it?**
It clones only the most recent commit (or N commits with `--depth N`) rather than the entire history, dramatically reducing clone time/size for very large repositories — commonly used in CI pipelines where full history isn't needed, just the current state to build/test from.

**Q28. What is the difference between a Git submodule and a Git subtree, at a high level?**
A submodule embeds another repository as a reference (a pointer to a specific commit) at a subdirectory path, keeping the two repositories' histories fully separate — the parent repo just tracks which commit of the submodule to use. A subtree actually merges the external repository's history and files directly into the parent repo's own history, avoiding the need for collaborators to separately manage/initialize a linked repo, at the cost of a larger, combined history.

**Q29. How do you handle a situation where you accidentally committed a large file or secret and need to remove it from history entirely (not just the latest commit)?**
Simply deleting the file in a new commit isn't enough since it remains in history; you need to rewrite history with a tool like `git filter-repo` (the modern recommended tool, replacing the older `git filter-branch` and BFG Repo-Cleaner) to strip the file/secret from every commit, then force-push the rewritten history — and critically, rotate/invalidate the leaked secret immediately regardless, since it may already have been cloned or scraped by automated bots before cleanup.

---

## Section 4: Undoing Changes & History Manipulation (Q30–Q39)

**Q30. What is the difference between `git reset`, `git revert`, and `git checkout`/`git restore` for undoing changes?**
`git reset` moves the branch pointer (and optionally the index/working directory) backward, effectively rewriting history — appropriate for local, unpushed commits. `git revert` creates a *new* commit that undoes the changes of a previous commit, preserving history — the safe choice for undoing something already pushed/shared. `git restore` reverts specific file(s) in the working directory or index to a previous state without touching commit history at all.

**Q31. Explain the difference between `git reset --soft`, `--mixed`, and `--hard`.**
`--soft` moves the branch pointer but leaves the staging area and working directory untouched (changes from the "undone" commits become staged). `--mixed` (the default) moves the pointer and resets the staging area, but leaves working directory files untouched (changes become unstaged). `--hard` moves the pointer *and* resets both the staging area and working directory to match — discarding all those changes entirely, which is destructive and unrecoverable except via reflog.

**Q32. When would you use `git revert` instead of `git reset` to undo a bad commit?**
Whenever the commit has already been pushed and potentially pulled by others — `revert` adds a new, safe, shareable commit undoing the change, whereas `reset` would rewrite history that others have already based work on, causing the same shared-history problems as rebasing a public branch (Q19).

**Q33. What is `git stash`, and what problem does it solve?**
`git stash` temporarily shelves uncommitted changes (both staged and unstaged) so you can switch context (e.g., to fix an urgent bug on another branch) with a clean working directory, then later restore them with `git stash pop` or `git stash apply` — useful when you're not ready to commit but need to switch branches cleanly.

**Q34. What is the difference between `git stash pop` and `git stash apply`?**
`git stash apply` restores the stashed changes but keeps the stash entry in the stash list (in case you need it again or want to apply it to multiple branches). `git stash pop` restores the changes *and* removes that stash entry from the list in one step.

**Q35. What is `git cherry-pick`, and when is it useful?**
`git cherry-pick <commit-hash>` applies the changes from a single specific commit onto your current branch, without merging the entire branch it came from — useful for backporting a specific bug fix to a release branch, or pulling in one useful commit from a branch you don't want to fully merge yet.

**Q36. What is an interactive rebase (`git rebase -i`), and what can you do with it?**
`git rebase -i <base>` opens an editable list of commits, letting you reorder, squash (combine multiple commits into one), reword (edit commit messages), edit (pause to amend a commit's content), or drop commits entirely — commonly used to clean up a messy feature branch's history (squashing "fix typo" commits, etc.) before merging/opening a PR.

**Q37. What is `git commit --amend`, and what's an important caveat about using it?**
It modifies the *most recent* commit — adding staged changes to it and/or editing its message — rather than creating a new commit. The caveat: like rebasing, it changes the commit's hash, so it should only be used on commits that haven't been pushed/shared yet, or you'll create the same shared-history divergence problem.

**Q38. How do you find which commit introduced a specific bug, efficiently, in a large history?**
`git bisect` performs a binary search through commit history — you mark a known-good and known-bad commit, and Git checks out commits in between for you to test and mark good/bad at each step, converging on the exact commit that introduced the regression in O(log n) steps rather than checking every commit linearly.

**Q39. How do you recover a commit that seems "lost" after a bad `reset --hard` or an accidentally deleted branch?**
Use `git reflog` to find the commit hash HEAD pointed to before the destructive operation, then `git checkout <hash>` (or `git branch recovery-branch <hash>`) to recover it — as long as Git's garbage collection hasn't run and purged unreachable objects yet (which it doesn't do immediately), the commit data is still recoverable.

---

## Section 5: Workflows & Best Practices (Q40–Q50)

**Q40. What is a common branching strategy (e.g., Git Flow, GitHub Flow, trunk-based development), and how do they differ?**
Git Flow uses long-lived `develop` and `main` branches plus dedicated `feature/`, `release/`, and `hotfix/` branches — structured but heavier, suited to scheduled release cycles. GitHub Flow is simpler: `main` is always deployable, feature branches are short-lived and merged via PR directly into `main`, suited to continuous deployment. Trunk-based development takes this further — very short-lived branches (or direct small commits) merged into a single trunk multiple times a day, relying heavily on feature flags rather than long-lived branches to manage incomplete work.

**Q41. What makes a good commit message, and why does it matter for a team?**
A good commit message has a short, imperative-mood summary line (e.g., "Fix null pointer in user lookup," not "fixed bug") under ~50 characters, optionally followed by a blank line and a more detailed body explaining *why* the change was made (not just what — the diff already shows what). This matters because commit history becomes a searchable project record used for code review, `git blame` investigations, and changelogs long after the author has forgotten the details.

**Q42. What is `git blame`, and what's a common limitation/gotcha when using it?**
`git blame <file>` shows which commit and author last modified each line of a file — useful for understanding the history/rationale behind a specific line. The common gotcha: a purely cosmetic change (reformatting, renaming) shows up as "blame" on whoever did the reformatting rather than the original logical author; `git blame -w` (ignore whitespace) and `git log --follow` help mitigate this, and Git also supports a `.git-blame-ignore-revs` file to skip specific "noise" commits entirely.

**Q43. What is a Git hook, and give an example of a commonly used one?**
A hook is a script Git automatically runs at specific points in the workflow (stored in `.git/hooks/`) — e.g., a `pre-commit` hook that runs linters/tests before allowing a commit, or a `commit-msg` hook that enforces a commit message format (like requiring a ticket number). Tools like Husky or `pre-commit` are commonly used to manage and share hooks across a team, since raw `.git/hooks/` scripts aren't versioned/shared automatically by cloning.

**Q44. What is the difference between `.gitattributes` and `.gitignore`?**
`.gitignore` controls which files Git ignores entirely (never tracked). `.gitattributes` controls how Git *handles* tracked files — e.g., forcing consistent line-ending normalization (`* text=auto`), marking certain file types as binary (preventing bad diff/merge attempts on them), or configuring diff/merge drivers for specific extensions.

**Q45. How do you handle line-ending (CRLF vs LF) issues in a cross-platform team (Windows + Mac/Linux)?**
Configure `core.autocrlf` appropriately (`true` on Windows to convert to LF in the repo and CRLF in the working directory; `input` on Mac/Linux to convert CRLF to LF on commit only) and/or standardize behavior explicitly via a `.gitattributes` file (`* text=auto`) so the setting isn't dependent on each individual developer's local Git config — the `.gitattributes` approach is generally more reliable for teams since it travels with the repo.

**Q46. What is the difference between squash merging, rebase merging, and a regular (merge commit) merge on a platform like GitHub?**
A regular merge creates a merge commit preserving all individual commits from the feature branch. Squash merging combines all the feature branch's commits into a single new commit on the target branch (cleaner history, loses individual commit granularity). Rebase merging replays the feature branch's individual commits directly onto the target branch with no merge commit at all (linear history, individual commits preserved) — teams pick based on how much commit-level granularity they want to preserve in `main`'s history.

**Q47. How do you keep a long-lived feature branch up to date with `main` without losing work?**
Regularly merge `main` into the feature branch (`git merge main` while on the feature branch) for a low-risk approach that preserves full history, or regularly rebase the feature branch onto `main` (`git rebase main`) for a cleaner linear history if the branch hasn't been shared with others yet — the choice follows the same merge-vs-rebase trade-off from Q18, just applied incrementally throughout the branch's life rather than only at the final merge.

**Q48. What is a monorepo, and what specific Git challenges does it introduce at scale?**
A monorepo holds multiple projects/services in a single Git repository rather than splitting them across many repos. Challenges include: repository size/clone time growing very large over time, `git blame`/`log` becoming slower on huge histories, and the need for tooling (sparse-checkout, partial clone, or specialized systems like Git virtual filesystems) to let developers work with only the subset of the repo relevant to them rather than the entire codebase.

**Q49. How would you set up branch protection rules in a team repository, and what do they typically enforce?**
Branch protection (a platform feature on GitHub/GitLab/Bitbucket) typically enforces: requiring PR review approval(s) before merge, requiring passing CI status checks, disallowing direct pushes to the protected branch (e.g., `main`), and optionally requiring linear history (no merge commits) or signed commits — collectively preventing untested or unreviewed code from landing directly on critical branches.

**Q50. Describe a real Git problem you've had to troubleshoot (e.g., a bad merge, lost commits, or a messy history) and how you resolved it.**
A strong answer at this level names a concrete, specific incident — e.g., recovering commits after a teammate force-pushed over shared history (using reflog to identify and cherry-pick the lost commits back), or untangling a merge conflict caused by two branches independently refactoring the same file (resolving by understanding both branches' intent rather than blindly picking one side) — and explains what process change followed (e.g., adopting `--force-with-lease`, branch protection rules, or smaller/more frequent merges) to prevent recurrence, since interviewers are listening for judgment and follow-through, not just command recall.

---

*Tip: At 3 years, expect at least one "explain what happened and how you'd fix it" scenario rather than pure definitions — e.g., being shown a conflicted `git status` output or a confusing `git log --graph` and asked to narrate what's going on. Comfort reading Git's own output (status, log, diff, reflog) under a bit of pressure is usually more telling to interviewers than reciting command syntax from memory.*
