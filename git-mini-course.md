# Git Mini Course: From Comfortable Solo Work to Confident Conflict Handling

This guide assumes you already know the everyday Git commands:

- `git clone`
- `git add`
- `git commit`
- `git push`
- `git pull`
- basic `git merge`
- basic `git rebase`
- basic `git stash`

The goal is to make Git feel less mysterious when the workflow gets messy: pulling while you have local changes, using stash safely, resolving merge conflicts, understanding merge vs rebase, undoing mistakes, and practicing everything with a tiny HTML project.

---

## 1. The Core Mental Model

Git is mostly about snapshots and pointers.

Every commit is a snapshot of your project at one moment. A branch is a movable label pointing at one commit. `HEAD` means "where I am right now."

When you run:

```bash
git status
```

Git is comparing three places:

1. Your working tree: the files you are editing.
2. The staging area: what you have prepared for the next commit.
3. The current commit: the last saved snapshot on your branch.

Most confusion gets easier when you ask:

> Is this change only in my files, staged for commit, committed locally, or pushed to the remote?

Useful inspection commands:

```bash
git status
git log --oneline --graph --decorate --all
git diff
git diff --staged
```

What they tell you:

- `git status`: what changed and what Git expects next.
- `git log --oneline --graph --decorate --all`: branch history as a map.
- `git diff`: unstaged file changes.
- `git diff --staged`: staged changes waiting to be committed.

---

## 2. Practice Repo: Tiny HTML Project

Create a folder anywhere safe:

```bash
mkdir git-practice
cd git-practice
git init
```

Create `index.html`:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Git Practice</title>
  </head>
  <body>
    <h1>Git Practice</h1>
    <p>This is my first version.</p>
  </body>
</html>
```

Commit it:

```bash
git add index.html
git commit -m "Create practice page"
```

Now make another small change:

```html
<p>This is my second version.</p>
```

Inspect it:

```bash
git status
git diff
```

Commit it:

```bash
git add index.html
git commit -m "Update page copy"
```

Look at history:

```bash
git log --oneline --graph --decorate --all
```

That log command is one of the best habits you can build.

---

## 3. Branches Without Fear

A branch lets you try work without touching your main line.

```bash
git switch -c add-styles
```

Add a `style.css` file:

```css
body {
  font-family: Arial, sans-serif;
  max-width: 720px;
  margin: 40px auto;
  line-height: 1.5;
}
```

Link it in `index.html`:

```html
<link rel="stylesheet" href="style.css" />
```

Commit:

```bash
git add index.html style.css
git commit -m "Add basic styles"
```

Switch back:

```bash
git switch main
```

Merge the branch:

```bash
git merge add-styles
```

If Git can simply move `main` forward, it performs a fast-forward merge. That means `main` had no new commits of its own, so Git just moves the branch pointer.

Delete the branch after merging:

```bash
git branch -d add-styles
```

Use lowercase `-d` when the branch is merged. Use uppercase `-D` only when you intentionally want to delete an unmerged branch.

---

## 4. Merge vs Rebase

Both merge and rebase combine work. They do it differently.

### Merge

Merge says:

> Keep both histories and create a commit that ties them together.

Example:

```bash
git switch main
git merge feature-branch
```

Pros:

- Preserves exact history.
- Safer for shared branches.
- Easier when you want to show that two lines of work came together.

Cons:

- History can get busier with merge commits.

### Rebase

Rebase says:

> Replay my commits on top of another branch as if I started from there.

Example:

```bash
git switch feature-branch
git rebase main
```

Pros:

- Creates a cleaner, more linear history.
- Nice for local feature branches before opening or updating a PR.

Cons:

- Rewrites commit history.
- Risky if other people already pulled your branch.

### Rule of Thumb

Use merge when:

- You are combining shared branches.
- You want the safest team workflow.
- You do not care about perfectly linear history.

Use rebase when:

- You are cleaning up your own local branch.
- You want your feature branch updated on top of latest `main`.
- Nobody else is depending on your branch history.

Avoid rebasing public/shared branches unless your team explicitly uses that workflow.

---

## 5. Pull: What It Really Does

This:

```bash
git pull
```

is really shorthand for:

```bash
git fetch
git merge origin/current-branch
```

Or, if configured:

```bash
git fetch
git rebase origin/current-branch
```

Safer habit:

```bash
git fetch
git status
git log --oneline --graph --decorate --all
```

Then decide:

```bash
git merge origin/main
```

or:

```bash
git rebase origin/main
```

Why `fetch` is nice:

- It updates your knowledge of the remote.
- It does not modify your working files.
- You can inspect before merging or rebasing.

---

## 6. Pulling When You Have Local Changes

This is one of the most common real-life Git moments.

You try:

```bash
git pull
```

Git says something like:

```text
error: Your local changes to the following files would be overwritten by merge
```

Git is protecting your uncommitted work.

You have three main options.

### Option A: Commit First

Best when the work is real and worth saving as a checkpoint.

```bash
git add .
git commit -m "WIP: save local changes"
git pull
```

Later you can clean up the commit if needed.

### Option B: Stash First

Best when your changes are temporary or unfinished.

```bash
git stash push -m "WIP before pulling main"
git pull
git stash pop
```

What happens:

- `git stash push` stores your local changes and cleans your working tree.
- `git pull` updates your branch.
- `git stash pop` reapplies your saved changes and removes that stash if successful.

If `stash pop` causes conflicts, Git keeps the stash until you resolve the conflict, so your work is not lost.

### Option C: Discard Local Changes

Only use this when you truly do not want the local edits.

Discard changes in one file:

```bash
git restore index.html
```

Discard all unstaged changes:

```bash
git restore .
```

Discard staged changes back to unstaged:

```bash
git restore --staged .
```

Discard staged and unstaged changes:

```bash
git restore --staged .
git restore .
```

Be careful: restoring uncommitted changes throws them away.

---

## 7. Stash: The Practical Guide

Stash is a temporary shelf for uncommitted work.

### Save Changes

```bash
git stash push -m "message describing the stash"
```

Include untracked files too:

```bash
git stash push -u -m "include new files too"
```

By default, `git stash` does not include new untracked files. That catches people all the time.

### List Stashes

```bash
git stash list
```

Example:

```text
stash@{0}: On main: WIP before pulling main
stash@{1}: On feature: navbar experiment
```

`stash@{0}` is the newest.

### See What Is in a Stash

Summary:

```bash
git stash show stash@{0}
```

Detailed patch:

```bash
git stash show -p stash@{0}
```

### Reapply a Stash and Keep It

```bash
git stash apply stash@{0}
```

Use this when you want to test applying it without deleting the stash.

### Reapply a Stash and Remove It

```bash
git stash pop stash@{0}
```

Use this when you are confident you want it back.

### Delete One Stash

```bash
git stash drop stash@{0}
```

### Delete All Stashes

```bash
git stash clear
```

Use `clear` carefully. It removes every stash.

### Make a Branch From a Stash

This is underrated:

```bash
git stash branch recover-work stash@{0}
```

Git creates a new branch from where the stash was originally made, applies the stash, and drops it if successful.

This is great when a stash does not apply cleanly to your current branch.

---

## 8. Merge Conflicts: What They Mean

A conflict happens when Git cannot safely combine changes.

Example:

Branch A changed a line to:

```html
<p>Welcome to my Git practice page.</p>
```

Branch B changed the same line to:

```html
<p>This page teaches merge conflicts.</p>
```

Git does not know which one you want, so it stops and asks you to choose.

Conflict markers look like this:

```html
<<<<<<< HEAD
<p>Welcome to my Git practice page.</p>
=======
<p>This page teaches merge conflicts.</p>
>>>>>>> feature-branch
```

Meaning:

- `HEAD`: your current branch version.
- middle section after `=======`: the incoming version.
- `>>>>>>> feature-branch`: the branch being merged or rebased.

Your job is to edit the file into the final version you actually want.

For example:

```html
<p>Welcome to my Git practice page. This page teaches merge conflicts.</p>
```

Then:

```bash
git add index.html
git commit
```

If you were rebasing:

```bash
git add index.html
git rebase --continue
```

---

## 9. Merge Conflict Drill

Start clean:

```bash
git status
```

Create a branch:

```bash
git switch -c version-a
```

Change the paragraph in `index.html`:

```html
<p>This is version A of the page.</p>
```

Commit:

```bash
git add index.html
git commit -m "Write version A paragraph"
```

Go back to main:

```bash
git switch main
```

Create another branch:

```bash
git switch -c version-b
```

Change the same paragraph differently:

```html
<p>This is version B of the page.</p>
```

Commit:

```bash
git add index.html
git commit -m "Write version B paragraph"
```

Now merge `version-a` into `version-b`:

```bash
git merge version-a
```

You should get a conflict.

Open `index.html`, remove the conflict markers, and make the final paragraph:

```html
<p>This page combines version A and version B.</p>
```

Finish:

```bash
git add index.html
git commit
```

Inspect:

```bash
git log --oneline --graph --decorate --all
```

You just resolved a merge conflict.

---

## 10. Rebase Conflict Drill

Create a new branch from `main`:

```bash
git switch main
git switch -c rebase-practice
```

Change the paragraph:

```html
<p>This paragraph was changed on the rebase-practice branch.</p>
```

Commit:

```bash
git add index.html
git commit -m "Change paragraph on rebase branch"
```

Switch to `main`:

```bash
git switch main
```

Change the same paragraph differently:

```html
<p>This paragraph was changed directly on main.</p>
```

Commit:

```bash
git add index.html
git commit -m "Change paragraph on main"
```

Now rebase:

```bash
git switch rebase-practice
git rebase main
```

You should get a conflict.

Open `index.html`, choose the final text, remove conflict markers, then:

```bash
git add index.html
git rebase --continue
```

If you panic:

```bash
git rebase --abort
```

That returns you to where you were before the rebase attempt.

---

## 11. Conflict Commands You Should Know

When Git is in a conflicted state:

```bash
git status
```

This tells you exactly which files need attention and what command to run next.

See conflicted files:

```bash
git diff --name-only --diff-filter=U
```

See conflict details:

```bash
git diff
```

During a merge, abort:

```bash
git merge --abort
```

During a rebase, abort:

```bash
git rebase --abort
```

During a cherry-pick, abort:

```bash
git cherry-pick --abort
```

Take your current branch's version of a conflicted file:

```bash
git checkout --ours index.html
git add index.html
```

Take the incoming version:

```bash
git checkout --theirs index.html
git add index.html
```

Important: during rebase, "ours" and "theirs" can feel reversed because Git is replaying commits. Prefer manually editing conflicts during rebase until you are comfortable.

---

## 12. Undoing Mistakes

### Undo Unstaged File Changes

```bash
git restore index.html
```

### Unstage a File

```bash
git restore --staged index.html
```

### Amend the Last Commit

Use this when you forgot a file or want to fix the last commit message.

```bash
git add forgotten-file.html
git commit --amend
```

Change only the message:

```bash
git commit --amend -m "Better commit message"
```

Avoid amending commits you already pushed if other people may have pulled them.

### Revert a Pushed Commit

Safest way to undo a public commit:

```bash
git revert <commit-sha>
```

This creates a new commit that reverses the old one.

### Reset Local Commits

Use reset mostly for local, unpushed work.

Move branch back but keep changes staged:

```bash
git reset --soft HEAD~1
```

Move branch back and keep changes unstaged:

```bash
git reset HEAD~1
```

Move branch back and discard changes:

```bash
git reset --hard HEAD~1
```

Be very careful with `--hard`. It discards work.

---

## 13. Recovery: Reflog Is Your Safety Net

If you think you lost a commit:

```bash
git reflog
```

Reflog records where `HEAD` has been.

Example:

```text
abc1234 HEAD@{0}: reset: moving to HEAD~1
def5678 HEAD@{1}: commit: Add important work
```

Recover by making a branch at the old commit:

```bash
git switch -c recovery def5678
```

This is one reason Git is less scary than it seems. Many "lost" commits are recoverable if they were committed at least once.

---

## 14. A Good Everyday Workflow

When starting work:

```bash
git switch main
git fetch
git pull
git switch -c my-feature
```

While working:

```bash
git status
git diff
git add .
git commit -m "Clear message"
```

Before pushing:

```bash
git fetch
git rebase origin/main
```

Resolve conflicts if needed, then:

```bash
git push -u origin my-feature
```

For solo work, this is usually pleasant:

- Use feature branches for experiments.
- Commit often.
- Use `fetch` before big branch operations.
- Use `stash -u` when you need to pull but are not ready to commit.
- Use `reflog` when something feels lost.

---

## 15. Common Scenarios and Recipes

### I Need to Pull but Have Unfinished Work

```bash
git stash push -u -m "WIP before pull"
git pull
git stash pop
```

If conflicts happen:

```bash
git status
```

Fix files, then:

```bash
git add .
git commit
```

or continue the operation Git tells you about.

### I Popped a Stash and Got Conflicts

```bash
git status
```

Open conflicted files, fix them, then:

```bash
git add .
git commit -m "Apply stashed changes"
```

Check stash list:

```bash
git stash list
```

If the stash is still there and you no longer need it:

```bash
git stash drop stash@{0}
```

### I Want to Update My Feature Branch With Main

Merge style:

```bash
git switch my-feature
git fetch
git merge origin/main
```

Rebase style:

```bash
git switch my-feature
git fetch
git rebase origin/main
```

For solo feature branches, rebase is often tidy. For shared branches, merge is safer.

### I Want to Throw Away One File's Changes

```bash
git restore path/to/file
```

### I Want to Throw Away All Local Uncommitted Changes

```bash
git restore --staged .
git restore .
```

This does not remove untracked files. To preview untracked cleanup:

```bash
git clean -nd
```

To actually remove untracked files:

```bash
git clean -fd
```

Be careful with `git clean -fd`; it deletes untracked files.

### I Committed to the Wrong Branch

Suppose you committed on `main` but meant to commit on `my-feature`.

Create the branch at your current commit:

```bash
git switch -c my-feature
```

Now move `main` back:

```bash
git switch main
git reset --hard HEAD~1
```

Only do this if the mistaken commit was not pushed to a shared remote.

### I Need to See What Changed in One Commit

```bash
git show <commit-sha>
```

### I Need to Compare Branches

```bash
git diff main..my-feature
```

See commits on your branch that are not on main:

```bash
git log main..my-feature --oneline
```

---

## 16. Reading Git Status Like Instructions

When in doubt, trust:

```bash
git status
```

Git usually tells you:

- Which operation is in progress.
- Which files are conflicted.
- Which files are staged.
- Which files are unstaged.
- Which command continues or aborts the operation.

Example during rebase:

```text
You are currently rebasing branch 'feature' on 'abc1234'.
fix conflicts and then run "git rebase --continue"
use "git rebase --abort" to check out the original branch
```

That is your checklist.

---

## 17. Suggested Practice Schedule

### Day 1: Inspecting and Committing

Practice:

```bash
git status
git diff
git diff --staged
git log --oneline --graph --decorate --all
```

Goal: know where your changes are.

### Day 2: Branches and Merge

Practice:

```bash
git switch -c branch-name
git merge branch-name
git branch -d branch-name
```

Goal: understand branch pointers and fast-forward merges.

### Day 3: Stash

Practice:

```bash
git stash push -u -m "practice stash"
git stash list
git stash show -p stash@{0}
git stash apply stash@{0}
git stash drop stash@{0}
```

Goal: safely pause and resume unfinished work.

### Day 4: Merge Conflicts

Run the merge conflict drill.

Goal: become comfortable editing conflict markers.

### Day 5: Rebase Conflicts

Run the rebase conflict drill.

Goal: understand `rebase --continue` and `rebase --abort`.

### Day 6: Recovery

Practice:

```bash
git commit --amend
git reset --soft HEAD~1
git reflog
```

Goal: know how to recover from common local mistakes.

---

## 18. The Git Commands Worth Memorizing

Daily:

```bash
git status
git diff
git add .
git commit -m "message"
git fetch
git pull
git push
git log --oneline --graph --decorate --all
```

Branches:

```bash
git switch main
git switch -c feature-name
git branch
git branch -d feature-name
```

Stash:

```bash
git stash push -u -m "message"
git stash list
git stash show -p stash@{0}
git stash apply stash@{0}
git stash pop stash@{0}
git stash drop stash@{0}
```

Conflicts:

```bash
git merge --abort
git rebase --abort
git rebase --continue
git diff --name-only --diff-filter=U
```

Undo:

```bash
git restore file
git restore --staged file
git commit --amend
git revert <commit-sha>
git reflog
```

Danger zone:

```bash
git reset --hard
git clean -fd
git push --force
```

Use those only when you understand exactly what will be discarded or rewritten.

---

## 19. Final Mental Checklist

Before doing a Git operation that feels risky, ask:

1. Is my work committed, stashed, or disposable?
2. Am I on the branch I think I am on?
3. Have I run `git status`?
4. Have I looked at the branch graph?
5. Is this branch shared with anyone?
6. Am I rewriting history or only adding new history?

The most important habit is not memorizing every command. It is pausing long enough to inspect the state before changing it.

When Git feels confusing, run:

```bash
git status
git log --oneline --graph --decorate --all
```

Those two commands usually tell you where you are and what happened.
