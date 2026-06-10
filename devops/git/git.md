# Git: Local Version Control Fundamentals

## 1. Introduction to Version Control & Git

### What is Version Control?
A Version Control System (VCS) tracks changes made to files over time. It allows you to revert files or projects back to a previous state, compare changes, and see who modified something that caused an issue.

### What is Git?
Git is a **Distributed Version Control System (DVCS)**. Unlike centralized systems where users get only one main copy of the code from a central server, every developer working with Git gets a full clone of the repository history on their local machine. If a server dies, any client repository can be used to restore it.

---

## 2. The Three States of Git

Git manages files in three distinct areas:
1. **Working Directory**: The actual folder on your computer where you edit files.
2. **Staging Area (Index)**: A middle ground file/memory cache where you draft the changes you want to include in your next snapshot (commit).
3. **Local Directory (Repository)**: The permanent database of your project’s history, stored inside the hidden `.git` folder.

```text
  Working Directory        Staging Area        Local Repository
┌──────────────────┐     ┌──────────────┐     ┌────────────────┐
│   Edit Files     │ ──> │   git add    │ ──> │   git commit   │
└──────────────────┘     └──────────────┘     └────────────────┘
```

---

## 3. Basic Setup & Configuration

Configure your identity before you start committing. These settings will be attached to your commits.
- **Set global username**:
  ```bash
  git config --global user.name "John Doe"
  ```
- **Set global email**:
  ```bash
  git config --global user.email "johndoe@example.com"
  ```
- **Check config settings**:
  ```bash
  git config --list
  ```

---

## 4. Local Repository Operations

### Initialize a Repository (`git init`)
Turn any folder on your machine into a Git repository.
- **Usage**:
  ```bash
  mkdir my-project
  cd my-project
  git init
  ```
  *This creates a hidden `.git` folder in your project root directory.*

### Checking Status (`git status`)
View which files have changes, which are staged, and which are untracked.
- **Usage**: `git status`

### Tracking/Staging Files (`git add`)
Before you save changes, you must explicitly stage them.
- **Stage a specific file**: `git add index.html`
- **Stage all modified/new files**: `git add .` or `git add -A`

### Comparing Changes (`git diff`)
See exactly what lines you added, changed, or deleted.
- **Compare working directory with staging area**: `git diff`
- **Compare staged files with repository**: `git diff --staged` or `git diff --cached`

### Committing Changes (`git commit`)
Save your staged snapshot permanently to the repository history.
- **Commit with inline message**:
  ```bash
  git commit -m "feat: add index html homepage"
  ```
- **Automatically stage and commit (skips `git add` for modified files)**:
  ```bash
  git commit -am "fix: correct typo in footer"
  ```

---

## 5. Undoing & Reverting Changes

### Discarding Working Directory Changes (`git restore`)
If you made changes to a file but want to discard them and revert to the last committed version:
- **Usage**: `git restore filename.txt`

### Unstaging a File (`git restore --staged`)
If you accidentally staged a file with `git add` and want to unstage it:
- **Usage**: `git restore --staged filename.txt`

### Reset vs. Revert
If you have already committed changes and want to go back:

| Command | How it works | History Effect | Safety |
|---|---|---|---|
| **`git reset`** | Moves the branch pointer backward to a previous commit. | **Deletes** commit history after that point. | **Dangerous** if commits are pushed to a shared remote. |
| **`git revert`** | Creates a **new** commit that performs the exact opposite changes of the specified commit. | **Preserves** history, keeping old commits visible. | **Safe** for shared remote branches. |

- **Reset to a previous commit ID (keeps files in working directory)**:
  ```bash
  git reset --soft <commit-id>
  ```
- **Reset to a previous commit ID (destroys all changes in staging & working directory)**:
  ```bash
  git reset --hard <commit-id>
  ```
- **Revert a specific commit**:
  ```bash
  git revert <commit-id>
  ```

### Removing Files (`git rm`)
- **Delete file from system and Git tracking**: `git rm file.txt`
- **Remove file from Git tracking but keep the local file**: `git rm --cached file.txt`

---

## 6. Branching & Merging

Branches represent isolated development environments where you can write features or fixes without affecting the stable `main`/`master` branch.

### Creating & Listing Branches
- **List local branches**: `git branch`
- **Create a new branch**: `git branch feature/login`

### Switching Branches (`git checkout` / `git switch`)
- **Switch to branch**: `git checkout feature/login` or `git switch feature/login`
- **Create and switch to new branch**: `git checkout -b feature/dashboard` or `git switch -c feature/dashboard`

### Merging Branches
Once a feature is ready, you merge it back into your main branch.
1. Switch to the destination branch:
   ```bash
   git checkout main
   ```
2. Run the merge command:
   ```bash
   git merge feature/login
   ```

#### Resolving Merge Conflicts
If Git cannot automatically merge changes (e.g., the same line was changed in both branches), it stops and prints a merge conflict.
1. Open the conflicted files.
2. Resolve conflicts by keeping desired code and deleting the Git markers (`<<<<<<<`, `=======`, `>>>>>>>`).
3. Stage the resolved files (`git add .`).
4. Complete the merge commit (`git commit -m "Merge resolved"`).

---

## 7. Inspection & Tagging

### Viewing History (`git log`)
- **Standard log**: `git log`
- **One-line summary**: `git log --oneline`
- **Visual graph of branches**: `git log --oneline --graph --all`

### Tagging Releases (`git tag`)
Tags are flags pointing to specific commits, typically used to mark releases (e.g., `v1.0.0`).
- **Create an annotated tag**:
  ```bash
  git tag -a v1.0 -m "Release version 1.0"
  ```
- **List tags**: `git tag`