### **What is a Namespace in Kubernetes?**  
A **namespace** in Kubernetes is a **logical isolation mechanism** that allows you to group resources within a cluster. It is useful for **organizing and managing** workloads, especially in **multi-team or multi-environment setups**.

---

### **How is a Namespace Useful?**
Namespaces help in **separating and organizing** different workloads efficiently:

 **Multi-Tenancy** – Different teams can work in **isolated environments** within the same cluster.  
 **Resource Quotas** – Limits can be set per namespace to **control CPU and memory usage**.  
 **Security & Access Control** – RBAC (Role-Based Access Control) can be used to **restrict access** to specific namespaces.  
 **Environment Separation** – Useful for **staging, development, and production** environments.  

---

### **Challenges Solved by Namespaces**
1. **Avoids Resource Conflicts**  
   - Multiple applications can run on the same cluster **without conflicting** over resource names.  
   - Example: Two teams can **use the same deployment name** (`frontend`) in different namespaces (`team-a`, `team-b`).

2. **Better Organization**  
   - Large clusters with **hundreds of services** can be **grouped logically** (e.g., `dev`, `staging`, `prod`).  

3. **Fine-Grained Access Control**  
   - Using **RBAC**, you can give developers **access only to their namespace** instead of the entire cluster.  

4. **Simplifies Resource Management**  
   - Kubernetes allows **setting quotas per namespace** to prevent a single team from consuming all cluster resources.

---


## **Default Namespaces in Kubernetes (EKS)**
When you create an EKS cluster, it comes with the following **default namespaces**:

| Namespace       | Purpose |
|----------------|---------|
| **default**    | Default namespace for all resources when no namespace is specified. |
| **kube-system** | Contains Kubernetes system components (like CoreDNS, kube-proxy, AWS VPC CNI). Do not delete or modify resources here. |
| **kube-public** | Publicly readable namespace, often used for cluster-wide info (e.g., discovery). |
| **kube-node-lease** | Manages node heartbeats to track node availability efficiently. |
| **aws-observability** (EKS) | Stores AWS Observability components (e.g., Fluent Bit for logging). Only in AWS EKS. |

---



### **Demo Deployment File Using Namespace**  
Below is a **sample YAML file** that creates a **namespace** and deploys an **nginx application** inside it.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev-namespace  # Custom namespace name

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: dev-namespace  # Assigning the deployment to "my-namespace"
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: dev-namespace  # Assigning the service to "my-namespace"
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

---

### **Steps to Apply & Verify**
 **Apply the namespace and deployment:**  
   ```sh
   kubectl apply -f namespace-deployment.yaml
   ```

 **Check created namespaces:**  
   ```sh
   kubectl get namespaces
   ```

 **List resources inside the namespace:**  
   ```sh
   kubectl get deployments,pods,services -n my-namespace
   ```

 **Delete the namespace (cleanup):**  
   ```sh
   kubectl delete namespace my-namespace
   ```

---

### **Test the ResourceQuota in a Kubernetes Cluster**

---

## **Step 1: Set Namespace-Level Resource Limits (LimitRange)**  
To enforce resource constraints **at the namespace level**, create a **LimitRange**.

### **Create a LimitRange for `dev-namespace`**  
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: dev-namespace-limits
  namespace: dev-namespace
spec:
  limits:
  - default:
      cpu: "1"      # Max CPU a container can use
      memory: "1Gi" # Max memory a container can use
    defaultRequest:
      cpu: "0.5"    # Default requested CPU
      memory: "512Mi" # Default requested memory
    type: Container
```

 **Apply the LimitRange**  
```sh
kubectl apply -f limitrange.yaml
```

 **Verify the LimitRange**  
```sh
kubectl get limitrange -n dev-namespace
kubectl describe limitrange dev-namespace-limits -n dev-namespace
```

---

## **Step 2: Create a Pod (Success Scenario)**
Now, deploy a pod that **respects** the namespace limits.

### **Success Scenario: Pod within Limits**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod-success
  namespace: dev-namespace
spec:
  containers:
    - name: nginx
      image: nginx
      resources:
        requests:
          cpu: "0.5" # Within the allowed limit
          memory: "512Mi" # Within the allowed limit
        limits:
          cpu: "1" # Allowed max CPU
          memory: "1Gi" # Allowed max memory
```

**Apply the Pod**  
```sh
kubectl apply -f test-pod-success.yaml
```

**Check Pod Status**  
```sh
kubectl get pod -n dev-namespace
kubectl describe pod test-pod-success -n dev-namespace
```

---

## **Step 3: Create a Pod (Fail Scenario)**
Now, deploy a pod that **exceeds** the namespace limits.

### **Fail Scenario: Pod Exceeding Limits**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod-fail
  namespace: dev-namespace
spec:
  containers:
    - name: nginx
      image: nginx
      resources:
        requests:
          cpu: "2" # Exceeds the allowed limit (1)
          memory: "2Gi" # Exceeds the allowed limit (1Gi)
        limits:
          cpu: "3" # Exceeds the allowed limit (1)
          memory: "2Gi" # Exceeds the allowed limit (1Gi)
```

**Apply the Pod**  
```sh
kubectl apply -f test-pod-fail.yaml
```

**Check Pod Status**  
```sh
kubectl get pod -n dev-namespace
kubectl describe pod test-pod-fail -n dev-namespace
```

---
