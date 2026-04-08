# Collaborative Git Workflows: Part 2

* Go back to the feature branch: `git checkout feature/issue-number-description`
* Confirm that the remote branch is up to date with the branch in the origin by running `git status`

---

## 11. Moving Work Around (`git cherry-pick`)

**Member 2** is required to simulate a common mistake: committing work to the **wrong branch**.

1. **The Mistake:** Imagine you accidentally committed an update in the wrong feature branch.
2. **Get the ID:** Find the **Commit ID (SHA value)** of the misplaced commit in the wrong feature branch.
    * You need to be in the branch that contains the misplaced commit first: `git checkout feature/issue-number-description-of-wrong-branch`
    * Then view the commit IDs (SHA values) using `git log --oneline`.
3. **Switch and Copy:**

    * Switch back to the correct feature branch first then perform the cherry-picking:

    ```bash
    git checkout feature/issue-number-description
    git cherry-pick <Commit-ID-from-Step-2>
    ```

    *Note: Git applies those changes to your current branch and creates a **new commit ID**, while the original commit remains in the old branch.*

4. **Clean up:** Switch back to the branch that had the misplaced commit and use `git reset --hard HEAD~1` (see the next Section) to remove the duplicate commit from the wrong location.

---

## 12. Reversing Changes (`git revert` vs. `git reset`)

The team must distinguish between local "undos" and shared history corrections.

### A. Local Undo (`git reset`)

Use this **only for commits that have not been pushed** to GitHub.

* **Member 3:** Make a dummy commit locally, then "land" back on the previous state:

    ```bash
    git reset --hard HEAD~1
    ```

*Note: This moves your HEAD backward, unstaging the changes and losing them.*

The accidental commit should no longer appear. **This is safe only because the commit was never pushed to GitHub**. Had you pushed it, `--hard` would still remove it locally, but the commit would persist on the remote and reappear the next time you ran git pull. In that case, you would need git revert instead (see the next Section).

### The Three Modes of `git reset`

`git reset` moves the `HEAD` pointer backward, but what happens to your changes depends on the flag you use.

| Flag | Commit removed from log? | Staging area | Working directory files |
|-----|-----|-----|-----|
| `--soft` | Yes | Changes remain staged | Files unchanged |
| `--mixed` (default) | Yes | Changes are unstaged | Files unchanged |
| `--hard` | Yes | Cleared permanently | Deleted permanently |

* The default (`--mixed`) is the most gentle option: your work survives; it simply needs to be re-staged.
* Use `--hard` only when you are certain you want to discard the changes entirely.
* If you accidentally use `--hard`, `git reflog` can often recover the commit within approximately **30 days** on your local machine.

### B. Shared Undo (`git revert`)

Use this for **shared, remote branches** to ensure you do not rewrite history that others have pulled.

* **Member 4:** If an error is discovered in a file already merged into `main`, do not delete it. Instead, create an "undo" commit:

    ```bash
    git revert <Commit-ID-to-undo>
    ```

*Note: This creates a **new commit** that applies the exact opposite changes, keeping the original history intact for audit purposes.*

---

## 13. The Safety Net (`git reflog`)

If you ever lose a commit due to a bad reset or rebase, use Git's "personal diary".

1. **View HEAD Movements:** Run `git reflog --relative-date`.
2. **Restore:** You can return to exactly where you were 10 minutes ago, even if the commits are no longer in the standard `git log`:

    ```bash
    git reset HEAD@{10.minutes.ago}
    ```

*Note: **Reflog is local only**; it cannot help teammates restore deleted commits on their own machines.*

---

## 14. Cleaning Local History (`git rebase -i`)

Before opening a Pull Request, **Member 5** should "clean up" messy intermediate commits (like "typo fix" used to fix minor typing errors or "WIP" used to indicate work in progress).

1. **Start Interactive Rebase:**

    ```bash
    git rebase -i HEAD~3
    ```

2. **Modify History:** In the editor that pops up, change `pick` to `squash` for minor commits to fold them into a main feature commit.
3. **Warning:** **Never rebase a branch that others are working on**, as rewriting shared history causes painful reconciliation for the team.

---
