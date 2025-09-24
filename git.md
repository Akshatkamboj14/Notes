# Git

## git init
git init is the command to create a new, empty Git repository in your current directory.

Meaning

- It initializes Git in the current folder.

- Creates a .git/ directory (hidden) which stores:

    - commit history

    - branches

    - configuration

    - staging area (index)


## git status

git status is the inspection command in Git. It shows you the current state of your working directory and staging area.

- What it does

When you run:
```
git status
```

Git tells you:

1. Which branch you’re on

    - e.g. On branch master or On branch main

    - Whether the branch is ahead/behind remote.

2. Untracked files

    - Files Git doesn’t know about yet.

    - They won’t be included unless you git add them.

3. Changes not staged for commit

    - Files that are tracked but modified.

    - You must git add <file> to stage them.

4. Changes staged for commit

    - Files that are ready to be committed (after git add).

5. Clean working directory

    - If nothing has changed, Git will say:
    ```
    nothing to commit, working tree clean
    ```


## Clone

git clone is the Git command used to copy an existing repository (usually from a remote like GitHub, GitLab, Bitbucket) onto your local machine.

It’s the opposite of git init (which starts a brand-new repo locally).


- What happens when you run git clone

1. Creates a new directory with the repo’s name.

2. Copies all files + commit history from the remote.

3. Sets the remote origin automatically to the source repo.

4. Checks out the default branch (usually main or master).


## git add

It’s how you stage changes (tell Git which files you want included in the next commit).

- Meaning

Git has three areas:

1. Working directory → where you edit files.

2. Staging area (Index) → where you collect changes you want to commit.

3. Repository (.git) → permanent history after commit.

👉 ``git add`` moves changes from working directory → staging area.
Without ``git add``, ``git commit`` won’t include your edits.


- Usage
1. Add a specific file
```
git add file1.txt
```

→ Stages file1.txt only.

2. Add multiple files
```
git add file1.txt file2.txt
```

3. Add everything in current directory
```
git add .
```

or
```
git add -A
```

→ Stages all changes (new, modified, deleted).

4. Add files by pattern
```
git add *.py
```

→ Adds all .py files.



# Git commit 

git commit is the command that permanently saves your staged changes into the repository history (inside .git/).

## What it does

- Takes the changes staged with git add.

- Creates a new commit object in Git:

    - contains snapshot of files

    - commit message

    - author & timestamp

    - parent commit reference

- Moves your branch pointer (HEAD) to this new commit.



- Usage
1. Commit with a message
```
git commit -m "Add login API"
```

✅ Best practice: clear, concise message.



2. Commit all staged changes
```
git commit
```

→ Opens editor (e.g., Vim) for you to type message.


3. Skip staging (commit all modified & tracked files directly)
```
git commit -am "Fix bug in cart logic"
```

⚠️ Note: -a only works for already tracked files, not new/untracked files.



4. Amend the last commit
```
git commit --amend -m "Better message"
```

→ Rewrites the most recent commit (message or staged files).


## Example Flow
```
$ echo "console.log('hi')" > app.js
$ git add app.js
$ git commit -m "Add app.js with hello world"

[main 1a2b3c4] Add app.js with hello world
 1 file changed, 1 insertion(+)
 create mode 100644 app.js
```


- They’re the building blocks of version history.

- Each commit has a unique SHA-1 hash (e.g., 1a2b3c4).

- You can roll back, branch, merge, or cherry-pick using commits.




## Git cherry pick

Git cherry-pick is a command that lets you apply a specific commit from one branch into another branch, without merging the entire branch.

Think of it like saying: “I only want that one cherry (commit) from the cake (branch), not the whole cake.” 🍒🎂

🔹 Syntax
```
git cherry-pick <commit-hash>
```
Example

Let’s say you have this history:
```
main:       A---B---C
feature:        \---D---E
```

- A, B, C = commits on main

- D, E = commits on feature

Now you’re on main and want just commit D (from feature).
```
git checkout main
git cherry-pick <commit-hash-of-D>
```

Result:
```
main:   A---B---C---D'
feature:    \---D---E
```

👉 Notice:

- Commit D was copied to main as a new commit (D') with a new hash.

- You didn’t bring commit E.

### When to use cherry-pick

✅ Backporting a bug fix from dev to main.\
✅ Applying a hotfix from one branch without merging all WIP code.\
✅ Picking a single useful commit from a messy feature branch.

🔹 Things to know

- New commit hash → Even though the content is same, cherry-picked commits are new objects in Git.

- Conflicts can happen → If the commit touches the same lines as your current branch, you must resolve conflicts.

- Not for large feature merges → Use git merge or git rebase when you need whole branches.

🔹 Workflow Example
```
# On main branch
git log --oneline
# Suppose commit abc123 is the one you need

git cherry-pick abc123
# Applies that commit onto current branch

```


# HOW GIT TRACK THE CHANGES

Git’s power comes from how it internally tracks changes. Unlike traditional version control (like SVN) that stores differences (deltas) between file versions, Git stores everything in a content-addressable object database.


1. Git Data Model = Directed Acyclic Graph (DAG) of Commits

    - Every commit points to:

        - A snapshot of your project (not just diffs).

        - Metadata (author, message, timestamp).

        - Parent commit(s).

Think of commits as nodes in a graph, and branches/tags as pointers (references) to those commits.



### Git Object Types

Inside `.git/objects`, Git stores 4 fundamental objects:

| Object Type | Purpose                                               | Example                                                                 |
|-------------|-------------------------------------------------------|-------------------------------------------------------------------------|
| **Blob**    | Stores file content (like “binary large object”).     | A single file’s contents (e.g., `main.py` code).                        |
| **Tree**    | Stores directory structure (file names → blobs/trees).| A folder snapshot.                                                      |
| **Commit**  | Stores metadata + reference to a tree + parent commit(s). | “Commit message, author, parent=xyz, tree=abc.”                     |
| **Tag**     | Human-readable name pointing to a commit.             | `v1.0` tag points to commit `abc123`.                                   |

👉 Each object is identified by a **SHA-1/SHA-256 hash** of its contents.  
This means if a file hasn’t changed, Git reuses the blob instead of storing it again.


### How Git Tracks Changes (Step by Step)

a) Working Directory

- Where you edit files (main.py, index.html, etc.).

b) Staging Area (Index)

- Temporary area where you collect changes before committing.

- Controlled via:
    ```
    git add file.txt
    ```

c) Repository (.git/objects + refs)

- Permanent storage of commits, blobs, and trees.



## Example of What Happens Internally

Suppose you run:
```
echo "Hello" > file.txt
git add file.txt
git commit -m "Add file"
```

Git does this internally:

1. Blob: Stores "Hello" as a blob object → abc123.

2. Tree: Creates a tree object representing the root folder, mapping file.txt → abc123.

3. Commit: Creates a commit object with:

    - Pointer to the tree.

    - Metadata (author, timestamp, message).

    - Parent (if any).

4. Ref update: master now points to that commit.



### Deltas & Compression

- While commits store snapshots, Git optimizes storage:

    - Blobs are reused if file content is same.

    - Packfiles compress objects and sometimes store deltas between them for efficiency.


## Viewing Internals

You can peek inside Git’s storage:
```
git cat-file -p <hash>   # Inspect an object (blob, tree, commit, tag)
git log --graph --oneline --all  # Show commit DAG
```

Example:
```
git cat-file -p HEAD
```
Might show:
```
tree 5f3c2d...
parent a1b2c3...
author Akshat <akshat@example.com>
committer Akshat <akshat@example.com>

Add file
```


# Branch in Git?

```
- A branch = pointer to a commit.

- HEAD = pointer to the current branch (or commit if detached).

- Branches enable parallel development and safe experimentation.
```

- A branch is just a pointer (reference) to a commit.

- Default branch after git init is usually master or main.

- Branching lets you diverge from the main line of development, work independently, and then merge changes back.

👉 Think of it as “a movable pointer to a commit in the history graph.”

## How Branches Work Internally

- A branch is stored in .git/refs/heads/<branch-name>.

- It contains a 40-character SHA hash of the commit it points to.

- Example:
```
.git/refs/heads/feature-login → 1a2b3c4d
```

HEAD is a special pointer that tells Git which branch (or commit) you’re currently on.

## How Branches Work Internally

- A branch is stored in ``.git/refs/heads/<branch-name>``.

- It contains a 40-character SHA hash of the commit it points to.

Example:
```
.git/refs/heads/feature-login → 1a2b3c4d
```

- ``HEAD`` is a special pointer that tells Git which branch (or commit) you’re currently on.


## Branch Commands

1. Create a Branch
```
git branch feature-login
```

- Creates a new pointer (feature-login) at the current commit.

- Does not switch to it.

2. Switch to a Branch
```
git checkout feature-login
# or modern command
git switch feature-login
```

- Moves HEAD to point to feature-login.

3. Create and Switch in One Step
```
git checkout -b feature-login
# or
git switch -c feature-login
```
4. List Branches
```
git branch
```

- Shows all local branches, highlights current with *.

5. Rename a Branch
```
git branch -m old-name new-name
```

6. Delete a Branch

```
git branch -d feature-login     # delete if merged
git branch -D feature-login     # force delete
```



Example Workflow

```
# Start on main
git checkout -b feature-login   # create branch
# (make changes, commit)
git commit -m "Add login feature"
# Switch back to main
git checkout main
# Merge changes
git merge feature-login
```

## Merging Branches

- Fast-forward merge
If no new commits on main since branching:
```
main → moves pointer forward
```

- 3-way merge
If both main and branch have diverged:
```
Git creates a new merge commit combining both histories.
```


## Viewing Branch History
```
git log --oneline --graph --all
```

Example output:
```
* a7b9cde (feature-login) Add login UI
| * 92c0fea (main) Fix bug in dashboard
|/
* 1f2d3e4 Initial commit
```
## Why Branching is Powerful in Git

✅ Lightweight — just pointers, not copies of code.
✅ Cheap to create/delete — no extra overhead.
✅ Encourages workflows — like Git Flow, Feature Branching, Trunk-based development.
✅ Safe experimentation — try things in isolation, merge when stable.


## Merging Statergies

- A merge strategy is the method Git uses to integrate changes from one branch into another.

- It decides whether to move pointers, create a new commit, or rebase history.

- Git picks automatically in most cases, but you can override with flags.

### Common Merge Strategies
1. Fast-Forward Merge

- Happens when the target branch has not diverged.

- Git just moves the pointer forward — no extra commit is made.
```
git checkout main
git merge feature-login
```

If main has no new commits, Git simply updates main to point to feature-login.

✅ Clean history\
❌ Loses context of branch (looks like linear commits)

2. ``3-Way Merge (Default Merge Commit)``

- Used when branches have diverged.

- Git finds the common ancestor commit, compares changes in both, and creates a new merge commit.
```
git merge feature-login
```

History looks like:
```
*   M (merge commit)
|\
| * feature commits
* | main commits
|/
* base commit
```

✅ Preserves full history\
❌ History can look “noisy” with many merge commits

3. Squash Merge

Combines all commits from the branch into one single commit on the target branch.

Doesn’t preserve the branch history.

git merge --squash feature-login
git commit -m "Add login feature"


✅ Clean history (only one commit per feature)
❌ Loses commit granularity

4. Rebase (not technically a merge)

Instead of merging, you replay commits from one branch on top of another.

History becomes linear.

git checkout feature-login
git rebase main


✅ Very clean history (no merge commits)
❌ Rewrites commit hashes → dangerous if branch is shared



3. Squash Merge

- Combines all commits from the branch into one single commit on the target branch.

- Doesn’t preserve the branch history.

```
git merge --squash feature-login
git commit -m "Add login feature"
```

✅ Clean history (only one commit per feature)
❌ Loses commit granularity


4. Rebase (not technically a merge)

Instead of merging, you replay commits from one branch on top of another.

History becomes linear.
```
git checkout feature-login
git rebase main
```

✅ Very clean history (no merge commits)
❌ Rewrites commit hashes → dangerous if branch is shared




1. Gitflow Workflow

Proposed by Vincent Driessen (2010), Gitflow is a heavy branching model.
It defines strict roles for branches and how they interact.

🔸 Branches in Gitflow

main → always production-ready.

develop → integration branch, holds the latest development code.

feature/* → new features, branched from develop.

release/* → staging for production release.

hotfix/* → urgent fixes for main.

🔸 Workflow

Developers branch feature/X from develop.

When done → merge back into develop.

Before release → create release/X branch → test/stabilize.

Release → merge release/X into both main and develop.

For urgent prod bugs → create hotfix/X from main, merge back into both.

✅ Pros

Structured, clear process.

Good for large teams with formal releases.

Separation of concerns (features, releases, hotfixes).

❌ Cons

Complex, many long-lived branches.

Lots of merging overhead.

Slows down continuous integration/delivery.

🔹 2. Trunk-Based Development

Modern lightweight approach focused on speed + CI/CD.

🔸 Rules

Everyone commits to a single shared branch: main (or trunk).

No long-lived branches.

Feature branches exist but are short-lived (a few hours/days max).

Use feature flags/toggles to hide incomplete features.

🔸 Workflow

Developer creates a short-lived branch for a change.

Commit, test, and merge into main quickly (often multiple times a day).

CI/CD automatically builds, tests, and deploys.

✅ Pros

Simpler model → fewer branches.

Encourages continuous integration.

Works well with automation (CI/CD pipelines).

Faster delivery to production.

❌ Cons

Needs discipline + strong CI/CD.

Risky without automated tests.

Large teams may struggle without feature flags.




## Gitflow 🌀 vs Trunk-Based 🌳

| Feature / Aspect       | Gitflow 🌀                                                                 | Trunk-Based 🌳                                     |
|-------------------------|---------------------------------------------------------------------------|---------------------------------------------------|
| **Branching Model**     | Many branches (`main`, `develop`, `feature`, `release`, `hotfix`)        | Single `main` branch + short-lived branches       |
| **Release Cadence**     | Planned, versioned releases                                               | Continuous deployment / frequent releases         |
| **Complexity**          | High (lots of merges, coordination)                                      | Low (simple branch model)                         |
| **Suitable For**        | Large teams, enterprise, strict QA                                       | Modern agile teams, CI/CD-driven                  |
| **Feature Toggles Needed?** | Not required                                                         | Essential for incomplete features                 |
| **Merge Frequency**     | Infrequent (long-lived branches)                                         | Very frequent (daily/hourly merges)               |
| **History Style**       | Preserves full branch history                                            | Linear, clean history                             |
| **Risk of Merge Conflicts** | Higher (long-lived branches diverge)                                 | Lower (short-lived branches)                      |


# Git conflict resolution

Conflict Resolution Workshop in Git context.
This is usually done in teams to practice resolving merge conflicts (which happen when multiple developers edit the same parts of code differently).

🛠️ Conflict Resolution Workshop (Git)
🔹 1. Why Conflicts Happen

Conflicts occur when:

Two developers edit the same line in a file.

One developer deletes a file, another modifies it.

Reordering / formatting changes overlap with feature edits.

Git can auto-merge many changes, but if it cannot decide, it marks a conflict.

🔹 2. Workshop Setup

To simulate conflicts, follow these steps:
Step 1: Create Repo
```
git init conflict-demo
cd conflict-demo
echo "Hello World" > demo.txt
git add .
git commit -m "Initial commit"
```
Step 2: Create Branch A
```
git checkout -b feature-A
echo "Feature A Line" >> demo.txt
git commit -am "Added Feature A"
```
Step 3: Create Branch B (from main)
```
git checkout main
git checkout -b feature-B
echo "Feature B Line" >> demo.txt
git commit -am "Added Feature B"
```

Now both branches edited the same file (same section).

🔹 3. Simulating Conflict

Merge feature-A into main:
```
git checkout main
git merge feature-A
```

Then merge feature-B:
```
git merge feature-B
```

👉 Git will throw a merge conflict in demo.txt.

🔹 4. Resolving Conflicts

Open the conflicted file:
```
Hello World
<<<<<<< HEAD
Feature A Line
=======
Feature B Line
>>>>>>> feature-B

```
Markers:

<<<<<<< HEAD → current branch’s changes.

======= → separator.

>>>>>>> branch → incoming branch’s changes.

Options:

Keep A
```
Hello World
Feature A Line
```

Keep B
```
Hello World
Feature B Line
```


Merge Both
```
Hello World
Feature A Line
Feature B Line
```

Save → then stage + commit:
```
git add demo.txt
git commit -m "Resolved conflict between Feature A and Feature B"
```

5. Best Practices for Conflict Resolution

Communicate → ask teammate before discarding changes.

Use tools → VS Code, git mergetool, IntelliJ, etc.

Keep commits small → easier to resolve.

Rebase frequently → reduces conflict risk.

Use feature flags → avoid long-lived diverging branches.

🔹 6. Workshop Exercises

Two developers both edit the same line → resolve.

One deletes file, other edits it → resolve.

Conflicts in JSON/YAML configs (trickier).

Multi-file conflict resolution with tools.





# Git Pre-Commit Hooks
🔹 What is a Pre-Commit Hook?

A hook = a script Git runs automatically when certain actions happen.

Pre-commit hook runs before a commit is finalized.

Location: .git/hooks/pre-commit in every repo.

Purpose: Enforce checks before code enters history.

🔹 Typical Uses of Pre-Commit Hooks

Code Quality

Run eslint, flake8, pylint, prettier, etc.

Example: block commits with lint errors.

Security

Prevent committing secrets (API keys, passwords).

Run git-secrets, trufflehog.

Tests

Run unit tests; block if failing.

Formatting

Auto-format code with black, prettier, gofmt.

Validate Commit Message (via commit-msg hook).

🔹 How Pre-Commit Hook Works

Git looks for an executable file at .git/hooks/pre-commit.
If it exists and returns a non-zero exit code, the commit is aborted.

Example flow:

$ git commit -m "Add feature"
Running pre-commit hook...
Lint failed: 2 errors
Commit aborted






## Difference Between `git fetch` and `git pull`

| Feature               | `git fetch`                                                                 | `git pull`                                                                 |
|-----------------------|------------------------------------------------------------------------------|-----------------------------------------------------------------------------|
| **What it does**      | Downloads latest commits, branches, tags from remote → updates local remote tracking branches (`origin/main`, etc.) | Does everything `fetch` does **+ merges (or rebases)** into your current branch |
| **Effect on branch**  | ❌ No changes to your working branch (safe)                                  | ✅ Updates your current branch (may create merge commits / conflicts)        |
| **When to use**       | To see what's new on remote **without touching your work**                   | To update your branch with the latest changes from remote                   |
| **Risk of conflicts** | None (only updates refs)                                                     | Possible (merges into your branch)                                          |
| **Command flow**      | `git fetch` → check with `git log origin/main` or `git diff main origin/main` | Equivalent to `git fetch` + `git merge origin/main` (default)               |





# PR Lifecycle

Pull Request Lifecycle
1. Branch Creation

Developer creates a feature branch from main/develop.

Work is done locally (git add, git commit).

Branch is pushed to remote:

git push origin feature/login

2. Open Pull Request (PR)

Developer opens a PR in GitHub UI (or CLI).

A PR specifies:

Source branch → e.g., feature/login

Target branch → e.g., main

Description (what, why, how).

PR gets a unique ID (e.g., #42).

3. Automated Checks (CI/CD)

On PR creation/update:

CI pipelines run → lint, tests, security scans.

Status reported back in PR UI.

PR may be blocked until checks pass.

4. Code Review

Teammates review the changes:

Add comments/suggestions.

Request changes or approve.

Reviews ensure code quality, security, and consistency.

5. Author Updates

Developer makes fixes per feedback:

git commit -am "fix: update error handling"
git push origin feature/login


New commits are automatically added to the PR.

6. Final Approval

Once reviewers approve & checks pass:

PR marked as Approved.

Some repos enforce minimum approvals (e.g., 2 reviewers).

7. Merge

Maintainer merges PR → strategies:

Merge commit: preserves full history.

Squash merge: combines commits into one.

Rebase + merge: linear history.

Example in GitHub:

Button: “Merge Pull Request”.

8. Post-Merge Cleanup

Feature branch often deleted (to keep repo clean).

CI/CD pipeline may deploy merged code (e.g., staging/prod).

9. Close PR

PR is automatically closed after merge.

If not merged (rejected/abandoned), PR can be manually closed.






Quick mental model

History (git log) tells why things changed (intent, reasoning).

Diffs (git diff) tell what actually changed (the surface).
Use both together: read the commit message and then the code patch. Good history makes diffs easy to understand.

Useful git commands (what to run during review)
Inspect commits / history

git log --oneline --graph --decorate -n 20 — quick picture of recent commits and branches.

git log --stat <range> — files changed + insert/delete counts.

git show <hash> — patch + metadata for a single commit.

git show --name-only <hash> — files touched by a commit.

Look at PR / branch changes

git fetch origin then:

git diff origin/main...HEAD — all changes in your branch vs main (three-dot shows combined diff).

git diff --name-only origin/main...HEAD — list changed files.

git diff origin/main...HEAD -- <path> — diff for one file.

Read patches with context

git diff -U5 — more context lines (default is 3).

git diff --word-diff — highlights word-level changes (good for long lines).

git show --patch-with-stat <hash> — commit with stats and patch.

Inspect who/when changed a line

git blame --line-porcelain <file> -- <path> — who last touched each line (useful to ask reviewers/owners).

git log -L :<symbol>:<file> — history of a specific function/line range.

Narrow/interactive review

git add -p or git add --patch — stage/inspect hunks interactively (good for author to split commits).

git diff --staged — diff of staged changes (before commit).

Debug regressions

git bisect start / git bisect bad / git bisect good — find the commit that introduced a bug.

How to read a PR efficiently (step-by-step)

High-level check

Read PR title & description. Are purpose and scope clear? Are linked issues / tests / design docs referenced?

Ensure PR is appropriately sized. If huge, request it be split.

Commit hygiene

git log --oneline for this branch: are commits atomic and meaningful?

Look for noisy commits like “WIP” or auto-generated files mixed with logic — ask for cleanup or squash.

Files changed list

git diff --name-only — scan file list. Beware changes across many unrelated modules.

Read commit messages + corresponding patch

For each commit: read message → git show <commit> to verify message explains why and the patch matches intent.

Good commit = small, one concern, descriptive message.

Read diffs in logical order

Prefer commit-by-commit review (helps understand intent) if author preserved logical commits.

If author squashed, read the whole diff but try to mentally group changes.

Focus areas

Public API changes (breaking changes?) — require extra scrutiny + docs.

Security-sensitive areas (auth, input validation, secrets) — audit carefully.

Performance hotspots (loops, DB queries) — ask for benchmarks or complexity analysis.

Tests — are there unit/integration tests covering changes? Run them locally when necessary.

Behavioral checks

Does the change alter behavior without tests? Ask for tests or a manual verification plan.

Any potential data migrations / backward-incompatibility? Require migration plan or feature flag.

Run the code locally (if needed)

Build, run unit tests, lint. Confirm nothing unexpectedly breaks.

What to look for in the diff (concrete checklist)

Use this checklist when scanning patches:

Correctness

Does the code do what the PR claims? Any off-by-one, null/None issues, edge cases?

Tests

Added/updated tests? Do tests cover edge cases and failure modes?

Clarity

Is the code readable and idiomatic? Prefer clarity over cleverness.

API stability

Any breaking public API changes? Are they documented and versioned?

Security

Input validation, authentication, authorization, secrets, injection risks.

Performance

Any new synchronous/blocking IO in hot paths? N+1 DB queries? Large allocations?

Error handling

Are errors handled/logged meaningfully? No swallowing exceptions silently.

Resource management

File/DB/connection closing, goroutine/thread cleanup, memory leaks.

Duplication

Is logic duplicated that should be extracted?

Formatting & lint

Matches project style; avoid unnecessary whitespace/noisy reformatting in big diffs.

Dep changes

New deps added? Are they necessary and trusted? Any license/security flags?

Build and CI

Does CI pass? If not, ask for fixes. For optional failing checks, request explicit rationale.

Reviewer best practices (how to give useful reviews)

Read commits + context before diving into nitpicks — prioritize correctness & design over style.

Leave actionable comments: show expected vs observed behavior and suggest concrete code changes.

Ask clarifying questions when intent unclear — “What is the intended behavior when X happens?”

Use inline comments sparingly and group related comments in summary.

If many small style fixes, suggest an automated fix (prettier/clang-format) instead of manual comments.

Call out missing tests and ask the author to add them before merging.

When you approve, mention scope of the approval: e.g., “LGTM for correctness; please address linter warnings before merge.”

Author best practices to make reviews fast

Small, focused PRs — one logical change per PR.

Descriptive PR title & body — purpose, what changed, why, how to test, screenshots/logs if UI.

Atomic commits — each commit addresses one concern; use git rebase -i to clean up before pushing.

Add tests for behavior changes.

Document migrations or backward incompatibilities.

Run linters/tests locally and fix obvious issues before opening PR.

Tag reviewers who own the modules (use git blame to find owners for complex areas).

If the PR must be large, add a review guide: which files to prioritize and a high-level summary of design decisions.

Handling messy/difficult diffs

If diff includes large reformatting or generated files, ask to split reformatting into a separate PR — otherwise it buries logic in noise.

For large refactors, prefer review by feature/owner pair and consider high-level design review first (docs + diagrams) before line-by-line.

If commits are messy: ask author to rebase -i and squash irrelevant WIP commits.