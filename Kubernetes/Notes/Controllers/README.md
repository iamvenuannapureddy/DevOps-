<h1>Controllers</h1>

In Kubernetes, **controllers** are processes that watch the state of the cluster through the API server and make changes to ensure the desired state matches the actual state. Controllers are part of the control plane and manage specific resources or workloads.

---

### **List of Kubernetes Controllers**

#### **Workload Controllers**
1. **Deployment Controller**  
   - Manages Deployments and ensures the desired number of replicas are running with rolling updates or rollbacks.

2. **ReplicaSet Controller**  
   - Ensures a specified number of Pod replicas are running.

3. **StatefulSet Controller**  
   - Manages stateful applications with stable network identities and persistent storage.

4. **DaemonSet Controller**  
   - Ensures a Pod is running on every (or specific) node in the cluster.

5. **Job Controller**  
   - Manages Jobs to ensure Pods complete a specific task successfully.

6. **CronJob Controller**  
   - Manages CronJobs to run Jobs on a scheduled basis.

7. **ReplicationController**  
   - Ensures a specified number of Pod replicas are running (superseded by ReplicaSet).

---

#### **Service and Networking Controllers**
8. **Service Controller**  
   - Manages Kubernetes Services and ensures networking rules are applied for communication between Pods.

9. **Ingress Controller** *(not built-in, typically implemented using third-party tools like NGINX, Traefik, etc.)*  
   - Manages Ingress resources to provide HTTP and HTTPS routing.

10. **EndpointSlice Controller**  
    - Manages EndpointSlices for efficient network discovery and scalability.

---

#### **Volume and Storage Controllers**
11. **PersistentVolume Controller**  
    - Manages PersistentVolume lifecycle, ensuring they are provisioned and bound to PersistentVolumeClaims.

12. **PersistentVolumeClaim Controller**  
    - Ensures that PersistentVolumeClaims are correctly bound to PersistentVolumes.

13. **StorageClass Controller**  
    - Manages dynamic provisioning of PersistentVolumes based on StorageClasses.

---

#### **Node and Resource Controllers**
14. **Node Controller**  
    - Manages node lifecycle and handles node unavailability.

15. **ResourceQuota Controller**  
    - Enforces resource limits (e.g., CPU, memory) across namespaces.

16. **LimitRange Controller**  
    - Enforces resource request/limit constraints on Pods and Containers in a namespace.

---

#### **Custom Controllers**
17. **Horizontal Pod Autoscaler (HPA) Controller**  
    - Scales the number of Pods horizontally based on CPU/memory or custom metrics.

18. **Vertical Pod Autoscaler (VPA) Controller**  
    - Adjusts resource requests and limits for Pods automatically.

19. **Custom Controller** *(built using the Kubernetes Controller Framework)*  
    - Allows users to define and manage custom resources (CRDs) and their behavior.

---

### Built-in vs. External Controllers
- **Built-in Controllers:** Native to Kubernetes, such as ReplicaSet, Deployment, Job, etc.
- **External Controllers:** Added through third-party tools, such as Ingress controllers (NGINX, Traefik), service mesh controllers (Istio), and operators.

Would you like a deep dive into any specific controller or how they work?
