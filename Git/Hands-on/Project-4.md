<h1>Project-4</h1>

Setting up Git with Continuous Integration (CI) tools allows you to automate the testing and building process every time code is committed or a pull request (PR) is made. Below is a comprehensive guide on how to set up CI using GitHub with different CI tools such as Jenkins, CircleCI, and GitHub Actions.

### Step-by-Step Guide: Git with Continuous Integration (CI) Tools

---

### **1. Set Up a Repository in GitHub**

1. **Create a New Repository**:
   - Go to GitHub and click on the **+** icon at the top right corner.
   - Select **New repository**.
   - Fill in the repository name and description.
   - Choose visibility (public or private).
   - Optionally, initialize the repository with a README.
   - Click **Create repository**.

2. **Clone the Repository Locally**:
   ```bash
   git clone https://github.com/your-username/your-repository.git
   cd your-repository
   ```

---

### **2. Integrate CI Tools**

#### Option A: **Using GitHub Actions**

GitHub Actions is built into GitHub and allows you to automate workflows directly in your repository.

1. **Create a Workflow File**:
   - In your repository, navigate to the **Actions** tab.
   - GitHub will suggest workflows based on your project. Choose one or select **set up a workflow yourself**.
   - Alternatively, create a new file in `.github/workflows/` (e.g., `ci.yml`).

2. **Define the Workflow**:
   Here’s an example of a basic workflow to run tests on every push or PR:
   ```yaml
   name: CI Pipeline

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
           run: npm install

         - name: Run tests
           run: npm test
   ```

3. **Commit and Push the Workflow**:
   ```bash
   git add .github/workflows/ci.yml
   git commit -m "Add CI workflow"
   git push origin main
   ```

4. **Trigger CI**:
   Every time you push code or create a PR to `main`, the tests defined in your workflow will run automatically.

---

#### Option B: **Using Jenkins**

1. **Set Up Jenkins**:
   - Install Jenkins on your server or use a cloud service.
   - Open Jenkins in your browser and create an admin user.

2. **Install Git Plugin**:
   - Navigate to **Manage Jenkins** > **Manage Plugins**.
   - In the **Available** tab, search for **Git Plugin** and install it.

3. **Create a New Pipeline Job**:
   - Click on **New Item**.
   - Enter a name for your job and select **Pipeline**. Click **OK**.

4. **Configure the Pipeline**:
   - In the job configuration, under **Pipeline**, select **Pipeline script**.
   - Use the following example script in your `Jenkinsfile`:
   ```groovy
   pipeline {
       agent any
       stages {
           stage('Checkout') {
               steps {
                   git 'https://github.com/your-username/your-repository.git'
               }
           }
           stage('Build') {
               steps {
                   sh 'npm install'
               }
           }
           stage('Test') {
               steps {
                   sh 'npm test'
               }
           }
       }
   }
   ```

5. **Trigger Builds**:
   - Under **Build Triggers**, check **GitHub hook trigger for GITScm polling** to trigger builds on commits or PRs.

6. **Push Changes to GitHub**:
   - Whenever you push changes or create a PR, Jenkins will automatically pull the changes and run the defined pipeline.

---

#### Option C: **Using CircleCI**

1. **Sign Up for CircleCI**:
   - Go to [CircleCI](https://circleci.com/) and sign up with GitHub.

2. **Add Your Project**:
   - From the CircleCI dashboard, click on **Add Projects** and select your repository.

3. **Create a `.circleci/config.yml` File**:
   In your repository, create a `.circleci` directory and add a `config.yml` file with the following content:
   ```yaml
   version: 2.1

   jobs:
     build:
       docker:
         - image: circleci/node:14
       steps:
         - checkout
         - run:
             name: Install dependencies
             command: npm install
         - run:
             name: Run tests
             command: npm test

   workflows:
     version: 2
     test:
       jobs:
         - build:
             filters:
               branches:
                 only: main
   ```

4. **Commit and Push**:
   ```bash
   git add .circleci/config.yml
   git commit -m "Add CircleCI configuration"
   git push origin main
   ```

5. **Trigger CI**:
   CircleCI will automatically run builds for every commit and PR based on your configuration.

---

### **3. Define Build/Test Steps in the Pipeline**

In each CI tool, you define the build and test steps in the respective configuration files (`Jenkinsfile` for Jenkins, `.circleci/config.yml` for CircleCI, and `.github/workflows/*.yml` for GitHub Actions).

### **4. Trigger Automated Builds and Tests**

Once everything is set up:
- Any commits pushed to the repository or PRs created will trigger the CI pipelines as defined.
- You can check the build status in the respective CI tool's dashboard (GitHub Actions, Jenkins, CircleCI).

---

### Summary of Steps:

1. **Set up a repository on GitHub**.
2. **Integrate CI tools**:
   - GitHub Actions: Create `.github/workflows/ci.yml`.
   - Jenkins: Create a `Jenkinsfile` and configure a pipeline job.
   - CircleCI: Create a `.circleci/config.yml`.
3. **Define build/test steps**.
4. **Trigger automated builds/tests** on commits and PRs.

By following this guide, you can set up CI to automate testing and building processes efficiently. Let me know if you have any questions or need further assistance!
