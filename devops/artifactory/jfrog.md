# JFrog Artifactory: Binary Repository Management

## 1. Introduction to JFrog Artifactory

In modern CI/CD pipelines, source code is compiled into binaries (such as `.jar`, `.war`, `.zip`, `.tar.gz`, Docker images, NPM packages, or Python wheels). Storing these built binaries directly inside source control (like Git) is bad practice because Git is not optimized for large, immutable binary files.

**JFrog Artifactory** is a universal binary repository manager. It acts as a central hub for hosting, managing, and distributing build artifacts throughout the software development lifecycle.

### Core Concepts

- **Local Repositories**: Private, physical repositories hosted on the Artifactory server. They are used to upload and share internal team builds.
- **Remote Repositories**: Act as a local caching proxy for public repositories (like Maven Central, npmjs, or Docker Hub). This speeds up builds and ensures availability even if the public registry goes down.
- **Virtual Repositories**: A logical grouping that combines multiple Local and Remote repositories under a single URL. Developers only need to configure one URL to resolve dependencies and upload builds.

### Port Mappings
- **`8081` (REST API)**: Used for internal communications, build tools (like Maven/Gradle/npm), and automated APIs.
- **`8082` (UI Console / Router)**: The web UI dashboard where users log in, manage configurations, and inspect repository files.

---

## 2. Local Installation using Docker & PostgreSQL

We will install JFrog Artifactory alongside a PostgreSQL database using Docker.

### Step 1: Prepare the Host Filesystem and Permissions
Artifactory runs inside the Docker container under user ID `1030`. To prevent permission errors when saving files to the host machine, set the owner of the data directory to `1030:1030` first.

```bash
# Define home environment variable
export JFROG_HOME=/home/username/jfrog

# Create config directories
mkdir -p $JFROG_HOME/artifactory/var/etc/
cd $JFROG_HOME/artifactory/var/etc/

# Create system.yaml configuration file
touch ./system.yaml

# Recursively change directory ownership to Artifactory user ID (1030)
chown -R 1030:1030 $JFROG_HOME/artifactory/var
```

### Step 2: Configure PostgreSQL Database Connection
Edit the created `system.yaml` file (`nano system.yaml`) and insert the database configuration block to connect Artifactory to your external database:

```yaml
shared:
    database:
        driver: org.postgresql.Driver
        type: postgresql
        url: jdbc:postgresql://postgres:5432/postgresdb
        username: user
        password: password
```

### Step 3: Run the Containers
Ensure both containers run on the same shared Docker network (`qtree`) so they can resolve each other by name.

1. **Start PostgreSQL Database** (if not already running):
   ```bash
   docker run --name postgres -d \
     -e POSTGRES_USER=user \
     -e POSTGRES_PASSWORD=password \
     -e POSTGRES_DB=postgresdb \
     -p 5432:5432 \
     --network qtree \
     postgres:15
   ```
2. **Start JFrog Artifactory**:
   ```bash
   docker run -d --name artifactory \
     -v $JFROG_HOME/artifactory/var/:/var/opt/jfrog/artifactory \
     -p 8081:8081 \
     -p 8082:8082 \
     --network qtree \
     releases-docker.jfrog.io/jfrog/artifactory-oss:7.90.7
   ```

### Step 4: Login to the Dashboard
1. Open your browser and navigate to: `http://localhost:8082`
2. **Default Credentials**:
   - Username: `admin`
   - Password: `password`
3. Complete the initial setup wizard and change your administrator password.

---

## 3. Jenkins CI/CD Integration

We can automate uploading compilation builds to Artifactory inside a Jenkins pipeline.

### Prerequisites in JFrog UI
Before running the pipeline, create a **Generic Local Repository** in Artifactory:
1. Navigate to **Administration > Repositories > Local**.
2. Click **New Local Repository** and select **Generic** as the Package Type.
3. Set the **Repository Key** to `test-repo`.

### Prerequisites in Jenkins
1. Install the **JFrog** (or Artifactory) plugin in Jenkins (**Manage Jenkins > Plugins**).
2. Configure your Artifactory server details in **Manage Jenkins > System > JFrog Integration**. Name the server connection identifier `jfrog`.

### Complete Jenkinsfile Example
This script creates a dummy file and uploads it to the `test-repo` repository in JFrog Artifactory.

```groovy
pipeline {
    agent any
    stages {
        stage('Compile & Upload to JFrog') {
            steps {
                script {
                    // Create a dummy shell script build artifact
                    sh 'echo "echo hello" > test.sh'
                    
                    // Connect to the configured JFrog server instance
                    def server = Artifactory.server 'jfrog'
                    
                    // Define the upload specification (JSON)
                    def uploadSpec = """{
                        "files" : [{
                            "pattern": "test.sh",
                            "target" : "test-repo/"
                        }]
                    }"""
                    
                    // Execute the upload
                    server.upload(uploadSpec)
                }
            }
        }
    }
}
```

### Upload Spec JSON Keys:
- **`pattern`**: The local file path pattern to scan for upload (e.g. `*.war` or `target/app.jar`).
- **`target`**: The destination path inside the Artifactory repository (must point to `<repository-key>/<optional-subfolders>/`).