# GitHub: Collaboration & Remote Repositories

## 1. Git vs. GitHub

- **Git**: A local command-line tool designed to manage file history and perform version control on your computer. It does not require internet access.
- **GitHub**: A cloud-hosted platform that hosts Git repositories online. It provides a web interface, user authentication, and collaboration features like Pull Requests, Issue tracking, and automated CI/CD pipelines.

---

## 2. Connecting to Remote Repositories

To upload or download code from GitHub, you connect your local Git repo to a remote repository on GitHub using one of two protocols:

- **HTTPS**: Simple authentication using GitHub Personal Access Tokens (PATs) as passwords.
  *URL Format*: `https://github.com/username/repository.git`
- **SSH**: Secure, passwordless authentication using cryptographic keys.
  *URL Format*: `git@github.com:username/repository.git`

### Managing Remote Links
- **List remote links connected to local repo**:
  ```bash
  git remote -v
  ```
- **Add a remote repository link (usually named `origin`)**:
  ```bash
  git remote add origin git@github.com:username/repository.git
  ```
- **Remove a remote link**:
  ```bash
  git remote remove origin
  ```

---

## 3. Sharing & Synchronizing Code

### Cloning a Repository (`git clone`)
Download an existing online project and all of its version history to your computer.
- **Usage**:
  ```bash
  git clone https://github.com/username/repository.git
  ```

### Downloading Remote Changes (`git fetch` vs. `git pull`)
When collaborating, other developers will push code online. You need to pull their changes to stay updated.

- **`git fetch`**: Downloads the latest commits, branches, and tags from the remote repository, but **does not modify** your working directory files. It updates your remote tracking branches (e.g., `origin/main`).
  *Usage*: `git fetch`
- **`git pull`**: Downloads remote changes and **immediately merges** them into your current local branch. It is essentially a combination of `git fetch` followed by `git merge`.
  *Usage*: `git pull`

### Uploading Local Changes (`git push`)
Upload your local commits to the remote GitHub repository.
- **Pushing for the first time (sets tracking branch)**:
  ```bash
  git push -u origin main
  ```
  *The `-u` (upstream) flag links your local branch to the remote branch, allowing you to just run `git push` next time.*
- **Subsequent pushes**:
  ```bash
  git push
  ```
- **Force Push (`git push -f`)**:
  > [!CAUTION]
  > Force pushing overrides the remote history with your local history. Never use `git push -f` on shared main branches because it can delete other developers' commits.

---

## 4. Intermediate Workflows: PRs & Forking

- **Forking**: Creating a personal copy of someone else's repository on your GitHub account. This is common when contributing to open-source projects.
- **Pull Request (PR)**: A request sent from your branch to the original project owner, asking them to review and merge your changes.

---

## 5. Managing Multiple GitHub Accounts

If you use one computer for both personal projects (e.g., using your personal GitHub account) and work projects (e.g., using your company's GitHub account), you must configure Git to use the correct credentials and SSH keys for each project.

### Step 1: Global vs. Local Configuration
Git uses global configs by default, but you can override them on a per-repository basis.

- **Set Global Identity (e.g., for Work/Primary account)**:
  ```bash
  git config --global user.name "John Work"
  ```
  ```bash
  git config --global user.email "jwork@company.com"
  ```
- **Set Local Identity Override (e.g., in a Personal project directory)**:
  1. Navigate to your personal repository: `cd ~/projects/personal-app`
  2. Set local configurations (these will overwrite the global configs inside this folder only):
     ```bash
     git config --local user.name "John Personal"
     ```
     ```bash
     git config --local user.email "jpersonal@gmail.com"
     ```

### Step 2: Configure SSH Keys for Two Accounts
If you push using SSH, your computer needs to present the correct SSH key for each account.

1. **Generate separate SSH keys**:
   - Work key: `ssh-keygen -t ed25519 -C "jwork@company.com" -f ~/.ssh/id_ed25519_work`
   - Personal key: `ssh-keygen -t ed25519 -C "jpersonal@gmail.com" -f ~/.ssh/id_ed25519_personal`
2. **Add both public keys to their respective GitHub Accounts** under Settings > SSH and GPG keys.
3. **Configure SSH Host Aliases**: Create or edit your SSH config file (`~/.ssh/config`):
   ```text
   # Work Account (Default)
   Host github.com-work
       HostName github.com
       User git
       IdentityFile ~/.ssh/id_ed25519_work
       IdentitiesOnly yes

   # Personal Account
   Host github.com-personal
       HostName github.com
       User git
       IdentityFile ~/.ssh/id_ed25519_personal
       IdentitiesOnly yes
   ```

### Step 3: Cloning and Pushing using SSH Aliases
When you interact with remote repositories, modify the SSH domain URL to use your configured host alias instead of standard `github.com`.

- **Clone Work Repo**:
  ```bash
  git clone git@github.com-work:company-org/work-project.git
  ```
- **Clone Personal Repo**:
  ```bash
  git clone git@github.com-personal:your-username/personal-project.git
  ```
- **Updating Remote URL on an Existing Repository**:
  If you already cloned a repo normally and want to update its connection to use your personal SSH alias:
  ```bash
  git remote set-url origin git@github.com-personal:your-username/personal-project.git
  ```