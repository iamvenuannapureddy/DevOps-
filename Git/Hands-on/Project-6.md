<h1>Project-6</h1>
**GitOps with Kubernetes** is an operational framework that applies DevOps best practices to infrastructure automation, leveraging Git as the single source of truth. When you integrate Git with Kubernetes through GitOps, changes committed to a Git repository (like Kubernetes manifests or Helm charts) trigger automatic updates to the Kubernetes environment using tools like ArgoCD or Flux.

### Steps for Setting Up GitOps with Kubernetes

---

### **1. Set Up a Git Repository with Kubernetes Manifests or Helm Charts**

1. **Create a Git repository** (on GitHub, GitLab, Bitbucket, etc.) to store your Kubernetes configuration files.
   - This can be Kubernetes YAML manifests, Helm charts, or a combination of both.
   - These files define the desired state of your infrastructure and applications.

   Example folder structure:

   ```
   my-kubernetes-repo/
   ├── manifests/
   │   ├── deployment.yaml
   │   └── service.yaml
   └── helm/
       └── my-app-chart/
           ├── Chart.yaml
           ├── values.yaml
           └── templates/
               ├── deployment.yaml
               └── service.yaml
   ```

2. **Add your Kubernetes manifests or Helm charts** to this repository.

3. **Commit and push** the changes to the repository:
   ```bash
   git add .
   git commit -m "Initial commit of Kubernetes manifests"
   git push origin main
   ```

---

### **2. Set Up a Kubernetes Cluster**

If you haven't already set up a Kubernetes cluster, you can do so using managed services like **Amazon EKS**, **Google GKE**, or **Azure AKS**, or you can install a local Kubernetes cluster using **Minikube** or **Kind**.

---

### **3. Install and Configure ArgoCD or Flux**

#### **Option A: Using ArgoCD**

**ArgoCD** is a popular GitOps tool that continuously monitors your Git repository and automatically syncs changes to your Kubernetes cluster.

1. **Install ArgoCD** on your Kubernetes cluster:
   - You can install ArgoCD by applying the following YAML:
     ```bash
     kubectl create namespace argocd
     kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
     ```

2. **Access the ArgoCD UI**:
   - Forward the ArgoCD server port to localhost:
     ```bash
     kubectl port-forward svc/argocd-server -n argocd 8080:443
     ```
   - You can access the ArgoCD UI by visiting `https://localhost:8080` in your browser.

3. **Log in to ArgoCD**:
   - Get the initial admin password:
     ```bash
     kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode
     ```
   - Username: `admin`
   - Password: (use the output from the above command)

4. **Configure an ArgoCD application** to track your Git repository:
   - Open the ArgoCD UI.
   - Click on **New App** and provide the necessary information, like your Git repository URL, the path to your Kubernetes manifests, and the target cluster.

5. **Sync the Application**:
   - Once configured, ArgoCD will automatically apply the Kubernetes manifests or Helm charts from your repository to the cluster whenever changes are pushed to the Git repository.

#### **Option B: Using Flux**

**Flux** is another GitOps operator that continuously applies the desired state from your Git repository to your Kubernetes cluster.

1. **Install Flux** using the `flux` CLI:
   - Install the `flux` CLI:
     ```bash
     curl -s https://fluxcd.io/install.sh | sudo bash
     ```

2. **Bootstrap Flux** with your Git repository:
   - Flux will link the Git repository to your Kubernetes cluster:
     ```bash
     flux bootstrap github \
       --owner=<github-username> \
       --repository=<repo-name> \
       --branch=main \
       --path=./clusters/my-cluster
     ```

   This will:
   - Create the Flux controllers in the Kubernetes cluster.
   - Link the Git repository and configure Flux to watch for changes.

3. **Deploy Infrastructure Automatically**:
   - Once the setup is complete, Flux will automatically deploy the manifests or Helm charts from your Git repository to the Kubernetes cluster whenever you push changes to the repository.

---

### **4. Automate Infrastructure Changes**

With either **ArgoCD** or **Flux** in place, the flow for applying infrastructure changes automatically is:

1. **Make changes to your Kubernetes manifests or Helm charts** in the Git repository.
   
   For example, if you're updating an application version in a `deployment.yaml` file:
   ```yaml
   spec:
     containers:
     - name: my-app
       image: my-app-image:v2.0.0
   ```

2. **Commit and push the changes**:
   ```bash
   git add .
   git commit -m "Updated app version to v2.0.0"
   git push origin main
   ```

3. **ArgoCD or Flux** will detect the changes in the repository and automatically apply them to the Kubernetes cluster, updating the infrastructure or application accordingly.

---

### **Summary**

1. **Set up a Git repository** with Kubernetes manifests or Helm charts.
2. **Install a GitOps tool** like **ArgoCD** or **Flux** in your Kubernetes cluster.
3. **Link the Git repository** to the GitOps tool.
4. Whenever a commit is made to the Git repository:
   - **ArgoCD** or **Flux** will automatically sync the changes to the Kubernetes cluster, ensuring that the cluster matches the desired state defined in the Git repository.

With GitOps, you ensure that your Kubernetes infrastructure and applications are deployed and managed via simple Git commits, providing version control and auditability. Let me know if you'd like help with any specific parts of this process!
