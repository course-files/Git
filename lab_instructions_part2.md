# Collaborative Git Workflows: Part 2

* Go back to the feature branch: `git checkout feature/lab-number/description`
* Confirm that the remote branch is up to date with the branch in the origin by running `git status`
* If the branch is behind, then fetch the latest changes from the main branch in origin and merge them into your local branch:

```bash
git checkout feature/lab-number/description
git fetch origin
git merge origin/main
```

---

## 11. Moving Work Around (`git cherry-pick`) (Member 2)

**Member 2** is required to simulate a common mistake: committing work to the **wrong branch**.

To list all the local branches, use:

```bash
git branch
```

To list all the branches in the origin (remote branches), use:

```bash
git branch -r
```

To show which branch you are currently on, use:

```bash
git status
```

1. **The Simulated Mistake:** Commit an update in the wrong branch.
2. **Get the ID:** Find the **Commit ID (SHA value)** of the misplaced commit in the wrong feature branch.
    * Note that you need to be in the branch that contains the misplaced commit first: `git checkout feature/lab-number/description-of-wrong-branch`
    * Then view the commit IDs (SHA values) using `git log --oneline`.
3. **Switch and Copy:**

    * Confirm that the commit is not in the correct branch by switching to it and checking the log:

    ```bash
    git checkout feature/lab-number/description-of-correct-branch
    git log --oneline
    ```

    * Perform the cherry-picking to copy the commit from the wrong branch to the correct branch:

    ```bash
    git cherry-pick <Commit-ID-from-Step-2>
    ```

    *Note: Git applies those changes to your current branch and creates a **new commit ID**, while the original commit remains in the old branch.*

4. **Clean up:** Switch back to the branch that had the misplaced commit and use `git reset --hard HEAD~1` (see the next Section) to remove the duplicate commit from the wrong location.

---

## 12. Reversing Changes (`git revert` vs. `git reset`) (Member 3 and Member 4)

The team must distinguish between local "undos" and shared history corrections.

### A. Local Undo (`git reset`)

Use this **only for commits that have not been pushed** to GitHub.

* **Member 3:** Make a dummy commit locally, then "land" back on the previous state:

    ```bash
    git reset --hard HEAD~1
    ```

*Note: This moves your HEAD backward, unstaging the changes and losing them.*

### The Three Modes of `git reset`

`git reset` moves the `HEAD` pointer backward, but what happens to your changes depends on the flag you use.

| Flag | Commit removed from log? | Staging area | Working directory files |
| ----- | ----- | ----- | ----- |
| `--soft` | Yes | Changes remain staged | Files unchanged |
| `--mixed` (default) | Yes | Changes are unstaged | Files unchanged |
| `--hard` | Yes | Cleared permanently | Deleted permanently |

* The default (`--mixed`) is the most gentle option: your work survives; it simply needs to be re-staged.
* Use `--hard` only when you are certain you want to discard the changes entirely.
* If you accidentally use `--hard`, `git reflog` can often recover the commit within approximately **30 days** on your local machine.

The accidental commit should no longer appear. **This is safe only because the commit was never pushed to GitHub**. Had you pushed it, `--hard` would still remove it locally, but the commit would persist in origin and reappear the next time you execute `git pull`. In that case, you would need git revert instead (see the next Section).

### B. Shared Undo (`git revert`)

Use this for **shared, remote branches** to ensure you do not rewrite history that others have pulled.

* **Member 4:** If an error is discovered in a file already merged into `main`, do not delete it. Instead, create an "undo" commit:

    ```bash
    git revert <Commit-ID-to-undo>
    git push origin
    ```

*Note: This creates a **new commit** that applies the exact opposite changes, keeping the original history intact for audit purposes.*

---

## 13. The Safety Net (`git reflog`) (Member 3)

If you ever lose a commit due to a bad reset or rebase, use Git's "personal diary".

1. **View HEAD Movements:** Run `git reflog --relative-date`.
2. **Restore:** You can return to exactly where you were 10 minutes ago, even if the commits are no longer in the standard `git log`:

    ```bash
    git reset HEAD@{10.minutes.ago}
    ```

*Note: **Reflog is local only**; it cannot help teammates restore deleted commits on their own machines.*

---

## 14. Cleaning Local History (git rebase -i) (Member 5)

Before opening a Pull Request, Member 5 should "clean up" messy intermediate commits, such as "typo fix" or "WIP" (work in progress). This process provides a clear and concise commit history, which is preferred in professional environments, although it does reduce traceability by discarding individual commit details.

**Start Interactive Rebase:** Run the following command to view and edit the last three commits:

```bash
git rebase -i HEAD~3
```

*Note: In Git, HEAD is the pointer that identifies your current location in the repository's history.*

**Modify History in the Editor:** The editor will open showing a list of commits, each starting with the word pick. You can replace pick with one of the following commands to transform your history accordingly:

* **reword:** Keep the commit's changes but change the commit message itself.
* **squash:** Fold the commit into the previous one and combine the commit messages. In an actual corporate environment, your manager would want a clean history (Squash). In an academic environment, the lecturer would want to see your mistakes and how you fixed them (Merge).
* **fixup:** Fold the commit into the previous one but discard that commit's message (ideal for minor "typo fixes").
* **drop:** Delete the commit entirely.

**Finalize the Messages:** If you chose squash, an editor window will appear, allowing you to edit the combined message into one cohesive "technical lab note" that explains the reasoning behind your changes.

For example:

```text
pick 14190ac # Commit-Message-1
squash fa1d322 # Commit-Message-2
squash 80adcfe # Commit-Message-3
```

This will squash the last two commits into the first one, resulting in a single commit with a combined message. The interactive rebase editor (the "todo" list) is one of the few places in Git where commits are listed from oldest (top) to newest (bottom). We, therefore, squash "upwards." You cannot squash the very first (top) commit because there is nothing before it to squash into.

You can then save and close the editor to finalize the rebase. If there are any conflicts during the rebase, Git will pause and allow you to resolve them before continuing.

The second editor window will show the combined commit messages, which you can edit to create a clear and concise message that explains the reasoning behind your changes.

**Conclusion:**

**If successful:** You do nothing. Git will message "**Successfully rebased and updated.**"

**If conflicts occur:** Only then do you fix the files, this is done by executing:

```bash
git add .
git rebase --continue
```

---

**Warning:** Never rebase a branch that others have based their work upon, i.e., a branch with commits that have already been pushed to the origin. *Analogy: Never change the past that others have already seen.* Rewriting shared history forces everyone else on the team into a "painful reconciliation". Interactive rebase is strictly for your own unpublished/local work.

* **The Problem:** If you push a branch to a shared repository in the origin, your teammates will "pull" the branch's commit history and start building their own work on top of your specific commits.
* **The Conflict:** If you then rebase that shared branch on your machine and force-push it back to the origin, you have deleted the old history and replaced it with your new version.
* **The Result:** When your teammates try to update their work, their computer will see that the "foundation" they were building on has vanished. They are then forced to manually and laboriously figure out how to attach their work to your new, rewritten history.

---

## 15. When the Workflow May Be Overridden

The GitHub Flow process defined in this lab — feature branches, pull requests, mandatory reviews, and --no-ff merges — exists to protect the integrity of main. However, rigid adherence to a process can itself become a liability when the context changes. You need to know when to override rigid rules.

In such cases where the rules are overriden, the team must still be accountable for the decision. The override should be documented in the commit message and/or PR description, and the team should be prepared to justify the decision in a retrospective review. The goal is not to bypass accountability but to adapt to exceptional circumstances while maintaining transparency.

**Legitimate override contexts include:**

* **Production hotfix under active incident:** If a critical defect is causing data loss or system unavailability in production, the cost of waiting for a full review cycle may exceed the risk of a direct, fast-tracked push. In this case, the branch protection rule may be temporarily suspended by an administrator, with the understanding that a retrospective review is conducted immediately after the incident is resolved. This is not an exemption from accountability — it is a deferral of it.
* **Sole maintainer on a private or personal repository:** Branch protection and mandatory reviews presuppose a team. On a repository where only one person will ever commit, enforcing a self-review PR cycle adds friction without adding safety. The workflow scales down legitimately in this context.
