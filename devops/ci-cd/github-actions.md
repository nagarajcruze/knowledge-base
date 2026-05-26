# GitHub Actions Notes

GitHub Actions is an API-driven automation platform built directly into GitHub. It allows you to build, test, and deploy code directly from repository events.

---

## 1. Core GitHub Actions Concepts

- **Workflow**: An automated procedure added to your repository (defined in a YAML file under `.github/workflows/`). A workflow contains one or more jobs.
- **Event**: A specific activity that triggers a workflow (e.g., a code push, a pull request creation, a cron schedule, or a manual trigger).
- **Job**: A set of steps that execute on the same runner. By default, jobs run in parallel, but you can configure dependencies using the `needs` keyword.
- **Step**: An individual task that runs commands. Steps can run shell scripts/commands (`run`) or call reusable Actions (`uses`).
- **Action**: A standalone, reusable application that performs a complex, common task (e.g., checking out code, setting up Node.js, or logging into AWS).
- **Runner**: The server that executes the jobs. GitHub provides hosted runners (like `ubuntu-latest`, `windows-latest`), or you can connect your own self-hosted runners.
- **Secrets**: Encrypted variables stored in repository settings to manage API keys, passwords, or deployment tokens securely.

---

## 2. GitHub Actions Workflow Example

Below is a production-like GitHub Actions YAML workflow (`.github/workflows/ci-cd.yml`) that triggers on pushes, caches Node.js packages to speed up builds, runs tests, and deploys securely using secrets:

```yaml
name: CI/CD Pipeline

# 1. Define events that trigger this workflow
on:
  push:
    branches: [ main, dev ]
  pull_request:
    branches: [ main ]
  # Allow manual execution from the GitHub Actions tab
  workflow_dispatch:

# 2. Define jobs to run
jobs:
  # Job 1: Build and Test
  build-and-test:
    name: Build & Test Application
    runs-on: ubuntu-latest

    steps:
      # Step A: Checkout code from repository
      - name: Checkout Code
        uses: actions/checkout@v4

      # Step B: Setup Node.js environment
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm' # Automatically cache npm packages

      # Step C: Install dependencies
      - name: Install Dependencies
        run: npm ci

      # Step D: Run code linter
      - name: Run Linter
        run: npm run lint

      # Step E: Run unit tests
      - name: Run Tests
        run: npm test

      # Step F: Compile application code
      - name: Build Application
        run: npm run build
```
