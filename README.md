# DevSecOps: Netflix Clone CI/CD with Monitoring & Email

![Project Banner](https://imgur.com/vORuBnK.png)

<p align="center">
  <a href="https://github.com/yourusername/DevOps-Project-09/stargazers"><img src="https://img.shields.io/github/stars/yourusername/DevOps-Project-09?style=social" alt="Stars"></a>
  <a href="https://github.com/yourusername/DevOps-Project-09/network/members"><img src="https://img.shields.io/github/forks/yourusername/DevOps-Project-09?style=social" alt="Forks"></a>
  <a href="https://github.com/yourusername/DevOps-Project-09"><img src="https://img.shields.io/github/license/yourusername/DevOps-Project-09" alt="License"></a>
</p>

---

## 📖 Project Overview

This project demonstrates a complete DevSecOps pipeline for deploying a Netflix clone application. It covers every stage from code integration, automated testing, security scanning, containerization, and deployment to Kubernetes, with robust monitoring and notification systems. The goal is to provide a real-world, production-grade CI/CD workflow with security and observability best practices.

---

## 🎯 Goals

- **Automate** the build, test, and deployment process using Jenkins.
- **Secure** the application pipeline with static and dynamic analysis (SonarQube, Trivy, OWASP).
- **Containerize** the app using Docker and orchestrate with Kubernetes.
- **Monitor** infrastructure and application health with Prometheus and Grafana.
- **Notify** stakeholders of build and deployment status via email.

---

## 📝 Prerequisites

- AWS account (or any cloud provider for VM provisioning)
- Basic knowledge of Linux command line
- Familiarity with Docker, Jenkins, and Kubernetes concepts
- [Optional] DockerHub account for image storage
- [Optional] TMDB API key for the Netflix clone app

---

## 🗺️ Project Architecture

Below is a high-level architecture of the CI/CD pipeline and deployment:

```
[ Developer ]
     |
     v
[ GitHub Repo ]
     |
     v
[ Jenkins CI/CD ] --(Security Scans)--> [ SonarQube | Trivy | OWASP ]
     |
     v
[ Docker Build & Push ]
     |
     v
[ Kubernetes Cluster ]
     |
     v
[ Monitoring: Prometheus & Grafana ]
     |
     v
[ Email Notifications ]
```

*You can replace this with a real diagram if you wish!*

---

## 🛠️ Tech Stack

- **Jenkins**: CI/CD automation
- **Docker**: Containerization
- **Kubernetes**: Orchestration
- **SonarQube**: Code quality and static analysis
- **Trivy**: Container vulnerability scanning
- **OWASP Dependency-Check**: Dependency vulnerability scanning
- **Prometheus**: Monitoring and metrics
- **Grafana**: Visualization and dashboards
- **Email Extension**: Build notifications

---

## 🚀 Quickstart

### 1. Clone the Repository
```bash
git clone https://github.com/yourusername/DevOps-Project-09.git
cd DevOps-Project-09
```

### 2. Review the Project Steps
This README contains a detailed, step-by-step guide to set up the entire pipeline from scratch. Each step includes context, commands, and troubleshooting tips.

### 3. Run the Pipeline
Once setup is complete, trigger builds in Jenkins and monitor deployments and metrics in Grafana.

---

## 📋 Project Steps (Detailed)

### Step 1: Launch an Ubuntu Server
- **Purpose:** Host Jenkins, Docker, and other tools.
- **Instructions:**
  - Launch an Ubuntu 22.04 VM (T2 Large recommended) on AWS EC2 or your preferred cloud.
  - Open inbound ports: 22 (SSH), 80/443 (HTTP/HTTPS), 8080 (Jenkins), 9000 (SonarQube), 3000 (Grafana), 9090 (Prometheus).
- **Tip:** Use a strong key pair and restrict access to your IP for security.

### Step 2: Install Jenkins, Docker, and Trivy; Set Up SonarQube
- **Purpose:** Set up the CI/CD engine, container runtime, security scanner, and code quality tool.
- **Instructions:**
  - Update and install dependencies:
    ```bash
    sudo apt update -y
    sudo apt install -y openjdk-17-jdk docker.io wget curl
    sudo systemctl start docker && sudo systemctl enable docker
    sudo usermod -aG docker $USER
    newgrp docker
    ```
  - Install Jenkins:
    ```bash
    wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
    sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
    sudo apt update -y
    sudo apt install -y jenkins
    sudo systemctl start jenkins && sudo systemctl enable jenkins
    ```
  - Install Trivy:
    ```bash
    sudo apt-get install wget apt-transport-https gnupg lsb-release -y
    wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
    echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
    sudo apt-get update
    sudo apt-get install trivy -y
    ```
  - Run SonarQube in Docker:
    ```bash
    docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
    ```
- **Troubleshooting:**
  - If Jenkins or SonarQube do not start, check logs with `sudo journalctl -u jenkins` or `docker logs sonar`.

### Step 3: Obtain a TMDB API Key
- **Purpose:** Required for the Netflix clone app to fetch movie data.
- **Instructions:**
  - Register at [TMDB](https://www.themoviedb.org/).
  - Navigate to your profile > Settings > API > Create API Key.
  - Save the key for later use in Docker build.

### Step 4: Install Prometheus and Grafana
- **Purpose:** Enable monitoring and visualization of your infrastructure and app.
- **Instructions:**
  - Download and extract Prometheus and Node Exporter from [prometheus.io](https://prometheus.io/download/).
  - Move binaries to `/usr/local/bin` and set up systemd services (see [Prometheus docs](https://prometheus.io/docs/introduction/overview/)).
  - Download and install Grafana from [grafana.com](https://grafana.com/grafana/download).
  - Start both services and access Grafana at `http://<EC2-Public-IP>:3000` (default: admin/admin).
- **Tip:** Add Prometheus as a data source in Grafana and import dashboards for Jenkins/Kubernetes.

### Step 5: Integrate Jenkins with Prometheus
- **Purpose:** Expose Jenkins metrics for monitoring.
- **Instructions:**
  - Install the Prometheus plugin in Jenkins (Manage Jenkins > Plugins).
  - Edit Prometheus config (`prometheus.yml`) to add Jenkins as a target:
    ```yaml
    - job_name: 'jenkins'
      metrics_path: '/prometheus'
      static_configs:
        - targets: ['<jenkins-ip>:8080']
    ```
  - Reload Prometheus config and verify Jenkins appears in the targets list.

### Step 6: Configure Email Notifications in Jenkins
- **Purpose:** Receive build and deployment status via email.
- **Instructions:**
  - Install the Email Extension plugin in Jenkins.
  - In "Manage Jenkins" > "Configure System", set up SMTP (e.g., Gmail with app password).
  - Add your email and credentials.
- **Tip:** Test email configuration before using in pipelines.

### Step 7: Install and Configure Required Jenkins Plugins
- **Purpose:** Enable all necessary integrations for the pipeline.
- **Instructions:**
  - Install: Eclipse Temurin Installer, SonarQube Scanner, NodeJS, OWASP Dependency-Check, Docker, Docker Pipeline, Docker API.
  - Configure JDK, NodeJS, SonarQube, and Docker in "Global Tool Configuration".

### Step 8: Create a Jenkins Pipeline Project
- **Purpose:** Automate the build, test, and deployment process.
- **Instructions:**
  - Create a new Pipeline job in Jenkins.
  - In the pipeline configuration, select 'Pipeline script from SCM' and point to this repository.
  - Jenkins will automatically use the provided `Jenkinsfile` in the root of the repository for the pipeline definition.
  - Replace any placeholders (e.g., repo URLs, credentials) in your Jenkins and repository settings as needed.
- **Troubleshooting:**
  - If a stage fails, check the Jenkins console output for error details.

### Step 9: Add Security Scans (OWASP, Trivy)
- **Purpose:** Ensure code and dependencies are free from known vulnerabilities.
- **Instructions:**
  - Add these stages to your pipeline:
    ```groovy
    stage('OWASP FS SCAN') {
      steps {
        dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
      }
    }
    stage('TRIVY FS SCAN') { steps { sh 'trivy fs . > trivyfs.txt' } }
    ```
- **Tip:** Review scan reports in Jenkins and address any critical issues before proceeding.

### Step 10: Build and Push Docker Image
- **Purpose:** Package the app for deployment and store it in a registry.
- **Instructions:**
  - Add Docker build and push stages to your pipeline:
    ```groovy
    stage('Docker Build & Push') {
      steps {
        script {
          withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
            sh 'docker build --build-arg TMDB_V3_API_KEY=your_api_key -t netflix .'
            sh 'docker tag netflix yourdockerhub/netflix:latest'
            sh 'docker push yourdockerhub/netflix:latest'
          }
        }
      }
    }
    stage('TRIVY') { steps { sh 'trivy image yourdockerhub/netflix:latest > trivyimage.txt' } }
    ```
- **Troubleshooting:**
  - If Docker build fails, check the Dockerfile and build context.
  - Ensure DockerHub credentials are correct and have push access.

### Step 11: Deploy with Docker and Kubernetes
- **Purpose:** Run the app in a scalable, production-like environment.
- **Instructions:**
  - For Docker:
    ```groovy
    stage('Deploy to container') { steps { sh 'docker run -d --name netflix -p 8081:80 yourdockerhub/netflix:latest' } }
    ```
  - For Kubernetes:
    - Set up master and worker nodes (see [Kubernetes docs](https://kubernetes.io/docs/setup/)).
    - Install `kubectl` and configure access.
    - Use manifests to deploy:
      ```groovy
      stage('Deploy to kubernetes') {
        steps {
          script {
            dir('Kubernetes') {
              withKubeConfig(credentialsId: 'k8s') {
                sh 'kubectl apply -f deployment.yml'
                sh 'kubectl apply -f service.yml'
              }
            }
          }
        }
      }
      ```
- **Tip:** Use `kubectl get all` and `kubectl get svc` to verify deployment status.

### Step 12: Access the Netflix Clone
- **Purpose:** Verify the deployment is successful.
- **Instructions:**
  - Open your browser and go to `http://<public-ip>:<service-port>`.
  - You should see the Netflix clone app running.

### Step 13: Clean Up Resources
- **Purpose:** Avoid unnecessary cloud charges.
- **Instructions:**
  - Terminate AWS EC2 instances and delete any cloud resources when finished.

---

## 📸 Screenshots

![Jenkins Pipeline](https://imgur.com/vORuBnK.png)
![Monitoring Dashboard](https://imgur.com/2j7GSPs.png)

---

## 👤 Author

**Dushyanth Bandaru**  
[LinkedIn](https://www.linkedin.com/in/dushyanthbandaru/)

---

## ⭐ Support

If you found this project helpful, please consider starring ⭐ the repository and sharing it with your network!

---

## 📢 Stay Connected

For updates, connect with me on [LinkedIn](https://www.linkedin.com/in/dushyanthbandaru/).

---
