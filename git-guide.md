# Practical Git Command Notes

A beginner-to-advanced reference for everyday Git usage — what each command does, how to use it, when to reach for it, and what to watch out for.

**Legend:** `WD` = Working Directory · `Staging` = Staging Area (index) · `Local` = Local repository · `Remote` = Remote repository

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
11. [Common Real-World Git Scenarios](#common-real-world-git-scenarios)
12. [Important Comparisons](#important-comparisons)
13. [Git Mental Model](#git-mental-model)
14. [Safety Warnings — Destructive Commands](#️-safety-warnings--destructive-commands)
15. [Git Rules of Thumb](#git-rules-of-thumb)
16. [Git Quick Reference](#git-quick-reference)

---

## 1. Git Basics

### `git --version`

- **What it does:** Prints the installed Git version.
- **How to use it:**
  ```bash
  git --version
  ```
- **When to use it:** Confirming Git is installed, or checking you meet a minimum version required by a tool or tutorial.
- **Important notes:** Very old Git versions may lack newer commands like `git switch` or `git restore` — worth checking if a tutorial's commands "don't exist" on your machine.

### `git config`

- **What it does:** Reads or sets Git configuration values (user identity, editor, aliases, behavior defaults) at system, global (per-user), or local (per-repo) scope.
- **How to use it:**
  ```bash
  git config --global user.name "Your Name"
  git config --global user.email "you@example.com"
  git config --list
  ```
- **When to use it:** First-time setup on a new machine, setting your commit identity, or customizing behavior (default branch name, editor, aliases) for one repo or all repos.
- **Important notes:** Use `--global` for settings that should apply everywhere; omit it (or use `--local`) to scope a setting to the current repo only. `git config --list --show-origin` helps find which file a setting came from.

### `git init`

- **What it does:** Creates a new, empty Git repository in the current directory (adds a hidden `.git` folder).
- **How to use it:**
  ```bash
  git init
  git init my-project
  ```
- **When to use it:** Starting version control on a brand-new project, or turning an existing folder of files into a Git repo.
- **Important notes:** Running it twice in the same folder is harmless (it won't wipe existing history), but double-check you're in the right directory — you don't want a stray `.git` folder nested inside another repo.

### `git clone`

- **What it does:** Copies an existing remote repository (and its full history) to your machine, automatically setting up a remote called `origin`.
- **How to use it:**
  ```bash
  git clone https://github.com/user/repo.git
  git clone https://github.com/user/repo.git my-folder-name
  ```
- **When to use it:** Starting work on a project that already exists elsewhere (GitHub, GitLab, a teammate's server, etc.).
- **Important notes:** By default this downloads the entire history. For very large repos, see shallow clones (`--depth`) in [Advanced Git](#10-advanced-git).

### `git status`

- **What it does:** Shows the current state of the working directory and staging area — which files are modified, staged, or untracked.
- **How to use it:**
  ```bash
  git status
  git status -s   # short format
  ```
- **When to use it:** Constantly. Run it before and after almost every Git operation to know exactly what state you're in.
- **Important notes:** It never changes anything — completely safe to run as often as you like.

### `git add`

- **What it does:** Moves changes from the working directory into the staging area, marking them to be included in the next commit.
- **How to use it:**
  ```bash
  git add file.txt
  git add .
  git add -p        # interactively stage parts of a file
  ```
- **When to use it:** After editing files, when you're ready to include those changes in your next commit.
- **Important notes:** `git add .` stages everything in the current directory, including files you might not have meant to include — check `git status` first. `git add -p` is great for splitting unrelated changes into separate commits.

### `git commit`

- **What it does:** Saves the currently staged changes as a new snapshot (commit) in the local repository's history.
- **How to use it:**
  ```bash
  git commit -m "Add login form validation"
  git commit -am "Fix typo in header"   # stage tracked changes + commit
  ```
- **When to use it:** After staging a logically complete, meaningful set of changes.
- **Important notes:** `-a` only stages files Git already tracks — it won't pick up new (untracked) files. Write clear, specific commit messages; "fix stuff" helps nobody later, including you.

### `git log`

- **What it does:** Shows the commit history for the current branch.
- **How to use it:**
  ```bash
  git log
  git log -5              # last 5 commits
  git log --author="Omkar"
  ```
- **When to use it:** Reviewing history, finding a specific commit, understanding what changed and when.
- **Important notes:** Default output is verbose. See `git log --oneline` and `git log --graph` in [Inspection & Debugging](#8-inspection--debugging) for more scannable views.

### `git diff`

- **What it does:** Shows line-by-line differences between two states — by default, working directory vs. the staging area (i.e., unstaged changes).
- **How to use it:**
  ```bash
  git diff                # unstaged changes
  git diff file.txt       # unstaged changes in one file
  git diff HEAD           # all changes vs. last commit (staged + unstaged)
  ```
- **When to use it:** Reviewing exactly what you've changed before staging or committing.
- **Important notes:** For staged-only changes, use `git diff --staged` (covered in [Inspection & Debugging](#8-inspection--debugging)).

### `git show`

- **What it does:** Displays the details and diff of a single commit (or other Git object like a tag).
- **How to use it:**
  ```bash
  git show HEAD
  git show a1b2c3d
  ```
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
- **When to use it:** Discarding local edits you don't want, or unstaging a file you added by mistake.
- **Important notes:** ⚠️ `git restore file.txt` permanently discards unstaged edits to that file — there's no undo once it runs (unless the content happens to still be cached elsewhere, e.g., an editor's local history).

### `git reset`

- **What it does:** Moves the current branch pointer (and optionally the staging area and working directory) to a different commit. Behavior depends heavily on the flag used — see [Commit Management](#7-commit-management) for `--soft` / `--mixed` / `--hard`.
- **How to use it:**
  ```bash
  git reset file.txt        # unstage a file, same effect as `restore --staged`
  git reset HEAD~1          # move branch back one commit (mixed by default)
  ```
- **When to use it:** Unstaging files, or rewinding your branch's history (locally).
- **Important notes:** ⚠️ Without `--soft`, `reset` alters your staging area; with `--hard` it also wipes working directory changes. Avoid resetting commits that have already been pushed and pulled by others — use `git revert` instead (see [Undoing & Recovering Changes](#5-undoing--recovering-changes)).

### `git clean`

- **What it does:** Deletes untracked files (files Git isn't tracking at all) from the working directory.
- **How to use it:**
  ```bash
  git clean -n     # dry run — preview what would be deleted
  git clean -fd    # actually delete untracked files and directories
  ```
- **When to use it:** Clearing out build artifacts, stray temp files, or other untracked clutter before a clean build or commit.
- **Important notes:** ⚠️ `git clean -fd` **permanently deletes** untracked files — they are not recoverable via Git since they were never committed. Always run `git clean -n` first to preview.

### `git rm`

- **What it does:** Removes a file from both the working directory and the staging area (schedules its deletion for the next commit).
- **How to use it:**
  ```bash
  git rm old-file.txt
  git rm --cached secrets.env   # untrack the file but keep it on disk
  ```
- **When to use it:** Deleting a file as part of a tracked change (rather than deleting it in your file explorer and then having to `git add` the deletion separately).
- **Important notes:** `--cached` is the go-to option when you accidentally committed something like a `.env` file and want Git to stop tracking it without deleting it locally. Remember to add it to `.gitignore` too.

### `git mv`

- **What it does:** Renames or moves a file and stages that change in one step.
- **How to use it:**
  ```bash
  git mv old-name.js new-name.js
  ```
- **When to use it:** Renaming or relocating tracked files.
- **Important notes:** Equivalent to `mv` + `git rm` + `git add`, just more convenient. Git detects renames automatically in most cases even with a plain `mv`, but `git mv` guarantees it's staged correctly right away.

### `git stash`

- **What it does:** Temporarily shelves (saves and removes) uncommitted changes, giving you a clean working directory.
- **How to use it:**
  ```bash
  git stash
  git stash push -m "WIP: folder modal styling"
  ```
- **When to use it:** You need to switch branches or pull changes but aren't ready to commit what you're working on.
- **Important notes:** By default, untracked files are **not** stashed — add `-u` to include them. Stashes are local only; they aren't pushed to remotes.

### `git stash pop`

- **What it does:** Reapplies the most recent stash to your working directory **and removes it** from the stash list.
- **How to use it:**
  ```bash
  git stash pop
  git stash pop stash@{2}
  ```
- **When to use it:** You've finished the interruption and want your shelved work back, and you're done with that stash entry.
- **Important notes:** ⚠️ Can produce merge conflicts if the working directory has since diverged from when the stash was made — resolve conflicts like a normal merge conflict.

### `git stash apply`

- **What it does:** Reapplies a stash to your working directory but **keeps it** in the stash list.
- **How to use it:**
  ```bash
  git stash apply
  git stash apply stash@{1}
  ```
- **When to use it:** You want to reuse the same stashed changes in more than one branch, or you're not fully confident yet and want a safety net before dropping the stash.
- **Important notes:** You'll need to `git stash drop` manually afterward if you no longer need it.

### `git stash list`

- **What it does:** Lists all currently saved stashes.
- **How to use it:**
  ```bash
  git stash list
  ```
- **When to use it:** Checking what you've stashed before applying, popping, or dropping.
- **Important notes:** Stashes are shown newest-first as `stash@{0}`, `stash@{1}`, etc. Old forgotten stashes are a common source of "wait, where did that code go?" confusion.

### `git stash drop`

- **What it does:** Deletes a specific stash entry without applying it.
- **How to use it:**
  ```bash
  git stash drop stash@{0}
  ```
- **When to use it:** Cleaning up stashes you no longer need (e.g., after confirming you already applied them, or you decided the changes weren't worth keeping).
- **Important notes:** ⚠️ Dropped stashes are hard to recover — though if truly needed, `git fsck --unreachable` can sometimes locate the dangling commit shortly after dropping.

---

## 3. Branches

### `git branch`

- **What it does:** Lists, creates, or deletes branches. With no arguments, lists local branches.
- **How to use it:**
  ```bash
  git branch                # list local branches
  git branch feature/login  # create a new branch (doesn't switch to it)
  git branch -a             # list local + remote-tracking branches
  ```
- **When to use it:** Checking what branches exist, or creating a branch without immediately switching to it.
- **Important notes:** Creating a branch is cheap and instant in Git — there's rarely a reason to avoid branching for a new piece of work.

### `git switch`

- **What it does:** Switches the working directory to an existing branch. A modern, more focused alternative to `git checkout` for branch switching.
- **How to use it:**
  ```bash
  git switch main
  git switch feature/login
  ```
- **When to use it:** Moving between existing branches.
- **Important notes:** Requires a clean-enough working directory (or non-conflicting changes) — Git will warn you if switching would overwrite uncommitted work. Stash or commit first if needed.

### `git checkout`

- **What it does:** The older, multi-purpose command for switching branches, restoring files, or checking out a specific commit/tag (detached HEAD).
- **How to use it:**
  ```bash
  git checkout main
  git checkout a1b2c3d       # detached HEAD at a specific commit
  git checkout -- file.txt   # discard changes to a file (legacy syntax)
  ```
- **When to use it:** Still common in older scripts/tutorials, or for checking out a specific commit/tag to inspect it.
- **Important notes:** Because it does so many different things based on its arguments, it's easy to use by mistake (e.g., typo a branch name and accidentally checkout a file). `git switch` and `git restore` split its responsibilities to reduce that risk — prefer them for everyday use.

### `git switch -c`

- **What it does:** Creates a new branch and switches to it in one step.
- **How to use it:**
  ```bash
  git switch -c feature/login
  git switch -c hotfix/nav-bug main   # branch off a specific starting point
  ```
- **When to use it:** Starting new work — this is the standard way to begin any feature, fix, or experiment.
- **Important notes:** Equivalent to `git checkout -b <name>` in older syntax.

### `git branch -d`

- **What it does:** Deletes a local branch, but only if it has already been fully merged.
- **How to use it:**
  ```bash
  git branch -d feature/login
  ```
- **When to use it:** Cleaning up branches after their work has been merged into `main` (or another target branch).
- **Important notes:** Git will refuse if the branch has unmerged commits — that's the safety this flag provides over `-D`.

### `git branch -D`

- **What it does:** Force-deletes a local branch, even if it has unmerged commits.
- **How to use it:**
  ```bash
  git branch -D experimental/old-idea
  ```
- **When to use it:** Discarding a branch you're certain you no longer need, merged or not.
- **Important notes:** ⚠️ Any commits that exist **only** on that branch become unreachable and are hard to recover (though `git reflog` can sometimes help shortly after).

### `git branch -m`

- **What it does:** Renames a branch.
- **How to use it:**
  ```bash
  git branch -m old-name new-name
  git branch -m new-name              # rename current branch
  ```
- **When to use it:** Fixing a typo in a branch name, or renaming to match a naming convention.
- **Important notes:** If the branch has already been pushed, you'll need to push the new name and delete the old one on the remote too (`git push origin --delete old-name` then `git push -u origin new-name`).

### `git merge`

- **What it does:** Combines the history of another branch into your current branch, creating a new merge commit (unless a fast-forward is possible).
- **How to use it:**
  ```bash
  git switch main
  git merge feature/login
  ```
- **When to use it:** Bringing completed work from a feature branch into `main` (or integrating any two branches) while preserving full branch history.
- **Important notes:** See the full [Merge & Rebase Workflows](#6-merge--rebase-workflows) section for fast-forward vs. normal merges, conflicts, and `--abort`.

### `git rebase`

- **What it does:** Replays your branch's commits on top of another branch's tip, producing a linear history instead of a merge commit.
- **How to use it:**
  ```bash
  git switch feature/login
  git rebase main
  ```
- **When to use it:** Keeping a feature branch's history clean and linear, or updating a feature branch with the latest `main` before merging.
- **Important notes:** ⚠️ Rebasing rewrites commit hashes. Never rebase commits that other people have already pulled and built on top of, unless the whole team has agreed to that workflow. See [Merge & Rebase Workflows](#6-merge--rebase-workflows) for interactive rebase, conflicts, and recovery flags.

---

## 4. Remote Repositories

### `git remote`

- **What it does:** Manages the set of remote repositories your local repo knows about.
- **How to use it:**
  ```bash
  git remote
  ```
- **When to use it:** Base command for listing, adding, renaming, or removing remotes (usually used with a subcommand — see below).
- **Important notes:** Most projects only need one remote, conventionally named `origin`.

### `git remote -v`

- **What it does:** Lists configured remotes along with their fetch/push URLs.
- **How to use it:**
  ```bash
  git remote -v
  ```
- **When to use it:** Confirming which remote URL you're pushing to/pulling from — especially useful after cloning a fork or when something pushes to the wrong place.
- **Important notes:** Fetch and push URLs can technically differ (rare, but possible in advanced setups) — this command shows both.

### `git remote add`

- **What it does:** Registers a new remote repository under a given name.
- **How to use it:**
  ```bash
  git remote add origin https://github.com/user/repo.git
  git remote add upstream https://github.com/original-owner/repo.git
  ```
- **When to use it:** Connecting a local repo (created with `git init`) to a remote for the first time, or adding a second remote (e.g., `upstream` for a forked repo).
- **Important notes:** The name (`origin`, `upstream`, etc.) is just a label — there's nothing magic about the word "origin" except convention.

### `git fetch`

- **What it does:** Downloads commits, branches, and tags from a remote **without** merging them into your local branches.
- **How to use it:**
  ```bash
  git fetch origin
  git fetch --all
  ```
- **When to use it:** Checking what's new on the remote before deciding how to integrate it, or updating remote-tracking branches for comparison (`git log main..origin/main`).
- **Important notes:** Completely safe — it never touches your working directory or local branches, only updates remote-tracking references like `origin/main`.

### `git pull`

- **What it does:** Downloads changes from a remote **and** integrates them into your current branch (fetch + merge, or fetch + rebase with `--rebase`).
- **How to use it:**
  ```bash
  git pull origin main
  git pull --rebase origin main
  ```
- **When to use it:** Bringing your local branch up to date with the remote in one step.
- **Important notes:** Because it merges (or rebases) automatically, it can produce conflicts you weren't expecting. If you want more control, `git fetch` followed by a manual `git merge`/`git rebase` is safer.

### `git push`

- **What it does:** Uploads your local commits to a remote repository.
- **How to use it:**
  ```bash
  git push origin main
  git push
  ```
- **When to use it:** Sharing your committed work with others, or backing it up remotely.
- **Important notes:** Git will refuse a push that isn't a fast-forward on the remote (i.e., someone else pushed first) — pull/rebase first rather than force-pushing by reflex.

### `git push -u`

- **What it does:** Pushes a branch and sets up tracking, so future `git push`/`git pull` on that branch don't need the remote/branch name specified.
- **How to use it:**
  ```bash
  git push -u origin feature/login
  ```
- **When to use it:** The first time you push a newly created local branch.
- **Important notes:** `-u` is short for `--set-upstream`. After this, plain `git push` and `git pull` on that branch know where to go.

### `git push --force-with-lease`

- **What it does:** Force-pushes, but refuses if the remote branch has commits you haven't seen yet (i.e., someone else pushed since your last fetch).
- **How to use it:**
  ```bash
  git push --force-with-lease origin feature/login
  ```
- **When to use it:** Pushing after rewriting history (e.g., `rebase` or `commit --amend`) on a branch you're confident you "own," such as your own feature branch.
- **Important notes:** ⚠️ Still overwrites remote history — just safer than plain `--force` because it won't silently clobber a teammate's unseen work. See the [force vs. force-with-lease comparison](#important-comparisons) below.

### Tracking Branches & Upstream

- **What it does:** A "tracking branch" (or "upstream branch") is a local branch linked to a specific remote branch, so Git knows what to compare against and where plain `push`/`pull`/`status` should operate.
- **How to use it:**
  ```bash
  git branch -u origin/main            # set upstream for current branch
  git branch -vv                       # see tracking relationships
  git push -u origin feature/login     # push + set tracking in one step
  ```
- **When to use it:** After creating a new branch you intend to push regularly, or when `git status` reports your branch is "ahead/behind" and you want to know relative to what.
- **Important notes:** `git status` and `git branch -vv` both show ahead/behind counts based on the tracking relationship — a fast way to sanity-check sync state before pushing or pulling.

---

## 5. Undoing & Recovering Changes

Git gives you several "undo" tools, and picking the right one depends on **where** the change lives: working directory, staging area, local history, or already-shared (pushed) history.

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
- **When to use it:** Undoing a commit that has already been pushed and possibly pulled by others — reverting is additive and doesn't rewrite history, so it's safe for shared branches.
- **Important notes:** Reverting a merge commit requires specifying which parent to revert to (`-m 1`), which trips people up the first time.

### `git reflog`

- **What it does:** Shows a log of everywhere `HEAD` has pointed locally — commits, checkouts, resets, rebases — even ones no longer reachable from any branch.
- **How to use it:**
  ```bash
  git reflog
  git reset --hard HEAD@{2}   # jump back to a previous HEAD position
  ```
- **When to use it:** Your local "safety net" — recovering a commit after a bad `reset --hard`, an accidental branch deletion, or a rebase that went wrong.
- **Important notes:** Reflog entries are local-only (not shared via push/clone) and eventually expire (default ~90 days for reachable entries, ~30 for unreachable) — it's a recovery window, not permanent storage.

### Practical Undo Examples

**Undo unstaged changes to a file** (discard edits, keep them gone):
```bash
git restore file.txt
```

**Unstage a file** (keep the edits, just remove from staging):
```bash
git restore --staged file.txt
```

**Undo the last commit but keep the changes** (uncommit, keep edits staged):
```bash
git reset --soft HEAD~1
```

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
git merge feature/login   # fast-forwards if main hasn't moved since branching
```

**Normal (three-way) merge:** When both branches have new commits, Git creates a dedicated merge commit with two parents, combining both histories.
```bash
git switch main
git merge feature/login   # creates a merge commit
```

**Merge conflicts:** Happen when the same lines (or a file's existence) were changed differently on both branches. Git pauses the merge and marks the conflicting files.
```bash
git merge feature/login
# CONFLICT (content): Merge conflict in src/app.js
```

**Resolving conflicts:** Open each conflicted file, look for `<<<<<<<`, `=======`, `>>>>>>>` markers, edit to the correct final content, then stage and finish.
```bash
# edit the file(s) to resolve conflicts
git add src/app.js
git commit                # finishes the merge
```

**`git merge --abort`**
- **What it does:** Cancels an in-progress merge and restores the pre-merge state.
- **How to use it:**
  ```bash
  git merge --abort
  ```
- **When to use it:** A merge conflict is messier than expected and you'd rather back out and try a different approach.
- **Important notes:** Only works while the merge is still in a conflicted, uncommitted state — not after you've completed it with `git commit`.

**Interactive rebase (`git rebase -i`):** Replays and lets you edit, reorder, squash, or drop commits one by one.
```bash
git rebase -i HEAD~4       # interactively edit the last 4 commits
```
An editor opens listing the commits with actions like `pick`, `reword`, `squash`, `fixup`, `drop`, and `edit` — change the word next to a commit to change what happens to it.

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
- **When to use it:** Fixing a typo in the last commit message, or adding changes you forgot to include in the previous commit (before it's shared).
- **Important notes:** ⚠️ Replaces the last commit with a brand-new commit (new hash). Never amend commits others have already pulled, unless force-pushing is an agreed part of your workflow. `--no-edit` keeps the existing message.

### `git cherry-pick`

- **What it does:** Copies a specific commit from another branch onto your current branch.
- **How to use it:**
  ```bash
  git cherry-pick a1b2c3d
  git cherry-pick a1b2c3d d4e5f6g        # pick several commits in order
  ```
- **When to use it:** You need one particular fix or feature from another branch without merging the whole branch.
- **Important notes:** Can produce conflicts — resolve them, `git add` the files, then `git cherry-pick --continue` (or `--abort` to back out). The copied commit gets a new hash.

### `git revert`

- **What it does:** Creates a new commit that undoes the changes introduced by a specified earlier commit, leaving history intact.
- **How to use it:**
  ```bash
  git revert a1b2c3d
  git revert HEAD
  ```
- **When to use it:** The safe way to undo commits that have already been pushed and shared.
- **Important notes:** See [Undoing & Recovering Changes](#5-undoing--recovering-changes) for the full walkthrough. Reverting a merge commit needs `-m 1` to choose which parent to revert to.

### `git reset --soft`

- **What it does:** Moves the branch pointer back while keeping all changes in the staging area.
- **How to use it:**
  ```bash
  git reset --soft HEAD~1
  ```
- **When to use it:** "Uncommit" the last commit but keep the changes staged, so you can recommit differently.
- **Important notes:** The least destructive reset — working directory is untouched.

### `git reset --mixed`

- **What it does:** Moves the branch pointer back *and* unstages changes, but keeps them in the working directory.
- **How to use it:**
  ```bash
  git reset HEAD~1
  ```
- **When to use it:** Uncommit and unstage, keeping your edits on disk so you can re-stage selectively.
- **Important notes:** This is the **default** mode — `git reset` with no flag behaves this way.

### `git reset --hard`

- **What it does:** Moves the branch pointer back, clears the staging area, and wipes changes from the working directory.
- **How to use it:**
  ```bash
  git reset --hard HEAD~1
  git reset --hard origin/main           # force your local branch to match the remote
  ```
- **When to use it:** Discarding commits *and* all of their changes completely — e.g., abandoning a bad local experiment.
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
- **When to use it:** Reviewing your edits before staging. See [Git Basics](#1-git-basics).
- **Important notes:** `git diff HEAD` compares against the last commit (staged + unstaged combined); `git diff --staged` shows only staged changes.

### `git diff --staged`

- **What it does:** Shows only the changes that are staged and will go into the next commit.
- **How to use it:**
  ```bash
  git diff --staged
  git diff --cached                      # alias for the same thing
  ```
- **When to use it:** Reviewing exactly what your next commit will contain, before committing.
- **Important notes:** If nothing is staged, it outputs nothing.

### `git log --oneline`

- **What it does:** Shows commit history as one line per commit (short hash + subject).
- **How to use it:**
  ```bash
  git log --oneline
  git log --oneline -10
  ```
- **When to use it:** A fast, scannable overview of history.
- **Important notes:** Pairs well with `--graph` and `--all`.

### `git log --graph`

- **What it does:** Renders history as an ASCII graph showing branches and merges.
- **How to use it:**
  ```bash
  git log --graph --oneline --all
  ```
- **When to use it:** Visualizing how branches diverged and where merges happened.
- **Important notes:** `--all` includes every branch and ref, not just the current branch.

### `git show`

- **What it does:** Displays the details and diff of a single commit, tag, or object.
- **How to use it:**
  ```bash
  git show HEAD
  git show v1.0.0
  ```
- **When to use it:** Inspecting exactly what a specific commit changed. See [Git Basics](#1-git-basics).
- **Important notes:** Read-only — safe to run on anything.

### `git blame`

- **What it does:** Shows, line by line, which commit last modified each line of a file, and by whom.
- **How to use it:**
  ```bash
  git blame src/app.js
  git blame -L 10,20 src/app.js          # lines 10–20 only
  ```
- **When to use it:** Finding out who introduced a specific line and when — great for hunting a bug or asking for context on a change.
- **Important notes:** Blames the last commit that *touched* each line, not necessarily the author of the logic — a later reformat can muddy the picture.

### `git reflog`

- **What it does:** A local log of everywhere HEAD has pointed — the safety net for lost commits and bad resets.
- **How to use it:**
  ```bash
  git reflog
  git reset --hard HEAD@{1}
  ```
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
- **When to use it:** Marking release points (v1.0.0, v2.3.1) or any important commit you want to reference by name.
- **Important notes:** Lightweight tags carry no author, date, or message. For releases, prefer annotated tags (below).

### `git tag -a`

- **What it does:** Creates an annotated tag — a full Git object with a tagger, date, and message.
- **How to use it:**
  ```bash
  git tag -a v1.0.0 -m "Release version 1.0.0"
  git tag -a v1.0.0 a1b2c3d -m "Bugfix release"
  ```
- **When to use it:** Releasing software where release notes and a tagger identity matter.
- **Important notes:** Annotated tags can also be signed (`-s`) — see [Advanced Git](#10-advanced-git). Recommended over lightweight tags for releases.

### `git push --tags`

- **What it does:** Pushes tags to the remote repository.
- **How to use it:**
  ```bash
  git push --tags
  git push origin v1.0.0               # push one specific tag
  ```
- **When to use it:** After tagging a release locally, to share the tag with the team and CI systems.
- **Important notes:** A plain `git push` does **not** push tags. Tags live under `refs/tags/`.

### Deleting tags

- **What it does:** Removes a tag locally and/or on the remote.
- **How to use it:**
  ```bash
  git tag -d v1.0.0                    # delete local tag
  git push origin --delete v1.0.0      # delete remote tag
  ```
- **When to use it:** Fixing a mistagged release or cleaning up.
- **Important notes:** Deleting a remote tag affects anyone else who has it — coordinate first if the tag marks a real release.

### Checking out tags

- **What it does:** Inspects the code at a specific tagged commit.
- **How to use it:**
  ```bash
  git checkout v1.0.0                  # detached HEAD at the tagged commit
  git switch -c release-1.0 v1.0.0     # create a branch from a tag
  ```
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
- **When to use it:** Porting a hotfix to multiple branches (e.g., the same bug fixed on `main` and on a release branch). Full details in [Commit Management](#7-commit-management).

### `git worktree`

- **What it does:** Lets you have multiple working directories attached to the same repository, each on a different branch.
- **How to use it:**
  ```bash
  git worktree add ../hotfix hotfix/urgent
  git worktree list
  git worktree remove ../hotfix
  ```
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
- **When to use it:** A regression appeared somewhere in history and you want to find it fast. See [Inspection & Debugging](#8-inspection--debugging) for the full walkthrough.

### `git reflog`

- **What it does:** A local-only history of every position HEAD has held.
- **How to use it:**
  ```bash
  git reflog
  git reset --hard HEAD@{2}
  ```
- **When to use it:** Rescuing commits "lost" to resets, rebases, or deleted branches. See [Undoing & Recovering Changes](#5-undoing--recovering-changes).
- **Important notes:** Never pushed or cloned; entries expire after roughly 30–90 days.

### `git filter-repo`

- **What it does:** Rewrites history at scale — removes files from all of history, renames authors/emails, or splits a repo. It is an **external tool** (install it separately; the built-in `git filter-branch` is the deprecated alternative).
- **How to use it:**
  ```bash
  git filter-repo --path secrets.env --invert-paths
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
- **When to use it:** Pinning an external dependency to an exact version, or managing a monorepo split across multiple repos.
- **Important notes:** Submodules add complexity — teammates must run `git submodule update` after pulling. The parent repo only records *which commit* the submodule points to, not its contents. Many teams prefer package managers for dependencies and reserve submodules for cases that truly need them.

### Hooks

- **What it does:** Local scripts Git runs automatically on events like `commit`, `push`, or `merge`.
- **How to use it:**
  ```bash
  # inside .git/hooks/, rename pre-commit.sample to pre-commit and add your script
  ```
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
- **When to use it:** Projects with high security requirements (kernels, many open-source projects), or any repo where the provenance of code matters.
- **Important notes:** Requires a GPG or SSH key set up first. Configure your hosting provider (GitHub/GitLab) so signed commits show a "Verified" badge. Adds setup cost, so don't enable it casually.

### Shallow clones

- **What it does:** Clones with truncated history — only the most recent commits.
- **How to use it:**
  ```bash
  git clone --depth 1 https://github.com/user/repo.git
  git clone --depth 10 --branch v2.0.0 https://github.com/user/repo.git
  ```
- **When to use it:** Quickly getting a copy of a huge repo for building or reading.
- **Important notes:** Limited history breaks `git log`, `git blame`, and `git bisect` beyond the cloned depth. `git fetch --unshallow` upgrades it to a full clone later.

### Sparse checkout

- **What it does:** Checks out only a subset of directories instead of the whole repository.
- **How to use it:**
  ```bash
  git clone --filter=blob:none --sparse https://github.com/user/monorepo.git
  git sparse-checkout set apps/web
  ```
- **When to use it:** Monorepos where you only work on part of the code — saves disk space and checkout time.
- **Important notes:** Combined with `--depth` it makes very large repos usable. Minus `--filter=blob:none` it still downloads all blobs, so include it for the full benefit.

---

## Common Real-World Git Scenarios

### Starting a new project with Git
```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/user/project.git
git push -u origin main
```

### Cloning an existing project
```bash
git clone https://github.com/user/project.git
cd project
git status
```

### Making and committing changes
```bash
git status              # see what changed
git add .
git commit -m "Add login form"
git log --oneline -5    # verify the commit landed
```

### Creating a feature branch
```bash
git switch main
git switch -c feature/login
```

### Updating a branch with the latest changes
```bash
git fetch origin
git rebase origin/main          # replay your work on top of latest main
# or, with a merge workflow:
git merge origin/main
```

### Merging a feature branch
```bash
git switch main
git merge feature/login
git branch -d feature/login     # clean up once merged
```

### Rebasing a feature branch
```bash
git switch feature/login
git rebase main
```

### Pushing a new branch
```bash
git push -u origin feature/login
```

### Pulling changes from GitHub
```bash
git pull --rebase origin main
```

### Fixing a merge conflict
```bash
git merge feature/login
# CONFLICT in src/app.js — open the file, resolve the <<<<<<< / ======= / >>>>>>> markers,
# keeping the correct final content
git add src/app.js
git commit
```

### Changing the last commit message
```bash
git commit --amend -m "Improved commit message"
```

### Adding forgotten changes to the last commit
```bash
git add forgotten-file.txt
git commit --amend --no-edit
```

### Unstaging files
```bash
git restore --staged file.txt
# or, equivalently:
git reset file.txt
```

### Discarding local changes
```bash
git restore file.txt     # a single file
git restore .            # everything tracked
```

### Undoing the last commit while keeping the files
```bash
git reset --soft HEAD~1          # keep changes staged
git reset HEAD~1                 # also unstage, keeping edits on disk
```

### Undoing a pushed commit safely
```bash
git revert <commit-hash>
git push origin main
```

### Recovering a deleted commit
```bash
git reflog
git branch recovered-work <commit-hash>   # restore it as a branch
```

### Recovering from an accidental `git reset --hard`
```bash
git reflog
git reset --hard HEAD@{1}        # back to the position before the reset
```

### Moving changes to another branch
```bash
git stash
git switch feature/other
git stash pop
```

### Copying a commit from another branch (`cherry-pick`)
```bash
git switch main
git cherry-pick a1b2c3d          # the commit hash from the other branch
```

### Cleaning untracked files
```bash
git clean -n                     # preview first — always
git clean -fd                    # then actually delete
```

---

## Important Comparisons

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

**Which to choose:** Merge to integrate shared work and keep history truthful about when things merged. Rebase to keep *your* feature branch clean and linear on top of the latest `main` — but only while the commits are local and unpushed (see [Rules of Thumb](#git-rules-of-thumb)).

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

## Git Mental Model

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

## Git Rules of Thumb

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

## Git Quick Reference

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