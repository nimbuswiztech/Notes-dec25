# git reset

Git reset is a command in the Git version control system used to reset the current state of the repository to a specified point. It's a powerful tool that allows you to undo changes, unstage files, and move the HEAD pointer to a different commit or branch.

The `git reset` command is used to undo changes in your working directory and get back to a specific commit while discarding all the commits made after that one. Before using `git reset`, consider the type of changes you plan to make — using the wrong mode can cause data loss.

The main modes are: `git reset --soft`, `git reset --mixed` (the default), and `git reset --hard`.

{% stepper %}
{% step %}
### Soft reset

This mode preserves the changes in your working directory and staging area but moves the HEAD pointer to a different commit. It effectively "undoes" commits without deleting any changes.

Usage examples:

* Move HEAD to a specific commit:

```
git reset --soft <commit ID>
```

* Move HEAD back n commits:

```
git reset --soft HEAD~n
```

* Move HEAD back one commit:

```
git reset --soft HEAD~1
```

Example workflow (fixing a commit where you forgot to add a file):

Initial commands (example):

```
git add test.py data.py
git commit -m "added test file and data file"
```

Example commit result:

```
[master (root-commit) faf864e] added test file and data file
2 files changed, 0 insertions(+), 0 deletions(-)
create mode 100644 test.py
create mode 100644 data.py
```

If you forgot to add `demofile.py`, check status:

```
git status
# On branch master
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#     demofile.py
# nothing added to commit but untracked files present (use "git add" to track)
```

Fix it by:

```
git reset --soft HEAD
git add demofile.py
git commit -m "added all the python scripts"
```

Example final commit result:

```
[master (root-commit) faf864e] added all the python scripts
3 files changed, 0 insertions(+), 0 deletions(-)
create mode 100644 test.py
create mode 100644 data.py
create mode 100644 demofile.py
```
{% endstep %}

{% step %}
### Mixed reset (default)

This is the default mode. It resets the HEAD pointer to a specified commit and resets the staging area to match that commit, but leaves the working directory unchanged. The effect is that changes remain in your files but are unstaged.

Typical scenario — you accidentally staged or committed a file you don't want to include:

* Unstage staged/committed files:

```
git reset HEAD
```

Example status after unstaging:

```
git status
# On branch master
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#     test.py
#     data.py
#     demofile.py
# nothing added to commit but untracked files present (use "git add" to track)
```

Then add only the files you want and commit:

```
git add data.py demofile.py
git commit -m "Removed the test.py from the commit"
```
{% endstep %}

{% step %}
### Hard reset

This mode resets the HEAD pointer to a specified commit and discards all changes in both the staging area and the working directory — reverting the repository to the exact state of that commit.

{% hint style="warning" %}
`git reset --hard` is potentially dangerous: it permanently discards uncommitted changes. Be sure you want to remove those changes before running it.
{% endhint %}

Example scenario:

Commit history:

```
* 7856f73 (HEAD -> main) Commit D
* a6b2c9e Commit C
* e75a4d2 Commit B
* d93cfe1 Commit A
```

To reset the branch `main` back to Commit B (`e75a4d2`):

```
git checkout main
git reset --hard e75a4d2
```

After this, the repository (working directory and staging area) matches Commit B and any changes after that are discarded.
{% endstep %}
{% endstepper %}

## Restore a single file from a past commit

To restore a specific file to its state in a particular commit without changing the rest of the working directory, use:

```
git checkout <commit> -- <file>
```

Example: restore `example.txt` to its state in commit `d5e6f7g`:

```
git checkout d5e6f7g -- example.txt
```

This affects only the specified file and does not modify commit history.

<details>

<summary>Longer example for file restore</summary>

Suppose your commit history is:

```
* c1a2b3d (HEAD -> main) Commit D
* a3b4c5e Commit C
* d5e6f7g Commit B
* f8g9h0i Commit A
```

If `example.txt` was changed in Commit D and you want the version from Commit B:

```
git checkout d5e6f7g -- example.txt
```

After running this, `example.txt` will be reverted to its state in Commit B. You can then stage and commit that change if desired:

```
git add example.txt
git commit -m "Revert example.txt to state from Commit B"
```

</details>
