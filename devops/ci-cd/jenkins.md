# Jenkins Notes

Jenkins is a popular open-source automation server used to implement CI/CD (Continuous Integration and Continuous Delivery/Deployment) pipelines.

---

## 1. Core Jenkins Concepts

- **Continuous Integration (CI)**: The practice of automatically building and testing code changes frequently to detect errors early.
- **Continuous Delivery/Deployment (CD)**: Automatically preparing builds (Delivery) or pushing builds directly to production (Deployment) after passing tests.
- **Declarative vs. Scripted Pipelines**:
  - **Declarative Pipeline (Recommended)**: Uses a structured, pre-defined syntax with a `pipeline` block. It is easier to write, read, and maintain.
  - **Scripted Pipeline**: Uses a more imperative Groovy scripting syntax inside a `node` block. It offers more flexibility but is harder to manage.
- **Agent**: Defines where the pipeline will execute (e.g., on a specific virtual machine, a docker container, or any available executor).
- **Stage**: A block containing a group of steps (e.g., "Build", "Test", "Deploy"). It helps visualize the pipeline's progress in the Jenkins UI.
- **Steps**: The actual commands or tasks executed within a stage (e.g., running shell scripts, pulling git repos, sending emails).

---

## 2. Declarative Pipeline Example

Below is a production-like Declarative Pipeline that sets environment variables, binds credentials securely, runs test stages, and handles post-execution notifications:

```groovy
pipeline {
    // Run on any available Jenkins agent
    agent any

    // Define environment variables used throughout the script
    environment {
        APP_NAME      = 'my-nodejs-app'
        REGISTRY_URL  = 'docker.io/myusername'
        DB_CREDENTIALS = credentials('database-login-secret') // Bind secure credentials
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code from Git...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing npm dependencies...'
                sh 'npm ci'
            }
        }

        stage('Code Analysis & Lint') {
            steps {
                echo 'Running linter...'
                sh 'npm run lint'
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image ${REGISTRY_URL}/${APP_NAME}:latest..."
                sh "docker build -t ${REGISTRY_URL}/${APP_NAME}:latest ."
            }
        }
    }

    // Define actions to perform after the stages finish
    post {
        always {
            echo 'Pipeline execution finished. Cleaning up workspace...'
            cleanWs()
        }
        success {
            echo 'Pipeline succeeded!'
            // E.g., send slack notifications or success email
        }
        failure {
            echo 'Pipeline failed! Please check console logs.'
        }
    }
}
```
