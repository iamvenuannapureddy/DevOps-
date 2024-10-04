<h1>Project-3</h1>
Developing and deploying a **full-stack web application** involves creating both the **frontend** and **backend** of the application, managing code through GitHub, and automating the deployment process using CI/CD pipelines. Here's a detailed **end-to-end guide** for building and deploying a full-stack web app using **Node.js, React** (as an example stack), and integrating **GitHub Actions** for CI/CD.

---

### **Steps to Develop and Deploy a Full-Stack Web Application**

---

### **1. Create a GitHub Repository for the Project**

Start by creating a GitHub repository to host your full-stack web application’s code.

#### **Step 1.1: Initialize a GitHub Repository**

1. Go to [GitHub](https://github.com) and log into your account.
2. Create a new repository and name it something like `full-stack-app`.
3. Choose to initialize the repository with a README file, and optionally, add a `.gitignore` file for Node.js or Python if applicable.

#### **Step 1.2: Clone the Repository Locally**

Now clone the repository to your local machine:

```bash
git clone https://github.com/your-username/full-stack-app.git
cd full-stack-app
```

---

### **2. Set Up Backend and Frontend Code Separately**

We will use **Node.js** for the backend and **React** for the frontend.

#### **Step 2.1: Set Up Backend (Node.js)**

1. Navigate to your project directory and create a `backend` folder:

    ```bash
    mkdir backend
    cd backend
    ```

2. Initialize a new Node.js project:

    ```bash
    npm init -y
    ```

3. Install necessary dependencies for a basic API (e.g., **Express.js** for the server):

    ```bash
    npm install express
    ```

4. Create a basic server in `backend/index.js`:

    ```js
    const express = require('express');
    const app = express();
    const PORT = process.env.PORT || 5000;

    app.get('/', (req, res) => {
        res.send('Hello from the backend!');
    });

    app.listen(PORT, () => {
        console.log(`Server is running on port ${PORT}`);
    });
    ```

5. In the `backend/package.json`, add the following script to run your server:

    ```json
    "scripts": {
        "start": "node index.js"
    }
    ```

6. Test the backend locally by running:

    ```bash
    npm start
    ```

You should be able to access the backend at `http://localhost:5000`.

#### **Step 2.2: Set Up Frontend (React)**

1. Navigate back to the root directory and create a `frontend` folder:

    ```bash
    cd ..
    npx create-react-app frontend
    ```

2. Start the React development server:

    ```bash
    cd frontend
    npm start
    ```

You should see the default React app running at `http://localhost:3000`.

---

### **3. Commit Backend and Frontend Code Separately**

It’s a good practice to manage your **frontend** and **backend** code in separate branches to keep your work organized.

#### **Step 3.1: Commit Backend Code**

1. Navigate to the backend directory and add all the files:

    ```bash
    cd backend
    git add .
    ```

2. Commit the backend changes:

    ```bash
    git commit -m "Initial commit - backend"
    ```

3. Create a new branch for the backend:

    ```bash
    git checkout -b backend
    ```

4. Push the backend branch to GitHub:

    ```bash
    git push origin backend
    ```

#### **Step 3.2: Commit Frontend Code**

1. Navigate to the frontend directory and add all the files:

    ```bash
    cd ../frontend
    git add .
    ```

2. Commit the frontend changes:

    ```bash
    git commit -m "Initial commit - frontend"
    ```

3. Create a new branch for the frontend:

    ```bash
    git checkout -b frontend
    ```

4. Push the frontend branch to GitHub:

    ```bash
    git push origin frontend
    ```

---

### **4. Set Up GitHub Actions for CI/CD**

We’ll use **GitHub Actions** to automate the deployment of both the backend and frontend.

#### **Step 4.1: Set Up GitHub Actions for Backend**

1. In the root of your repository, create a directory for GitHub Actions workflows:

    ```bash
    mkdir -p .github/workflows
    ```

2. Create a `backend-deployment.yml` file in `.github/workflows`:

    ```yaml
    name: Backend Deployment

    on:
      push:
        branches:
          - backend

    jobs:
      build:
        runs-on: ubuntu-latest

        steps:
          - name: Checkout repository
            uses: actions/checkout@v2

          - name: Set up Node.js
            uses: actions/setup-node@v2
            with:
              node-version: '14'

          - name: Install dependencies
            run: |
              cd backend
              npm install

          - name: Run backend server
            run: |
              cd backend
              npm start
    ```

This action will install dependencies and run the backend whenever changes are pushed to the `backend` branch.

#### **Step 4.2: Set Up GitHub Actions for Frontend**

1. In the same `.github/workflows` directory, create a `frontend-deployment.yml` file:

    ```yaml
    name: Frontend Deployment

    on:
      push:
        branches:
          - frontend

    jobs:
      build:
        runs-on: ubuntu-latest

        steps:
          - name: Checkout repository
            uses: actions/checkout@v2

          - name: Set up Node.js
            uses: actions/setup-node@v2
            with:
              node-version: '14'

          - name: Install dependencies
            run: |
              cd frontend
              npm install

          - name: Build frontend
            run: |
              cd frontend
              npm run build
    ```

This action will build the frontend whenever changes are pushed to the `frontend` branch.

---

### **5. Deploy the Application**

#### **Step 5.1: Backend Deployment**

For backend deployment, you can:

- Use platforms like **Heroku**, **AWS Elastic Beanstalk**, or **DigitalOcean**.
- You’ll typically need to set up a **Procfile** for Heroku or a Dockerfile for other cloud providers.

#### **Step 5.2: Frontend Deployment**

For frontend deployment:

- You can use **Netlify**, **Vercel**, or **GitHub Pages** to deploy the React app.
- If using GitHub Pages, enable it in your repository’s settings, or use a deployment action for Netlify/Vercel.

---

### **6. Integrate GitHub Secrets for Managing API Keys**

If your full-stack app requires API keys or sensitive data, use **GitHub Secrets** to securely store and manage them.

#### **Step 6.1: Add Secrets in GitHub**

1. Go to the repository’s **Settings**.
2. Click on **Secrets** on the left sidebar.
3. Click on **New repository secret**.
4. Add the name and value of your secret (e.g., `API_KEY`).

#### **Step 6.2: Use Secrets in GitHub Actions**

To use secrets in your GitHub Actions workflows, reference them like this:

```yaml
env:
  API_KEY: ${{ secrets.API_KEY }}
```

This ensures your API keys and sensitive data are secure during the build and deployment process.

---

### **Summary of Steps**

1. **Create a GitHub repository** to host your project.
2. **Set up backend and frontend code** separately in branches.
3. **Commit and push** changes to GitHub.
4. **Configure GitHub Actions** to automate CI/CD for both frontend and backend.
5. **Deploy the application** to platforms like Heroku, Netlify, or Vercel.
6. **Use GitHub Secrets** to manage sensitive data like API keys.

By following these steps, you’ll have a fully functional full-stack web application with automated deployment. Let me know if you need assistance with any of the steps!
