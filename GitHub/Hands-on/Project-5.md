<h1>Project-5</h1>
Here's a detailed guide for building and deploying **containerized applications** using **GitHub**, **Docker**, and **GitHub Actions**. The goal is to automate the entire process, from building Docker images to deploying them to a container orchestration platform like AWS **Elastic Container Service (ECS)**, **Kubernetes (EKS)**, or **Heroku**.

### **1. Project Directory Structure**

```
containerized-app/
│
├── .github/
│   └── workflows/
│       └── docker-deployment.yml         # GitHub Actions workflow file
│
├── app/
│   ├── Dockerfile                       # Dockerfile for the application
│   ├── app.js                           # Simple Node.js application
│   ├── package.json                     # Node.js application dependencies
│   ├── package-lock.json                # Node.js lock file
│   └── .env                             # Environment variables (should be in .gitignore)
│
├── .gitignore                           # Git ignore file
└── README.md                            # Project documentation
```

### **2. Project Files Content**

#### **2.1. app/Dockerfile**

This is the Dockerfile that will package your application into a container:

```dockerfile
# Use the official Node.js image.
FROM node:14

# Set the working directory in the container.
WORKDIR /usr/src/app

# Copy package.json and package-lock.json.
COPY package*.json ./

# Install the dependencies.
RUN npm install

# Copy the application files.
COPY . .

# Expose the application port.
EXPOSE 3000

# Command to run the application.
CMD ["node", "app.js"]
```

#### **2.2. app/app.js**

This is a simple Node.js application:

```javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
    res.send('Hello, Dockerized Application!');
});

app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
});
```

#### **2.3. app/package.json**

Defines the dependencies for the Node.js application:

```json
{
  "name": "containerized-app",
  "version": "1.0.0",
  "description": "A simple Node.js app for demonstrating containerization and CI/CD",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.17.1"
  },
  "author": "Your Name",
  "license": "ISC"
}
```

#### **2.4. .github/workflows/docker-deployment.yml**

This GitHub Actions workflow automatically builds and pushes your Docker image to **Docker Hub** and deploys it to **AWS ECS** or **EKS**. 

You can modify it to deploy to other platforms like **Heroku** if needed.

```yaml
name: Docker Build and Deploy

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Log in to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build Docker image
        run: |
          docker build -t your-dockerhub-username/containerized-app:latest -f app/Dockerfile .

      - name: Push Docker image to Docker Hub
        run: |
          docker push your-dockerhub-username/containerized-app:latest

  deploy-to-ecs:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to AWS ECS
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: ecs-task-definition.json
          service: your-ecs-service-name
          cluster: your-ecs-cluster-name
          wait-for-service-stability: true
```

> **Notes**:
- **AWS Secrets** (like AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY) must be set in your GitHub Secrets.
- Modify the `ecs-task-definition.json` and `service`, `cluster` values according to your AWS ECS setup.

#### **2.5. app/.env** (create this file but do not commit)

```env
# Environment variables for your app
PORT=3000
```

#### **2.6. .gitignore**

```
# Node.js
node_modules/
npm-debug.log

# Environment variables
.env

# Docker
*.tar
```

#### **2.7. README.md**

```markdown
# Containerized Application

## Overview

This is a simple Node.js application that is Dockerized and uses GitHub Actions to build, push, and deploy the container.

## Setup

### Prerequisites

- Docker installed on your local machine.
- Docker Hub account.
- AWS account (if deploying to ECS or EKS).

### Local Development

1. Clone the repository:

   ```bash
   git clone https://github.com/your-username/containerized-app.git
   cd containerized-app
   ```

2. Build and run the Docker image locally:

   ```bash
   docker build -t containerized-app .
   docker run -p 3000:3000 containerized-app
   ```

3. Access the app at `http://localhost:3000`.

## CI/CD Pipeline

The CI/CD pipeline is configured to:

- **Build the Docker image** upon each push to the `main` branch.
- **Push the Docker image to Docker Hub**.
- **Deploy the Docker image to AWS ECS** (or EKS, Heroku as needed).

## Secrets Configuration

To securely configure your Docker Hub and AWS credentials, add the following secrets in your GitHub repository:

- `DOCKER_USERNAME`: Your Docker Hub username.
- `DOCKER_PASSWORD`: Your Docker Hub password.
- `AWS_ACCESS_KEY_ID`: Your AWS access key.
- `AWS_SECRET_ACCESS_KEY`: Your AWS secret key.

## Deployment

The app is automatically deployed to AWS ECS whenever code is pushed to the `main` branch.
```

### **3. Steps to Save and Deploy the Project**

1. **Create a New GitHub Repository**: Go to GitHub and create a new repository for your project.
2. **Clone the Repository**: Clone your new GitHub repository to your local machine.

   ```bash
   git clone https://github.com/your-username/containerized-app.git
   cd containerized-app
   ```

3. **Create the Directory Structure**: Create the required directories and files based on the structure provided above.
4. **Add Content to Files**: Copy and paste the provided content into the respective files.
5. **Initialize Git**: If this is a new project, initialize Git:

   ```bash
   git init
   git add .
   git commit -m "Initial project setup for Docker CI/CD pipeline"
   ```

6. **Push to GitHub**: Push your local changes to the GitHub repository.

   ```bash
   git remote add origin https://github.com/your-username/containerized-app.git
   git branch -M main
   git push -u origin main
   ```

7. **Configure GitHub Secrets**: Go to your GitHub repository settings and add the following secrets under the "Secrets and variables" section:
   - `DOCKER_USERNAME`: Your Docker Hub username.
   - `DOCKER_PASSWORD`: Your Docker Hub password.
   - `AWS_ACCESS_KEY_ID`: Your AWS access key.
   - `AWS_SECRET_ACCESS_KEY`: Your AWS secret key.

8. **Test the CI/CD Pipeline**: Push changes to the `main` branch to trigger the CI/CD pipeline. Modify the `app.js` file and push the changes to see the automated testing, building, and deployment process.

### **4. How the CI/CD Pipeline Works**

- **Trigger Events**: The pipeline runs on pushes to the `main` branch.
- **Build Job**: It checks out the code, builds the Docker image, and pushes it to Docker Hub.
- **Deploy Job**: Deploys the Docker image to AWS ECS. You can modify the workflow to deploy to AWS EKS, Heroku, or any other platform.

With this setup, you have a fully automated **CI/CD pipeline** that builds, pushes, and deploys your Dockerized application. Let me know if you have any questions or need further assistance!
