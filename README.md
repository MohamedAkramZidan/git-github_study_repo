# Git and GitHub Study Repository
This repository contains study notes and practical exercises for understanding Git architecture and daily workflow.
## Core Concepts
### The 3-Tree Architecture
Git manages files across three distinct areas:
* Working Tree (WT): The local directory where files are actively edited.
* Staging Area (Index): The preparation zone where files are added before committing.
* Repository (Repo / .git): The local database storing permanent snapshots.
### File States
Files in Git transition through different states:
* Untracked (U): New files not yet watched by Git.
* Tracked: Files Git is monitoring.
  * Unmodified: No changes since the last commit.
  * Modified (M): File has been changed.
    * M (Red): Modified but not in the index (not staged).
    * M (Green): Modified and added to the index (staged).
### Git Objects
Git generates a unique SHA-1 hash for everything and stores them in the .git/objects folder. The primary objects are:
* Blob: Stores raw file content and metadata.
* Tree: Stores directory structure and groups of blobs.
* Commit: Stores snapshot metadata (author, date, message) and points to a tree.
* Tagged Annotation: Used to mark specific commits.
## Standard Command Workflow (full bash history save in obsidian this summary)
* `git init`<br>
  Initializes a new repository by creating the .git directory structure.
* `git status`<br>
  Shows the current state of the working directory and staging area.
* `git status -s`<br>
  Provides a condensed, short status summary.<br>
  Example output:<br>
  M file.txt
* `git ls-files`<br>
  Lists all files currently tracked in the repository.<br>
  Example output:<br>
  README.md<br>
  file.txt
* `git add .`<br>
  Moves all modified and untracked files into the Staging Area.
* `git commit -m "message"`<br>
  Saves the staging area as a permanent commit in the repository.
* `git log`<br>
  Displays the commit history, including the SHA-1 hash, author details, date, and message.

## Inspecting the .git Directory
Exploring `.git/` directly shows the internal structure Git relies on:
* `.git/COMMIT_EDITMSG` — the last commit message used.
* `.git/HEAD` — pointer to the current branch reference.
* `.git/config` — repository-specific configuration.
* `.git/index` — the staging area itself, stored as a binary file.
* `.git/objects/` — contains subfolders (named by the first 2 hash characters, e.g. `17/`, `92/`, `f4/`) holding compressed blob/tree/commit objects, plus `info/` and `pack/` for packed data.
* `.git/refs/` and `.git/logs/` — branch references and their history.

## Committing Changes — Case Walkthrough
* Created a file, checked untracked status, staged, then committed:<br>
  `echo "text" >> file.txt` → `git status` (shows Untracked files) → `git add .` → `git status` (shows Changes to be committed) → `git commit -m "Initial commit"`.
* **CRLF/LF warning:** on Windows, `git add` frequently prints:<br>
  `warning: in the working copy of 'file.txt', LF will be replaced by CRLF the next time Git touches it`<br>
  This is Git's `core.autocrlf` line-ending normalization and is not an error.
* After committing, `git status` shows `Your branch is ahead of 'origin/main' by N commit(s)` — meaning local commits exist that haven't been pushed yet.

## Viewing History
* `git log`<br>
  Full history with hash, author, date, message for every commit.
* `git log --oneline`<br>
  Condensed one-line-per-commit view.<br>
  Example: `f1be401 (HEAD -> main) confim`
* `git log --oneline -2`<br>
  Limits output to the last 2 commits. Note: the count flag (`-2`) must come **before** any file path argument, otherwise Git errors with:<br>
  `fatal: option '-2' must come before non-option arguments`
* `git log --oneline file.txt`<br>
  Shows only commits that touched `file.txt`.
* `git log --oneline --graph`<br>
  Adds an ASCII graph showing branch/merge topology alongside each commit.

## Comparing Changes
* `git diff`<br>
  Shows unstaged changes (working tree vs. staging area). Empty output means no unstaged differences.
* `git diff --staged`<br>
  Shows staged changes (staging area vs. last commit).
* `git diff <commit1>..<commit2>`<br>
  Compares two arbitrary commits directly, e.g. `git diff dcd89a9..f1be401`.
* `git show`<br>
  Shows the diff introduced by the most recent commit (HEAD), including metadata.
* `git show <commit>`<br>
  Shows the diff introduced by a specific commit, e.g. `git show dcd89a9`.

## Undoing / Restoring Changes
* `git restore --staged <file>`<br>
  Unstages a file, moving it from the index back to modified-but-not-staged. (Note: `git restore -- staged file.txt` fails — `staged` is interpreted as a filename, producing `error: pathspec 'staged' did not match any file(s) known to git`.)
* `git restore <file>`<br>
  Discards uncommitted working-directory changes, reverting the file to match the last commit/staged state.
* `git reset HEAD~1`<br>
  Moves HEAD (and the branch pointer) back one commit, keeping the changes from that commit as **unstaged** modifications in the working tree (a mixed reset).
* `git reset --hard HEAD~1`<br>
  Moves HEAD back one commit and **discards** all working-tree and staged changes to match that commit exactly.
* `git reset --hard <commit-or-tag>`<br>
  Hard-resets the branch to any specific commit or tag, e.g. `git reset --hard v2.0` or `git reset --hard 199d17c`.
* Case observed: committing with `git commit -am "message"` when nano was left with a blank/incomplete edit produced a commit with a huge block of blank lines in the message before the real message — a reminder to save/exit the editor cleanly.

## Amending and Rewriting History
* `git commit --amend`<br>
  Replaces the most recent commit with a new one (new message and/or new staged changes), effectively rewriting history. Seen in the reflog as `commit (amend): ...`.
* `git revert`<br>
  (Referenced as an alternative to resetting) — creates a **new** commit that undoes the changes of a previous commit, without rewriting history. Safer than reset/amend for shared branches.
* `git checkout HEAD file.txt`<br>
  (Older equivalent of `git restore`) — restores a file from the HEAD commit into the working tree.

## Reflog — Recovering History
* `git reflog`<br>
  Shows a chronological log of every place HEAD has pointed (commits, resets, amends, merges, pulls), even ones no longer reachable from the current branch tip. Each entry has a `HEAD@{N}` reference.<br>
  Used throughout this session to recover from `reset --hard` operations, confirm what a `commit --amend` replaced, and trace exactly how the branch moved after merges and force-pushes.

## Tagging
* `git tag -a v2.0 -m "Version 2"`<br>
  Creates an **annotated** tag (`-a`) with a message (`-m`) at the current commit. Annotated tags store tagger name, date, and message as their own object (unlike lightweight tags).
* `git show v2.0`<br>
  Displays the tag's metadata (tagger, date, message) followed by the diff of the commit it points to.
* Tags can be used anywhere a commit hash is expected, e.g. `git reset --hard v2.0`.

## Working with Remotes
* `git push origin main`<br>
  Pushes local commits on `main` to the `origin` remote. On success shows the range of commits pushed (e.g. `3fc1cb7..199d17c main -> main`).
* `git push --force-with-lease origin main`<br>
  Force-pushes, but only if the remote hasn't changed since your last fetch (safer than `--force`, which would blindly overwrite). Used after `reset --hard` to make the remote match a rewritten local history.
* Case: a plain `git push origin main` after a local `reset --hard` to an older commit was **rejected**:<br>
  `! [rejected] main -> main (non-fast-forward)` with the hint to `git pull` first — Git refuses to lose remote-only commits with a normal push.
* `git pull`<br>
  Fetches from `origin` and merges (or fast-forwards) into the current branch. Resolved the rejected-push case above.
* Case: after deleting a local branch (`test`) that had already been pushed, a later `git push origin main` failed with:<br>
  `! [rejected] main -> main (fetch first)` — remote had commits not present locally — resolved again with `git pull` (fast-forward).
* `git fetch origin`<br>
  Downloads new commits/refs from the remote **without** merging them into the working branch (unlike `pull`). Updates remote-tracking refs like `origin/master`.
* `git branch -r`<br>
  Lists remote-tracking branches, e.g. `origin/HEAD -> origin/master`, `origin/master`.
* `git branch -vv`<br>
  Shows local branches with their upstream tracking branch and ahead/behind status, e.g.<br>
  `* master 83faf4d [origin/master: ahead 2] second`
* `git branch -v`<br>
  Shows local branches with the latest commit hash and message, plus ahead/behind summary.
* `git clone <path-or-url> <target-dir>`<br>
  Creates a full local copy of a repository (used here to clone a local `remote-repo/` into `local-repo/`), automatically setting up `origin` and tracking branches.

## Branching
* `git branch <name>`<br>
  Creates a new branch pointing at the current commit (does not switch to it).
* `git branch`<br>
  Lists local branches, marking the current one with `*`.
* `git switch <branch>`<br>
  Switches the working directory/HEAD to another branch (modern replacement for `git checkout <branch>`).
* `git branch -d <branch>`<br>
  Deletes a branch **safely** — refuses if the branch has unmerged commits:<br>
  `error: the branch 'test' is not fully merged` with a hint to use `-D`.<br>
  Also refuses if the branch is checked out in a worktree:<br>
  `error: cannot delete branch 'test' used by worktree at '...'`
* `git branch -D <branch>`<br>
  Force-deletes a branch regardless of merge status.
* `git branch --merged`<br>
  Lists branches already fully merged into the current branch.
* `git push origin <branch>`<br>
  Pushes a new branch to the remote for the first time; GitHub responds with a suggested pull-request URL, e.g.:<br>
  `remote: Create a pull request for 'testing' on GitHub by visiting: .../pull/new/testing` and reports `* [new branch] testing -> testing`.
* `git push --set-upstream origin <branch>`<br>
  Pushes a branch and links it to the matching remote branch so future plain `git push`/`git pull` work without specifying remote/branch. Needed when pushing a branch that has no upstream yet:<br>
  `fatal: The current branch feature has no upstream branch.`
* `git push origin test` after the local `test` branch was deleted<br>
  Failed with `error: src refspec test does not match any` — there is no local branch named `test` to push.

## Merging
* `git merge <branch>`<br>
  Merges another branch into the current one.
  * **Fast-forward merge:** if the current branch has no new commits since it diverged, Git simply moves the branch pointer forward (`Fast-forward` message), no merge commit is created.
  * **Three-way merge:** if both branches have diverged with new commits, Git creates a merge commit (`Merge made by the 'ort' strategy.`), which can show as a long block of blank lines in the editor if not handled carefully.
* `git merge` (no branch argument, on a branch tracking a remote)<br>
  Merges in the fetched changes from the configured upstream — fails with `fatal: No remote for the current branch.` if the branch (e.g. a local-only `master` in the bare "remote" repo) has no tracking remote configured.
* Merge commits are shown in `git log` with two parent hashes, e.g.:<br>
  `Merge: a41f27c 0a0ea04`

## Merge Conflicts
* Case: pulling from `origin` when both local and remote had independently modified the same lines of `remote.txt` produced:<br>
  ```
  Auto-merging remote.txt
  CONFLICT (content): Merge conflict in remote.txt
  Automatic merge failed; fix conflicts and then commit the result.
  ```
* `git status` during a conflict shows the branch state as `(master|MERGING)` and lists:<br>
  `Unmerged paths: ... both modified: remote.txt`<br>
  with guidance to either fix and `git add`, or run `git merge --abort`.
* **Resolution workflow used:**
  1. Open the conflicted file in an editor (`nano remote.txt`) and manually resolve the conflict markers.
  2. `git add .` — marks the conflict as resolved.
  3. `git commit -m "message"` — completes the merge (no `-m` argument is required for a merge commit if you accept the default merge message, but a custom one was supplied here).
* After resolving, `git status` shows the branch ahead of origin by the number of commits involved in the merge (e.g. by 2 commits: the incoming remote commit plus the merge/resolution commit).

## Working with Multiple Local Repos (Simulating Remote/Local)
* Created a second local repository (`remote-repo/`) to act as a stand-in "remote", then cloned it into `local-repo/` to simulate a real developer/remote relationship — useful for practicing `push`/`pull`/`fetch`/`merge` without needing GitHub.
* Demonstrated that changes made directly in the "remote" repo (`remote-repo/`) are only reflected in `local-repo/` after `git fetch` + `git merge`, or `git pull` (fetch + merge combined).
