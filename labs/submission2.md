# Lab 2


## Task 1 — Exploring the Git Object Model with `git cat-file`

### Goal

Inspect low–level Git objects (commit, tree, and blob) to understand how Git stores data internally.

### Commands and outputs

I first created a new feature branch and a simple test file:

```bash
git switch -c feature/lab2_new
echo "Add test file" > test.txt
git add test.txt
git commit -m "Add test file"
```

Then I inspected the latest commit:

```bash
git log --oneline -1
```

_Output (example):_

```text
29d3fa4 (HEAD -> feature/lab2_new) Add test file
```

To see the full commit object, I used:

```bash
git cat-file -p HEAD
```

_Output (abridged):_

```text
tree 00f17be68d5103171596f1ecb5ed87516c979ce5
parent 56df3fb900088a19e184c5a53f5510846b16287
author Ilyas Galiev <...>
committer Ilyas Galiev <...>

Add test file
```

From the `tree` line I took the tree hash and inspected the tree object:

```bash
git cat-file -p 00f17be68d5103171596f1ecb5ed87516c979ce5
```

_Output (abridged):_

```text
100644 blob <README_blob_hash>    README.md
040000 tree <app_tree_hash>       app
040000 tree <labs_tree_hash>      labs
040000 tree <lectures_tree_hash>  lectures
100644 blob <test_blob_hash>      test.txt
```

Finally I displayed the blob for `test.txt`:

```bash
git cat-file -p <test_blob_hash>
```

_Output:_

```text
Add test file
```

### Explanation

- A **blob** object stores the raw contents of a single file (no filename, no directory structure).
- A **tree** object represents a directory: it contains filenames and references to blobs and other trees.
- A **commit** object points to a single tree (the root of the project), plus metadata (author, date, message) and references to parent commits.

Together, commits, trees, and blobs form a graph addressed by hashes. Git does not store “diffs” by default; instead, it stores immutable snapshots where each snapshot is identified by the SHA hash of its contents.

---

## Task 2 — Using `git reset` and Recovering with `git reflog`

### Goal

Practice different reset modes (`--soft`, `--hard`) and use `git reflog` to safely recover previous states.

### Commands and outputs

I created a dedicated practice branch and three commits:

```bash
git switch -c git-reset-practice2

echo "First commit" > file.txt
git add file.txt
git commit -m "First commit"

echo "Second commit" >> file.txt
git add file.txt
git commit -m "Second commit"

echo "Third commit" >> file.txt
git add file.txt
git commit -m "Third commit"
```

Then I checked the history:

```bash
git log --oneline
```

_Output (top part):_

```text
e3be50a (HEAD -> git-reset-practice2) Third commit
a84cc1a Second commit
d5161a6 First commit
29d3fa4 (feature/lab2_new) Add test file
56df3fb  chore: add lab2 submission skeleton
87810a0  feat: remove old Exam Exemption Policy
...
```

#### `git reset --soft`

I moved `HEAD` one commit back, keeping changes staged:

```bash
git reset --soft HEAD~1
git status
```

_Output:_

```text
On branch git-reset-practice2
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   file.txt

Untracked files:
        .idea/
```

Interpretation:  
- `HEAD` now points to `Second commit`.  
- The changes from `Third commit` are still present and staged in the index.

#### `git reset --hard`

Next, I moved `HEAD` one more commit back and discarded changes:

```bash
git reset --hard HEAD~1
git status
```

_Output:_

```text
HEAD is now at d5161a6 First commit
On branch git-reset-practice2
Untracked files:
        .idea/

nothing added to commit but untracked files present (use "git add" to track)
```

Interpretation:  
- `HEAD` now points to `First commit`.  
- Both `Second` and `Third` commit changes have been removed from the index and working tree.  
- Untracked files (like `.idea/`) are not affected by `reset`.

#### Recovery with `git reflog`

To recover the previous state with all three commits, I used `git reflog`:

```bash
git reflog
```

_Output (relevant part):_

```text
d5161a6 (HEAD -> git-reset-practice2) HEAD@{0}: reset: moving to HEAD~1
a84cc1a HEAD@{1}: reset: moving to HEAD~1
e3be50a HEAD@{2}: commit: Third commit
a84cc1a HEAD@{3}: commit: Second commit
d5161a6 HEAD@{4}: commit: First commit
...
```

Then I reset back to the commit with all three changes:

```bash
git reset --hard e3be50a
git log --oneline
```

_Output (top part):_

```text
e3be50a (HEAD -> git-reset-practice2) Third commit
a84cc1a Second commit
d5161a6 First commit
...
```

### Explanation

- `git reset --soft <target>`:
  - Moves `HEAD` to `<target>`.
  - Leaves both the index (staging area) and working directory unchanged.
  - Effectively “uncommits” changes but keeps them staged.
- `git reset --hard <target>`:
  - Moves `HEAD` to `<target>`.
  - Resets both index and working directory to match `<target>`.
  - Discards local changes (tracked files).

`git reflog` keeps a local history of where `HEAD` has pointed (including resets, checkouts, merges).  
Even if commits become unreachable from any branch, they still appear in the reflog for some time, so you can recover them by resetting or creating a new branch at the desired hash.

---

## Task 3 — Visualizing Commit History with `git log --graph`

### Goal

Use `git log --graph --all` to visualize branches and understand how commits relate to each other.

### Commands and outputs

From `feature/lab2_new` I created a side branch and added one commit:

```bash
git switch feature/lab2_new
git switch -c side-branch2

echo "Branch commit" >> history.txt
git add history.txt
git commit -m "Side branch commit"
```

Then I returned to `feature/lab2_new`:

```bash
git switch -
```

Finally, I visualized the commit history with all branches:

```bash
git log --oneline --graph --all
```

_Output (abridged):_

```text
* 463d934 (side-branch2) Side branch commit
* d5161a6 (git-reset-practice2) First commit
* 29d3fa4 (HEAD -> feature/lab2_new) Add test file
* 56df3fb chore: add lab2 submission skeleton
* 6fc682c (origin/feature/lab2, feature/lab2) docs: add lab12 submission
| * 4043245 (feature/lab11) Merge branch 'main' of ...
|/
* 87810a0 (upstream/main, origin/main, origin/HEAD, main) feat: remove old Exam Exemption Policy
...
```

### Explanation

- `git log --graph --all --oneline` draws an ASCII graph on the left side, where:
  - Each line with `*` represents a commit.
  - Vertical and diagonal lines show branching and merging relations.
- This visualization makes it easier to see:
  - Where side branches diverge from `main`.
  - How feature branches are merged.
  - Which commit is currently checked out (`HEAD -> ...` annotation).

For this task, I can clearly see:
- The `side-branch2` branch with its own commit on top.
- The `git-reset-practice2` branch diverging earlier.
- The `feature/lab2_new` branch sitting on top of the lab2-related changes.

---

## Task 4 — Tagging Commits with Semantic Versions

### Goal

Create and push annotated points in history using Git tags, suitable for marking releases such as `v1.0.0`.

### Commands and outputs

After completing the lab tasks on `feature/lab2_new`, I created a lightweight tag for the current state:

```bash
git tag v1.0.2
```

Then I pushed the tag to my fork:

```bash
git push origin v1.0.2
```

_Output:_

```text
Enumerating objects: 9, done.
Counting objects: 100% (9/9), done.
Delta compression using up to 8 threads
Compressing objects: 100% (6/6), done.
Writing objects: 100% (7/7), 1.12 KiB | 1.12 MiB/s, done.
Total 7 (delta 3), reused 0 (delta 0), pack-reused 0 (from 0)
To github.com:Ily17as/F25-DevOps-Intro.git
 * [new tag]         v1.0.2 -> v1.0.2
```

I verified the tag and the associated commit:

```bash
git show v1.0.2
```

_Output (abridged):_

```text
tag v1.0.2
Tagger: Ilyas Galiev <...>

commit 29d3fa4 (HEAD -> feature/lab2_new, tag: v1.0.2)
Author: Ilyas Galiev <...>

    Add test file
```

### Explanation

- A **tag** is a named reference to a specific commit.  
  It is typically used to mark release points (for example `v1.0.0`, `v1.1.0`).
- Tags are immutable markers:
  - They do not move when new commits are added (unlike branches).
  - This makes them ideal for CI/CD pipelines, changelogs, and release management.
- Pushing tags with `git push origin <tag>` or `git push origin --tags` ensures that the same release boundaries exist in the remote repository.

In this lab, tag `v1.0.2` documents the state of the repository after finishing the main Lab 2 Git exercises.

---

## Task 5 — Comparing `git switch`, `git checkout`, and `git restore`

### Goal

Demonstrate modern Git commands `git switch` and `git restore` and compare them to the legacy `git checkout`.

### Commands and outputs

#### Working with branches using `git switch` and `git checkout`

To create and move between branches, I used both `switch` and `checkout`:

```bash
# Create and move to a new branch using switch
git switch -c cmd-compare

# List branches
git branch

# Jump back to the previous branch
git switch -

# Create another branch using checkout (legacy style)
git checkout -b cmd-compare-2

# Return to feature branch
git switch feature/lab2_new
```

`git branch` showed all branches, with `*` marking the current branch.

#### Restoring files with `git restore`

I then experimented with `git restore` on a demo file:

```bash
echo "scratch" >> demo.txt
git status
```

_Output (abridged):_

```text
On branch feature/lab2_new
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
        modified:   demo.txt
```

To discard the change from the working tree and go back to the last committed version:

```bash
git restore demo.txt
git status
```

_Output:_

```text
On branch feature/lab2_new
nothing to commit, working tree clean
```

Next, I tried restoring from the index:

```bash
echo "scratch2" >> demo.txt
git add demo.txt
git status
```

_Output (abridged):_

```text
Changes to be committed:
        modified:   demo.txt
```

Then I removed the file from the index but kept the change in the working tree:

```bash
git restore --staged demo.txt
git status
```

_Output (abridged):_

```text
Changes not staged for commit:
        modified:   demo.txt
```

Optionally, I could also restore to an older commit:

```bash
git restore --source=HEAD~1 demo.txt
```

### Explanation

- `git switch`:
  - Designed specifically for **branch operations**.
  - `git switch <branch>` — move to an existing branch.  
  - `git switch -c <branch>` — create a new branch and switch to it.
  - More explicit and less error-prone than `git checkout` for branches.

- `git restore`:
  - Designed for **file content operations**.
  - `git restore <path>` — reset file in working directory to the index (discard local changes).
  - `git restore --staged <path>` — unstage changes (move them from index back to working tree).
  - `git restore --source=<rev> <path>` — restore a file from a specific commit.

- `git checkout`:
  - Older, overloaded command that can:
    - Switch branches: `git checkout <branch>`.
    - Create branches: `git checkout -b <branch>`.
    - Restore files: `git checkout -- <path>`.
  - Because it mixes both branch and file responsibilities, it is easier to misuse.  
    Modern Git encourages using:
    - `git switch` for branches.
    - `git restore` for file-level resets.

In this lab I used all three commands, but for future work I will prefer the clearer separation:
- `switch` for branch navigation and creation.
- `restore` for undoing file changes or unstaging.
- `checkout` only when needed for older scripts or when working with repositories that still rely on it.

---

## Summary


1. Explored Git’s internal object model (commits, trees, blobs) using `git cat-file`.
2. Practiced `git reset --soft` and `git reset --hard`, and used `git reflog` to safely recover previous states.
3. Visualized the commit history with `git log --graph --all` to understand branching and merging.
4. Marked a release point with a semantic tag `v1.0.2` and pushed it to the remote repository.
5. Compared `git switch`, `git checkout`, and `git restore`, and adopted the modern split between branch management and file restoration.