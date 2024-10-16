<h1>Projects-1</h1>

### **Basic Jenkins Pipeline Setup**

#### **Goal:**
Automate the build and test processes using Jenkins, triggered by each code commit.

---

### **Project Directory Structure**

```
jenkins-pipeline-setup/
├── Jenkinsfile                 # Pipeline script
├── src/
│   └── main.py                 # Example application source code
└── tests/
    └── test_main.py            # Example test cases for the application
```

---

### **1. Install Jenkins and Configure a Freestyle Project**

#### **Steps to Install Jenkins:**

1. Install Jenkins on your server using the package manager for your OS (e.g., `apt` for Ubuntu or `yum` for CentOS).
   
   ```bash
   sudo apt update
   sudo apt install openjdk-11-jdk -y
   wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
   sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
   sudo apt update
   sudo apt install jenkins -y
   ```

2. Start Jenkins and access it via `http://<your-server-ip>:8080`:

   ```bash
   sudo systemctl start jenkins
   sudo systemctl enable jenkins
   ```

#### **Configure Jenkins:**

- Log in to Jenkins using the initial admin password (`/var/lib/jenkins/secrets/initialAdminPassword`).
- Install recommended plugins.
- Set up an admin user and complete the setup.

---

### **2. Set Up Source Code Management (SCM)**

To set up source code management in Jenkins, integrate it with GitHub or Git.

1. Go to **Manage Jenkins > Manage Plugins** and install the **Git** plugin.
2. Create a **new Freestyle project** in Jenkins.
3. Under **Source Code Management**, select **Git**, and provide the URL of your GitHub repository.

---

### **3. Write a Basic Jenkinsfile**

The `Jenkinsfile` defines a pipeline with stages like Build, Test, and Deploy. This file is saved in the root of your project.

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building the application...'
                sh 'python --version'  // For Python projects
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'pytest tests/'  // Running tests using pytest
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                // Add deployment scripts here (e.g., copy files, push Docker image)
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
            emailext (
              subject: "Build Success: Job '${env.JOB_NAME}' (${env.BUILD_NUMBER})",
              body: "Build passed.",
              recipientProviders: [[$class: 'DevelopersRecipientProvider']]
            )
        }
        failure {
            echo 'Pipeline failed!'
            emailext (
              subject: "Build Failure: Job '${env.JOB_NAME}' (${env.BUILD_NUMBER})",
              body: "Build failed. Please check the Jenkins logs.",
              recipientProviders: [[$class: 'DevelopersRecipientProvider']]
            )
        }
    }
}
```

#### **Explanation of Jenkinsfile:**

- **pipeline**: Defines the Jenkins pipeline.
- **agent any**: The pipeline can run on any available Jenkins agent.
- **stages**: Defines different stages (Build, Test, Deploy) of the pipeline.
- **steps**: The actual commands run during each stage.
- **post**: Defines actions to take after the pipeline finishes (e.g., send email notifications).

---

### **4. Configure Build Triggers**

To trigger the Jenkins pipeline on every commit or pull request, set up a **Webhook** in GitHub:

1. Go to your GitHub repository and navigate to **Settings > Webhooks**.
2. Add a new webhook with the following details:
   - **Payload URL**: `http://<your-jenkins-server>/github-webhook/`
   - **Content Type**: `application/json`
   - **Event Trigger**: "Just the push event" (or select based on your needs).
3. In Jenkins, go to **Build Triggers** and select **GitHub hook trigger for GITScm polling**.

---

### **5. Add Notifications**

To notify team members on build failures or successes, configure Jenkins for email or Slack notifications.

#### **Email Notification:**

1. Go to **Manage Jenkins > Configure System**.
2. Under **Extended E-mail Notification**, provide SMTP server details and set up default email content.
3. In the **Jenkinsfile**, add the `emailext` function to send emails on success or failure.

#### **Slack Notification:**

1. Install the **Slack Notification** plugin in Jenkins.
2. Configure Slack in **Manage Jenkins > Configure System**.
3. Use the `slackSend` step in the `post` section of the Jenkinsfile:

   ```groovy
   post {
       success {
           echo 'Pipeline completed successfully!'
           slackSend(channel: '#ci-cd', color: 'good', message: "Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) succeeded.")
       }
       failure {
           echo 'Pipeline failed!'
           slackSend(channel: '#ci-cd', color: 'danger', message: "Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) failed.")
       }
   }
   ```

---

### **6. Running the Pipeline**

1. Commit and push the `Jenkinsfile` to your repository.
2. In Jenkins, go to the pipeline job and click **Build Now**.
3. The pipeline will execute the `Build`, `Test`, and `Deploy` stages defined in the `Jenkinsfile`.

---

### **Example Use Case:**

- **Build**: For a Python application, you can check the Python version and install dependencies.
- **Test**: Use `pytest` to run automated tests.
- **Deploy**: Push the code to a production environment or trigger the deployment process.

This Jenkins pipeline setup is highly customizable and can be extended to handle more complex workflows like Docker image builds, Kubernetes deployments, and integration with cloud services (e.g., AWS, GCP).
