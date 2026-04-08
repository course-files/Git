# Part 2: Advanced Git Commands for Team Collaboration

* Go back to the feature branch: `git checkout feature/issue-number-description`
* Confirm that the remote branch is up to date with the branch in the origin by running `git status`

---

## 11. Moving Work Around (`git cherry-pick`)

**Member 2** will simulate a common mistake: committing work to the **wrong branch**.

1. **The Mistake:** Imagine you accidentally committed an update directly in the `main` branch instead of through a `feature` branch.
2. **Get the ID:** Find the **Commit ID (SHA value)** of the misplaced commit in the `main` branch.
    * You need to be in the branch that contains the misplaced commit first: `git checkout main`
    * Then view the commit IDs (SHA values) using `git log --oneline`.
3. **Switch and Copy:**

    * Switch back to the correct feature branch first then perform the cherry-pick:

    ```bash
    git checkout feature/issue-number-description
    git cherry-pick <Commit-ID-from-Step-2>
    ```

    *Note: Git applies those changes to your current branch and creates a **new commit ID**, while the original commit remains in the old branch.*

4. **Clean up:** Switch back to the branch that had the misplaced commit and use `git reset` (see Section 9) to remove the duplicate commit from the wrong location.

---

## 9. Reversing Changes (`git revert` vs. `git reset`)

The team must distinguish between local "undos" and shared history corrections.

### A. Local Undo (`git reset`)
Use this **only for commits that have not been pushed** to GitHub.
*   **Member 3:** Make a dummy commit locally, then "land" back on the previous state:
    ```bash
    git reset HEAD~1
    ```
    *Note: This moves your HEAD backward, unstaging the changes without losing them.*

### B. Shared Undo (`git revert`)
Use this for **shared, remote branches** to ensure you don't rewrite history that others have pulled.
*   **Member 4:** If an error is discovered in a file already merged into `main`, do not delete it. Instead, create an "undo" commit:
    ```bash
    git revert <Commit-ID-to-undo>
    ```
    *Note: This creates a **new commit** that applies the exact opposite changes, keeping the original history intact for audit purposes.*

## 10. The Safety Net (`git reflog`)
If you ever lose a commit due to a bad reset or rebase, use Git's "personal diary".

1.  **View HEAD Movements:** Run `git reflog --relative-date`.
2.  **Restore:** You can return to exactly where you were 10 minutes ago, even if the commits are no longer in the standard `git log`:
    ```bash
    git reset HEAD@{10.minutes.ago}
    ```
    *Note: **Reflog is local only**; it cannot help teammates restore deleted commits on their own machines.*

## 11. Cleaning Local History (`git rebase -i`)
Before opening a Pull Request, **Member 5** should "clean up" messy intermediate commits (like "typo fix" or "WIP").

1.  **Start Interactive Rebase:**
    ```bash
    git rebase -i HEAD~3
    ```
2.  **Modify History:** In the editor that pops up, change `pick` to `squash` for minor commits to fold them into a main feature commit.
3.  **Warning:** **Never rebase a branch that others are working on**, as rewriting shared history causes painful reconciliation for the team.

---

**Final Submission Check:** Ensure your `main` branch follows **Three-Part Versioning** (e.g., v1.0.0). Use a **PATCH version** (e.g., v1.0.1) for bug fixes and a **MINOR version** (e.g., v1.1.0) for new backward-compatible feature implementations.