<h1>Project-3</h1>
The **Feature Branch Workflow** is a common Git practice that allows you to work on individual features or tasks in isolation, ensuring that changes are properly tested before being integrated into the main codebase. This approach is particularly useful for teams, as it minimizes the risk of conflicts and bugs. Below is a detailed step-by-step guide to using the feature branch workflow.

### Step-by-Step Guide: Feature Branch Workflow

---

### **1. Create a Feature Branch**

You begin by creating a new branch that isolates the feature or task you're working on from the main codebase. This ensures that your work doesn't affect the main branch until it's ready to be merged.

1. Create and switch to a new feature branch:
   ```bash
   git checkout -b feature/branch-name
   ```
   - Replace `branch-name` with a descriptive name for the feature (e.g., `feature/login-auth`, `feature/ui-update`).
   - This command creates a new branch and switches to it in one step.

2. Verify that you're on the new branch:
   ```bash
   git branch
   ```
   The branch you are on will have an asterisk (`*`) next to it.

---

### **2. Make Your Changes**

Work on the feature in isolation within the `feature/branch-name` branch. Once you've made some changes, you can stage and commit them.

1. Check the status of your changes:
   ```bash
   git status
   ```

2. Stage the files you’ve modified or added:
   ```bash
   git add <file1> <file2>
   ```
   Or to add all changed files:
   ```bash
   git add .
   ```

3. Commit the changes with a descriptive message:
   ```bash
   git commit -m "Implemented feature X for user authentication"
   ```

---

### **3. Continue Working on the Feature**

If you're working on a complex feature, you'll likely make multiple commits before the feature is complete. After each set of changes, repeat the process:
- **Check status**: `git status`
- **Stage changes**: `git add <file>`
- **Commit changes**: `git commit -m "Your message"`

---

### **4. Merge the Feature Branch into `main`**

Once the feature is complete, tested, and ready to be integrated into the main branch, you'll merge the feature branch into the `main` (or `master`) branch.

#### **Step 4.1: Switch to the Main Branch**
Before merging, switch to the `main` branch:
```bash
git checkout main
```

#### **Step 4.2: Pull the Latest Changes from Remote**
Ensure your `main` branch is up to date with the latest changes from the remote repository:
```bash
git pull origin main
```

#### **Step 4.3: Merge the Feature Branch**
Now, merge your feature branch into `main`:
```bash
git merge feature/branch-name
```

If there are no conflicts, the feature branch will be merged into `main`. If conflicts arise, Git will prompt you to resolve them manually.

#### **Step 4.4: Push the Changes to Remote**
After the merge, push the updated `main` branch to the remote repository:
```bash
git push origin main
```

---

### **5. Delete the Feature Branch (Optional)**

After successfully merging the feature branch, you can delete it to keep your branches clean and organized:

1. Delete the branch locally:
   ```bash
   git branch -d feature/branch-name
   ```

2. If you want to delete the branch from the remote repository as well:
   ```bash
   git push origin --delete feature/branch-name
   ```

---

### **6. Submitting a Pull Request (PR) for Team Collaboration**

If you're working in a team and using a collaborative workflow, instead of merging your feature branch directly, you should submit a **Pull Request (PR)** to have your changes reviewed before merging.

#### Steps to Submit a Pull Request:
1. **Push the feature branch to the remote repository**:
   ```bash
   git push origin feature/branch-name
   ```

2. **Create a PR on GitHub**:
   - Go to your GitHub repository.
   - Select the feature branch you pushed.
   - Click the **New Pull Request** button.
   - Provide a description and submit the PR.

3. Once the PR is approved by your team, merge it into `main`.

---

### **Summary of Key Commands**
1. **Create a feature branch**:
   ```bash
   git checkout -b feature/branch-name
   ```

2. **Stage and commit changes**:
   ```bash
   git add .
   git commit -m "Your commit message"
   ```

3. **Merge the feature branch** into `main`:
   ```bash
   git checkout main
   git pull origin main
   git merge feature/branch-name
   git push origin main
   ```

4. **Delete the feature branch** (optional):
   ```bash
   git branch -d feature/branch-name
   git push origin --delete feature/branch-name
   ```

5. **Submit a PR** (if working in a team):
   - Push the feature branch and create a Pull Request on GitHub.

By using the feature branch workflow, you can work on new features in isolation, avoid conflicts, and ensure your changes are reviewed before merging them into the main branch.
