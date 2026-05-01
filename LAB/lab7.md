# Experiment 7: CI/CD using Jenkins, GitHub and Docker Hub

**Name:** Aviral Jain 
**Subject:** Containerization and DevOps  

---

## Aim

To design and implement a complete CI/CD pipeline using Jenkins, integrating source code from GitHub, and building & pushing Docker images to Docker Hub.

---

## Objectives

- Understand CI/CD workflow using Jenkins (GUI-based tool)
- Create a structured GitHub repository with application and Jenkinsfile
- Build Docker images from source code
- Securely store Docker Hub credentials in Jenkins
- Automate build and push process using webhook triggers
- Use same host (Docker) as Jenkins agent

---

## Theory

### 1. What is Jenkins?

Jenkins is a web-based GUI automation server used to build applications, test code, and deploy software. It provides a browser-based dashboard, a plugin ecosystem (GitHub, Docker, etc.), and Pipeline as Code using Jenkinsfile.

### 2. What is CI/CD?

**Continuous Integration (CI):** Code is automatically built and tested after each commit.

**Continuous Deployment (CD):** Built artifacts (Docker images) are automatically delivered or deployed.

### 3. Workflow Overview

```
Developer → GitHub → Webhook → Jenkins → Build → Docker Hub
```

### 4. Prerequisites

- Docker and Docker Compose installed
- GitHub account
- Docker Hub account
- Basic Linux command knowledge

---

## Part A: GitHub Repository Setup

### Project Structure

```
My-App/
├── app.py
├── requirements.txt
├── Dockerfile
└── Jenkinsfile
```

### Application Code

**app.py**
```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def home():
    return "Hello from CI/CD Pipeline!"

app.run(host="0.0.0.0", port=80)
```

**requirements.txt**
```
flask
```

### Dockerfile

```dockerfile
FROM python:3.10-slim

WORKDIR /app
COPY . .

RUN pip install -r requirements.txt

EXPOSE 80
CMD ["python", "app.py"]
```

### Jenkinsfile

```groovy
pipeline {
    agent any

    environment {
        IMAGE_NAME = "jasman1201/myapp"
    }

    stages {

        stage('Clone Source') {
            steps {
                git branch: 'main', url: 'https://github.com/JasmanCodes/My-App'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub-token', variable: 'DOCKER_TOKEN')]) {
                    sh 'echo $DOCKER_TOKEN | docker login -u jasman1201 --password-stdin'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh 'docker push $IMAGE_NAME:latest'
            }
        }
    }
}
```

---

## Part B: Jenkins Setup using Docker

### docker-compose.yml

```yaml
version: '3.8'

services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    restart: always
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    user: root

volumes:
  jenkins_home:
```

### Screenshot 1 — Jenkins Container Started Successfully

![Jenkins Docker Compose Up](../Screenshots/Lab7_s/lab7.1.png)
> `docker compose up -d` was run in WSL inside the `jenkins-setup` folder. All 14 layers pulled successfully. Container `jenkins` was created and started on port 8080.

### Screenshot 2 — Jenkins Unlock Screen

![Jenkins Unlock Screen](../Screenshots/Lab7_s/lab7.2.png)

> Accessed Jenkins at `http://localhost:8080`. The initial admin password was retrieved using:
> ```bash
> docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
> ```

### Screenshot 3 — Plugin Installation

![Jenkins Plugin Installation](../Screenshots/Lab7_s/lab7.3.png)

> Clicked "Install suggested plugins". Jenkins installed all required plugins including Pipeline, Git, GitHub Branch Source, Credentials Binding, and others.

### Screenshot 4 — Docker CLI Installed Inside Jenkins

![Docker Installed in Jenkins](../Screenshots/Lab7_s/lab7.4.png)

> Entered Jenkins container using `docker exec -it jenkins bash` and ran `apt-get update && apt-get install -y docker.io` to install Docker CLI inside Jenkins so it can run `docker build` and `docker push` commands.

### Screenshot 5 — Docker ps Inside Jenkins Container

![Docker ps Inside Jenkins](../Screenshots/Lab7_s/lab7.5.png)

> Verified that Docker socket mount is working correctly. Running `docker ps` inside the Jenkins container shows the host's running containers — confirming Jenkins can control the host Docker daemon directly.

---

## Part C: Jenkins Configuration

### Screenshot 6 — Docker Hub Credentials Added

![Credentials Added](../Screenshots/Lab7_s/lab7.6.png)

> Added Docker Hub Access Token as a Secret Text credential in Jenkins with ID `dockerhub-token`. Path followed: Manage Jenkins → Credentials → System → Global credentials → Add Credentials.

### Screenshot 7 — Pipeline Job Configured

![Pipeline Job Configure](../Screenshots/Lab7_s/lab7.7.png)

> Created a new Pipeline job named `Pipeline`. Configured it with:
> - Definition: Pipeline script from SCM
> - SCM: Git
> - Repository URL: `https://github.com/JasmanCodes/My-App`
> - Branch Specifier: `*/main`
> - Script Path: `Jenkinsfile`

### Screenshot 8 — Pipeline Build Console Output

![Console Output](../Screenshots/Lab7_s/lab7.8.png)
> Pipeline successfully cloned the repository from GitHub. The console shows Jenkins reading the Jenkinsfile from `https://github.com/JasmanCodes/My-App` and executing each stage in sequence.

---

## Execution Flow (Stage-wise)

| Stage | Action | Status |
|-------|--------|--------|
| Clone Source | Pulled latest code from GitHub (branch: main) | ✅ Success |
| Build Docker Image | Built image `jasman1201/myapp:latest` using Dockerfile | ✅ Success |
| Login to Docker Hub | Authenticated using stored `dockerhub-token` credential | ✅ Success |
| Push to Docker Hub | Pushed image to Docker Hub repository | ✅ Success |

---

## Role of Same Host Agent

Jenkins runs inside Docker with the Docker socket mounted:

```
/var/run/docker.sock → /var/run/docker.sock
```

This allows Jenkins to directly control the host Docker engine — building and pushing images without needing a separate agent node.

---

## Observations

- Jenkins GUI simplifies CI/CD management — no manual build commands needed
- GitHub acts as both source code repository and pipeline definition (Jenkinsfile)
- Docker ensures consistent, reproducible builds every time
- Credentials are stored securely in Jenkins — never hardcoded in the pipeline
- Docker socket mounting allows Jenkins to act as its own Docker agent

---

## Result

Successfully implemented a complete CI/CD pipeline where:

- Source code and pipeline definition are maintained in GitHub (`JasmanCodes/My-App`)
- Jenkins automatically detects changes and triggers builds
- Docker image `jasman1201/myapp:latest` is built on the host agent
- Image is securely pushed to Docker Hub using stored credentials
