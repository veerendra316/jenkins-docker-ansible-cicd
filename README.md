# 🚀 Jenkins Docker Ansible CI/CD on AWS

A complete **automated CI/CD pipeline on AWS** that builds, tests, containerizes, stores, and deploys a web application to multiple EC2 instances.

The pipeline is triggered automatically whenever new code is pushed to GitHub.

---

## 📌 Project Overview

This project demonstrates an end-to-end DevOps workflow using:

* GitHub
* Jenkins
* Docker
* Amazon ECR
* Ansible
* Amazon EC2
* Nginx
* GitHub Webhooks

Whenever code is pushed to the GitHub repository, Jenkins automatically starts the pipeline.

The pipeline:

1. Checks out the latest source code
2. Builds a Docker image
3. Runs a Docker container for testing
4. Tests the website using `curl`
5. Pushes the Docker image to Amazon ECR
6. Uses Ansible to deploy the image
7. Deploys the application to multiple EC2 instances

---

## 🏗️ Architecture

```text
                    Developer
                       │
                       │ git push
                       ▼
                  ┌──────────┐
                  │  GitHub  │
                  └────┬─────┘
                       │
                       │ Webhook
                       ▼
                ┌──────────────┐
                │    Jenkins   │
                │   CI/CD      │
                └──────┬───────┘
                       │
             ┌─────────┴─────────┐
             │                   │
             ▼                   ▼
       Docker Build         Docker Test
             │                   │
             └─────────┬─────────┘
                       │
                       ▼
                ┌──────────────┐
                │ Amazon ECR   │
                │ Docker Image │
                └──────┬───────┘
                       │
                       ▼
                 ┌───────────┐
                 │  Ansible  │
                 └─────┬─────┘
                       │
                ┌──────┴──────┐
                │             │
                ▼             ▼
          ┌──────────┐   ┌──────────┐
          │ EC2 #2   │   │ EC2 #3   │
          │ Docker   │   │ Docker   │
          │ Nginx    │   │ Nginx    │
          └────┬─────┘   └────┬─────┘
               │              │
               └──────┬───────┘
                      ▼
                  Web Browser
```

---

## 🛠️ Technologies Used

| Technology      | Purpose                      |
| --------------- | ---------------------------- |
| Git             | Version control              |
| GitHub          | Source code repository       |
| GitHub Webhooks | Automatic Jenkins triggering |
| Jenkins         | CI/CD automation             |
| Docker          | Application containerization |
| Amazon ECR      | Docker image registry        |
| Ansible         | Automated deployment         |
| Amazon EC2      | Application servers          |
| Nginx           | Web server                   |
| Linux/Ubuntu    | Server operating system      |
| AWS IAM         | AWS permissions and access   |

---

## 🔄 CI/CD Workflow

### 1. Developer Push

The developer modifies the website:

```text
index.html
```

Then pushes the changes:

```bash
git add .
git commit -m "Update website"
git push origin main
```

---

### 2. GitHub Webhook

GitHub sends a webhook request to Jenkins:

```text
GitHub
   ↓
Jenkins /github-webhook/
```

Jenkins automatically starts the pipeline.

---

### 3. Docker Build

Jenkins builds the Docker image:

```bash
docker build -t jenkins-cicd-web:${BUILD_NUMBER} .
```

The image uses:

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

---

### 4. Docker Test

Jenkins starts a temporary container:

```bash
docker run -d \
  --name jenkins-cicd-test \
  -p 8081:80 \
  jenkins-cicd-web:${BUILD_NUMBER}
```

The website is tested:

```bash
curl -f http://localhost:8081
```

If the test fails, the pipeline stops.

---

### 5. Push to Amazon ECR

After successful testing, Jenkins authenticates with Amazon ECR:

```bash
aws ecr get-login-password --region us-east-1 \
| docker login \
--username AWS \
--password-stdin \
592011499817.dkr.ecr.us-east-1.amazonaws.com
```

The image is tagged using the Jenkins build number:

```text
jenkins-cicd-web:BUILD_NUMBER
```

and pushed to ECR.

Example:

```text
592011499817.dkr.ecr.us-east-1.amazonaws.com/jenkins-cicd-web:10
```

The `latest` tag is also updated.

---

### 6. Ansible Deployment

Jenkins runs:

```bash
ansible-playbook -i inventory deploy.yml
```

Ansible connects to the application EC2 instances through SSH.

The deployment performs tasks such as:

* Installing Docker
* Logging into Amazon ECR
* Pulling the latest Docker image
* Removing the previous container
* Starting the new container
* Exposing the application on port 80

---

## 🐳 Docker Container

The application runs using Nginx:

```text
Docker Container
       │
       │ Port 80
       ▼
     Nginx
       │
       ▼
  index.html
```

The container is exposed on:

```text
EC2_PUBLIC_IP:80
```

---

## 📁 Project Structure

```text
jenkins-docker-ansible-cicd/
│
├── Jenkinsfile
├── Dockerfile
├── index.html
├── README.md
│
└── ansible/
    ├── inventory
    └── deploy.yml
```

---

## ⚙️ Jenkins Pipeline Stages

The Jenkins pipeline contains the following stages:

```text
Checkout
   ↓
Docker Build
   ↓
Docker Test
   ↓
Push to ECR
   ↓
Deploy with Ansible
```

### Checkout

Downloads the latest source code from GitHub.

### Docker Build

Creates a new Docker image using the application source.

### Docker Test

Runs the container and verifies that the website responds successfully.

### Push to ECR

Pushes the tested Docker image to Amazon ECR.

### Deploy with Ansible

Deploys the ECR image to multiple EC2 application servers.

---

## 🔐 AWS Configuration

The Jenkins EC2 instance requires permissions to interact with Amazon ECR.

The application EC2 instances require permission to pull images from ECR.

Example IAM policy:

```text
AmazonEC2ContainerRegistryReadOnly
```

The Jenkins server requires appropriate ECR permissions for pushing images.

---

## 🔑 SSH Configuration

Ansible uses an SSH private key to connect to the application servers.

The Jenkins server stores the key securely:

```text
/var/lib/jenkins/.ssh/Ansible2.pem
```

The Jenkins user must have permission to read the key.

---

## 🌐 Deployment Result

The application is deployed to multiple EC2 servers:

```text
EC2 #2
   │
   └── Docker + Nginx
           │
           └── Website

EC2 #3
   │
   └── Docker + Nginx
           │
           └── Website
```

Both servers serve the same application image from Amazon ECR.

---

## 🧪 Testing

Local Docker test:

```bash
docker build -t jenkins-cicd-web .
```

Run:

```bash
docker run -d \
  --name jenkins-cicd-web \
  -p 80:80 \
  jenkins-cicd-web
```

Test:

```bash
curl http://localhost
```

Expected result:

```text
Jenkins CI/CD Pipeline
```

---

## 🔁 Automatic Deployment

After the initial configuration, deployment is automatic.

```text
Modify Code
    ↓
git push
    ↓
GitHub
    ↓
Webhook
    ↓
Jenkins
    ↓
Docker Build
    ↓
Docker Test
    ↓
ECR
    ↓
Ansible
    ↓
EC2 #2 + EC2 #3
```

No manual **Build Now** action is required.

---

## 📚 Key DevOps Concepts Demonstrated

This project demonstrates practical experience with:

* CI/CD pipelines
* Infrastructure automation
* Configuration management
* Docker containerization
* Docker image versioning
* Amazon ECR
* AWS EC2
* AWS IAM
* SSH authentication
* Ansible playbooks
* GitHub webhooks
* Jenkins pipelines
* Automated testing
* Multi-server deployment
* Nginx
* Linux administration

---

## 🎯 Project Outcome

The project successfully implements an automated deployment workflow where a developer can push a code change to GitHub and the application is automatically:

```text
Built → Tested → Containerized → Stored → Deployed
```

to multiple AWS EC2 application servers.

---

## 👨‍💻 Author

**Veerendra Sai Perabathula**

Cloud / DevOps Engineer Fresher

Skills demonstrated:

```text
AWS | Linux | Docker | Jenkins | Ansible | Terraform |
Git | GitHub | Kubernetes | Amazon ECR | EC2
```
