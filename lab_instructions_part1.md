# Lab: Mastering Collaborative Git Workflows

This lab is designed for a team of five members to practice **GitHub Flow** while understanding alternative Git workflows. You will focus on **team governance**, **branch protection**, and **explicit merging** without fast-forwarding to maintain a clear audit trail.

---

## 1. Overview of Git Workflows

Before starting, review these three primary workflows used in the industry:

* **GitHub Flow (Our Focus):** Ideal for agile teams and web applications. It relies on a single `main` branch that must always be in a deployable state. Feature branches are used for all changes.
* **Git Flow:** A more structured approach with multiple long-lived branches (`main`, `dev`, `release`, `feature`, and `bug`). It is ideal for installable applications (like mobile apps) where multiple versions run simultaneously.
* **Trunk-Based Development (TBD):** The current industry standard (2026) for large-scale SaaS. It uses direct commits to `main` or very short-lived feature branches (~4 hours) and relies on "feature flags" and bulletproof automated testing.

---

## 2. Lab Setup (Member 1)

**Member 1** will act as the "team lead" for this lab.

1. **Clone the Repository:** `git clone <repository-url> <destination-folder>`.

2. **Confirm that your Team Members are Collaborators:** Go to: **Settings > Collaborators**. You should see all five members listed with "Write" access or "Admin" access.

3. **Configure Branch Protection:**
    * Go to **Settings > Branches > Add branch ruleset**.
    * Create a rule named `pr-and-1-approval-required-to-merge`.
    * Set the enforcement status to **Active**.
    * Include the default branch as the target.
    * **Enable:** "Require a pull request before merging" and set "Required approvals" to **1**.
    * This ensures that no team member can bypass the review process. At least 1 other team member must agree with the changes before they can be merged into `main`.
    * Got to **Settings > Rulesets** and confirm that the new ruleset is active and correctly configured.

---

## 3. Create a Project (Member 1)

1. Go to the **Projects** tab in the repository and click on **New Project**.
2. Select the **Iterative development** template and name it "**2026 Business Intelligence Labs**".
3. Ensure that your team members are added to the project as collaborators with **Admin** rights. This can be done in the project **Settings** > **Manage access**.
4. Create iterations and specify the start and end dates for each iteration. This is available under the project **Settings** > **Iterations**.

---

## 4. Create Milestones (Member 1)

1. Go to the **Issues** tab and click on **Milestones**.
2. Create 3 milestones and:
   * Assign appropriate due dates for each to align with the iterations you set up in the project.
   * Add an appropriate description for each milestone to clarify the goals and expectations
     * **50% Complete Milestone**
     * **75% Complete Milestone**
     * **100% Complete Milestone**

---

## 5. Creating the Task Backlog (All Members)

1. Each member must create a "GitHub Issue" to represent a specific technical task.
2. All the issues should be assigned to the respective member of the team.
3. All the issues should be assigned the label "**enhancement**".
4. All the issues should be assigned to the "**2026 Business Intelligence Labs**" project.

* **Member 1:** Issue #1 - 50% Complete Milestone - Title: Add project README with team roles.
* **Member 2:** Issue #2 - 50% Complete Milestone - Title: Create a `data_source.md` file listing BI source types.
* **Member 3:** Issue #3 - 50% Complete Milestone - Title: Create a `warehouse_schema.md` describing a Star Schema.
* **Member 4:** Issue #4 - 75% Complete Milestone - Title: Create a `data_pipeline.md` explaining ETL vs. ELT.
* **Member 5:** Issue #5 - 100% Complete Milestone - Title: Add a `governance.md` file regarding PII and data auditability.

---

## 6. Assigning Issues to Iterations

1. Go to the **Projects** tab and open the "**2026 Business Intelligence Labs**" project. Navigate to the **Status Board** view where you can see all the issues in the backlog.
2. For each issue, click on the issue card and assign it to the appropriate iteration based on its milestone. Example:
   * Issues #1, #2, and #3 are planned for Iteration 1, which corresponds to the **50% Complete Milestone**.
   * Issue #4 is planned for Iteration 2, which corresponds to the **75% Complete Milestone**.
   * Issue #5 is planned for Iteration 3, which corresponds to the **100% Complete Milestone**.

---

## 7. The GitHub Flow Cycle (Members 2-5)

Each member must follow these steps for their respective issue. **Member 1** will follow these steps last.

### Step A: Branching

On your local machine, create and switch to a new branch for your task.

*Note: The branch name should follow the format `feature/issue-number-description` to maintain clarity and consistency across the team. For example, if you are working on Issue #2, your branch name could be `feature/issue-2-data-source`. This naming convention helps everyone on the team quickly understand the purpose of the branch and its connection to the corresponding issue.*

```bash
git checkout -b feature/issue-number-description
```

*Note: A branch is an independent line of development used for team governance*.

### Step B: Working and Committing

Create your assigned file and confirm that there is a modification ready to be committed.

```bash
git status
```

Once ready, stage and commit it:

```bash
git add .
git commit
```

**Important:** Your commit message should include a subject line (< 72 characters) and a body explaining the academic value/reasoning of the change.

### Step C: Pushing and Pull Request

Push your branch to GitHub:

```bash
git push origin feature/issue-number-description
```

1. Go to GitHub and open a **Pull Request (PR)** from your branch to `main`. There should be a green button prompting you to "Compare & pull request" after pushing.
2. Link the PR to the corresponding issue by including `#issue-number` in the PR description. This creates a connection between the changes and the issue it addresses.
3. Assign a teammate to perform a **Code Review**. If this was your research/project, then your research supervisor would be the reviewer.

### Step D: The Code Review

* **The Reviewer:** Check the **Files changed** for clarity and accuracy based on the source material. Approve the PR if it meets requirements. If not, request changes and provide specific feedback in the comments.
* **The Author:** Address any feedback provided during the review.

### Step E: Merging without Fast-Forward (`--no-ff`)

Once approved, the merge must be performed. To ensure the process is visible for audit purposes, we will avoid "Fast-Forward" merging.

In your terminal:

```bash
git checkout main
git pull origin main
git merge --no-ff feature/issue-number-description
git push origin main
```

**Why `--no-ff`?** This option always creates a "merge commit," documenting the integration as a deliberate event in history. This is ideal in academic contexts so that research supervisors can see the branch evolution and your collaborative process. It also enables the lecturer to see the history of changes in a more structured way, which is beneficial for grading and feedback.

---

## 8. Handling Merge Conflicts (Member 5)

By the time **Member 5** merges, the `main` branch will have moved forward significantly. If a conflict occurs:

1. Git will flag the files that have "natural" collaboration conflicts.
2. Open the file, choose the correct version (or combine them), and save your changes.
3. Stage the resolved file: `git add <filename>`.
4. Complete the merge with `git commit`.

---

## 9. Summary of Merging Techniques

While we used **Git Merge (`--no-ff`)**, be aware of these alternatives:

* **Git Rebase:** Rewrites history by moving the base of your branch to the tip of `main`. It creates a clean, linear history but reduces traceability.
* **Squash Commits:** Combines all commits from a feature branch into one single commit on `main`. This is useful for cleaning up "messy" intermediate commits but discards individual commit details.

---

## 10. Auditing Repository History (`git log`)

**All Members** should practice viewing the "technical lab notes" of the project to ensure the process is transparent.

* Go back to the main branch: `git checkout main`
* Then run `git log --oneline --graph` to view the repository history. You should see a series of "knots" representing the deliberate merge commits made by each team member.

* **View a Graph of Merges:** To see how branches have evolved and joined, run:

    ```bash
    git log --merges
    ```

    *Note: This provides a quick overview of the branch evolution and is ideal for supervisors to see your collaborative process.*

* **Inspect Recent Changes:** To see the full details of the most recent commit, including the reasoning in the commit body, use:

    ```bash
    git show HEAD
    ```

    *Note: The **HEAD** pointer identifies your current location in the repository's history.*
