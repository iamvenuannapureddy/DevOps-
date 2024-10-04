<h1>Project-4</h1>

Here's a complete guide to set up a **CI/CD pipeline** using **GitHub Actions** for a **Dockerized application**. This example will automate the process of testing, building, and deploying your application. The setup includes a Docker image that will be pushed to Docker Hub upon each push to the main branch.

### **Project Directory Structure**

Let's start by establishing a directory structure for your project. Below is the structure with essential files needed for the project:

```
ci-cd-pipeline/
│
├── .github/
│   └── workflows/
│       └── ci-cd-pipeline.yml         # GitHub Actions workflow file
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

### **1. Project Files Content**

#### **1.1. app/Dockerfile**

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

#### **1.2. app/app.js**

```javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
    res.send('Hello, CI/CD Pipeline!');
});

app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
});
```

#### **1.3. app/package.json**

```json
{
  "name": "ci-cd-pipeline-app",
  "version": "1.0.0",
  "description": "A simple Node.js application for CI/CD pipeline demonstration",
  "main": "app.js",
  "scripts": {
    "start": "node app.js",
    "test": "echo \"No tests specified\" && exit 0"
  },
  "dependencies": {
    "express": "^4.17.1"
  },
  "author": "Your Name",
  "license": "ISC"
}
```

#### **1.4. app/.env** (create this file but do not commit)

```env
# Add your environment variables here
```

#### **1.5. .github/workflows/ci-cd-pipeline.yml**

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '14'

      - name: Install dependencies
        run: |
          cd app
          npm install

      - name: Run tests
        run: |
          cd app
          npm test

      - name: Build Docker image
        run: |
          cd app
          docker build -t your-dockerhub-username/your-image-name .

      - name: Log in to Docker Hub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

      - name: Push Docker image to Docker Hub
        run: |
          docker push your-dockerhub-username/your-image-name
```

#### **1.6. .gitignore**

```
# Node.js
node_modules/
npm-debug.log

# Environment variables
.env

# Docker
*.tar
```

#### **1.7. README.md**

```markdown
# CI/CD Pipeline Project

## Overview

This project demonstrates a CI/CD pipeline using GitHub Actions to automate the testing, building, and deployment of a Dockerized Node.js application.

## Getting Started

### Prerequisites

- Docker installed on your local machine.
- An account on Docker Hub.

### Setup

1. Clone the repository:

   ```bash
   git clone https://github.com/your-username/ci-cd-pipeline.git
   cd ci-cd-pipeline
   ```

2. Create your Docker Hub account and create a repository (replace `your-dockerhub-username` and `your-image-name` in the workflow file).
3. Add your Docker Hub credentials as GitHub secrets:
   - `DOCKER_USERNAME`: Your Docker Hub username.
   - `DOCKER_PASSWORD`: Your Docker Hub password.

4. Build the Docker image locally (optional):

   ```bash
   cd app
   docker build -t your-dockerhub-username/your-image-name .
   ```

5. Push changes to the `main` branch to trigger the CI/CD pipeline.
```

### **2. Steps to Save and Deploy the Project**

1. **Create a New GitHub Repository**: Go to GitHub and create a new repository for your project.
2. **Clone the Repository**: Clone your new GitHub repository to your local machine.

   ```bash
   git clone https://github.com/your-username/ci-cd-pipeline.git
   cd ci-cd-pipeline
   ```

3. **Create the Directory Structure**: Create the required directories and files based on the structure provided above.
4. **Add Content to Files**: Copy and paste the provided content into the respective files.
5. **Initialize Git**: If this is a new project, initialize Git:

   ```bash
   git init
   git add .
   git commit -m "Initial project setup for CI/CD pipeline"
   ```

6. **Push to GitHub**: Push your local changes to the GitHub repository.

   ```bash
   git remote add origin https://github.com/your-username/ci-cd-pipeline.git
   git branch -M main
   git push -u origin main
   ```

7. **Configure GitHub Secrets**: Go to your GitHub repository settings and add the following secrets under the "Secrets and variables" section:
   - `DOCKER_USERNAME`: Your Docker Hub username.
   - `DOCKER_PASSWORD`: Your Docker Hub password.

8. **Test the CI/CD Pipeline**: Push changes to the `main` branch to trigger the CI/CD pipeline. You can modify the `app.js` file and push the changes to see the automated testing, building, and deployment process.

### **3. How the CI/CD Pipeline Works**

- **Trigger Events**: The pipeline runs on pushes or pull requests to the `main` branch.
- **Build Job**: It checks out the code, sets up Node.js, installs dependencies, runs tests, builds the Docker image, logs into Docker Hub, and pushes the image.
- **Docker Hub**: The final Docker image is pushed to your Docker Hub account.

With this setup, you will have a CI/CD pipeline that automates the testing, building, and deployment of your Dockerized application. If you have any questions or need further assistance, feel free to ask!
