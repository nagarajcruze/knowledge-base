# SonarQube: Code Quality & Security Analysis

## 1. Introduction to SonarQube

SonarQube is an open-source, self-managed platform that enables developers and DevOps engineers to perform automatic code reviews. It analyzes source code to detect bugs, code smells, vulnerabilities, and security hotspots across 30+ programming languages.

### Core Concepts

- **Bugs**: Coding errors that will break code execution or cause incorrect behavior at runtime.
- **Code Smells**: Maintainability issues that do not break the program but make it hard to read, modify, or scale (e.g., duplicated code, overly complex methods).
- **Vulnerabilities**: Direct security risks that can be exploited by attackers (e.g., SQL injection, insecure cryptography).
- **Security Hotspots**: Code segments that require human review because they use security-sensitive APIs or configurations (e.g., hardcoded credentials, open CORS policies).
- **Quality Profiles**: A set of rules applied during analysis. Different languages use different profiles.
- **Quality Gates**: A set of boolean conditions that a project must meet before it can be released to production (e.g., "New coverage must be >= 80%", "0 New Critical Bugs").

### Architecture Overview

SonarQube operates in two main parts:
1. **SonarQube Server**: The central service that hosts the dashboard UI, runs the analyzer engine, tracks database metrics, and manages Quality Gate policies.
2. **SonarQube Scanner**: A client-side CLI tool or build-plugin (Jenkins, Maven, Gradle, GitHub Actions) that runs locally on the code repository, performs the static analysis, and uploads the results to the Server.

---

## 2. Local Installation using Docker

To run SonarQube locally, we will deploy it alongside a PostgreSQL database using Docker.

### Step 1: Create a Dedicated Docker Network
Both SonarQube and PostgreSQL need to communicate with each other. Running them on a shared network allows container name resolution.
```bash
docker network create qtree
```

### Step 2: Start PostgreSQL 15 Container
SonarQube requires a database to store configuration, users, rules, and historical scan results.
```bash
docker run --name postgres -d \
  -e POSTGRES_USER=user \
  -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=postgresdb \
  -p 5432:5432 \
  --network qtree \
  postgres:15
```

### Step 3: Start the SonarQube Container
Pass the database connection details as environment variables.
```bash
docker run -d --name sonarqube \
  -p 9000:9000 \
  -e sonar.jdbc.url=jdbc:postgresql://postgres/postgresdb \
  -e sonar.jdbc.username=user \
  -e sonar.jdbc.password=password \
  --network qtree \
  sonarqube:latest
```
> [!NOTE]
> SonarQube might take 1-2 minutes to bootstrap completely. You can monitor its startup logs using `docker logs -f sonarqube`.

### Step 4: Access the UI Dashboard
1. Open your browser and navigate to: `http://localhost:9000`
2. **Default Credentials**: 
   - Username: `admin`
   - Password: `admin`
3. Upon first login, SonarQube will prompt you to change the administrator password.

---

## 3. Running a Manual Scan (CLI)

Before automating scans in CI/CD pipelines, you can run a manual scan locally.

1. **Install Sonar Scanner**: Download the SonarQube Scanner CLI for your OS and add its `bin` directory to your system's `PATH`.
2. **Generate a Token**: In the SonarQube UI, go to **My Account > Security** and generate a new User Token.
3. **Configure Project Settings**: Create a file named `sonar-project.properties` in your project's root directory:
   ```properties
   sonar.projectKey=my-project-key
   sonar.projectName=My Sample Project
   sonar.projectVersion=1.0
   
   # Path to source directory (relative to this file)
   sonar.sources=.
   
   # Exclude test files from code metrics
   sonar.exclusions=**/tests/**,**/node_modules/**
   ```
4. **Execute Scan**: Run the scanner command pointing to your local server:
   ```bash
   sonar-scanner \
     -Dsonar.host.url=http://localhost:9000 \
     -Dsonar.token=your_generated_security_token
   ```

---

## 4. CI/CD Integration: Jenkins Pipeline

Integrating code analysis in a Jenkins pipeline ensures that every commit is vetted before going to production.

### Jenkins Prerequisites
1. Install the **SonarQube Scanner** plugin in Jenkins (**Manage Jenkins > Plugins**).
2. Add your SonarQube server token in Jenkins credentials.
3. Configure the SonarQube Server link in **Manage Jenkins > System** under "SonarQube installations". Name it `sonarqube-server`.
4. Configure the Scanner CLI tool in **Manage Jenkins > Tools** under "SonarQube Scanner installations". Name it `sonar-scanner`.

### Complete Jenkinsfile Example
This pipeline fetches a Flask application, executes code analysis, and halts the build if it fails the Quality Gate.

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'sonarqube-demo', url: 'https://github.com/nagarajcruze/flask_server.git'
            }
        }
        
        stage('SonarQube Code Analysis') {
            steps {
                dir("${WORKSPACE}") {
                    script {
                        // Retrieve path to local scanner installation
                        def scannerHome = tool name: 'sonar-scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                        
                        // Inject configuration configured in Jenkins system settings
                        withSonarQubeEnv('sonarqube-server') {
                            sh """${scannerHome}/bin/sonar-scanner \
                                -Dsonar.projectVersion=1.0-SNAPSHOT \
                                -Dsonar.projectName=sample-app \
                                -Dsonar.projectKey=sample-app \
                                -Dsonar.sources=.
                            """
                        }
                    }
                }
            }
        }

        stage('Quality Gate Status Check') {
            steps {
                script {
                    // Pause the pipeline and wait for SonarQube's webhook response
                    timeout(time: 10, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to Quality Gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
    }
}
```

### Key Parameters Explained:
- `sonar.projectKey`: Unique identifier for the project in the SonarQube system.
- `sonar.projectName`: The display name of the project.
- `sonar.sources`: Directory paths containing the source code files to analyze.
- `waitForQualityGate()`: Standard step from the SonarQube Jenkins plugin. It waits for the SonarQube server task to complete and returns the Quality Gate status (`OK`, `WARN`, or `ERROR`). Note: For this to work, you must configure a **Webhook** in SonarQube pointing to `http://<jenkins-url>/sonarqube-webhook/`.