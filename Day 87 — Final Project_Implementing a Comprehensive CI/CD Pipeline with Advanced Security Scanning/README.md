# Day 87 — Final Project: Implementing a Comprehensive CI/CD Pipeline with Advanced Security Scanning

## Project Overview

This final project brings together everything from the curriculum — Linux, Git, AWS, Docker, CI/CD, and monitoring — into a single, security-conscious CI/CD pipeline. Using **Jenkins** on an AWS EC2 instance, the pipeline automates the full software delivery lifecycle for a Node.js application: source checkout, static code analysis, dependency vulnerability scanning, image build, container image vulnerability scanning, image publishing, and deployment. The goal is not just automated deployment, but a pipeline that actively enforces code quality and security gates before anything reaches production.

Tools integrated:
- **Jenkins** — pipeline orchestration
- **SonarQube** — static code analysis and quality gates
- **OWASP Dependency-Check** — third-party dependency vulnerability scanning
- **Trivy** — container image vulnerability scanning
- **Docker / DockerHub** — image build and registry

---

## Pre-requisites

- An AWS EC2 instance — **t2.large** (needed for the combined memory footprint of Jenkins + SonarQube running on the same box).
- Docker, Jenkins, and Trivy installed on the instance.
- A DockerHub account (for image push).
- Target repository: [`LondheShubham153/node-todo-cicd`](https://github.com/LondheShubham153/node-todo-cicd.git)

### Environment Setup

```bash
# Update packages
sudo apt update -y

# --- Install Docker ---
sudo apt install -y docker.io
sudo usermod -aG docker $USER
sudo usermod -aG docker jenkins
newgrp docker

# --- Install Java (Jenkins prerequisite) ---
sudo apt install -y fontconfig openjdk-17-jre

# --- Install Jenkins ---
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update -y
sudo apt install -y jenkins
sudo systemctl enable --now jenkins

# --- Install Trivy ---
sudo apt install -y wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | \
  sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt update -y
sudo apt install -y trivy
trivy --version
```

- Opened inbound security group ports: `22` (SSH), `8080` (Jenkins), `9000` (SonarQube).

---

## Task Breakdown

### 1. Setup Environment

- Launched the `t2.large` EC2 instance in a dedicated VPC/subnet with a security group allowing SSH, Jenkins (8080), and SonarQube (9000).
- Installed Docker, Jenkins, and Trivy as shown above.
- Ran **SonarQube as a Docker container** on the same instance (simplest path for a single-node lab setup):

```bash
docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community
```

- Accessed SonarQube at `http://<EC2_PUBLIC_IP>:9000`, logged in with default credentials (`admin`/`admin`), and set a new password.

### 2. Configure Jenkins

- Retrieved the initial admin password and unlocked Jenkins:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

- Accessed Jenkins at `http://<EC2_PUBLIC_IP>:8080`, completed the setup wizard, and installed suggested plugins.
- Installed additional required plugins via **Manage Jenkins → Plugins → Available**:
  - **OWASP Dependency-Check Plugin**
  - **Docker Pipeline**
  - **SonarQube Scanner**
  - **Quality Gates** (bundled with SonarQube Scanner plugin — configured as a pipeline step, `waitForQualityGate`)
  - **Eclipse Temurin Installer** (JDK provisioning for SonarQube Scanner)

### 3. Integrations

**SonarQube ↔ Jenkins:**
1. In SonarQube: **Administration → Security → Users → Tokens** — generated a token for Jenkins authentication.
2. In Jenkins: **Manage Jenkins → System → SonarQube servers** — added a SonarQube server entry (`Name: sonar-server`, `Server URL: http://<EC2_PUBLIC_IP>:9000`, credential = the generated token).
3. In Jenkins: **Manage Jenkins → Tools → SonarQube Scanner installations** — added a scanner installation (`Name: sonar-scanner`).
4. In SonarQube: **Administration → Webhooks** — added a webhook pointing to `http://<EC2_PUBLIC_IP>:8080/sonarqube-webhook/` so SonarQube notifies Jenkins when the Quality Gate result is ready.

**OWASP Dependency-Check ↔ Jenkins:**
1. **Manage Jenkins → Tools → Dependency-Check installations** — added an automatic installation (`Name: DP-Check`).
2. Configured the pipeline to reference this tool name in the `Dependency-Check` build step.
3. Configured **Manage Jenkins → System → Dependency-Check** with an NVD API key to speed up the vulnerability database update (avoids long first-run download times).

**DockerHub ↔ Jenkins:**
1. Created a DockerHub access token (Account Settings → Security → New Access Token).
2. Added it in Jenkins as a **Username/Password** credential (`ID: docker-hub-cred`).

### 4. Create Jenkins Pipeline

- Created a new **Pipeline** job in Jenkins, linked to the GitHub project via **Pipeline script from SCM**, pointing at `https://github.com/LondheShubham153/node-todo-cicd.git`.
- Wrote the following declarative pipeline (`Jenkinsfile`):

```groovy
pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-cred')
        IMAGE_NAME = 'yourdockerhubuser/node-todo-cicd'
        IMAGE_TAG  = "${env.BUILD_NUMBER}"
    }

    stages {

        stage('Code Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/LondheShubham153/node-todo-cicd.git'
            }
        }

        stage('SonarQube Code Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=node-todo-cicd \
                        -Dsonar.projectKey=node-todo-cicd
                    '''
                }
            }
        }

        stage('SonarQube Quality Gates Check') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --format XML',
                                 odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Code Build') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                    trivy image --severity HIGH,CRITICAL \
                        --format table \
                        --output trivy-report.txt \
                        ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Image Push to DockerHub') {
            steps {
                sh '''
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                    docker push ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Code Deployment') {
            steps {
                sh '''
                    docker rm -f node-todo-app || true
                    docker run -d --name node-todo-app -p 3000:3000 ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'trivy-report.txt', allowEmptyArchive: true
            dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        }
        success {
            echo 'Pipeline completed successfully — application deployed.'
        }
        failure {
            echo 'Pipeline failed — check stage logs above for the failing gate.'
        }
    }
}
```

- Triggered the first build via **Build Now** and monitored the **Stage View** to confirm each stage executed in order.

---

## Deliverables

- ✅ A fully functional Jenkins pipeline executing all eight stages: Code Checkout → SonarQube Code Analysis → SonarQube Quality Gates Check → OWASP Dependency Check → Code Build → Trivy Image Scan → Image Push to DockerHub → Code Deployment.
- ✅ Documentation of each pipeline stage below, with emphasis on the security analysis stages.
- ✅ Evidence of the deployed application, accessible at `http://<EC2_PUBLIC_IP>:3000`.

### Stage-by-Stage Summary

| Stage | What It Does | Security/Quality Value |
|---|---|---|
| Code Checkout | Pulls latest source from the `node-todo-cicd` GitHub repo | Ensures pipeline always builds from source of truth |
| SonarQube Code Analysis | Runs static analysis (code smells, bugs, duplication, coverage) | Surfaces maintainability and correctness issues early |
| SonarQube Quality Gates Check | Blocks the pipeline if the analysis fails the configured quality thresholds | Prevents low-quality code from progressing further |
| OWASP Dependency Check | Scans `package.json`/`node_modules` dependency tree against the NVD CVE database | Flags known-vulnerable third-party libraries before build |
| Code Build | Builds the Docker image from the `Dockerfile` | Produces the deployable artifact |
| Trivy Image Scan | Scans the built image's OS packages and app dependencies for HIGH/CRITICAL CVEs | Catches vulnerabilities baked into the container layer, not just app code |
| Image Push to DockerHub | Tags and pushes the image (build number + `latest`) to the registry | Makes the vetted image available for deployment |
| Code Deployment | Runs the container from the freshly pushed image, replacing any prior instance | Completes the automated delivery to the running environment |

---

## Validation of Task Completion

- Confirmed the pipeline ran successfully end-to-end via the Jenkins **Stage View**, with every stage showing green.
- **SonarQube**: verified the project appeared in the SonarQube dashboard with analysis metrics (bugs, vulnerabilities, code smells, coverage) and that the Quality Gate status was visible and correctly gated the pipeline.
- **OWASP Dependency-Check**: verified the "Dependency-Check Results" trend graph and report populated in the Jenkins job, listing any CVEs found in dependencies.
- **Trivy**: verified `trivy-report.txt` was archived as a build artifact, listing scanned image layers and any HIGH/CRITICAL CVEs found.
- **Deployment**: confirmed the application was reachable at `http://<EC2_PUBLIC_IP>:3000` after the pipeline completed, and that `docker ps` showed the `node-todo-app` container running with the correct image tag.

---

## Document Your Progress

Notes and challenges encountered during this build:

- **Memory pressure on `t2.large`**: running Jenkins, SonarQube, and Docker builds concurrently on one instance is tight even at `t2.large`. SonarQube in particular needs a reasonable JVM heap; watched memory with `free -h` during builds and would size up (`t3.xlarge`) or split SonarQube onto its own instance for anything beyond a lab exercise.
- **SonarQube webhook timing**: the `waitForQualityGate` step will hang until SonarQube's webhook call reaches Jenkins — this only works if Jenkins's URL is reachable from the SonarQube container/host. Had to make sure the security group and the SonarQube webhook URL used the correct public/internal address.
- **OWASP Dependency-Check first-run latency**: the first scan downloads and updates the full NVD vulnerability database, which is slow and can hit NVD API rate limits without a personal API key. Registering an NVD API key and configuring it in Jenkins's Dependency-Check global settings significantly reduced update time on subsequent runs.
- **Trivy DB updates**: Trivy also maintains its own vulnerability DB cache; kept it warm on the instance so each pipeline run didn't re-download it from scratch.
- **Docker permissions for Jenkins**: the `jenkins` user needed to be added to the `docker` group (`sudo usermod -aG docker jenkins`, then restart Jenkins) before pipeline steps could run `docker build`/`docker push` without permission errors.

---

## Preparation for Future Projects

Reflections on extending and scaling this pipeline:

- **Additional stages to consider**: automated unit/integration test execution before the build stage, DAST (dynamic scanning, e.g. OWASP ZAP) against a staging deployment, infrastructure-as-code scanning (e.g. `tfsec`/`checkov` if Terraform is introduced), and Slack/email notifications on pipeline failure.
- **Scaling for teams**: move SonarQube and Jenkins onto separate, right-sized instances (or managed services); use Jenkins agents/executors on separate worker nodes rather than the controller itself; store the Jenkinsfile in the app repo (already done here) so pipeline changes go through the same PR review process as application code.
- **Secrets management**: migrate credentials from Jenkins's built-in credential store to a dedicated secrets manager (AWS Secrets Manager / HashiCorp Vault) as the setup grows beyond a single-user lab.
- **Deployment target**: replace the single `docker run` deployment step with a rolling deployment to ECS/Kubernetes for zero-downtime releases and easier rollback.

---

## Conclusion

Completing this project solidified an end-to-end understanding of CI/CD pipelines with a strong emphasis on security and code quality, not just automated deployment. Integrating SonarQube, OWASP Dependency-Check, and Trivy into a single Jenkins pipeline demonstrated how static analysis, dependency scanning, and container image scanning each catch a different class of issue — code-level, supply-chain, and container-layer — before an application reaches production. This project represents the capstone of the DevOps Transformation curriculum and a concrete, security-aware CI/CD artifact for a professional portfolio.

---

## Additional Resources

- [Trivy Installation and Implementation Guide](https://github.com/DevMadhup/Trivy_Installation_and_implementation)
- [Jenkins Pipeline Documentation](https://www.jenkins.io/doc/book/pipeline/syntax/)
- [Target application repository — node-todo-cicd](https://github.com/LondheShubham153/node-todo-cicd.git)

---
