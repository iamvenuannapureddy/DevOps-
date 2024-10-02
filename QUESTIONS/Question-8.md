<h1>Can you describe a scenario where you've automated repetitive tasks using shell scripting, Python, or another scripting language?</h1>

Certainly! Automating repetitive tasks is a key aspect of DevOps and system administration. Here’s a scenario where I automated a repetitive process using **Python** and **shell scripting** to streamline the deployment of Docker containers across multiple environments.

### **Scenario: Automating Docker Container Deployment Across Multiple Environments**

#### **Problem:**
We had a microservices-based application deployed in **development**, **staging**, and **production** environments. Each time a developer pushed new code to the repository, the following repetitive tasks needed to be done manually:
1. **Build** a Docker image.
2. **Tag** the image with the appropriate version number and environment tag.
3. **Push** the image to a **Docker registry** (Docker Hub).
4. **Deploy** the updated container to the respective environment (development, staging, or production).

This process was time-consuming and error-prone, especially when it came to managing multiple microservices and environments.

#### **Solution:**
I automated this process using a combination of **Python** and **Bash scripting**. The automation covered the following steps:
- Pull the latest code from the Git repository.
- Build and tag Docker images based on the environment (development, staging, or production).
- Push the images to Docker Hub.
- Trigger the deployment to the respective environment using Kubernetes (`kubectl`) or Docker Swarm (`docker stack deploy`).

#### **Steps for Automation:**

### **1. Pulling the Latest Code and Setting Up Variables**
Using a shell script, I created environment-specific variables for building and deploying Docker containers. Each environment (dev, staging, prod) had its own branch in Git.

```bash
#!/bin/bash

# Variables
ENV=$1
GIT_BRANCH=""
DOCKER_IMAGE_NAME="my-app"
DOCKER_REPO="mydockerhub/my-app"
VERSION_TAG=$(date +%Y%m%d%H%M%S)

# Set Git branch based on the environment
if [ "$ENV" == "dev" ]; then
    GIT_BRANCH="develop"
elif [ "$ENV" == "staging" ]; then
    GIT_BRANCH="staging"
elif [ "$ENV" == "prod" ]; then
    GIT_BRANCH="master"
else
    echo "Unknown environment: $ENV"
    exit 1
fi

# Pull the latest code
git checkout $GIT_BRANCH
git pull origin $GIT_BRANCH
```

### **2. Automating Docker Build and Push:**
Using **Python** to handle Docker operations and managing tags for versioning. The script builds the Docker image with an appropriate version and environment tag, and then pushes it to Docker Hub.

```python
import os
import subprocess
import sys

# Get environment and version tag from shell script
env = sys.argv[1]
version_tag = sys.argv[2]
docker_repo = "mydockerhub/my-app"

# Function to run shell commands
def run_command(command):
    result = subprocess.run(command, shell=True, check=True)
    return result

# Build Docker image
image_tag = f"{docker_repo}:{env}-{version_tag}"
print(f"Building Docker image with tag: {image_tag}")
run_command(f"docker build -t {image_tag} .")

# Push Docker image to Docker Hub
print(f"Pushing image to Docker Hub: {image_tag}")
run_command(f"docker push {image_tag}")
```

### **3. Deploying to Kubernetes or Docker Swarm:**
Depending on the environment, the script deploys the Docker image using **Kubernetes** or **Docker Swarm**. Here’s an example for deploying to Kubernetes.

```bash
#!/bin/bash

NAMESPACE=$1
DOCKER_IMAGE_TAG=$2

# Deploy to Kubernetes
echo "Deploying to Kubernetes in namespace: $NAMESPACE"
kubectl set image deployment/my-app-deployment my-app-container=$DOCKER_IMAGE_TAG -n $NAMESPACE

# Check the status of the rollout
kubectl rollout status deployment/my-app-deployment -n $NAMESPACE
```

### **4. Integrating into CI/CD Pipeline:**
To complete the automation, I integrated this scripting process into **Jenkins**. When a developer pushes code to a specific branch, Jenkins triggers this automated script to:
1. Pull the latest code.
2. Build and push the Docker image.
3. Deploy the container to the appropriate environment.

The **Jenkins pipeline** was configured as follows:

```groovy
pipeline {
    agent any
    parameters {
        string(name: 'ENV', defaultValue: 'dev', description: 'Target environment (dev, staging, prod)')
    }
    stages {
        stage('Pull Latest Code') {
            steps {
                script {
                    sh './deploy.sh ${params.ENV}'
                }
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                script {
                    sh 'python3 build_and_push.py ${params.ENV} ${BUILD_NUMBER}'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh './deploy_k8s.sh ${params.ENV} ${BUILD_NUMBER}'
                }
            }
        }
    }
}
```

### **Outcome:**
This automated solution saved a significant amount of time and reduced manual errors. Developers no longer had to manually build, push, and deploy Docker images. The process was now fully automated, ensuring:
- **Consistency** in Docker image builds and versioning.
- **Faster deployments** by reducing manual steps.
- **Fewer errors**, especially in tagging and deploying to the correct environment.
- Seamless integration with our **CI/CD pipeline** (Jenkins), enabling continuous delivery.

#### **Tools Used:**
- **Shell Scripting** for basic environment setup and pulling Git branches.
- **Python** for handling Docker operations.
- **Docker Hub** for storing images.
- **Kubernetes** for orchestrating containerized applications.
- **Jenkins** for CI/CD pipeline automation.

This approach can be easily extended to handle other repetitive tasks like environment setup, application configuration, and resource scaling.
