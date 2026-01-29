# 🛠️ DEVOPS.md – DevOps Implementation & Troubleshooting

This document provides a **detailed DevOps guide** for setting up, running, deploying, and troubleshooting the application using **Docker, Docker Compose, GitHub Actions CI/CD, and a self-hosted runner on a Vagrant VM**.

---

## 1️⃣ Setup Guide

This section explains how to run the project **locally** and how it is deployed on the **server (Vagrant VM)** using CI/CD.

---

## 🔹 Prerequisites

Make sure the following tools are installed:

### Local / VM Requirements

* Docker
* Docker Compose
* Git
* Vagrant
* VirtualBox

### GitHub Requirements

* GitHub account
* Docker Hub account
* GitHub repository with Actions enabled

---

## 🔹 Running the Application Locally

### Step 1: Clone the Repository

```bash
git clone https://github.com/Shikha-1811/devops-assessment.git
cd devops-assessment
```

### Step 2: Build and Run Containers

```bash
docker-compose up -d --build
```

### Step 3: Verify Running Containers

```bash
docker ps
```

### Step 4: Access the Application

* **Frontend**:

  ```
  http://localhost:3000
  ```

* **Backend API**:

  ```
  http://localhost:8000/api/hello/
  ```

Expected response:

```json
{
  "message": "Hello World from Django Backend!"
}
```

---

## 🔹 Server Setup (Vagrant VM)

### Step 1: Start the Vagrant VM

```bash
vagrant up
vagrant ssh
```

### Step 2: Install Docker & Docker Compose (inside VM)

```bash
sudo apt update
sudo apt install -y docker.io docker-compose
```

### Step 3: Clone the Project on VM

```bash
git clone https://github.com/Shikha-1811/devops-assessment.git
cd devops-assessment
```

---

## 🔹 CI/CD Setup Using GitHub Actions

### Pipeline Overview

* Trigger: Push to `main` branch
* Build Docker images
* Push images to Docker Hub
* Deploy application on Vagrant VM using self-hosted runner

### 🔹 Why Self-Hosted Runner Is Used

In this project, a self-hosted GitHub Actions runner is used because the deployment target is a local Vagrant-based virtual machine, which is not directly accessible from GitHub-hosted runners. GitHub-hosted runners run in GitHub-managed infrastructure and cannot connect to private or local environments.

To solve this, the self-hosted runner was installed directly inside the Vagrant VM. This allows GitHub Actions to execute deployment steps such as Docker image pulling and Docker Compose commands directly on the target server. As a result, the CI/CD pipeline can automatically deploy the application whenever code is pushed to the main branch.


---

### GitHub Secrets Configuration

Configured under:
**Repository → Settings → Secrets and Variables → Actions**

| Secret Name        | Purpose                 |
| ------------------ | ----------------------- |
| DOCKERHUB_USERNAME | Docker Hub username     |
| DOCKERHUB_TOKEN    | Docker Hub access token |
| VM_HOST            | Vagrant VM IP           |
| VM_PORT            | SSH port                |
| VM_USER            | VM username             |
| VM_SSH_KEY         | SSH private key         |

---

### Self-Hosted Runner Setup (Vagrant VM)

```bash
mkdir actions-runner && cd actions-runner
curl -L -o actions-runner.tar.gz https://github.com/actions/runner/releases/download/v2.316.1/actions-runner-linux-x64-2.316.1.tar.gz
tar xzf actions-runner.tar.gz
./config.sh --url https://github.com/<your-username>/devops-assessment --token <TOKEN>
./run.sh
```

Once running, GitHub Actions will deploy directly to the VM.

---

## 2️⃣ Troubleshooting Log

This section documents **challenges faced during the assignment** and how they were resolved.

---

## ❌ Issue 1: Frontend Could Not Connect to Backend

### Problem

* Frontend UI loaded successfully
* API requests failed with "Connection Failed"

### Root Cause

* Django backend required a **trailing slash (`/`)** in API endpoints
* Frontend was calling `/api/hello` instead of `/api/hello/`

### Solution

* Updated frontend API call to include trailing slash
* Verified backend using `curl`

```bash
curl http://localhost:8000/api/hello/
```

---

## ❌ Issue 2: Docker Login Failed in GitHub Actions

### Problem

```
Error: Username and password required
```

### Root Cause

* Docker Hub password was used instead of access token
* Incorrect secret names referenced in workflow

### Solution

* Generated Docker Hub access token
* Updated GitHub secrets and workflow to use:

  * `DOCKERHUB_USERNAME`
  * `DOCKERHUB_TOKEN`

---

## ❌ Issue 3: Permission Denied to Docker Socket (Self-Hosted Runner)

### Problem

```
permission denied while trying to connect to the Docker daemon socket
```

### Root Cause

* GitHub runner user was not part of `docker` group

### Solution

```bash
sudo usermod -aG docker vagrant
newgrp docker
```

Restarted the runner and pipeline succeeded.

---

## ❌ Issue 4: Backend Image Not Found During Push

### Problem

```
No such image: backend:latest
```

### Root Cause

* Docker Compose image names did not match tag names

### Solution

* Used correct image names defined in `docker-compose.yaml`

---

## ✅ Final Outcome

* CI/CD pipeline successfully builds, pushes, and deploys containers
* Application is accessible via browser
* Issues were resolved using structured DevOps debugging practices

---

