# Practical Git Command Notes

A beginner-to-advanced reference for everyday Git usage — what each command does, how to use it, **what you'll actually see when you run it**, what changes in your repository, and what can go wrong.

**Legend:** `WD` = Working Directory · `Staging` = Staging Area (index) · `Local` = Local repository · `Remote` = Remote repository

Every command below follows the same practical pattern where it makes sense:

```text
Situation
    ↓
Command
    ↓
Terminal output
    ↓
Repository state changes
    ↓
Result
    ↓
Next step
```

> **About the examples:** All hashes, branch names, filenames, and dates are placeholders. Your exact output will differ — that's expected. Output shown as "example output" is illustrative of what real Git prints.

## Table of Contents

1. [Git Basics](#1-git-basics)
2. [Working with Changes](#2-working-with-changes)
3. [Branches](#3-branches)
4. [Remote Repositories](#4-remote-repositories)
5. [Undoing & Recovering Changes](#5-undoing--recovering-changes)
6. [Merge & Rebase Workflows](#6-merge--rebase-workflows)
7. [Commit Management](#7-commit-management)
8. [Inspection & Debugging](#8-inspection--debugging)
9. [Tags & Releases](#9-tags--releases)
10. [Advanced Git](#10-advanced-git)
11. [Common Real-World Git Scenarios](#11-common-real-world-git-scenarios)
12. [Important Comparisons](#12-important-comparisons)
13. [Git Mental Model](#13-git-mental-model)
14. [⚠️ Safety Warnings — Destructive Commands](#️-safety-warnings--destructive-commands)
15. [Git Rules of Thumb](#15-git-rules-of-thumb)
16. [Which Command Should I Use?](#16-which-command-should-i-use)
17. [Git Quick Reference](#17-git-quick-reference)

---

## 1. Git Basics

### `git --version`

- **What it does:** Prints the installed Git version.
- **How to use it:**
  ```bash
  git --version
  ```
- **Example output:**
  ```text
  $ git --version
  git version 2.50.1
  ```
- **Result:** Confirms Git is installed and shows the version.
- **When to use it:** Confirming Git is installed, or checking you meet a minimum version required by a tool or tutorial.
- **Important notes:** Very old Git versions may lack newer commands like `git switch` or `git restore` — worth checking if a tutorial's commands "don't exist" on your machine. If `git --version` prints an error like `'git' is not recognized`, Git isn't installed or isn't on your PATH.

### `git config`

- **What it does:** Reads or sets Git configuration values (user identity, editor, aliases, behavior defaults) at system, global (per-user), or local (per-repo) scope.
- **How to use it:**
  ```bash
  git config --global user.name "Your Name"
  git config --global user.email "you@example.com"
  git config --list
  ```
- **Example output** (`git config --list`, truncated):
  ```text
  $ git config --list
  user.name=Your Name
  user.email=you@example.com
  core.editor=code --wait
  init.defaultbranch=main
  ```
- **Result:** Your commits will now carry this identity and be attributed to you. Confirmed via `git config --list` or by checking the header of a new commit (`git show HEAD`).
- **When to use it:** First-time setup on a new machine, setting your commit identity, or customizing behavior (default branch name, editor, aliases) for one repo or all repos.
- **Common mistake:** Forgetting to set `user.name` / `user.email` — Git then refuses to commit with: `Author identity unknown`. Setting them is the fix:
  ```text
  $ git commit -m "First commit"
  *** Please tell me who you are.
  Run
    git config --global user.email "you@example.com"
    git config --global user.name "Your Name"
  to set your account's default identity.
  ```
- **Important notes:** Use `--global` for settings that should apply everywhere; omit it (or use `--local`) to scope a setting to the current repo only. `git config --list --show-origin` helps find which file a setting came from.

### `git init`

- **What it does:** Creates a new, empty Git repository in the current directory (adds a hidden `.git` folder).
- **How to use it:**
  ```bash
  git init
  git init my-project
  ```
- **Example output:**
  ```text
  $ git init
  hint: Using 'master' as the name for the initial branch...
  Initialized empty Git repository in /Users/you/project/.git/
  ```
- **Before:**
  ```text
  WD:       your files, untracked
  Local:    no repository yet (no .git folder)
  ```
- **Command:**
  ```bash
  git init
  ```
- **After:**
  ```text
  WD:       unchanged — your files are untouched
  Local:    .git/ created — empty repo, no commits yet
  ```
  `git status` now reports every file as untracked:
  ```text
  $ git status
  On branch main
  No commits yet

  Untracked files:
    (use "git add <file>..." to include in what will be committed)
      index.html
      style.css
  ```
- **Result:** The folder is now a Git repository. Nothing is tracked or committed yet — you still need `git add` and `git commit`.
- **When to use it:** Starting version control on a brand-new project, or turning an existing folder of files into a Git repo.
- **Important notes:** Running it twice in the same folder is harmless (it won't wipe existing history), but double-check you're in the right directory — you don't want a stray `.git` folder nested inside another repo. Use `git init -b main` (or set `init.defaultbranch=main`) to avoid the `master` naming hint.

### `git clone`

- **What it does:** Copies an existing remote repository (and its full history) to your machine, automatically setting up a remote called `origin`.
- **How to use it:**
  ```bash
  git clone https://github.com/user/repo.git
  git clone https://github.com/user/repo.git my-folder-name
  ```
- **Example output:**
  ```text
  $ git clone https://github.com/user/repo.git
  Cloning into 'repo'...
  remote: Enumerating objects: 42, done.
  remote: Counting objects: 100% (42/42), done.
  remote: Compressing objects: 100% (30/30), done.
  remote: Total 42 (delta 8), reused 42 (delta 8), pack-reused 0
  Receiving objects: 100% (42/42), done.
  Resolving deltas: 100% (8/8), done.
  ```
- **After:**
  ```text
  New folder:  repo/
  Local:       full history of the project
  Remote:      origin → https://github.com/user/repo.git
  WD:          the latest version of every file, checked out
  ```
- **Result:** A new folder (`repo/`) contains all files and the full history, with `origin` already configured.
- **When to use it:** Starting work on a project that already exists elsewhere (GitHub, GitLab, a teammate's server, etc.).
- **Important notes:** By default this downloads the entire history. For very large repos, see shallow clones (`--depth`) in [Advanced Git](#10-advanced-git).

### `git status`

- **What it does:** Shows the current state of the working directory and staging area — which files are modified, staged, or untracked.
- **How to use it:**
  ```bash
  git status
  git status -s   # short format
  ```
- **Example output** (after editing `index.html`):
  ```text
  $ git status
  On branch main
  Your branch is up to date with 'origin/main'.

  Changes not staged for commit:
    modified:   index.html

  no changes added to commit (use "git add" and/or "git commit -a")
  ```
- **Example output** (short format):
  ```text
  $ git status -s
   M index.html
  ?? new-file.txt       # M = modified, ?? = untracked
  ```
- **Example output** (clean):
  ```text
  $ git status
  On branch main
  Your branch is up to date with 'origin/main'.

  nothing to commit, working tree clean
  ```
- **Result:** Tells you exactly which files are modified (`M`), staged, or untracked (`??`), and whether you're ahead of or behind your remote-tracking branch.
- **When to use it:** Constantly. Run it before and after almost every Git operation to know exactly what state you're in.
- **Important notes:** It never changes anything — completely safe to run as often as you like. `git status` is read-only: nothing about the repo changes after running it.

### `git add`

- **What it does:** Moves changes from the working directory into the staging area, marking them to be included in the next commit.
- **How to use it:**
  ```bash
  git add file.txt
  git add .
  git add -p        # interactively stage parts of a file
  ```
- **Before:**
  ```text
  WD:       index.html → modified
  Staging:  empty
  Local:    previous commit
  ```
- **Command:**
  ```bash
  git add index.html
  ```
- **Example output:** `git add` normally prints **no output** on success — it succeeds silently. The result is visible in `git status`.
- **After (what `git status` now shows):**
  ```text
  $ git status
  On branch main

  Changes to be committed:
    modified:   index.html

  WD:       clean (index.html changes now staged)
  Staging:  index.html → staged
  Local:    unchanged
  ```
- **Result:** `index.html` is now prepared to be included in the next commit.
- **When to use it:** After editing files, when you're ready to include those changes in your next commit.
- **Common mistake:** `git add .` stages everything in the current directory — including files you didn't mean to include:
  ```bash
  git add .        # ⚠️ accidentally staged .env
  ```
  Fix it by unstaging:
  ```bash
  git restore --staged .env
  ```
- **When NOT to use it:** Don't blanket-`git add .` when your working directory contains secrets, build artifacts, or unrelated changes — stage specific files instead.
- **Important notes:** `git add -p` is great for splitting unrelated changes into separate commits.

### `git commit`

- **What it does:** Saves the currently staged changes as a new snapshot (commit) in the local repository's history.
- **How to use it:**
  ```bash
  git commit -m "Add login form validation"
  git commit -am "Fix typo in header"   # stage tracked changes + commit
  ```
- **Before:**
  ```text
  WD:       clean
  Staging:  index.html → staged
  Local:    AB12CD3 (last commit)
  ```
- **Command:**
  ```bash
  git commit -m "Add login form validation"
  ```
- **Example output:**
  ```text
  $ git commit -m "Add login form validation"
  [main 9f3a1b2] Add login form validation
   1 file changed, 12 insertions(+)
  ```
- **After:**
  ```text
  WD:       clean
  Staging:  empty
  Local:    9f3a1b2 (new commit) → now on top of AB12CD3

  $ git status
  On branch main
  nothing to commit, working tree clean
  ```
- **Result:** The staged changes are saved permanently as a new commit `9f3a1b2` on the current branch.
- **When to use it:** After staging a logically complete, meaningful set of changes.
- **Common mistake:** Committing without `-m` opens an editor (and if you forget `-M`, your message defaults to `Merge: ...` or the editor's placeholder). If nothing is staged, Git will refuse:
  ```text
  $ git commit -m "WIP"
  On branch main
  nothing to commit, working tree clean
  ```
- **Important notes:** `-a` only stages files Git already tracks — it won't pick up new (untracked) files. Write clear, specific commit messages; "fix stuff" helps nobody later, including you.

### `git log`

- **What it does:** Shows the commit history for the current branch.
- **How to use it:**
  ```bash
  git log
  git log -5              # last 5 commits
  git log --author="Omkar"
  ```
- **Example output:**
  ```text
  $ git log --oneline
  9f3a1b2 Add login form validation
  AB12CD3 Fix navbar alignment
  3c4d5e6 Initial commit
  ```
- **What you will see:** Newest commit at the top, oldest at the bottom. Full `git log` adds author, date, and the complete commit message per entry.
- **Result:** You can see what has been committed, when, and by whom.
- **When to use it:** Reviewing history, finding a specific commit, understanding what changed and when.
- **Important notes:** Default output is verbose. See `git log --oneline` and `git log --graph` in [Inspection & Debugging](#8-inspection--debugging) for more scannable views. `git log` is read-only.

### `git diff`

- **What it does:** Shows line-by-line differences between two states — by default, working directory vs. the staging area (i.e., unstaged changes).
- **How to use it:**
  ```bash
  git diff                # unstaged changes
  git diff file.txt       # unstaged changes in one file
  git diff HEAD           # all changes vs. last commit (staged + unstaged)
  ```
- **Example output** (after editing `index.html`):
  ```diff
  $ git diff
  diff --git a/index.html b/index.html
  index 1234567..89abcde 100644
  --- a/index.html
  +++ b/index.html
  @@ -1,5 +1,5 @@
  -<h1>Hello</h1>
  +<h1>Hello World</h1>
   <p>Welcome</p>
  ```
  Lines starting with `-` were removed; lines with `+` were added. (Hashes shown are examples only — yours will differ.)
- **What you will see:** The `--- a/` line is the "before" version, `+++ b/` is the "after". `@@ -1,5 +1,5 @@` says which lines are shown.
- **Result:** A precise preview of what has changed — nothing is modified by running it. `git diff` is read-only.
- **When to use it:** Reviewing exactly what you've changed before staging or committing.
- **Important notes:** For staged-only changes, use `git diff --staged` (covered in [Inspection & Debugging](#8-inspection--debugging)).

### `git show`

- **What it does:** Displays the details and diff of a single commit (or other Git object like a tag).
- **How to use it:**
  ```bash
  git show HEAD
  git show a1b2c3d
  ```
- **Example output** (truncated):
  ```diff
  $ git show HEAD
  commit a1b2c3d
  Author: Your Name <you@example.com>
  Date:   Mon Aug 12 10:00:00 2026 +0200

      Add login form validation

  diff --git a/src/login.js b/src/login.js
  new file mode 100644
  index 0000000..3c4d5e6
  --- /dev/null
  +++ b/src/login.js
  @@ -0,0 +1,23 @@
  +function validateLogin(form) {
  ```
- **What you will see:** The commit header (hash, author, date, message) followed by the commit's diff — what it added and removed.
- **Result:** You inspect exactly what a specific commit changed, without affecting your working directory. Read-only.
- **When to use it:** Inspecting exactly what a specific commit changed, without affecting your working directory.
- **Important notes:** Works on tags and blobs too, not just commits — e.g., `git show v1.0.0`.

---

## 2. Working with Changes

### `git restore`

- **What it does:** Restores files in the working directory (and optionally the staging area) to a previous state — the modern replacement for many `git checkout` file-level uses.
- **How to use it:**
  ```bash
  git restore file.txt              # discard unstaged changes to a file
  git restore --staged file.txt     # unstage a file (keep the edits)
  ```
- **Discard unstaged edits — Before:**
  ```text
  WD:       index.html → modified (unedited changes present)
  Staging:  empty
  Local:    last commit (index.html as committed)
  ```
- **Command:**
  ```bash
  git restore index.html
  ```
- **After:**
  ```text
  WD:       clean — index.html back to its committed state
  Staging:  empty
  Local:    unchanged

  $ git status
  On branch main
  nothing to commit, working tree clean
  ```
- **Unstage a file — Before:**
  ```text
  Staging:  index.html → staged
  WD:       index.html → no further edits
  ```
- **Command:**
  ```bash
  git restore --staged index.html
  ```
- **Example output:** No output on success — verify with `git status`.
- **After:**
  ```text
  Staging:  empty
  WD:       index.html → modified (edits still on disk)

  $ git status
  Changes not staged for commit:
    modified:   index.html
  ```
- **Result:** The file is back to its committed state (or, with `--staged`, un-staged but untouched on disk).
- **When to use it:** Discarding local edits you don't want, or unstaging a file you added by mistake.
- **When NOT to use it:** Don't use plain `git restore file.txt` to "unstage" — that *discards* your edits. Use `--staged` to keep them.
- **Important notes:** ⚠️ `git restore file.txt` permanently discards unstaged edits to that file — there's no undo once it runs (unless the content happens to still be cached elsewhere, e.g., an editor's local history).

### `git reset`

- **What it does:** Moves the current branch pointer (and optionally the staging area and working directory) to a different commit. Behavior depends heavily on the flag used — see [Commit Management](#7-commit-management) for `--soft` / `--mixed` / `--hard`.
- **How to use it:**
  ```bash
  git reset file.txt        # unstage a file, same effect as `restore --staged`
  git reset HEAD~1          # move branch back one commit (mixed by default)
  ```
- **Unstage a file — Before:**
  ```text
  Staging:  index.html → staged
  WD:       index.html → no further edits
  ```
- **Command:**
  ```bash
  git reset index.html
  ```
- **After:**
  ```text
  Staging:  empty
  WD:       index.html → modified (still on disk)
  ```
  `git status` now reports `index.html` as "*Changes not staged for commit*".
- **Result:** The file is unstaged but its edits remain.
- **When to use it:** Unstaging files, or rewinding your branch's history (locally).
- **When NOT to use it:** Avoid resetting commits that have already been pushed and pulled by others — use `git revert` instead (see [Undoing & Recovering Changes](#5-undoing--recovering-changes)).
- **Important notes:** ⚠️ Without `--soft`, `reset` alters your staging area; with `--hard` it also wipes working directory changes.

### `git clean`

- **What it does:** Deletes untracked files (files Git isn't tracking at all) from the working directory.
- **How to use it:**
  ```bash
  git clean -n     # dry run — preview what would be deleted
  git clean -fd    # actually delete untracked files and directories
  ```
- **Example output** (`git clean -n`):
  ```text
  $ git clean -n
  Would remove build/
  Would remove tmp-cache.dat
  ```
- **Before:**
  ```text
  WD:       untracked clutter (build/, tmp-cache.dat) sitting around
  ```
- **Command:**
  ```bash
  git clean -fd
  ```
- **After:**
  ```text
  WD:       build/ and tmp-cache.dat are gone; git status no longer lists them
  ```
  (Running `git clean -fd` yourself produces no per-file output; it just deletes. `-n` is the only way to preview.)
- **Result:** Your working directory is free of untracked clutter. ⚠️ The deleted files are gone permanently — Git never tracked them.
- **When to use it:** Clearing out build artifacts, stray temp files, or other untracked clutter before a clean build or commit.
- **When NOT to use it:** Never when you have untracked files you still want (e.g., a new file you haven't committed yet, a local `.env`). `git clean` won't spare them.
- **Important notes:** ⚠️ `git clean -fd` **permanently deletes** untracked files — they are not recoverable via Git since they were never committed. Always run `git clean -n` first to preview.

### `git rm`

- **What it does:** Removes a file from both the working directory and the staging area (schedules its deletion for the next commit).
- **How to use it:**
  ```bash
  git rm old-file.txt
  git rm --cached secrets.env   # untrack the file but keep it on disk
  ```
- **Example output:**
  ```text
  $ git rm old-file.txt
  rm 'old-file.txt'
  ```
- **After:**
  ```text
  WD:       old-file.txt deleted from disk
  Staging:  old-file.txt → staged for deletion in the next commit
  ```
  ```text
  $ git status
  Changes to be committed:
    deleted:    old-file.txt
  ```
- **Result:** The deletion is staged; the next `git commit` records the file as removed.
- **When to use it:** Deleting a file as part of a tracked change (rather than deleting it in your file explorer and then having to `git add` the deletion separately).
- **Common mistake:** Deleting a tracked file in your editor/explorer, forgetting to stage the deletion, then committing — the file stays tracked. Fix: `git add old-file.txt` or `git rm old-file.txt`.
- **Important notes:** `--cached` is the go-to option when you accidentally committed something like a `.env` file and want Git to stop tracking it without deleting it locally. Remember to add it to `.gitignore` too.

### `git mv`

- **What it does:** Renames or moves a file and stages that change in one step.
- **How to use it:**
  ```bash
  git mv old-name.js new-name.js
  ```
- **Example output:**
  ```text
  $ git mv old-name.js new-name.js
  ```
  (No output — the rename is staged. `git status` shows it as a rename.)
- **After:**
  ```text
  $ git status
  Changes to be committed:
    renamed:    old-name.js -> new-name.js
  ```
- **Result:** The rename is staged and will be recorded in the next commit.
- **When to use it:** Renaming or relocating tracked files.
- **Important notes:** Equivalent to `mv` + `git rm` + `git add`, just more convenient. Git detects renames automatically in most cases even with a plain `mv`, but `git mv` guarantees it's staged correctly right away.

### `git stash`

- **What it does:** Temporarily shelves (saves and removes) uncommitted changes, giving you a clean working directory.
- **How to use it:**
  ```bash
  git stash
  git stash push -m "WIP: folder modal styling"
  ```
- **Before:**
  ```text
  WD:       index.html → modified, modal.css → modified
  Staging:  empty (or some staged files)
  ```
- **Command:**
  ```bash
  git stash
  ```
- **Example output:**
  ```text
  $ git stash
  Saved working directory and index state WIP on main: 3c4d5e6 Add login form validation
  ```
- **After:**
  ```text
  WD:       clean (changes safely shelved)
  Staging:  empty
  Stash:    stash@{0} — holds your WIP

  $ git status
  On branch main
  nothing to commit, working tree clean
  ```
- **Result:** Your uncommitted work is safely saved as `stash@{0}` and your directory is clean, ready for another branch or a pull.
- **When to use it:** You need to switch branches or pull changes but aren't ready to commit what you're working on.
- **Common mistake:** Stash, then later `git stash drop` the wrong stash before reapplying your work — you lose it. Check `git stash list` before dropping.
- **Important notes:** By default, untracked files are **not** stashed — add `-u` to include them. Stashes are local only; they aren't pushed to remotes.

### `git stash pop`

- **What it does:** Reapplies the most recent stash to your working directory **and removes it** from the stash list.
- **How to use it:**
  ```bash
  git stash pop
  git stash pop stash@{2}
  ```
- **Example output:**
  ```text
  $ git stash pop
  On branch feature/other
  Changes not staged for commit:
    modified:   index.html

  Dropped refs/stash@{0} (2f1a4b9c3e8a1b2f...)
  ```
- **After:**
  ```text
  WD:       your shelved edits are back
  Stash:    stash@{0} removed from the list
  ```
- **Result:** Your interrupted work is restored and the stash entry is cleaned up.
- **When to use it:** You've finished the interruption and want your shelved work back, and you're done with that stash entry.
- **Important notes:** ⚠️ Can produce merge conflicts if the working directory has since diverged from when the stash was made — resolve conflicts like a normal merge conflict. At that point the stash isn't dropped automatically.

### `git stash apply`

- **What it does:** Reapplies a stash to your working directory but **keeps it** in the stash list.
- **How to use it:**
  ```bash
  git stash apply
  git stash apply stash@{1}
  ```
- **Example output:**
  ```text
  $ git stash apply
  On branch main
  Changes not staged for commit:
    modified:   index.html
  ```
- **After:**
  ```text
  WD:       shelved edits restored
  Stash:    unchanged — stash@{0} still exists
  ```
- **Result:** Your changes are back, and the stash remains as a backup.
- **When to use it:** You want to reuse the same stashed changes in more than one branch, or you're not fully confident yet and want a safety net before dropping the stash.
- **Important notes:** You'll need to `git stash drop` manually afterward if you no longer need it.

### `git stash list`

- **What it does:** Lists all currently saved stashes.
- **How to use it:**
  ```bash
  git stash list
  ```
- **Example output:**
  ```text
  $ git stash list
  stash@{0}: WIP on feature/other: 3c4d5e6 Add login form validation
  stash@{1}: WIP on main: 9f3a1b2 Fix navbar alignment
  ```
- **Result:** Shows every saved stash, newest first.
- **When to use it:** Checking what you've stashed before applying, popping, or dropping.
- **Important notes:** Stashes are shown newest-first as `stash@{0}`, `stash@{1}`, etc. Old forgotten stashes are a common source of "wait, where did that code go?" confusion.

### `git stash drop`

- **What it does:** Deletes a specific stash entry without applying it.
- **How to use it:**
  ```bash
  git stash drop stash@{0}
  ```
- **Example output:**
  ```text
  $ git stash drop stash@{0}
  Dropped refs/stash@{0} (2f1a4b9c3e8a1b2f...)
  ```
- **After:** The stash entry is gone; `git stash list` no longer shows it.
- **Result:** One fewer stash to keep track of. ⚠️ The shelved changes are gone.
- **When to use it:** Cleaning up stashes you no longer need (e.g., after confirming you already applied them, or you decided the changes weren't worth keeping).
- **Important notes:** ⚠️ Dropped stashes are hard to recover — though if truly needed, `git fsck --unreachable` can sometimes locate the dangling commit shortly after dropping.

---

## 3. Branches

Think of branches as labels that point at commits. A history with a feature branch looks like an ASCII diagram:

```text
A---B---C  main
     \
      D---E  feature
```

- Commits `A`, `B`, `C` are on `main`.
- `feature` branched off `B` and added `D` and `E`.
- Both labels are "active" pointers — Git doesn't duplicate any files when you branch; it's just a pointer.

### `git branch`

- **What it does:** Lists, creates, or deletes branches. With no arguments, lists local branches.
- **How to use it:**
  ```bash
  git branch                # list local branches
  git branch feature/login  # create a new branch (doesn't switch to it)
  git branch -a             # list local + remote-tracking branches
  ```
- **Example output** (listing):
  ```text
  $ git branch
  * main
    bugfix/navbar
    feature/login
  ```
  The `*` marks the branch you're currently on. An `ahead of 'origin/main'` note may appear under `git status` when your branch has unpushed commits.
- **After creating** `git branch feature/login`:
  ```text
  Local:    feature/login now points at the same commit as main
  WD:       unchanged — you are still on main
  ```
  ```text
  $ git branch
  * main
    feature/login
  ```
- **Result:** A new branch label exists; your checkout is unchanged until you `git switch` to it.
- **When to use it:** Checking what branches exist, or creating a branch without immediately switching to it.
- **Important notes:** Creating a branch is cheap and instant in Git — there's rarely a reason to avoid branching for a new piece of work.

### `git switch`

- **What it does:** Switches the working directory to an existing branch. A modern, more focused alternative to `git checkout` for branch switching.
- **How to use it:**
  ```bash
  git switch main
  git switch feature/login
  ```
- **Before:**
  ```text
  WD:       clean
  Current:  main
  Target:   feature/login
  ```
- **Command:**
  ```bash
  git switch feature/login
  ```
- **After:**
  ```text
  WD:       now contains feature/login's files
  Current:  feature/login
  Local:    branch pointer moved — HEAD is at feature/login
  ```
  ```text
  $ git status
  On branch feature/login
  ```
- **Result:** You're now working on `feature/login`.
- **Common mistake:** Switching with uncommitted changes gives a warning:
  ```text
  $ git switch main
  error: Your local changes to the following files would be overwritten by checkout:
          index.html
  Please commit your changes or stash them before you switch branches.
  ```
- **When to use it:** Moving between existing branches.
- **When NOT to use it:** Don't switch with uncommitted changes you care about — the switch may be refused, or (if the changes don't conflict) carried over unexpectedly. Stash or commit first.
- **Important notes:** Requires a clean-enough working directory (or non-conflicting changes) — Git will warn you if switching would overwrite uncommitted work. Stash or commit first if needed.

### `git checkout`

- **What it does:** The older, multi-purpose command for switching branches, restoring files, or checking out a specific commit/tag (detached HEAD).
- **How to use it:**
  ```bash
  git checkout main
  git checkout a1b2c3d       # detached HEAD at a specific commit
  git checkout -- file.txt   # discard changes to a file (legacy syntax)
  ```
- **Example output** (checking out a commit → detached HEAD):
  ```text
  $ git checkout a1b2c3d
  Note: switching to 'a1b2c3d'.

  You are in 'detached HEAD' state. You can look around, make experimental
  changes and commit them, and you can discard any commits you make in this
  state without impacting any branches by switching back...
  ```
- **Result:** Your files now reflect commit `a1b2c3d`, and `HEAD` is detached (not on any branch).
- **When to use it:** Still common in older scripts/tutorials, or for checking out a specific commit/tag to inspect it.
- **Important notes:** Because it does so many different things based on its arguments, it's easy to use by mistake (e.g., typo a branch name and accidentally checkout a file). `git switch` and `git restore` split its responsibilities to reduce that risk — prefer them for everyday use.

### `git switch -c`

- **What it does:** Creates a new branch and switches to it in one step.
- **How to use it:**
  ```bash
  git switch -c feature/login
  git switch -c hotfix/nav-bug main   # branch off a specific starting point
  ```
- **Before:**
  ```text
  A---B---C  main ← HEAD
  ```
- **Command:**
  ```bash
  git switch -c feature/login
  ```
- **After:**
  ```text
  A---B---C  main
            feature/login ← HEAD
  ```
  ```text
  $ git status
  On branch feature/login
  nothing to commit, working tree clean
  ```
- **Result:** A new branch diverges from `main` at `C` and you're on it, ready to make commits.
- **When to use it:** Starting new work — this is the standard way to begin any feature, fix, or experiment.
- **Important notes:** Equivalent to `git checkout -b <name>` in older syntax.

### `git branch -d`

- **What it does:** Deletes a local branch, but only if it has already been fully merged.
- **How to use it:**
  ```bash
  git branch -d feature/login
  ```
- **Example output:**
  ```text
  $ git branch -d feature/login
  Deleted branch feature/login (was 9f3a1b2).
  ```
- **After:**
  ```text
  Local:    feature/login label gone — its commits are reachable from main
  ```
- **Result:** A merged branch is cleaned up with no loss of history.
- **When to use it:** Cleaning up branches after their work has been merged into `main` (or another target branch).
- **Important notes:** Git will refuse if the branch has unmerged commits — that's the safety this flag provides over `-D`:
  ```text
  error: The branch 'feature/login' is not fully merged.
  If you are sure you want to delete it, run 'git branch -D feature/login'.
  ```

### `git branch -D`

- **What it does:** Force-deletes a local branch, even if it has unmerged commits.
- **How to use it:**
  ```bash
  git branch -D experimental/old-idea
  ```
- **Example output:**
  ```text
  $ git branch -D experimental/old-idea
  Deleted branch experimental/old-idea (was 8d1f2a3).
  ```
- **Result:** The branch label is removed. ⚠️ Commits that existed **only** on that branch can no longer be reached easily.
- **When to use it:** Discarding a branch you're certain you no longer need, merged or not.
- **When NOT to use it:** If there's any chance you'll want the branch's commits later — use `-d` first, or make sure `git reflog` still has them.
- **Important notes:** ⚠️ Any commits that exist **only** on that branch become unreachable and are hard to recover (though `git reflog` can sometimes help shortly after).

### `git branch -m`

- **What it does:** Renames a branch.
- **How to use it:**
  ```bash
  git branch -m old-name new-name
  git branch -m new-name              # rename current branch
  ```
- **Example output:**
  ```text
  $ git branch -m typo-featur typo-feature
  ```
  (No output on success — verify with `git branch`.)
- **Result:** The branch is renamed locally; its commits don't change.
- **When to use it:** Fixing a typo in a branch name, or renaming to match a naming convention.
- **Important notes:** If the branch has already been pushed, you'll need to push the new name and delete the old one on the remote too (`git push origin --delete old-name` then `git push -u origin new-name`).

### `git merge`

- **What it does:** Combines the history of another branch into your current branch, creating a new merge commit (unless a fast-forward is possible).
- **How to use it:**
  ```bash
  git switch main
  git merge feature/login
  ```
- **Before (a normal three-way merge — both branches moved):**
  ```text
  A---B---C  main
       \
        D---E  feature/login
  ```
- **Command:**
  ```bash
  git switch main
  git merge feature/login
  ```
- **Example output:**
  ```text
  $ git merge feature/login
  Merge made by the 'ort' strategy.
   app.js | 3 +++
   1 file changed, 3 insertions(+)
  ```
- **After:**
  ```text
  A---B---C-------M  main
       \         /
        D---E---/
  ```
  `M` is a new merge commit with two parents (the old `main` tip and `feature/login`'s tip).
- **Result:** `feature/login`'s work is now part of `main`.
- **When to use it:** Bringing completed work from a feature branch into `main` (or integrating any two branches) while preserving full branch history.
- **Important notes:** See the full [Merge & Rebase Workflows](#6-merge--rebase-workflows) section for fast-forward vs. normal merges, conflicts, and `--abort`.

### `git rebase`

- **What it does:** Replays your branch's commits on top of another branch's tip, producing a linear history instead of a merge commit.
- **How to use it:**
  ```bash
  git switch feature/login
  git rebase main
  ```
- **Before:**
  ```text
  A---B---C  main
       \
        D---E  feature/login
  ```
- **Command:**
  ```bash
  git switch feature/login
  git rebase main
  ```
- **After:**
  ```text
  A---B---C---D'---E'  feature/login
  ```
  ⚠️ `D'` and `E'` are **new commits with new hashes** — the originals `D` and `E` still exist but are orphaned.
- **Result:** Your feature commits now sit directly on top of the latest `main`, giving a straight line of history.
- **When to use it:** Keeping a feature branch's history clean and linear, or updating a feature branch with the latest `main` before merging.
- **When NOT to use it:** Never rebase commits that other people have already pulled and built on top of — the rewritten hashes will confuse and clobber their work. See [Merge & Rebase Workflows](#6-merge--rebase-workflows) for interactive rebase, conflicts, and recovery flags.
- **Important notes:** ⚠️ Rebasing rewrites commit hashes.

---

## 4. Remote Repositories

### `git remote`

- **What it does:** Manages the set of remote repositories your local repo knows about.
- **How to use it:**
  ```bash
  git remote
  ```
- **Example output:**
  ```text
  $ git remote
  origin
  ```
- **Result:** Lists the names of configured remotes (usually just `origin`).
- **When to use it:** Base command for listing, adding, renaming, or removing remotes (usually used with a subcommand — see below).
- **Important notes:** Most projects only need one remote, conventionally named `origin`.

### `git remote -v`

- **What it does:** Lists configured remotes along with their fetch/push URLs.
- **How to use it:**
  ```bash
  git remote -v
  ```
- **Example output:**
  ```text
  $ git remote -v
  origin  https://github.com/user/project.git (fetch)
  origin  https://github.com/user/project.git (push)
  ```
- **Result:** Confirms which URL this repo pushes to and fetches from.
- **When to use it:** Confirming which remote URL you're pushing to/pulling from — especially useful after cloning a fork or when something pushes to the wrong place.
- **Important notes:** Fetch and push URLs can technically differ (rare, but possible in advanced setups) — this command shows both.

### `git remote add`

- **What it does:** Registers a new remote repository under a given name.
- **How to use it:**
  ```bash
  git remote add origin https://github.com/user/repo.git
  git remote add upstream https://github.com/original-owner/repo.git
  ```
- **Example output:** No output on success — verify with `git remote -v`.
- **After:**
  ```text
  Remote:    origin → https://github.com/user/repo.git (now configured)

  $ git remote -v
  origin  https://github.com/user/repo.git (fetch)
  origin  https://github.com/user/repo.git (push)
  ```
- **Result:** The remote is registered; you can now `git push`/`git fetch` to it.
- **When to use it:** Connecting a local repo (created with `git init`) to a remote for the first time, or adding a second remote (e.g., `upstream` for a forked repo).
- **Important notes:** The name (`origin`, `upstream`, etc.) is just a label — there's nothing magic about the word "origin" except convention. Adding a remote doesn't download anything; run `git fetch` for that.

### `git fetch`

- **What it does:** Downloads commits, branches, and tags from a remote **without** merging them into your local branches.
- **How to use it:**
  ```bash
  git fetch origin
  git fetch --all
  ```
- **Example output:**
  ```text
  $ git fetch origin
  remote: Enumerating objects: 3, done.
  remote: Counting objects: 100% (3/3), done.
  remote: Compressing objects: 100% (2/2), done.
  remote: Total 3 (delta 0), reused 3 (delta 0), pack-reused 0
  Unpacking objects: 100% (3/3), done.
  From https://github.com/user/project.git
     9f3a1b2..d4e5f6g  main       -> origin/main
  ```
- **Before:**
  ```text
  Local:    9f3a1b2 on main
  Remote:   has d4e5f6g (a teammate pushed)
  ```
- **Command:**
  ```bash
  git fetch origin
  ```
- **After:**
  ```text
  Local:    9f3a1b2 on main — your working tree is UNCHANGED
  Remote:   origin/main updated to d4e5f6g
  ```
  ```
  $ git status
  On branch main
  Your branch is behind 'origin/main' by 1 commit, and can be fast-forwarded.
    (use "git pull" to update your local branch)
  ```
- **Result:** Git now knows about the remote's new commits, but your files and local branch are untouched.
- **When to use it:** Checking what's new on the remote before deciding how to integrate it, or updating remote-tracking branches for comparison (`git log main..origin/main`).
- **Important notes:** Completely safe — it never touches your working directory or local branches, only updates remote-tracking references like `origin/main`.

### `git pull`

- **What it does:** Downloads changes from a remote **and** integrates them into your current branch (fetch + merge, or fetch + rebase with `--rebase`).
- **How to use it:**
  ```bash
  git pull origin main
  git pull --rebase origin main
  ```
- **Example output** (when you have no local commits to lose):
  ```text
  $ git pull origin main
  Updating 9f3a1b2..d4e5f6g
  Fast-forward
   index.html | 4 +++-
   1 file changed, 3 insertions(+), 1 deletion(-)
  ```
- **Before:**
  ```text
  Local:    9f3a1b2 on main
  Remote:   d4e5f6g on origin/main
  ```
- **Command:**
  ```bash
  git pull origin main
  ```
- **After:**
  ```text
  Local:    d4e5f6g on main — your branch now includes the remote's new commits
  WD:       updated to the new state
  ```
- **Result:** Your local branch is now up to date with the remote.
- **When to use it:** Bringing your local branch up to date with the remote in one step.
- **Important notes:** Because it merges (or rebases) automatically, it can produce conflicts you weren't expecting. If you want more control, `git fetch` followed by a manual `git merge`/`git rebase` is safer.

### `git push`

- **What it does:** Uploads your local commits to a remote repository.
- **How to use it:**
  ```bash
  git push origin main
  git push
  ```
- **Example output:**
  ```text
  $ git push origin main
  Enumerating objects: 5, done.
  Counting objects: 100% (5/5), done.
  Writing objects: 100% (3/3), 1.2 KiB | 1.20 MiB/s, done.
  Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
  To https://github.com/user/project.git
     9f3a1b2..d4e5f6g  main -> main
  ```
- **Before:**
  ```text
  Local:   d4e5f6g on main
  Remote:  9f3a1b2 on origin/main
  ```
- **Command:**
  ```bash
  git push origin main
  ```
- **After:**
  ```text
  Local:   d4e5f6g on main
  Remote:  d4e5f6g on origin/main — now in sync
  ```
  ```text
  $ git status
  On branch main
  Your branch is up to date with 'origin/main'.
  ```
- **Result:** Your commits are now shared (or backed up) on the remote.
- **Common mistake:** Pushing when someone else pushed first — Git refuses with a non-fast-forward error:
  ```text
  ! [rejected]        main -> main (non-fast-forward)
  error: failed to push some refs to 'https://github.com/user/project.git'
  hint: Updates were rejected because the tip of your current branch is behind
  hint: its remote counterpart. Integrate the remote changes first
  hint: (e.g. 'git push ... && git pull') before pushing again.
  ```
  Fix: `git pull --rebase origin main` (or just `git pull`), resolve any conflicts, then push again.
- **When to use it:** Sharing your committed work with others, or backing it up remotely.
- **When NOT to use it:** Don't reach for `--force` to get out of the rejected-push situation — that's how teammates' work gets clobbered.
- **Important notes:** Git will refuse a push that isn't a fast-forward on the remote (i.e., someone else pushed first) — pull/rebase first rather than force-pushing by reflex.

### `git push -u`

- **What it does:** Pushes a branch and sets up tracking, so future `git push`/`git pull` on that branch don't need the remote/branch name specified.
- **How to use it:**
  ```bash
  git push -u origin feature/login
  ```
- **Example output:**
  ```text
  $ git push -u origin feature/login
  ...
  * [new branch]      feature/login -> feature/login
  branch 'feature/login' set up to track 'origin/feature/login'.
  ```
- **After:**
  ```text
  Remote:   feature/login now exists there
  Tracking: feature/login → origin/feature/login
  ```
  From now on, plain `git push` / `git pull` on this branch work without arguments.
- **Result:** The branch is on the remote and linked to it.
- **When to use it:** The first time you push a newly created local branch.
- **Important notes:** `-u` is short for `--set-upstream`. After this, plain `git push` and `git pull` on that branch know where to go.

### `git push --force-with-lease`

- **What it does:** Force-pushes, but refuses if the remote branch has commits you haven't seen yet (i.e., someone else pushed since your last fetch).
- **How to use it:**
  ```bash
  git push --force-with-lease origin feature/login
  ```
- **Example output:**
  ```text
  $ git push --force-with-lease origin feature/login
  ...
   + 9f3a1b2...d4e5f6g feature/login -> feature/login (forced update)
  ```
  (A `+` and "(forced update)" indicate history was rewritten on the remote.)
- **After:** The remote branch now matches your rewritten local history.
- **Result:** Your rewritten feature branch is uploaded — safely, because Git confirmed no one else pushed since your last fetch.
- **When to use it:** Pushing after rewriting history (e.g., `rebase` or `commit --amend`) on a branch you're confident you "own," such as your own feature branch.
- **Common mistake:** Pushing with plain `--force` clobbers a teammate's unseen work silently. `--force-with-lease` refuses instead:
  ```text
  ! [rejected]        feature/login -> feature/login (stale info)
  ```
- **When NOT to use it:** Never on a shared branch (like `main`) where others may be building on top of history you're rewriting.
- **Important notes:** ⚠️ Still overwrites remote history — just safer than plain `--force` because it won't silently clobber a teammate's unseen work. See the [force vs. force-with-lease comparison](#12-important-comparisons) below.

### Tracking Branches & Upstream

- **What it does:** A "tracking branch" (or "upstream branch") is a local branch linked to a specific remote branch, so Git knows what to compare against and where plain `push`/`pull`/`status` should operate.
- **How to use it:**
  ```bash
  git branch -u origin/main            # set upstream for current branch
  git branch -vv                       # see tracking relationships
  git push -u origin feature/login     # push + set tracking in one step
  ```
- **Example output** (`git branch -vv`):
  ```text
  $ git branch -vv
  * main            9f3a1b2 [origin/main] Add login form validation
    feature/login   d4e5f6g [origin/feature/login: ahead 2] Add favorites page
  ```
- **What you will see:** `[origin/main]` means `main` tracks `origin/main`; `ahead 2` means your local branch has 2 commits the remote doesn't. `git status` shows the same sync info.
- **When to use it:** After creating a new branch you intend to push regularly, or when `git status` reports your branch is "ahead/behind" and you want to know relative to what.
- **Important notes:** `git status` and `git branch -vv` both show ahead/behind counts based on the tracking relationship — a fast way to sanity-check sync state before pushing or pulling.

---

## 5. Undoing & Recovering Changes

Git gives you several "undo" tools, and picking the right one depends on **where** the change lives: working directory, staging area, local history, or already-shared (pushed) history.

The undo commands move work in the **reverse** direction along the mental model:

```text
WD ──git add──▶ Staging ──git commit──▶ Local ──git push──▶ Remote
  ▲                ▲                        ▲                   ▲
git restore      git restore --staged     git reset          git revert
                / git reset <file>      git revert
```

| Command | Undoes | Rewrites history? | Safe on shared/pushed commits? |
|---|---|---|---|
| `git restore` | Uncommitted changes (WD / staging) | No (nothing was committed) | N/A — local only |
| `git reset` | Staging state, and optionally commits + WD | Yes, moves branch pointer | ❌ No — avoid on pushed commits |
| `git revert` | A specific commit's changes, via a new commit | No, adds new history | ✅ Yes — the standard safe undo |
| `git clean` | Untracked files | No (they were never tracked) | N/A — local only |
| `git reflog` | (Recovery tool, not an "undo" itself) | No | N/A — local only |

### `git revert`

- **What it does:** Creates a **new commit** that reverses the changes introduced by a specific earlier commit, leaving history intact.
- **How to use it:**
  ```bash
  git revert a1b2c3d
  git revert HEAD              # revert the most recent commit
  git revert --no-commit a1b2c3d..HEAD   # revert a range, stage only
  ```
- **Before:**
  ```text
  Local:    A --- B --- C
                    ↑
                   HEAD
  ```
  Commit `B` contains a bug you want to undo.
- **Command:**
  ```bash
  git revert B1B2C3D
  ```
- **Example output:**
  ```text
  $ git revert b1b2c3d
  [main e5f6a7b] Revert "Add broken coupon logic"
   1 file changed, 8 deletions(-)
  ```
- **After:**
  ```text
  Local:    A --- B --- C --- e5f6a7b
                            ↑
                           HEAD
  ```
  The new commit `e5f6a7b` reverses `B`'s changes. `B` and `C` stay exactly as they were.
- **Result:** The bad change is undone with an additive commit, and history is untouched — safe to push.
- **When to use it:** Undoing a commit that has already been pushed and possibly pulled by others — reverting is additive and doesn't rewrite history, so it's safe for shared branches.
- **Common mistake:** Reverting a merge commit without choosing a parent — Git errors:
  ```text
  error: commit x is a merge but no -m option was given.
  ```
  Fix: `git revert -m 1 <merge-hash>` (parent `1` = the branch you checked out when merging).
- **Important notes:** Reverting a merge commit requires specifying which parent to revert to (`-m 1`), which trips people up the first time.

### `git reflog`

- **What it does:** Shows a log of everywhere `HEAD` has pointed locally — commits, checkouts, resets, rebases — even ones no longer reachable from any branch.
- **How to use it:**
  ```bash
  git reflog
  git reset --hard HEAD@{2}   # jump back to a previous HEAD position
  ```
- **Example output:**
  ```text
  $ git reflog
  d4e5f6g HEAD@{0}: reset: moving to HEAD@{1}
  9f3a1b2 HEAD@{1}: commit: Add login form validation
  a1b2c3d HEAD@{2}: reset: moving to HEAD@{0}
  c1d2e3f HEAD@{3}: commit (initial): Initial commit
  ```
- **What you will see:** `HEAD@{n}` positions, each labeled with the action that moved there. `HEAD@{0}` is where you are now.
- **Result:** A breadcrumb trail of your local history — including commits you "lost" to a reset or a deleted branch.
- **When to use it:** Your local "safety net" — recovering a commit after a bad `reset --hard`, an accidental branch deletion, or a rebase that went wrong.
- **Common mistake:** Assuming reflog exists forever. It's a **recovery window**, not permanent storage:
  ```text
  $ git reflog
  ... (nothing left after the entry expired)
  ```
- **Important notes:** Reflog entries are local-only (not shared via push/clone) and eventually expire (default ~90 days for reachable entries, ~30 for unreachable).

### Practical Undo Examples

Each example follows the same shape: **Before → Command → After**.

**Undo unstaged changes to a file** (discard edits, keep them gone):
```bash
git restore file.txt
```
- Before: `WD: file.txt modified`/ After: `WD: clean — file.txt back to committed state`.

**Unstage a file** (keep the edits, just remove from staging):
```bash
git restore --staged file.txt
```
- Before: `Staging: file.txt staged` / After: `Staging: empty`, `WD: file.txt modified (edits kept)`.

**Undo the last commit but keep the changes** (uncommit, keep edits staged):
```bash
git reset --soft HEAD~1
```
- Before: `Local: A──B──C (HEAD at C)`, `WD: clean` / After: `Local: A──B (HEAD at B)`, `Staging: C's changes still staged`.

**Undo the last commit and remove the changes entirely:**
```bash
git reset --hard HEAD~1
```
> ⚠️ This permanently discards the commit's changes from your working directory too.

**Undo a commit that's already been pushed (safely):**
```bash
git revert <commit-hash>
git push origin main
```
- After: `Local: A──B──C──R (new revert commit)` — history intact and pushable.

**Recover a deleted or "lost" commit:**
```bash
git reflog                    # find the commit hash before it was lost
git checkout <commit-hash>    # inspect it
git branch recovered-work <commit-hash>   # or restore it onto a branch
```

**Recover from an accidental `git reset --hard`:**
```bash
git reflog                    # find HEAD@{n} from just before the reset
git reset --hard HEAD@{1}     # move back to that point
```

---

## 6. Merge & Rebase Workflows

**Fast-forward merge:** When your current branch has no new commits since the branch you're merging diverged from it, Git simply moves the branch pointer forward — no merge commit is created.
```bash
git switch main
git merge feature/login
```
- Before:
  ```text
  A---B  main
       \
        C---D  feature/login
  ```
- After:
  ```text
  A---B---C---D  main
  ```
- Example output:
  ```text
  $ git merge feature/login
  Updating AB12CD3..D4E5F6G
  Fast-forward
   login.js | 5 +++--
   1 file changed, 3 insertions(+), 2 deletions(-)
  ```

**Normal (three-way) merge:** When both branches have new commits, Git creates a dedicated merge commit with two parents, combining both histories.
```bash
git switch main
git merge feature/login   # creates a merge commit
```
- Before:
  ```text
  A---B---C  main
       \
        D---E  feature/login
  ```
- After:
  ```text
  A---B---C-------M  main
       \         /
        D---E---/
  ```
- Example output:
  ```text
  $ git merge feature/login
  Merge made by the 'ort' strategy.
   login.js | 10 ++++++----
   1 file changed, 6 insertions(+), 4 deletions(-)
  ```

**Merge conflicts:** Happen when the same lines (or a file's existence) were changed differently on both branches. Git pauses the merge and marks the conflicting files.
```text
$ git merge feature/login

Auto-merging app.js
CONFLICT (content): Merge conflict in app.js
Automatic merge failed; fix conflicts and then commit the result.
```
`git status` now shows the conflicted file:
```text
$ git status
On branch main
You have unmerged paths.
  (fix conflicts and run "git commit")

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   app.js
```
The file itself contains conflict markers:
```text
<<<<<<< HEAD
const port = 3000;
=======
const port = 5000;
>>>>>>> feature/login
```
- `<<<<<<< HEAD` … `=======` = your current branch's version.
- `=======` … `>>>>>>> feature/login` = the incoming branch's version.
- Choose the correct final content and **delete the markers**.

**Resolving conflicts:** Open each conflicted file, look for `<<<<<<<`, `=======`, `>>>>>>>` markers, edit to the correct final content, then stage and finish.
```bash
# edit the file(s) to resolve conflicts
git add src/app.js
git commit                # finishes the merge
```
```text
$ git commit
[main 7f8a9b0] Merge branch 'feature/login'
```
- After: `Local: A---B---C---M  main` — the merge commit records the resolution.

**`git merge --abort`**
- **What it does:** Cancels an in-progress merge and restores the pre-merge state.
- **How to use it:**
  ```bash
  git merge --abort
  ```
- **Example output:**
  ```text
  $ git merge --abort
  ```
  (No output on success — `git status` confirms the repo is back to normal.)
- **Result:** The repo returns to exactly how it was before the merge attempt; any conflicts and partial merges are discarded.
- **When to use it:** A merge conflict is messier than expected and you'd rather back out and try a different approach.
- **Important notes:** Only works while the merge is still in a conflicted, uncommitted state — not after you've completed it with `git commit`.

**Interactive rebase (`git rebase -i`):** Replays and lets you edit, reorder, squash, or drop commits one by one.
```bash
git rebase -i HEAD~4       # interactively edit the last 4 commits
```
- Example output (the editor you're dropped into):
  ```text
  pick 9f3a1b2 Add login form validation
  pick a1b2c3d Refactor error handling
  pick 8hf3k2s Fix heading styles
  pick m3x4n5p Update README
  ```
  Change the action word next to a commit (`pick`, `reword`, `squash`, `fixup`, `drop`, `edit`), save, and close the editor.
- After (if you squashed the last two):
  ```text
  pick 9f3a1b2 Add login form validation
  pick a1b2c3d Refactor error handling
  squash 8hf3k2s Fix heading styles
  squash m3x4n5p Update README
  ```
  → `a1b2c3d` and `8hf3k2s`/`m3x4n5p` combine into one commit.

**`git rebase --abort`**
- **What it does:** Cancels an in-progress rebase and restores the branch to its pre-rebase state.
- **How to use it:**
  ```bash
  git rebase --abort
  ```
- **When to use it:** A rebase hits conflicts you don't want to resolve right now, or you realize you rebased onto the wrong branch.

**`git rebase --continue`**
- **What it does:** Resumes a paused rebase after you've resolved the current conflict (and staged the fix).
- **How to use it:**
  ```bash
  git add resolved-file.txt
  git rebase --continue
  ```
- **Example output:**
  ```text
  $ git rebase --continue
  [detached HEAD 4ab5cd6] Refactor error handling
   1 file changed, 3 insertions(+)
  Successfully rebased and updated refs/heads/feature/login.
  ```
- **When to use it:** After fixing conflicts during a rebase, to move on to the next commit being replayed.

**`git rebase --skip`**
- **What it does:** Skips the commit currently being replayed entirely, discarding its changes.
- **How to use it:**
  ```bash
  git rebase --skip
  ```
- **When to use it:** The current commit's changes are no longer needed (e.g., already covered by an earlier change) and resolving its conflict isn't worthwhile.
- **Important notes:** ⚠️ The skipped commit's changes are dropped from the branch — make sure that's actually what you want.

**Squashing commits:** Combine several commits into one, usually via interactive rebase.
```bash
git rebase -i HEAD~3
# mark the 2nd and 3rd commits as "squash" (or "s") instead of "pick"
```

**Reordering commits:** In the interactive rebase editor, simply change the order of the lines — Git replays them in the new order.

**Editing commit history:** Mark a commit as `edit` in interactive rebase to pause there, make changes, then continue.
```bash
git rebase -i HEAD~3
# mark a commit as "edit"
# ... make your changes ...
git add .
git commit --amend
git rebase --continue
```
> ⚠️ Any of the interactive rebase operations above rewrite commit hashes for that commit and everything after it. Safe on local/unpushed commits; avoid on commits others have already pulled unless force-pushing is an agreed-upon part of your team's workflow.

---

## 7. Commit Management

### `git commit --amend`

- **What it does:** Rewrites the most recent commit — letting you change its message or fold additional staged changes into it.
- **How to use it:**
  ```bash
  git commit --amend -m "Fix typo in login error message"
  git add forgotten-file.txt
  git commit --amend --no-edit      # add a file without changing the message
  ```
- **Before:**
  ```text
  Local:    A --- B (last commit, wrong message / missing a file)
                  ↑
                 HEAD
  ```
- **Command:**
  ```bash
  git commit --amend -m "Fix typo in login error message"
  ```
- **Example output:**
  ```text
  $ git commit --amend -m "Fix typo in login error message"
  [main 9f3a1b2] Fix typo in login error message
   1 file changed, 2 insertions(+), 2 deletions(-)
  ```
- **After:**
  ```text
  Local:    A --- B' (B replaced; B' has the new message/hash)
                ↑
               HEAD
  ```
  ⚠️ `B` is gone and `B'` has a **new hash**.
- **Result:** Your last commit now correct — same content position, different hash.
- **When to use it:** Fixing a typo in the last commit message, or adding changes you forgot to include in the previous commit (before it's shared).
- **When NOT to use it:** Never amend commits others have already pulled — you'd rewrite history they're building on. `--no-edit` keeps the existing message.
- **Important notes:** ⚠️ Replaces the last commit with a brand-new commit (new hash).

### `git cherry-pick`

- **What it does:** Copies a specific commit from another branch onto your current branch.
- **How to use it:**
  ```bash
  git cherry-pick a1b2c3d
  git cherry-pick a1b2c3d d4e5f6g        # pick several commits in order
  ```
- **Before:**
  ```text
  A---B---C  main ← HEAD     D---E  feature (E = the bugfix you need)
  ```
- **Command:**
  ```bash
  git cherry-pick D4E5F6G
  ```
- **Example output:**
  ```text
  $ git cherry-pick d4e5f6g
  [main 5a6b7c8] Fix duplicated login request
   1 file changed, 2 insertions(+), 1 deletion(-)
  ```
- **After:**
  ```text
  A---B---C---5a6b7c8  main
                        ↑
                       HEAD
  ```
  ⚠️ The copied commit `5a6b7c8` is **new** — `d4e5f6g` still exists on `feature`.
- **Result:** The fix is on `main` without merging the whole branch.
- **When to use it:** You need one particular fix or feature from another branch without merging the whole branch.
- **Common mistake:** Cherry-picking a commit that conflicts — commit `d4e5f6g` touches the same lines you've changed:
  ```text
  error: could not apply d4e5f6g... Fix duplicated login request
  hint: after resolving the conflicts, mark them with
  hint: "git add <paths>" then run "git cherry-pick --continue"
  ```
  Resolve, `git add`, then `git cherry-pick --continue` (or `--abort` to back out).
- **Important notes:** Can produce conflicts — resolve them, `git add` the files, then `git cherry-pick --continue` (or `--abort` to back out). The copied commit gets a new hash.

### `git revert`

- **What it does:** Creates a new commit that undoes the changes introduced by a specified earlier commit, leaving history intact.
- **How to use it:**
  ```bash
  git revert a1b2c3d
  git revert HEAD
  ```
- **Result:** A new "revert" commit negates the target commit; history is unchanged (additive).
- **When to use it:** The safe way to undo commits that have already been pushed and shared.
- **Important notes:** See [Undoing & Recovering Changes](#5-undoing--recovering-changes) for the full walkthrough. Reverting a merge commit needs `-m 1` to choose which parent to revert to.

### `git reset --soft`

- **What it does:** Moves the branch pointer back while keeping all changes in the staging area.
- **How to use it:**
  ```bash
  git reset --soft HEAD~1
  ```
- **Before:**
  ```text
  Local:
  A --- B --- C
            ↑
           HEAD

  WD:    clean
  ```
- **Command:**
  ```bash
  git reset --soft HEAD~1
  ```
- **After:**
  ```text
  Local:
  A --- B
        ↑
       HEAD

  C's changes:
  → still staged
  ```
  ```text
  $ git status
  On branch main
  Changes to be committed:
    modified:   index.html
  ```
- **Result:** The last commit is removed from the branch history, but its changes remain staged — recommit differently if you like.
- **When to use it:** "Uncommit" the last commit but keep the changes staged, so you can recommit differently.
- **Important notes:** The least destructive reset — working directory is untouched. Safe because nothing is discarded.

### `git reset --mixed`

- **What it does:** Moves the branch pointer back *and* unstages changes, but keeps them in the working directory.
- **How to use it:**
  ```bash
  git reset HEAD~1
  ```
- **Before:**
  ```text
  Local:
  A --- B --- C
            ↑
           HEAD
  ```
- **Command:**
  ```bash
  git reset HEAD~1
  ```
- **After:**
  ```text
  Local:    A --- B  (HEAD at B)
  Staging:  empty
  WD:       C's changes are still on disk, now unstaged
  ```
  ```text
  $ git status
  On branch main
  Changes not staged for commit:
    modified:   index.html
  ```
- **Result:** History rewound, changes unstaged but kept, ready to re-stage selectively.
- **When to use it:** Uncommit and unstage, keeping your edits on disk so you can re-stage selectively.
- **Important notes:** This is the **default** mode — `git reset` with no flag behaves this way.

### `git reset --hard`

- **What it does:** Moves the branch pointer back, clears the staging area, and wipes changes from the working directory.
- **How to use it:**
  ```bash
  git reset --hard HEAD~1
  git reset --hard origin/main           # force your local branch to match the remote
  ```
- **Before:**
  ```text
  Local:    A --- B --- C  (HEAD at C)
  WD:       contains C's changes + any uncommitted edits
  Staging:  one or more files staged
  ```
- **Command:**
  ```bash
  git reset --hard HEAD~1
  ```
- **Example output:**
  ```text
  $ git reset --hard HEAD~1
  HEAD is now at a1b2c3d  Fix navbar alignment
  ```
- **After:**
  ```text
  Local:    A --- B  (HEAD at B)
  Staging:  empty
  WD:       clean — C's changes and all uncommitted edits are GONE
  ```
- **Result:** Everything up to and including commit `C` is discarded from your working tree. ⚠️ There is no confirmation prompt.
- **When to use it:** Discarding commits *and* all of their changes completely — e.g., abandoning a bad local experiment.
- **When NOT to use it:** Do not use it when:
  - you have uncommitted work you still need (it wipes it),
  - you're working on a shared branch (it rewrites/diverges history),
  - you don't fully understand what will be deleted.
- **Important notes:** ⚠️ **Destructive.** The reverted commits' changes and any uncommitted work are gone from your working tree. Recoverable only via `git reflog` shortly after.

### Interactive rebase (commit editing)

To change commit messages, squash, reorder, or split commits, use `git rebase -i` — see [Merge & Rebase Workflows](#6-merge--rebase-workflows). It's the tool for editing history beyond just the last commit.

### Commit comparison

- **What it does:** Compare any two commits or branches to see what changed.
- **How to use it:**
  ```bash
  git log --oneline main..feature       # commits on feature but not on main
  git diff main..feature                # file changes between the two tips
  git show a1b2c3d                      # one commit's changes in full
  ```
- **Example output** (`git log --oneline main..feature`):
  ```text
  $ git log --oneline main..feature
  d4e5f6g Add favorites page
  8hf3k2s Fix heading styles
  ```
- **Result:** A clear picture of exactly which commits (and which file changes) a branch would bring in.
- **When to use it:** Reviewing what a branch adds before merging, or debugging when a behavior changed.
- **Important notes:** `A..B` = "commits reachable from B but not from A"; `A...B` (three dots) = symmetric difference (changes on both sides).

---

## 8. Inspection & Debugging

### `git diff`

- **What it does:** Shows line-by-line differences between two states — by default, unstaged changes (working directory vs. staging area).
- **How to use it:**
  ```bash
  git diff
  git diff file.txt
  git diff HEAD
  ```
- **Result:** A precise read-only preview of your edits. See [Git Basics](#1-git-basics).
- **Important notes:** `git diff HEAD` compares against the last commit (staged + unstaged combined); `git diff --staged` shows only staged changes.

### `git diff --staged`

- **What it does:** Shows only the changes that are staged and will go into the next commit.
- **How to use it:**
  ```bash
  git diff --staged
  git diff --cached                      # alias for the same thing
  ```
- **Example output:**
  ```diff
  $ git diff --staged
  diff --git a/login.js b/login.js
  index 89abcde..f012345 100644
  --- a/login.js
  +++ b/login.js
  @@ -9,6 +9,7 @@
   function validateLogin(form) {
  +  if (!form.password) return false;
  ```
- **Result:** Confirms exactly what the next commit will contain.
- **When to use it:** Reviewing exactly what your next commit will contain, before committing.
- **Important notes:** If nothing is staged, it outputs nothing.

### `git log --oneline`

- **What it does:** Shows commit history as one line per commit (short hash + subject).
- **How to use it:**
  ```bash
  git log --oneline
  git log --oneline -10
  ```
- **Example output:**
  ```text
  $ git log --oneline
  9f3a1b2 (HEAD -> main) Add login form validation
  AB12CD3 Fix navbar alignment
  3c4d5e6 Initial commit
  ```
- **Result:** A fast, scannable history — the newest commit is first, and `HEAD -> main` marks your current position.
- **When to use it:** A fast, scannable overview of history.
- **Important notes:** Pairs well with `--graph` and `--all`.

### `git log --graph`

- **What it does:** Renders history as an ASCII graph showing branches and merges.
- **How to use it:**
  ```bash
  git log --graph --oneline --all
  ```
- **Example output:**
  ```text
  $ git log --graph --oneline --all
  * 7f8a9b0 (HEAD -> main) Merge branch 'feature/login'
  |\
  | * d4e5f6g (origin/feature/login) Add favorites page
  | * 8hf3k2s Fix heading styles
  * | 9f3a1b2 Add login form validation
  |/
  * a1b2c3d Initial commit
  ```
- **Result:** You can *see* where branches diverged and merged.
- **When to use it:** Visualizing how branches diverged and where merges happened.
- **Important notes:** `--all` includes every branch and ref, not just the current branch.

### `git show`

- **What it does:** Displays the details and diff of a single commit, tag, or object.
- **How to use it:**
  ```bash
  git show HEAD
  git show v1.0.0
  ```
- **Result:** A full picture of one commit's author, date, message, and diff. See [Git Basics](#1-git-basics).
- **When to use it:** Inspecting exactly what a specific commit changed. See [Git Basics](#1-git-basics).
- **Important notes:** Read-only — safe to run on anything.

### `git blame`

- **What it does:** Shows, line by line, which commit last modified each line of a file, and by whom.
- **How to use it:**
  ```bash
  git blame src/app.js
  git blame -L 10,20 src/app.js          # lines 10–20 only
  ```
- **Example output:**
  ```text
  $ git blame src/app.js
  ^a1b2c3d (Your Name 2026-06-01 10:00:00 +0200 1) import express from "express";
  9f3a1b2 (Team Mate 2026-07-15 14:22:00 +0200 2) const port = process.env.PORT || 3000;
  ```
  Format: commit hash, author, date, line number (of the current file) and the line's content.
- **Result:** You know who last touched each line and in which commit.
- **When to use it:** Finding out who introduced a specific line and when — great for hunting a bug or asking for context on a change.
- **Important notes:** Blames the last commit that *touched* each line, not necessarily the author of the logic — a later reformat can muddy the picture.

### `git reflog`

- **What it does:** A local log of everywhere HEAD has pointed — the safety net for lost commits and bad resets.
- **How to use it:**
  ```bash
  git reflog
  git reset --hard HEAD@{1}
  ```
- **Example output:**
  ```text
  $ git reflog
  d4e5f6g HEAD@{0}: reset: moving to HEAD~1
  9f3a1b2 HEAD@{1}: commit: Add login form validation
  ```
- **Result:** The pre-`reset` tip `9f3a1b2` is still listed, so you can recover it with `git reset --hard HEAD@{1}`.
- **When to use it:** Recovering from accidental resets, branch deletions, or botched rebases. See [Undoing & Recovering Changes](#5-undoing--recovering-changes).
- **Important notes:** Local-only (never pushed) and expires after roughly 30–90 days.

### `git bisect`

- **What it does:** Binary-searches history to find the exact commit that introduced a bug.
- **How to use it:**
  ```bash
  git bisect start
  git bisect bad                 # current commit is bad
  git bisect good v1.0.0         # this old commit was good
  # Git checks out a middle commit — you test it, then:
  git bisect good                # or: git bisect bad
  # repeat until Git names the culprit
  git bisect reset               # return to your original branch
  ```
- **Example output:**
  ```text
  $ git bisect start
  status: waiting for both good and bad commits

  $ git bisect bad
  status: waiting for good commit(s), bad commit known

  $ git bisect good v1.0.0
  Bisecting: 4 revisions left to test after this (roughly 2 steps)
  [a1b2c3d] Refactor error handling

  ... (Git checks out a commit at a time; you mark each good or bad) ...

  a1b2c3d is the first bad commit
  commit a1b2c3d
  Author: ...
  Date: ...
      Refactor error handling
  ```
- **Result:** Git narrows the bug down to the exact introducing commit, usually in a handful of tests.
- **When to use it:** A regression appeared somewhere in a long range of commits and you don't know where.
- **Important notes:** ~10 tests cover ~1000 commits. `git bisect run <script>` automates it if a script can decide good/bad for you. Always run `git bisect reset` when done.

---

## 9. Tags & Releases

### `git tag`

- **What it does:** Lists tags, or creates a lightweight tag — essentially just a name pointing at a commit.
- **How to use it:**
  ```bash
  git tag
  git tag v1.0.0
  git tag v1.0.0 a1b2c3d              # tag a specific commit
  ```
- **Example output** (listing):
  ```text
  $ git tag
  v1.0.0
  v2.3.1
  ```
- **After creating:**
  ```text
  Local:    refs/tags/v1.0.0 → points at the current (or specified) commit
  ```
  (No output on creation.)
- **Result:** You can now reference that exact commit by name: `git show v1.0.0`, `git checkout v1.0.0`.
- **When to use it:** Marking release points (v1.0.0, v2.3.1) or any important commit you want to reference by name.
- **Important notes:** Lightweight tags carry no author, date, or message. For releases, prefer annotated tags (below).

### `git tag -a`

- **What it does:** Creates an annotated tag — a full Git object with a tagger, date, and message.
- **How to use it:**
  ```bash
  git tag -a v1.0.0 -m "Release version 1.0.0"
  git tag -a v1.0.0 a1b2c3d -m "Bugfix release"
  ```
- **Example output** (verifying):
  ```text
  $ git show v1.0.0
  tag v1.0.0
  Tagger: Your Name <you@example.com>
  Date:   Mon Aug 12 10:00:00 2026 +0200

  Release version 1.0.0

  commit a1b2c3d
  ...
  ```
- **Result:** A permanent, annotated marker of a particular commit, visible in history with its release notes.
- **When to use it:** Releasing software where release notes and a tagger identity matter.
- **Important notes:** Annotated tags can also be signed (`-s`) — see [Advanced Git](#10-advanced-git). Recommended over lightweight tags for releases.

### `git push --tags`

- **What it does:** Pushes tags to the remote repository.
- **How to use it:**
  ```bash
  git push --tags
  git push origin v1.0.0               # push one specific tag
  ```
- **Example output:**
  ```text
  $ git push --tags
  Enumerating objects: 1, done.
  ...
  * [new tag]         v1.0.0 -> v1.0.0
  ```
- **After:**
  ```text
  Remote:    v1.0.0 now exists on the server
  ```
- **Result:** The release tag is shared with the team and CI systems.
- **When to use it:** After tagging a release locally, to share the tag with the team and CI systems.
- **Important notes:** A plain `git push` does **not** push tags. Tags live under `refs/tags/`.

### Deleting tags

- **What it does:** Removes a tag locally and/or on the remote.
- **How to use it:**
  ```bash
  git tag -d v1.0.0                    # delete local tag
  git push origin --delete v1.0.0      # delete remote tag
  ```
- **Example output:**
  ```text
  $ git tag -d v1.0.0
  Deleted tag 'v1.0.0' (was a1b2c3d)

  $ git push origin --delete v1.0.0
  To https://github.com/user/project.git
   - [deleted]         v1.0.0
  ```
- **After:** The tag is gone locally and/or remotely. The tagged commit itself is unaffected.
- **When to use it:** Fixing a mistagged release or cleaning up.
- **Important notes:** Deleting a remote tag affects anyone else who has it — coordinate first if the tag marks a real release.

### Checking out tags

- **What it does:** Inspects the code at a specific tagged commit.
- **How to use it:**
  ```bash
  git checkout v1.0.0                  # detached HEAD at the tagged commit
  git switch -c release-1.0 v1.0.0     # create a branch from a tag
  ```
- **Example output:**
  ```text
  $ git checkout v1.0.0
  Note: switching to 'v1.0.0'.

  You are in 'detached HEAD' state. ...
  ```
- **Result:** Your files reflect the code exactly as it was at that release.
- **When to use it:** Reproducing a release, hotfixing an old version, or reviewing code as it was at release time.
- **Important notes:** `git checkout <tag>` puts you in **detached HEAD** — you're not on a branch, so new commits can easily be lost. Create a branch (`git switch -c`) if you intend to make commits.

---

## 10. Advanced Git

### `git cherry-pick`

- **What it does:** Applies a commit from another branch onto your current one.
- **How to use it:**
  ```bash
  git cherry-pick a1b2c3d
  git cherry-pick a1b2c3d^..d4e5f6g    # pick a range of commits
  ```
- **Result:** The chosen commit(s) are replayed onto `HEAD` with new hashes. See [Commit Management](#7-commit-management) for a full walkthrough with conflicts.
- **When to use it:** Porting a hotfix to multiple branches (e.g., the same bug fixed on `main` and on a release branch). Full details in [Commit Management](#7-commit-management).

### `git worktree`

- **What it does:** Lets you have multiple working directories attached to the same repository, each on a different branch.
- **How to use it:**
  ```bash
  git worktree add ../hotfix hotfix/urgent
  git worktree list
  git worktree remove ../hotfix
  ```
- **Example output:**
  ```text
  $ git worktree list
  /Users/you/project            main
  /Users/you/hotfix              hotfix/urgent
  ```
- **After:** A second directory `../hotfix` is checked out on `hotfix/urgent` — entirely separate from your main working tree.
- **Result:** Two branches checked out simultaneously, no stashing or cloning needed.
- **When to use it:** You need to work on two branches at once (e.g., review a PR while keeping your feature branch checked out) without stashing or cloning again.
- **Important notes:** The same branch can't be checked out in two worktrees at once. Remove the worktree before deleting its folder.

### `git bisect`

- **What it does:** Binary-search tool for finding the commit that broke something.
- **How to use it:**
  ```bash
  git bisect start
  git bisect bad HEAD
  git bisect good v1.0.0
  # ... test the checked-out commit and mark good/bad ...
  git bisect reset
  ```
- **Result:** Git identifies the offending commit in logarithmic steps. See [Inspection & Debugging](#8-inspection--debugging) for the full walkthrough.
- **When to use it:** A regression appeared somewhere in history and you want to find it fast. See [Inspection & Debugging](#8-inspection--debugging) for the full walkthrough.

### `git reflog`

- **What it does:** A local-only history of every position HEAD has held.
- **How to use it:**
  ```bash
  git reflog
  git reset --hard HEAD@{2}
  ```
- **Result:** A way back to any recent HEAD position, even after a botched reset.
- **When to use it:** Rescuing commits "lost" to resets, rebases, or deleted branches. See [Undoing & Recovering Changes](#5-undoing--recovering-changes).
- **Important notes:** Never pushed or cloned; entries expire after roughly 30–90 days.

### `git filter-repo`

- **What it does:** Rewrites history at scale — removes files from all of history, renames authors/emails, or splits a repo. It is an **external tool** (install it separately; the built-in `git filter-branch` is the deprecated alternative).
- **How to use it:**
  ```bash
  git filter-repo --path secrets.env --invert-paths
  ```
- **Example output:**
  ```text
  $ git filter-repo --path secrets.env --invert-paths
  Parsed 42 commits
  New history written in 0.20 seconds
  ...
  ```
- **Result:** `secrets.env` is gone from every commit in history, and every history-rewritten commit has a new hash.
- **After:**
  ```text
  Local:    all commit hashes changed (those touching the removed path)
  Remote:   origin removed as a safety measure
  ```
- **When to use it:** Purging a committed secret or a huge file from *every* commit in history.
- **Important notes:** ⚠️ This rewrites every affected commit hash and **breaks every existing clone**. Only do it on a history everyone agrees to rewrite, then force-push and ask all teammates to re-clone. It removes the `origin` remote automatically as a safety measure.

### Submodules

- **What it does:** Embeds another Git repository inside your repo, pinned to a specific commit.
- **How to use it:**
  ```bash
  git submodule add https://github.com/user/library.git vendor/library
  git clone --recurse-submodules <repo-url>     # clone + init submodules at once
  git submodule update --init --recursive
  ```
- **Example output** (adding):
  ```text
  $ git submodule add https://github.com/user/library.git vendor/library
  Cloning into '/Users/you/project/vendor/library'...
  ...
  ```
- **After:**
  ```text
  Local:    vendor/library is a full Git repo, pinned to a specific commit
  ```
- **When to use it:** Pinning an external dependency to an exact version, or managing a monorepo split across multiple repos.
- **Important notes:** Submodules add complexity — teammates must run `git submodule update` after pulling. The parent repo only records *which commit* the submodule points to, not its contents. Many teams prefer package managers for dependencies and reserve submodules for cases that truly need them.

### Hooks

- **What it does:** Local scripts Git runs automatically on events like `commit`, `push`, or `merge`.
- **How to use it:**
  ```bash
  # inside .git/hooks/, rename pre-commit.sample to pre-commit and add your script
  ```
- **Example** (a `.git/hooks/pre-commit` that blocks secrets):
  ```bash
  #!/bin/sh
  if git diff --cached --name-only | grep -q '^\.env$'; then
    echo "Refusing to commit .env" >&2
    exit 1
  fi
  ```
- **Example output** (when a hook rejects your commit):
  ```text
  $ git commit -m "Add config"
  Refusing to commit .env
  ```
  The commit is blocked until you fix it.
- **When to use it:** Enforcing conventions locally — running a linter or formatter before commit, blocking secrets, running tests before push.
- **Important notes:** Hooks live in `.git/hooks/` (not tracked), so every teammate must set them up. Tools like `pre-commit` or `husky` manage hooks across a team.

### Aliases

- **What it does:** Defines shortcuts for commands you type constantly.
- **How to use it:**
  ```bash
  git config --global alias.co checkout
  git config --global alias.st status
  git config --global alias.lg "log --oneline --graph --all"
  ```
- **After:** `git st` runs `git status`, `git lg` runs the log graph command.
- **Result:** Fewer keystrokes for your most-used commands.
- **When to use it:** Saving keystrokes and standardizing commands.
- **Important notes:** Use quotes for multi-word aliases. Aliases are config, so they're per-machine unless you share your config (e.g., via dotfiles).

### Signed commits and tags

- **What it does:** Cryptographically signs commits and tags so others can verify they came from you.
- **How to use it:**
  ```bash
  git config --global user.signingkey <your-key-id>
  git config --global commit.gpgsign true      # sign all commits from now on
  git commit -S -m "Signed commit"
  git tag -s v1.0.0 -m "Signed release tag"
  git verify-commit <commit-hash>
  ```
- **Example output** (verifying):
  ```text
  $ git verify-commit a1b2c3d
  gpg: Signature made Mon Aug 12 10:00:00 2026 CST
  gpg:                using RSA key <your-key-id>
  gpg: Good signature from "Your Name <you@example.com>"
  ```
- **Result:** Your commits/tags carry a verifiable cryptographic signature.
- **When to use it:** Projects with high security requirements (kernels, many open-source projects), or any repo where the provenance of code matters.
- **Important notes:** Requires a GPG or SSH key set up first. Configure your hosting provider (GitHub/GitLab) so signed commits show a "Verified" badge. Adds setup cost, so don't enable it casually.

### Shallow clones

- **What it does:** Clones with truncated history — only the most recent commits.
- **How to use it:**
  ```bash
  git clone --depth 1 https://github.com/user/repo.git
  git clone --depth 10 --branch v2.0.0 https://github.com/user/repo.git
  ```
- **Example output** (note `--depth` in the clone message):
  ```text
  $ git clone --depth 1 https://github.com/user/repo.git
  Cloning into 'repo'...
  remote: Total 1 (delta 0), reused 0 (delta 0), pack-reused 0
  Receiving objects: 100% (1/1), done.
  ```
- **Result:** A fast, small copy containing only recent history and the latest files.
- **When to use it:** Quickly getting a copy of a huge repo for building or reading.
- **Important notes:** Limited history breaks `git log`, `git blame`, and `git bisect` beyond the cloned depth. `git fetch --unshallow` upgrades it to a full clone later.

### Sparse checkout

- **What it does:** Checks out only a subset of directories instead of the whole repository.
- **How to use it:**
  ```bash
  git clone --filter=blob:none --sparse https://github.com/user/monorepo.git
  git sparse-checkout set apps/web
  ```
- **After:**
  ```text
  WD:       only apps/web is checked out; other paths are absent
  ```
- **Result:** Your working directory contains just the part you work on, saving disk space and checkout time.
- **When to use it:** Monorepos where you only work on part of the code — saves disk space and checkout time.
- **Important notes:** Combined with `--depth` it makes very large repos usable. Minus `--filter=blob:none` it still downloads all blobs, so include it for the full benefit.

---

## 11. Common Real-World Git Scenarios

Each scenario follows the pattern: **Situation → Current state → Commands → Terminal output → Final state → Next step**.

### Starting a new project with Git

**Situation:** You have a folder full of files and want to version-control it from scratch.

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/user/project.git
git push -u origin main
```

```text
$ git commit -m "Initial commit"
[main (root-commit) a1b2c3d] Initial commit
 5 files changed, 120 insertions(+)
```
**Final state:** All files committed on `main`; the project is also pushed to the remote. **Next step:** keep committing as you make changes.

### Cloning an existing project

**Situation:** You want a local copy of a repo that already exists on GitHub.

```bash
git clone https://github.com/user/project.git
cd project
git status
```

```text
$ git status
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```
**Final state:** Full history + latest files on `main`, tracking `origin/main`. **Next step:** create a feature branch and start working.

### Making and committing changes

**Situation:** You edited `login.html` and want to save it to history.

```bash
git status              # see what changed
git add .
git commit -m "Add login form"
git log --oneline -5    # verify the commit landed
```

```text
$ git status
Changes not staged for commit:
  modified:   login.html

$ git commit -m "Add login form"
[main 9f3a1b2] Add login form
 1 file changed, 12 insertions(+)

$ git log --oneline -5
9f3a1b2 (HEAD -> main) Add login form
a1b2c3d Initial commit
```
**Final state:** `login.html` edits are committed as `9f3a1b2`. **Next step:** push to share: `git push`.

### Creating a feature branch

**Situation:** You're starting new, isolated work (e.g., a login feature).

```bash
git switch main
git switch -c feature/login
```

```text
$ git switch -c feature/login
Switched to a new branch 'feature/login'
```
**Final state:** New branch `feature/login` checked out, identical to `main`. **Next step:** make commits here; `main` stays untouched.

### Updating a branch with the latest changes

**Situation:** `main` moved on the remote and your feature branch is behind.

```bash
git fetch origin
git rebase origin/main          # replay your work on top of latest main
# or, with a merge workflow:
git merge origin/main
```

```text
$ git fetch origin
From https://github.com/user/project.git
   a1b2c3d..d4e5f6g  main       -> origin/main

$ git rebase origin/main
Successfully rebased and updated refs/heads/feature/login.
```
**Final state:** Your feature commits now sit on top of the latest `main` (linear) — or a merge commit links the two histories. **Next step:** run tests; resolve conflicts if the rebase stopped.

### Merging a feature branch

**Situation:** `feature/login` is done and reviewed; bring it into `main`.

```bash
git switch main
git merge feature/login
git branch -d feature/login     # clean up once merged
```

```text
$ git merge feature/login
Merge made by the 'ort' strategy.
 login.html | 12 ++++++++++++
 1 file changed, 12 insertions(+)

$ git branch -d feature/login
Deleted branch feature/login (was d4e5f6g).
```
**Final state:** `feature/login`'s work is on `main`, and the branch is cleaned up. **Next step:** push `main`.

### Rebasing a feature branch

**Situation:** You want the feature's history to sit neatly on top of the latest `main`.

```bash
git switch feature/login
git rebase main
```
```text
$ git rebase main
Successfully rebased and updated refs/heads/feature/login.
```
**Final state:** Linear history: `main`'s commits followed by your feature's (with new hashes). **Next step:** `git push --force-with-lease origin feature/login` only if it was already pushed.

### Pushing a new branch

**Situation:** Your new branch needs to exist on the remote.

```bash
git push -u origin feature/login
```
```text
$ git push -u origin feature/login
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
remote:
remote: Create a pull request for 'feature/login' on GitHub by visiting:
remote:      https://github.com/user/project/pull/new/feature/login
remote:
 * [new branch]      feature/login -> feature/login
branch 'feature/login' set up to track 'origin/feature/login'.
```
**Final state:** Branch is on the remote and tracks `origin/feature/login`. **Next step:** open a pull request.

### Pulling changes from GitHub

**Situation:** Your local `main` is behind the remote, and you have no local commits to lose.

```bash
git pull --rebase origin main
```
```text
$ git pull --rebase origin main
From https://github.com/user/project.git
   a1b2c3d..d4e5f6g  main       -> origin/main
Current branch main is up to date.
```
(Sometimes you'll instead see `Successfully rebased and updated refs/heads/main.` when work had to be reapplied.)
**Final state:** Local `main` equals `origin/main`. **Next step:** keep going.

### Fixing a merge conflict

**Situation:** Both branches changed the same lines; the merge paused.

```bash
git merge feature/login
# CONFLICT in src/app.js — open the file, resolve the <<<<<<< / ======= / >>>>>>> markers,
# keeping the correct final content
git add src/app.js
git commit
```
```text
Auto-merging src/app.js
CONFLICT (content): Merge conflict in src/app.js
Automatic merge failed; fix conflicts and then commit the result.
```
After resolving and staging:
```text
$ git commit
[main 7f8a9b0] Merge branch 'feature/login'
```
**Final state:** Merge commit `7f8a9b0` records the resolution. **Next step:** run tests, then push.

### Changing the last commit message

**Situation:** The last commit's message is wrong (not yet pushed).

```bash
git commit --amend -m "Improved commit message"
```
```text
$ git commit --amend -m "Improved commit message"
[main 9f3a1b2] Improved commit message
```
**Final state:** Last commit rewritten with the corrected message. **Next step:** if already pushed, `git push --force-with-lease`.

### Adding forgotten changes to the last commit

**Situation:** A file was left out of the last commit.

```bash
git add forgotten-file.txt
git commit --amend --no-edit
```
**Final state:** The forgotten file is folded into the last commit (message unchanged). ⚠️ Only do this before pushing.

### Unstaging files

**Situation:** You staged too much; remove one file from the index.

```bash
git restore --staged file.txt
# or, equivalently:
git reset file.txt
```
```text
$ git restore --staged file.txt
```
(No output — `git status` shows `file.txt` back under "Changes not staged".)
**Final state:** File is unstaged; edits untouched on disk.

### Discarding local changes

**Situation:** You don't want your current edits anymore.

```bash
git restore file.txt     # a single file
git restore .            # everything tracked
```
**Final state:** The edited files are back to their committed state. ⚠️ The edits are gone permanently.

### Undoing the last commit while keeping the files

**Situation:** You committed the wrong thing and want a clean slate, keeping the work.

```bash
git reset --soft HEAD~1          # keep changes staged
git reset HEAD~1                 # also unstage, keeping edits on disk
```
```text
$ git reset --soft HEAD~1
$ git status
Changes to be committed:
  modified:   index.html
```
**Final state:** The commit is gone; changes remain (staged, or unstaged with `--mixed`). **Next step:** re-stage selectively and recommit.

### Undoing a pushed commit safely

**Situation:** Commit `a1b2c3d` was pushed and others may have it; undo its effects without rewriting history.

```bash
git revert <commit-hash>
git push origin main
```
```text
$ git revert a1b2c3d
[main e5f6a7b] Revert "Add broken coupon logic"
 1 file changed, 8 deletions(-)
```
**Final state:** A new revert commit (not a rewrite) reverses the change; both live in history. **Next step:** push the revert.

### Recovering a deleted commit

**Situation:** A branch was deleted or a commit became unreachable.

```bash
git reflog
git branch recovered-work <commit-hash>   # restore it as a branch
```
```text
$ git reflog
d4e5f6g HEAD@{0}: branch: Created from HEAD
...
$ git branch recovered-work d4e5f6g
```
**Final state:** The "lost" commit is reachable again via `recovered-work`.

### Recovering from an accidental `git reset --hard`

**Situation:** A `git reset --hard` wiped your last commit; you want it back.

```bash
git reflog
git reset --hard HEAD@{1}        # back to the position before the reset
```
```text
$ git reflog
d4e5f6g HEAD@{0}: reset: moving to HEAD~1
9f3a1b2 HEAD@{1}: commit: Add login form validation
$ git reset --hard HEAD@{1}
HEAD is now at 9f3a1b2  Add login form validation
```
**Final state:** Your branch (and working tree) are restored to `9f3a1b2`, the commit the reset undid.

### Moving changes to another branch

**Situation:** You started a change on the wrong branch.

```bash
git stash
git switch feature/other
git stash pop
```
```text
$ git stash
Saved working directory and index state WIP on main: 9f3a1b2 Add login form validation

$ git switch feature/other
Switched to branch 'feature/other'

$ git stash pop
On branch feature/other
Changes not staged for commit:
  modified:   index.html
```
**Final state:** Your work now lives on the correct branch. **Next step:** stage and commit it there.

### Copying a commit from another branch (`cherry-pick`)

**Situation:** A bugfix exists on `feature/login` but you're on `main`.

```bash
git switch main
git cherry-pick a1b2c3d          # the commit hash from the other branch
```
```text
$ git cherry-pick a1b2c3d
[main 5a6b7c8] Fix duplicated login request
 1 file changed, 2 insertions(+), 1 deletion(-)
```
**Final state:** The fix is now also on `main` (new hash). **Next step:** commit/push as usual.

### Cleaning untracked files

**Situation:** Your directory is full of build artifacts Git isn't tracking.

```bash
git clean -n                     # preview first — always
git clean -fd                    # then actually delete
```
```text
$ git clean -n
Would remove build/
Would remove tmp-cache.dat

$ git clean -fd
```
**Final state:** Untracked clutter is gone. ⚠️ Irreversible — these files were never tracked.

---

## 12. Important Comparisons

### `git fetch` vs `git pull`

| | `git fetch` | `git pull` |
|---|---|---|
| Downloads remote changes | ✅ | ✅ (includes fetch) |
| Merges into your branch | ❌ | ✅ |
| Can create conflicts | ❌ | ✅ |
| Touches working directory | ❌ | ✅ |

**Which to choose:** `git pull` = `git fetch` + integrate. Use plain `git fetch` when you want to inspect what changed (`git log main..origin/main`) before deciding how to integrate. Use `git pull` for the everyday "bring my branch up to date" case — and consider `git pull --rebase` for a linear history.

### `git merge` vs `git rebase`

| | `git merge` | `git rebase` |
|---|---|---|
| Creates a merge commit | ✅ (usually) | ❌ |
| History shape | Preserves branch topology | Linear |
| Rewrites commit hashes | ❌ | ✅ |
| Good for | Shared / integration branches | Your own unpublished feature work |

**Which to choose:** Merge to integrate shared work and keep history truthful about when things merged. Rebase to keep *your* feature branch clean and linear on top of the latest `main` — but only while the commits are local and unpushed (see [Rules of Thumb](#15-git-rules-of-thumb)).

### `git reset` vs `git revert` vs `git restore`

| | `git reset` | `git revert` | `git restore` |
|---|---|---|---|
| Scope | Branch pointer + staging (+ WD) | A specific commit | Files (WD / staging) |
| Adds a new commit | ❌ | ✅ | ❌ |
| Rewrites history | ✅ | ❌ | ❌ (nothing committed) |
| Safe on pushed commits | ❌ | ✅ | N/A (local only) |

**Which to choose:** `git restore` to fix the working directory or staging *before* committing. `git reset` to uncommit or rewrite *local* history. `git revert` to undo *shared* history without rewriting it.

### `git switch` vs `git checkout`

| | `git switch` | `git checkout` |
|---|---|---|
| Purpose | Branch switching only | Branch switching + file restore + detached HEAD |
| Safety | Focused, harder to misuse | Multi-purpose, easy to mistype into something else |
| Availability | Git 2.23+ | All versions |

**Which to choose:** Prefer `git switch` (and `git restore`) in everyday work — the narrower commands are harder to use by accident. Keep `git checkout` in mind for older tutorials and for checking out a specific commit or tag.

### `git stash apply` vs `git stash pop`

| | `git stash apply` | `git stash pop` |
|---|---|---|
| Reapplies changes | ✅ | ✅ |
| Keeps the stash in the list | ✅ | ❌ (removes it) |

**Which to choose:** Use `pop` when you're done with the stash (the common case). Use `apply` when you want to reuse the same stash on several branches, or keep a safety net until you're sure.

### `git reset --soft` vs `--mixed` vs `--hard`

| Mode | Branch pointer | Staging area | Working directory |
|---|---|---|---|
| `--soft` | ✅ moves | Kept | Untouched |
| `--mixed` (default) | ✅ moves | Unstaged | Untouched |
| `--hard` | ✅ moves | Cleared | **Wiped** ⚠️ |

**Which to choose:** `--soft` to uncommit but keep everything staged (e.g., to recommit differently). `--mixed` to uncommit and unstage while keeping edits on disk. `--hard` only when you truly want to discard commits *and* their changes.

### `git push --force` vs `git push --force-with-lease`

| | `--force` | `--force-with-lease` |
|---|---|---|
| Overwrites remote branch | ✅ | ✅ |
| Checks for unseen remote commits | ❌ | ✅ (refuses if others pushed) |
| Risk of clobbering teammates' work | High | Low |

**Which to choose:** Always prefer `--force-with-lease`. Plain `--force` silently overwrites commits you haven't seen; `--force-with-lease` fails loudly instead. Avoid both on shared branches unless the team agrees.

### `git clone` vs `git fetch`

| | `git clone` | `git fetch` |
|---|---|---|
| Creates a new repo | ✅ | ❌ |
| Downloads history | ✅ (full history by default) | Only new objects |
| Requires an existing local repo | ❌ | ✅ |
| Use case | Getting a repo the first time | Updating an existing repo |

**Which to choose:** `git clone` is the one-time "copy the repo down" command. After that, use `git fetch` (or `git pull`) to bring in updates.

---

## 13. Git Mental Model

```text
Working Directory             git add ─▶ Staging Area ── git commit ──▶ Commit
       │                                                           │
       │  git restore / git clean              git reset / branches ─▶ Local Branch
       ▼                                                           │
   your edits                                                      │  git push
                                                                   ▼
                                                        Remote Repository
                                                          ▲
                                               git fetch / git pull │
```

- **Working Directory:** the files on your disk. `git add` moves edits forward; `git restore` and `git clean` discard them.
- **Staging Area (index):** the "next commit" checklist. `git add` puts changes here; `git restore --staged` and `git reset` take them out.
- **Commit:** a permanent snapshot recorded on your local branch. `git commit` creates it; `git reset` and `git rebase` rewrite which commits a branch has.
- **Local Branch:** a movable pointer to a commit (and its history). `git switch`, `git branch`, and `git merge` manage these.
- **Remote Repository:** the shared copy on a server. `git push` uploads; `git fetch` / `git pull` download.

Files are edited in the **working directory**, get **staged**, get **committed** to a local branch, and finally get **pushed** to the remote. Undo commands work in the reverse direction: `git restore` (working directory) → `git reset` (staging/history) → `git revert` (shared history).

---

## ⚠️ Safety Warnings — Destructive Commands

These commands can **permanently discard work**. Always check `git status` and preview before running them.

| Command | What it can destroy | When it's appropriate |
|---|---|---|
| `git reset --hard` | Uncommitted changes (staging + working directory) and the commits it rewinds past | Discarding a local experiment you never want back |
| `git clean -fd` | **All untracked files** (they were never in Git) | Clearing build artifacts / clutter — always `git clean -n` first |
| `git branch -D` | Any commits that exist only on the deleted branch | Deleting a branch you're certain is abandoned |
| `git push --force` | Teammates' commits on the remote branch that you haven't seen | Almost never — use `--force-with-lease` instead, and only on branches you own |
| `git push --force-with-lease` | The remote branch's rewritten history | After rewriting *your own* feature branch, when no one else relies on it |

> ⚠️ Be careful: `git reset --hard` and `git clean -fd` can permanently discard local changes.

General rules:
- Preview before deleting: `git clean -n`, `git status`, `git log`.
- Nothing is truly "lost" immediately — `git reflog` can recover many mistakes within ~30–90 days.
- Don't rewrite or force-push shared branches without team agreement.

---

## 15. Git Rules of Thumb

- Check `git status` frequently — before and after almost every command.
- Review changes before committing: `git diff` and `git diff --staged`.
- Prefer small, meaningful commits with clear messages over one giant "wip" commit.
- Don't use `git reset --hard` without understanding exactly what will be deleted.
- Don't force-push shared branches unless you know the consequences.
- Prefer `--force-with-lease` over `--force` when rewriting a remote branch.
- Don't rebase commits that other people are already depending on, unless the workflow explicitly allows it.
- Use `git revert` when safely undoing changes that are already shared.
- Pull or rebase *before* pushing — don't reach for `--force` when a push is rejected.
- Stash or commit before switching branches to avoid conflicts and lost work.
- Write commit messages that explain *why*, not just *what*.
- Run `git fetch` (not just `git pull`) when you want to inspect remote changes before integrating.

---

## 16. Which Command Should I Use?

A quick decision guide based on what you're trying to accomplish.

| Situation | Use |
| --- | --- |
| I want to see what changed before staging | `git status`, then `git diff` |
| I want to see what's already staged | `git diff --staged` |
| I want to stage changes | `git add <file>` (prefer over `git add .`) |
| I want to stage parts of a file only | `git add -p` |
| I want to unstage a file (keep edits) | `git restore --staged <file>` |
| I want to discard local edits | `git restore <file>` |
| I want to discard untracked clutter | `git clean -n`, then `git clean -fd` |
| I want to save work temporarily | `git stash` |
| I want my stashed work back (and remove it) | `git stash pop` |
| I want my stashed work back (and keep it) | `git stash apply` |
| I want to remove a file and track the deletion | `git rm <file>` |
| I want to rename a tracked file | `git mv <old> <new>` |
| I want to undo a local commit but keep changes staged | `git reset --soft HEAD~1` |
| I want to undo a local commit but keep edits on disk | `git reset HEAD~1` (mixed) |
| I want to undo a local commit and discard its changes | `git reset --hard HEAD~1` |
| I want to undo a shared/pushed commit safely | `git revert <hash>` |
| I accidentally lost a commit | `git reflog`, then recover with `git reset` / `git branch` |
| I want to fix the last commit's message | `git commit --amend -m "..."` |
| I want to add a forgotten file to the last commit | `git add <file>` + `git commit --amend --no-edit` |
| I want to start new work on its own branch | `git switch -c <name>` |
| I want to copy one commit from another branch | `git cherry-pick <hash>` |
| I want a linear branch history | `git rebase <branch>` |
| I want to combine branch histories | `git merge <branch>` |
| I want to see how branches relate | `git log --graph --oneline --all` |
| I want to find who changed a line | `git blame <file>` |
| I want to find which commit broke something | `git bisect start` |
| I want to see remote-tracking sync state | `git status` or `git branch -vv` |
| I want to bring a branch up to date | `git fetch` then `git rebase origin/main` (or `git pull --rebase`) |
| I want to work on a stale local branch | `git fetch origin`, then `git reset --hard origin/main` (⚠️ discards local commits) |

---

## 17. Git Quick Reference

```text
SETUP
  git config --global user.name "Your Name"
  git config --global user.email "you@example.com"
  git init                          create a new repo
  git clone <url>                   copy an existing repo

DAILY WORKFLOW
  git status                        what's going on
  git diff                          unstaged changes
  git add <file> | .                stage changes
  git commit -m "msg"               commit staged changes
  git log --oneline --graph         history overview

BRANCHES
  git switch <branch>               change branch
  git switch -c <name>              new branch + switch
  git branch -d <name>              delete merged branch
  git merge <branch>                integrate a branch
  git rebase <branch>               replay your commits on another branch

REMOTE
  git remote -v                     list remotes
  git push -u origin <branch>       first push of a new branch
  git push                          push current branch
  git pull --rebase                 fetch + integrate
  git fetch                         download without merging

UNDO
  git restore <file>                discard unstaged edits
  git restore --staged <file>       unstage
  git reset --soft HEAD~1           undo commit, keep staged
  git reset --hard HEAD~1           undo commit + discard changes ⚠️
  git revert <commit>               safely undo a pushed commit
  git reflog                        recover "lost" commits

INSPECTION
  git show <commit>                 what a commit changed
  git diff --staged                 what will be committed
  git blame <file>                  who changed each line
  git bisect start                  find the buggy commit

STASH
  git stash                         save uncommitted work
  git stash pop                     restore + remove
  git stash apply                   restore + keep
  git stash list                    see saved stashes

MERGE / REBASE
  git merge --abort                 cancel a conflicted merge
  git rebase -i HEAD~3              edit history interactively
  git rebase --continue             resume after resolving
  git rebase --abort                cancel a rebase

ADVANCED
  git cherry-pick <commit>          copy a commit
  git worktree add <path> <branch>  second working directory
  git tag -a v1.0.0 -m "msg"        mark a release
  git push --tags                   share tags
  git clean -nfd                    preview / clean untracked files ⚠️
```