<h1>project-10</h1>

### **Project: DevOps Pipeline with Jenkins and GitHub Integration**

The goal of this project is to automate the build, test, and deployment process using **Jenkins** integrated with **GitHub**. The project will trigger Jenkins jobs on every code commit or pull request (PR), build Docker images, run tests, and deploy the application to an environment like **AWS**, **Google Cloud Platform (GCP)**, or **Kubernetes (EKS)**.

---

### **1. Project Directory Structure**

Here is a basic structure of the project, including source code, Docker files, and CI/CD configuration:

```bash
devops-project/
│
├── Jenkinsfile                       # Jenkins pipeline configuration
├── Dockerfile                        # Dockerfile for building the application image
├── src/                              # Application source code
│   └── app.py                        # Example Python app (can be any language)
├── requirements.txt                  # Example for Python dependencies
├── tests/                            # Directory for unit tests
│   └── test_app.py                   # Example test file
├── .github/                          # GitHub Actions (optional)
│   └── workflows/
│       └── ci.yml                    # GitHub Actions workflow (optional)
├── README.md                         # Project documentation
└── LICENSE                           # License file
```

---

### **2. Project Files Content**

#### **2.1. Jenkinsfile**

The **Jenkinsfile** defines the Jenkins pipeline stages: clone the repository, build the Docker image, run tests, and deploy the application.

```groovy
pipeline {
    agent any

    environment {
        REGISTRY = 'your-docker-registry'
        IMAGE = 'devops-project'
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/yourusername/devops-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${REGISTRY}/${IMAGE}:${env.BUILD_ID}")
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    docker.image("${REGISTRY}/${IMAGE}:${env.BUILD_ID}").inside {
                        sh 'pytest tests/'
                    }
                }
            }
        }

        stage('Push to Docker Registry') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${REGISTRY}/${IMAGE}:${env.BUILD_ID}").push()
                    }
                }
            }
        }

        stage('Deploy to Environment') {
            steps {
                echo 'Deploying application...'
                // Deployment commands go here (e.g., Kubernetes, AWS ECS)
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
```

---

#### **2.2. Dockerfile**

The **Dockerfile** describes how to build the Docker image for the application.

```Dockerfile
# Use Python base image
FROM python:3.9

# Set working directory
WORKDIR /app

# Copy application files
COPY . .

# Install dependencies
RUN pip install -r requirements.txt

# Expose the application port
EXPOSE 5000

# Start the application
CMD ["python", "src/app.py"]
```

---

#### **2.3. src/app.py**

A simple **Python** application (this can be replaced with any app written in Node.js, Java, etc.).

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello from DevOps Project!"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000)
```

---

#### **2.4. requirements.txt**

Dependencies for the Python application:

```txt
Flask==2.0.3
pytest==6.2.5
```

---

#### **2.5. tests/test_app.py**

An example unit test for the application:

```python
import pytest
from src.app import app

@pytest.fixture
def client():
    with app.test_client() as client:
        yield client

def test_hello(client):
    response = client.get('/')
    assert response.data == b"Hello from DevOps Project!"
```

---

### **3. Set Up GitHub Repository**

1. Create a new repository in GitHub (e.g., `devops-project`).
2. Clone the repository locally:

   ```bash
   git clone https://github.com/yourusername/devops-project.git
   cd devops-project
   ```

3. Add the necessary files and push them to GitHub:

   ```bash
   git add .
   git commit -m "Initial setup for DevOps project"
   git push origin main
   ```

---

### **4. Jenkins Setup**

#### **4.1. Install Jenkins**

1. Install **Jenkins** on your local machine or use a cloud-hosted solution.
2. Install necessary Jenkins plugins:
   - **Git Plugin** (for interacting with Git repositories)
   - **Docker Pipeline Plugin** (for building and managing Docker images)
   - **GitHub Plugin** (for linking Jenkins with GitHub)

#### **4.2. Configure Jenkins to Connect with GitHub**

1. In Jenkins, go to **Manage Jenkins** > **Configure System**.
2. Under **GitHub**, add your GitHub account or organization with credentials.
3. Set up webhooks in GitHub:
   - Go to your GitHub repository’s **Settings** > **Webhooks**.
   - Add a webhook with the Jenkins URL (e.g., `http://your-jenkins-server/github-webhook/`).
   - Select the events: push, pull request, etc.

#### **4.3. Create a Jenkins Pipeline Job**

1. Go to Jenkins and create a **New Item** of type **Pipeline**.
2. In the pipeline configuration:
   - Under **Pipeline**, select **Pipeline script from SCM**.
   - Enter your GitHub repository URL and specify the branch (e.g., `main`).
   - Jenkins will now use the **Jenkinsfile** from your repository.

---

### **5. Automate Docker Builds and Pushes**

- In the **Jenkinsfile**, the pipeline builds a Docker image with every commit and pushes it to Docker Hub.
- **Docker Hub** credentials should be stored securely in Jenkins:
   - Go to **Manage Jenkins** > **Credentials** and add Docker Hub credentials (`DOCKER_CREDENTIALS_ID`).
   - The Docker image will be tagged with the Jenkins build ID and pushed to your Docker Hub repository.

---

### **6. Deploy to Environment**

#### **6.1. Kubernetes Deployment (Optional)**

If deploying to **Kubernetes (EKS)**, the Jenkins pipeline can include steps to apply Kubernetes manifests:

```groovy
stage('Deploy to Kubernetes') {
    steps {
        script {
            sh 'kubectl apply -f k8s/deployment.yaml'
        }
    }
}
```

---

### **7. Summary of Steps**

1. **Set Up GitHub Repository**: Store your source code, Dockerfile, and Jenkins pipeline configuration.
2. **Jenkins Integration**: Use the **Jenkinsfile** to automate the CI/CD process: cloning the repo, building a Docker image, running tests, and deploying the application.
3. **Docker Automation**: The pipeline builds a Docker image for every commit and pushes it to Docker Hub or AWS ECR.
4. **Deployment**: Automate the deployment to an environment like Kubernetes, AWS ECS, or GCP.

This project demonstrates an end-to-end DevOps pipeline using Jenkins and GitHub. Let me know if you need further details or adjustments!
