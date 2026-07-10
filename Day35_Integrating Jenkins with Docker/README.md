# Day 35: Integrating Jenkins with Docker

> **DevOps Transformation** | CI/CD & Automation Series

---

## 📋 Overview

Today you bring together the two most important tools in this curriculum: **Jenkins** and **Docker**. Individually, each is powerful. Together, they create a CI/CD pipeline where every build runs in a clean, isolated container, Docker images are built and pushed automatically, and deployments happen without manual intervention.

By the end of today you will have Jenkins building Docker images from a Dockerfile in a Git repository, pushing those images to Docker Hub, running parameterized Docker commands, and automatically cleaning up old images on a schedule.

---

## 🗺️ What You'll Do Today

| Topic | What It Covers |
|---|---|
| Why Jenkins + Docker | The power of combining both tools |
| Running Jenkins in Docker | Setting up Jenkins with Docker socket access |
| Granting Docker Access | Mounting the Docker socket correctly |
| Task 1 | Dockerized Jenkins job with docker-compose |
| Task 2 | Build and push Docker image from Jenkins |
| Task 3 | Parameterized Docker commands |
| Task 4 | Schedule cleanup of old Docker images |

---

## 🏢 NexusCorp Scenario

> NexusCorp's CI pipeline currently builds code on the Jenkins server directly — which means every build pollutes the server with installed packages, running processes, and leftover files. Some builds interfere with others. When a developer's app needs Python 3.9 and another needs Python 3.11, the shared environment causes conflicts.
>
> The solution: every Jenkins build runs inside a Docker container. Builds are isolated, clean, and reproducible. And when a build passes, Jenkins pushes the resulting Docker image to the registry automatically. Today you build that system.

---

## Part 1: Why Integrate Jenkins with Docker?

### The Problem with Traditional Jenkins Builds

```
Traditional Jenkins build environment:

Jenkins Server (shared)
├── Python 3.9  ← Project A needs this
├── Python 3.11 ← Project B needs this (conflicts!)
├── Node 16     ← Project C
├── Node 20     ← Project D (conflicts!)
├── Leftover build artifacts from previous builds
├── Packages installed by one job that break another
└── "It worked yesterday" — environment drift over time

Result:
  → Flaky builds (work sometimes, fail sometimes)
  → "Works on Jenkins but not locally" — same problem as before
  → Hard to onboard new projects (careful not to break others)
  → Security risk — all builds share same system access
```

### The Docker Solution

```
Jenkins + Docker build environment:

Jenkins Server
  │
  ├── Job A triggered
  │     └── docker run python:3.9-slim → run build → container removed
  │
  ├── Job B triggered
  │     └── docker run python:3.11-slim → run build → container removed
  │
  ├── Job C triggered
  │     └── docker run node:16-alpine → run build → container removed
  │
  └── Job D triggered
        └── docker run node:20-alpine → run build → container removed

Result:
  → Every build starts fresh (clean container)
  → No environment conflicts between builds
  → Consistent environment between Jenkins and local dev
  → Easy to change runtime — just change the Docker image
  → Build artifact IS the Docker image — ready to deploy
```

### Key Benefits of Jenkins + Docker Integration

| Benefit | Description |
|---|---|
| **Isolated builds** | Each build runs in its own container — no interference |
| **Reproducible environments** | Same Docker image = same environment every time |
| **Clean builds** | Container is destroyed after each build — no leftover state |
| **Version-pinned runtimes** | `python:3.11-slim` is always exactly that — no drift |
| **Automated image building** | Jenkins builds the Docker image as part of CI |
| **Automated push** | Jenkins pushes the tested image to the registry |
| **Deployment automation** | Jenkins can pull and run the image on target servers |
| **Parallel environments** | Run the same build against multiple runtime versions simultaneously |

---

## Part 2: Setting Up Jenkins with Docker Access

### Option 1: Jenkins Running in Docker (with Docker socket)

The most common setup for learning: Jenkins runs inside a Docker container, with access to the host's Docker daemon via socket mounting.

```bash
# Step 1: Create required volumes and network
docker network create jenkins
docker volume create jenkins-data

# Step 2: Run Jenkins with Docker socket mounted
docker run -d \
    --name jenkins \
    --network jenkins \
    --restart unless-stopped \
    -p 8080:8080 \
    -p 50000:50000 \
    -v jenkins-data:/var/jenkins_home \
    -v /var/run/docker.sock:/var/run/docker.sock \
    jenkins/jenkins:lts-jdk17

# Step 3: Install Docker CLI inside the Jenkins container
docker exec -it --user root jenkins bash
apt-get update && apt-get install -y docker.io
exit

# Step 4: Add Jenkins user to docker group inside container
docker exec -it --user root jenkins bash
usermod -aG docker jenkins
exit

# Step 5: Restart the container to apply group changes
docker restart jenkins

# Step 6: Verify Docker access from inside Jenkins
docker exec -it jenkins docker ps
# Should list running containers (including Jenkins itself)
```

---

### Option 2: Build a Custom Jenkins Image with Docker Pre-installed

A more production-ready approach — bake Docker CLI into the Jenkins image:

```dockerfile
# Dockerfile.jenkins-docker
FROM jenkins/jenkins:lts-jdk17

# Switch to root to install packages
USER root

# Install Docker CLI
RUN apt-get update && apt-get install -y \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg \
        lsb-release \
    && curl -fsSL https://download.docker.com/linux/debian/gpg | \
       gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) \
       signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
       https://download.docker.com/linux/debian \
       $(lsb_release -cs) stable" | \
       tee /etc/apt/sources.list.d/docker.list > /dev/null \
    && apt-get update \
    && apt-get install -y docker-ce-cli \
    && rm -rf /var/lib/apt/lists/*

# Add Jenkins user to docker group
RUN groupadd -f docker && usermod -aG docker jenkins

# Switch back to Jenkins user
USER jenkins

# Pre-install essential plugins
RUN jenkins-plugin-cli --plugins \
    git \
    docker-workflow \
    docker-commons \
    credentials-binding \
    pipeline-stage-view \
    blueocean \
    slack
```

```bash
# Build the custom Jenkins image
docker build -f Dockerfile.jenkins-docker -t nexus-jenkins:latest .

# Run it
docker run -d \
    --name jenkins \
    --restart unless-stopped \
    -p 8080:8080 \
    -p 50000:50000 \
    -v jenkins-data:/var/jenkins_home \
    -v /var/run/docker.sock:/var/run/docker.sock \
    nexus-jenkins:latest
```

---

### Understanding Docker Socket Mounting

```
Host Machine
├── Docker Daemon (dockerd)
│     └── Listens on: /var/run/docker.sock
│
├── Jenkins Container
│     └── /var/run/docker.sock  ← mounted from host
│           │
│           └── Jenkins can now send Docker commands
│               to the HOST's Docker daemon
│
└── When Jenkins runs: docker build -t myapp .
      → Docker daemon on HOST builds the image
      → Image appears in HOST's docker images list
      → Image can be pushed from Jenkins to registry
```

**Security note:** Mounting the Docker socket gives Jenkins effectively root access to the host (anyone with Docker socket access can escalate to root). This is acceptable for personal learning and controlled CI environments, but in production use:
- A dedicated Jenkins build server (not shared with other services)
- Rootless Docker
- Docker-in-Docker (DinD) with careful configuration
- Kubernetes-based build agents (preferred in enterprise)

---

### Option 3: Jenkins Installed Directly on Linux (Recommended for Production Learning)

```bash
# On Ubuntu 22.04 EC2 instance:

# Step 1: Install Java
sudo apt update
sudo apt install -y fontconfig openjdk-17-jre

# Step 2: Install Jenkins
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
    https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/" | \
    sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update && sudo apt install -y jenkins

# Step 3: Install Docker
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker

# Step 4: Add Jenkins user to Docker group
sudo usermod -aG docker jenkins

# Step 5: Restart Jenkins
sudo systemctl restart jenkins
sudo systemctl enable jenkins

# Step 6: Verify
sudo -u jenkins docker ps
# Should work without permission errors

# Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

## Part 3: Installing Required Jenkins Plugins for Docker

```
Dashboard → Manage Jenkins → Plugins → Available plugins

Install these:
□ Docker Pipeline       → docker.build(), docker.withRegistry() in Jenkinsfile
□ Docker Commons        → Shared Docker utilities (dependency)
□ Docker plugin         → Docker-based build agents
□ Credentials Binding   → Inject Docker Hub credentials into builds
□ Pipeline Stage View   → See pipeline stages visually
□ Blue Ocean            → Modern pipeline UI

After installing → Restart Jenkins
```

---

### Add Docker Hub Credentials to Jenkins

```
Dashboard → Manage Jenkins → Credentials
→ System → Global credentials (unrestricted)
→ Add Credentials

Kind: Username with password
Scope: Global
Username: your-dockerhub-username
Password: your-dockerhub-password (or access token)
ID: dockerhub-credentials
Description: Docker Hub credentials for Jenkins CI

→ Create
```

**Best practice:** Use a Docker Hub **Access Token** instead of your password:
```
Docker Hub → Account Settings → Security → New Access Token
Name: jenkins-ci
Permissions: Read, Write, Delete
→ Generate → Copy the token
→ Use this as the password in Jenkins credentials
```

---

## Part 4: Practical Tasks

### Task 1: Dockerized Jenkins Job with Docker Compose

**Step 1: Create the project repository**

```bash
mkdir nexus-compose-demo
cd nexus-compose-demo
git init

# Create a simple Flask app
cat > app.py << 'EOF'
from flask import Flask, jsonify
import redis
import os

app = Flask(__name__)
r = redis.Redis(host=os.getenv('REDIS_HOST', 'redis'), port=6379, decode_responses=True)

@app.route('/')
def index():
    visits = r.incr('visits')
    return jsonify({'message': 'NexusCorp App', 'visits': visits})

@app.route('/health')
def health():
    return jsonify({'status': 'healthy'})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
EOF

# Create requirements.txt
cat > requirements.txt << 'EOF'
flask==3.0.0
redis==5.0.1
EOF

# Create Dockerfile
cat > Dockerfile << 'EOF'
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]
EOF

# Create docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: "3.8"
services:
  app:
    build: .
    container_name: nexus-app
    ports:
      - "5000:5000"
    environment:
      - REDIS_HOST=redis
    depends_on:
      - redis

  redis:
    image: redis:7-alpine
    container_name: nexus-redis
EOF

# Create Jenkinsfile
cat > Jenkinsfile << 'EOF'
pipeline {
    agent any

    environment {
        COMPOSE_PROJECT = "nexus-compose-${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Cloned: ${env.GIT_BRANCH} @ ${env.GIT_COMMIT?.take(8)}"
            }
        }

        stage('Start Services') {
            steps {
                sh '''
                    echo "Starting containers with docker-compose..."
                    docker-compose -p ${COMPOSE_PROJECT} up -d --build
                    echo "Waiting for services to initialize..."
                    sleep 10
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    echo "Checking application health..."
                    # Get the app container's port
                    curl -f http://localhost:5000/health || \
                        (echo "Health check failed!" && exit 1)
                    echo "Health check passed!"

                    echo "Testing main endpoint..."
                    curl -s http://localhost:5000/
                '''
            }
        }

        stage('Report') {
            steps {
                sh '''
                    echo "Running containers:"
                    docker-compose -p ${COMPOSE_PROJECT} ps
                    echo ""
                    echo "App logs:"
                    docker-compose -p ${COMPOSE_PROJECT} logs app --tail 20
                '''
            }
        }
    }

    post {
        always {
            sh '''
                echo "Cleaning up: stopping and removing containers..."
                docker-compose -p ${COMPOSE_PROJECT} down -v || true
                echo "Cleanup complete."
            '''
            cleanWs()
        }
        success {
            echo "All services started and passed health checks!"
        }
        failure {
            echo "Pipeline failed. Check console output for details."
        }
    }
}
EOF

git add .
git commit -m "Add NexusCorp compose demo with Jenkins pipeline"
```

**Step 2: Create Pipeline job in Jenkins**

```
Dashboard → New Item
Name: nexus-compose-demo
Type: Pipeline → OK

Pipeline:
  Definition: Pipeline script from SCM
  SCM: Git
  Repository URL: https://github.com/YOUR/nexus-compose-demo.git
  Credentials: github-credentials
  Branch: */main
  Script Path: Jenkinsfile

→ Save → Build Now
```

---

### Task 2: Docker Build and Push to Docker Hub

```groovy
// Jenkinsfile — Docker Build and Push
pipeline {
    agent any

    environment {
        DOCKER_REGISTRY   = 'docker.io'
        DOCKER_REPO       = 'YOUR_DOCKERHUB_USERNAME/nexus-api'
        IMAGE_TAG         = "${env.BUILD_NUMBER}"
        GIT_SHORT_SHA     = "${env.GIT_COMMIT?.take(8) ?: 'unknown'}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Building from commit: ${GIT_SHORT_SHA}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building Docker image..."
                    docker build \
                        --label "build.number=${BUILD_NUMBER}" \
                        --label "git.commit=${GIT_SHORT_SHA}" \
                        --label "git.branch=${GIT_BRANCH}" \
                        -t ${DOCKER_REPO}:${IMAGE_TAG} \
                        -t ${DOCKER_REPO}:latest \
                        .

                    echo ""
                    echo "Image built successfully:"
                    docker images ${DOCKER_REPO}
                '''
            }
        }

        stage('Test Image') {
            steps {
                sh '''
                    echo "Testing Docker image..."

                    # Run container from the newly built image
                    docker run -d \
                        --name nexus-test-${BUILD_NUMBER} \
                        -p 5001:5000 \
                        ${DOCKER_REPO}:${IMAGE_TAG}

                    # Wait for startup
                    sleep 8

                    # Run health check
                    curl -f http://localhost:5001/health || \
                        (docker rm -f nexus-test-${BUILD_NUMBER} && exit 1)

                    echo "Image test passed!"

                    # Cleanup test container
                    docker stop nexus-test-${BUILD_NUMBER}
                    docker rm nexus-test-${BUILD_NUMBER}
                '''
            }
        }

        stage('Push to Docker Hub') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "Logging in to Docker Hub..."
                        echo "${DOCKER_PASS}" | \
                            docker login -u "${DOCKER_USER}" --password-stdin

                        echo "Pushing image with build number tag..."
                        docker push ${DOCKER_REPO}:${IMAGE_TAG}

                        echo "Pushing latest tag..."
                        docker push ${DOCKER_REPO}:latest

                        echo "Logging out..."
                        docker logout

                        echo ""
                        echo "Successfully pushed:"
                        echo "  ${DOCKER_REPO}:${IMAGE_TAG}"
                        echo "  ${DOCKER_REPO}:latest"
                    '''
                }
            }
        }

        stage('Deploy to Staging') {
            when {
                branch 'main'
            }
            steps {
                sh '''
                    echo "Deploying to staging..."

                    # Stop existing staging container
                    docker stop nexus-staging 2>/dev/null || true
                    docker rm nexus-staging 2>/dev/null || true

                    # Run new version
                    docker run -d \
                        --name nexus-staging \
                        --restart unless-stopped \
                        -p 5000:5000 \
                        -e APP_ENV=staging \
                        ${DOCKER_REPO}:${IMAGE_TAG}

                    sleep 8

                    # Verify deployment
                    curl -f http://localhost:5000/health && \
                        echo "Staging deployment successful!" || \
                        echo "WARNING: Health check failed on staging"
                '''
            }
        }
    }

    post {
        always {
            sh '''
                # Clean up dangling images to save disk space
                docker image prune -f || true
            '''
            cleanWs()
        }
        success {
            echo "Build #${BUILD_NUMBER} succeeded — ${DOCKER_REPO}:${IMAGE_TAG} is live"
        }
        failure {
            // Clean up test container if it exists
            sh 'docker rm -f nexus-test-${BUILD_NUMBER} 2>/dev/null || true'
            echo "Build #${BUILD_NUMBER} failed — check console output"
        }
    }
}
```

---

### Task 3: Parameterized Docker Commands

Parameterized builds allow dynamic Docker operations — choose the image name, tag, and environment at build time.

**Create a Parameterized Jenkins Job:**

```
Dashboard → New Item
Name: nexus-parameterized-docker
Type: Pipeline → OK

✅ This project is parameterized → Add Parameter

Parameter 1: String Parameter
  Name: IMAGE_NAME
  Default: nexus-api
  Description: Docker image name to build

Parameter 2: String Parameter
  Name: TAG
  Default: latest
  Description: Docker image tag

Parameter 3: Choice Parameter
  Name: ENVIRONMENT
  Choices:
    development
    staging
    production
  Description: Target deployment environment

Parameter 4: Boolean Parameter
  Name: PUSH_TO_REGISTRY
  Default: false
  Description: Push image to Docker Hub after build

Parameter 5: String Parameter
  Name: DOCKER_HUB_USERNAME
  Default: yourusername
  Description: Docker Hub username for push

→ Pipeline: Pipeline script (paste below)
```

```groovy
pipeline {
    agent any

    stages {

        stage('Display Parameters') {
            steps {
                echo """
=== Build Parameters ===
Image Name:       ${params.IMAGE_NAME}
Tag:              ${params.TAG}
Environment:      ${params.ENVIRONMENT}
Push to Registry: ${params.PUSH_TO_REGISTRY}
Docker Hub User:  ${params.DOCKER_HUB_USERNAME}
Build Number:     ${BUILD_NUMBER}
========================
                """
            }
        }

        stage('Build Image') {
            steps {
                sh """
                    echo "Building: ${params.IMAGE_NAME}:${params.TAG}"

                    docker build \
                        --build-arg BUILD_ENV=${params.ENVIRONMENT} \
                        --label environment=${params.ENVIRONMENT} \
                        --label build.number=${BUILD_NUMBER} \
                        -t ${params.IMAGE_NAME}:${params.TAG} \
                        -t ${params.IMAGE_NAME}:${BUILD_NUMBER} \
                        -f Dockerfile .

                    echo "Build complete:"
                    docker images ${params.IMAGE_NAME}
                """
            }
        }

        stage('Run Container') {
            steps {
                sh """
                    CONTAINER_NAME="${params.IMAGE_NAME}-${params.ENVIRONMENT}-${BUILD_NUMBER}"

                    echo "Starting container: \$CONTAINER_NAME"
                    docker run -d \
                        --name "\$CONTAINER_NAME" \
                        -e APP_ENV=${params.ENVIRONMENT} \
                        -p 5002:5000 \
                        ${params.IMAGE_NAME}:${params.TAG}

                    sleep 5

                    echo "Container running:"
                    docker ps --filter name=\$CONTAINER_NAME

                    echo "Container logs:"
                    docker logs "\$CONTAINER_NAME"

                    echo "Stopping test container..."
                    docker stop "\$CONTAINER_NAME"
                    docker rm "\$CONTAINER_NAME"
                """
            }
        }

        stage('Push to Registry') {
            when {
                expression { return params.PUSH_TO_REGISTRY == true }
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        FULL_IMAGE="${params.DOCKER_HUB_USERNAME}/${params.IMAGE_NAME}:${params.TAG}"

                        echo "Tagging for registry: \$FULL_IMAGE"
                        docker tag ${params.IMAGE_NAME}:${params.TAG} "\$FULL_IMAGE"

                        echo "Pushing: \$FULL_IMAGE"
                        echo "\$DOCKER_PASS" | \
                            docker login -u "\$DOCKER_USER" --password-stdin
                        docker push "\$FULL_IMAGE"
                        docker logout
                        echo "Push complete: \$FULL_IMAGE"
                    """
                }
            }
        }
    }

    post {
        always {
            sh """
                docker image prune -f || true
            """
            cleanWs()
        }
    }
}
```

**How to run a parameterized build:**
```
Job page → "Build with Parameters"
→ Fill in:
  IMAGE_NAME: nexus-api
  TAG: v2.0.0
  ENVIRONMENT: staging
  PUSH_TO_REGISTRY: ✅ (checked)
→ Build
```

---

### Task 4: Schedule Cleanup of Old Docker Images

Create a scheduled maintenance job that runs weekly to clean up old Docker images and containers:

```groovy
// Jenkinsfile — Docker Cleanup Job
// Schedule: H 2 * * 0 (Every Sunday at 2 AM)

pipeline {
    agent any

    parameters {
        choice(
            name: 'CLEANUP_LEVEL',
            choices: ['conservative', 'moderate', 'aggressive'],
            description: 'How aggressively to clean up Docker resources'
        )
        booleanParam(
            name: 'DRY_RUN',
            defaultValue: true,
            description: 'Preview what would be deleted without actually deleting'
        )
    }

    stages {

        stage('Pre-cleanup Report') {
            steps {
                sh '''
                    echo "=== Docker Disk Usage BEFORE Cleanup ==="
                    docker system df
                    echo ""
                    echo "=== Stopped Containers ==="
                    docker ps -a --filter status=exited --format \
                        "table {{.ID}}\t{{.Names}}\t{{.Image}}\t{{.Status}}"
                    echo ""
                    echo "=== Dangling Images ==="
                    docker images --filter dangling=true --format \
                        "table {{.ID}}\t{{.Repository}}\t{{.Tag}}\t{{.Size}}"
                    echo ""
                    echo "=== Unused Volumes ==="
                    docker volume ls --filter dangling=true
                '''
            }
        }

        stage('Remove Stopped Containers') {
            steps {
                sh """
                    echo "=== Removing Stopped Containers ==="
                    COUNT=\$(docker ps -aq --filter status=exited | wc -l)
                    echo "Found \$COUNT stopped containers"

                    if [ "${params.DRY_RUN}" = "true" ]; then
                        echo "[DRY RUN] Would remove:"
                        docker ps -a --filter status=exited --format \
                            "  {{.Names}} ({{.Image}})"
                    else
                        docker container prune -f
                        echo "Removed \$COUNT stopped containers"
                    fi
                """
            }
        }

        stage('Remove Dangling Images') {
            steps {
                sh """
                    echo "=== Removing Dangling Images ==="
                    SIZE=\$(docker images --filter dangling=true -q | wc -l)
                    echo "Found \$SIZE dangling images"

                    if [ "${params.DRY_RUN}" = "true" ]; then
                        echo "[DRY RUN] Would remove dangling images:"
                        docker images --filter dangling=true
                    else
                        docker image prune -f
                        echo "Dangling images removed"
                    fi
                """
            }
        }

        stage('Remove Old Images') {
            when {
                expression {
                    return params.CLEANUP_LEVEL == 'moderate' ||
                           params.CLEANUP_LEVEL == 'aggressive'
                }
            }
            steps {
                sh """
                    echo "=== Removing Unused Images (${params.CLEANUP_LEVEL}) ==="

                    if [ "${params.DRY_RUN}" = "true" ]; then
                        echo "[DRY RUN] Would remove all unused images"
                        docker images --format \
                            "table {{.Repository}}\t{{.Tag}}\t{{.Size}}\t{{.CreatedSince}}"
                    else
                        docker image prune -a -f --filter \
                            "until=168h"    # Images older than 7 days
                        echo "Old images removed"
                    fi
                """
            }
        }

        stage('Remove Unused Volumes') {
            when {
                expression { return params.CLEANUP_LEVEL == 'aggressive' }
            }
            steps {
                sh """
                    echo "=== Removing Unused Volumes (aggressive mode) ==="

                    if [ "${params.DRY_RUN}" = "true" ]; then
                        echo "[DRY RUN] Would remove unused volumes:"
                        docker volume ls --filter dangling=true
                    else
                        docker volume prune -f
                        echo "Unused volumes removed"
                    fi
                """
            }
        }

        stage('Remove Unused Networks') {
            steps {
                sh """
                    echo "=== Removing Unused Networks ==="

                    if [ "${params.DRY_RUN}" = "true" ]; then
                        echo "[DRY RUN] Would remove unused networks:"
                        docker network ls --filter type=custom
                    else
                        docker network prune -f
                        echo "Unused networks removed"
                    fi
                """
            }
        }

        stage('Post-cleanup Report') {
            steps {
                sh '''
                    echo "=== Docker Disk Usage AFTER Cleanup ==="
                    docker system df
                    echo ""
                    echo "=== Remaining Images ==="
                    docker images --format \
                        "table {{.Repository}}\t{{.Tag}}\t{{.Size}}\t{{.CreatedSince}}"
                    echo ""
                    echo "=== Running Containers ==="
                    docker ps --format \
                        "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"
                '''
            }
        }
    }

    post {
        always {
            echo "Docker cleanup job finished"
        }
        success {
            echo "Cleanup completed successfully"
        }
    }
}
```

**Schedule the cleanup job:**
```
Job Configuration → Build Triggers
✅ Build periodically
Schedule: H 2 * * 0
          │ │ │ │ │
          │ │ │ │ └── Day of week: 0 = Sunday
          │ │ │ └──── Month: every month
          │ │ └────── Day of month: every day
          │ └──────── Hour: 2 AM
          └────────── Minute: random (H = hash-based)

Runs: Every Sunday at approximately 2 AM
```

---

## 🔧 Mini-Project: Complete NexusCorp Docker CI/CD Pipeline

**Scenario:** Build the full NexusCorp pipeline — code pushed to GitHub triggers Jenkins to test, build a Docker image, push it to Docker Hub, and deploy it to staging.

```groovy
// Jenkinsfile — NexusCorp Complete Docker CI/CD
pipeline {
    agent any

    environment {
        APP_NAME        = 'nexus-api'
        DOCKER_REPO     = "YOUR_DOCKERHUB_USERNAME/nexus-api"
        IMAGE_TAG       = "${BUILD_NUMBER}"
        GIT_SHORT_SHA   = "${env.GIT_COMMIT?.take(8) ?: 'unknown'}"
        STAGING_PORT    = "5000"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo """
=== NexusCorp CI/CD Pipeline ===
Branch: ${env.GIT_BRANCH}
Commit: ${GIT_SHORT_SHA}
Build:  #${BUILD_NUMBER}
================================
                """
            }
        }

        stage('Unit Tests') {
            agent {
                docker {
                    image 'python:3.11-slim'
                    args '--user root'
                }
            }
            steps {
                sh '''
                    pip install -q pytest
                    python3 -m pytest tests/ -v --tb=short || true
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build \
                        --build-arg BUILD_NUMBER=${BUILD_NUMBER} \
                        --build-arg GIT_SHA=${GIT_SHORT_SHA} \
                        -t ${DOCKER_REPO}:${IMAGE_TAG} \
                        -t ${DOCKER_REPO}:latest \
                        .
                """
            }
        }

        stage('Test Container') {
            steps {
                sh """
                    docker run -d \
                        --name nexus-ci-test-${BUILD_NUMBER} \
                        -p 5099:5000 \
                        ${DOCKER_REPO}:${IMAGE_TAG}

                    sleep 8

                    curl -f http://localhost:5099/health && \
                        echo "Container health check: PASSED" || \
                        (docker rm -f nexus-ci-test-${BUILD_NUMBER} && exit 1)

                    docker stop nexus-ci-test-${BUILD_NUMBER}
                    docker rm nexus-ci-test-${BUILD_NUMBER}
                """
            }
        }

        stage('Push to Registry') {
            when { branch 'main' }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_REPO}:${IMAGE_TAG}
                        docker push ${DOCKER_REPO}:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Staging') {
            when { branch 'main' }
            steps {
                sh """
                    docker stop ${APP_NAME}-staging 2>/dev/null || true
                    docker rm ${APP_NAME}-staging 2>/dev/null || true

                    docker run -d \
                        --name ${APP_NAME}-staging \
                        --restart unless-stopped \
                        -p ${STAGING_PORT}:5000 \
                        -e APP_ENV=staging \
                        ${DOCKER_REPO}:${IMAGE_TAG}

                    sleep 8

                    curl -f http://localhost:${STAGING_PORT}/health && \
                        echo "Staging deployment: LIVE" || \
                        echo "WARNING: Staging health check failed"
                """
            }
        }
    }

    post {
        always {
            sh """
                docker rm -f nexus-ci-test-${BUILD_NUMBER} 2>/dev/null || true
                docker image prune -f || true
            """
            cleanWs()
        }
        success {
            echo "Pipeline PASSED: ${DOCKER_REPO}:${IMAGE_TAG} deployed to staging"
        }
        failure {
            echo "Pipeline FAILED: Build #${BUILD_NUMBER}"
        }
    }
}
```

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Docker socket** | The Unix socket (`/var/run/docker.sock`) used to communicate with the Docker daemon |
| **Socket mounting** | Passing the host's Docker socket into a Jenkins container so Jenkins can run Docker commands |
| **Docker daemon** | The background service (`dockerd`) that manages Docker objects |
| **DinD (Docker-in-Docker)** | Running a Docker daemon inside a Docker container — more isolated but complex |
| **`docker build`** | Builds a Docker image from a Dockerfile |
| **`docker push`** | Uploads an image to a registry (Docker Hub, ECR, etc.) |
| **`docker pull`** | Downloads an image from a registry |
| **`docker compose up -d`** | Starts all services defined in `docker-compose.yml` in detached mode |
| **`docker compose down`** | Stops and removes all containers and networks defined in Compose file |
| **Parameterized build** | A Jenkins job that accepts input values before running |
| **`withCredentials()`** | Jenkinsfile step that injects stored credentials as environment variables |
| **`usernamePassword()`** | Credential type used to inject Docker Hub username and password |
| **`docker image prune`** | Removes dangling (untagged) images |
| **`docker system prune`** | Removes all unused Docker resources |
| **Dangling image** | An image with no tag — leftover from a build where the tag was reassigned |
| **`docker container prune`** | Removes all stopped containers |
| **`docker volume prune`** | Removes all volumes not used by any container |
| **`docker network prune`** | Removes all unused custom networks |
| **Docker Access Token** | A Docker Hub-generated token used instead of a password for authentication |
| **`--password-stdin`** | Docker login flag that reads password from stdin (safer than `-p`) |
| **Build artifact** | In Docker CI/CD, the artifact is the Docker image itself |
| **Staging deployment** | Running the new Docker image on a staging server for pre-production testing |
| **`docker run --restart unless-stopped`** | Restart policy: always restart unless explicitly stopped |
| **`docker ps --filter`** | Filter container list by status, name, or label |
| **`cleanWs()`** | Jenkins Pipeline step that removes the workspace directory after the build |
| **Dry run** | Running a job in preview mode — shows what would happen without making changes |
| **`cron` schedule** | Time-based trigger format used in Jenkins: `H 2 * * 0` = every Sunday at 2 AM |

---

## 📚 Resources

- [Jenkins + Docker Official Integration](https://www.jenkins.io/doc/book/pipeline/docker/) — using Docker with Jenkins pipelines
- [Docker Pipeline Plugin Documentation](https://plugins.jenkins.io/docker-workflow/) — plugin reference
- [Live DevOps Project — Jenkins CI/CD with GitHub Integration (YouTube)](https://www.youtube.com/results?search_query=Jenkins+CICD+GitHub+Docker+Integration+DevOps) — video walkthrough
- [Docker Hub Access Tokens](https://docs.docker.com/docker-hub/access-tokens/) — creating secure tokens for CI
- [Jenkins Credentials Documentation](https://www.jenkins.io/doc/book/using/using-credentials/) — managing secrets in Jenkins
- [Docker system prune Documentation](https://docs.docker.com/engine/reference/commandline/system_prune/) — cleanup reference

---

## 🔭 Day 36 Preview: Jenkins Pipeline as Code — Jenkinsfile Advanced

Today you ran Docker commands inside Jenkins jobs. Tomorrow you write production-grade Jenkinsfiles with advanced features.

You will learn:
- Parallel stages — run tests on multiple environments simultaneously
- Input and approval steps — human gates before production deployment
- Shared libraries — reuse pipeline code across multiple repositories
- `when` conditions — run stages only on specific branches, tags, or change sets
- Error handling with `try-catch` in pipelines
- A complete production pipeline: `git push` → test → build → push → staging → approval → production

This is the pipeline that every serious DevOps team runs — and you'll have built it from scratch.

---

> 💬 *"Jenkins without Docker is a CI server. Jenkins with Docker is a deployment factory. The difference is everything."*
>
> — DevOps Learning By Yukta
