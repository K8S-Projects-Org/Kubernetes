A **Pod is the smallest deployable unit in Kubernetes**. Architecturally, it is a logical wrapper around one or more containers that need to run together.

### 1. Pod Architecture Diagram

```
                    Kubernetes Cluster
                           │
                           ▼
                    ┌───────────────┐
                    │ Kubernetes    │
                    │   Node        │
                    │               │
                    │   ┌─────────┐ │
                    │   │   POD   │ │
                    │   │         │ │
                    │   │ ┌─────┐ │ │
                    │   │ │ App │ │ │
                    │   │ │Container│
                    │   │ └─────┘ │ │
                    │   │    │    │ │
                    │   │ ┌─────┐ │ │
                    │   │ │Side │ │ │
                    │   │ │car  │ │ │
                    │   │ │Container│
                    │   │ └─────┘ │ │
                    │   │         │ │
                    │   │ Shared  │ │
                    │   │ Network │ │
                    │   │ Shared  │ │
                    │   │ Volume  │ │
                    │   └─────────┘ │
                    └───────────────┘
```

## 2. Main Components of Pod Architecture

### A. Pod

The **Pod** is the Kubernetes abstraction that groups one or more containers.

For example:

```
Pod
├── Application Container
└── Sidecar Container
```

The Pod is scheduled onto a **single Node**.

---

### B. Containers

A Pod can contain:

```
1 container
```

or:

```
2+ containers
```

Most applications commonly use:

```
Pod
└── Application Container
```

Multiple containers are used when they are **tightly coupled**.

Example:

```
Pod
├── Java Application
└── Log Collector
```

---

### C. Pause / Infrastructure Container

One important intermediate-level concept is the **pause container**.

Conceptually:

```
                 Pod
                  │
        ┌─────────┴─────────┐
        │                   │
   Pause Container      App Container
        │                   │
        └──── Shared Network┘
```

The container runtime uses a small infrastructure container to establish and maintain the Pod's network namespace.

The application containers join that network namespace.

**Interview point:** You normally don't create or manage the pause container yourself; Kubernetes/container runtime handles it.

---

### D. Shared Network

All containers in a Pod share the **same network namespace**.

Therefore, they share:

- Pod IP address
- Network interface
- Port space

Example:

```
Pod IP: 10.244.1.20

┌──────────────────────────────┐
│             POD              │
│                              │
│ App Container    Sidecar     │
│ Port 8080        Port 9000   │
│       │              │       │
│       └── localhost ─┘       │
└──────────────────────────────┘
```

The containers can communicate using:

```
localhost:9000
```

instead of using the Pod IP.

**Important:** Because containers share the same network namespace, they cannot normally bind to the **same port**.

---

### E. Shared Storage

Containers inside the same Pod can mount the same volume.

```
              Pod
               │
       ┌───────┴────────┐
       ▼                ▼
 App Container     Sidecar Container
       │                │
       └───────┬────────┘
               ▼
         Shared Volume
```

Example:

```
App → writes logs → /shared/logs
                         ↑
                         │
                  Sidecar reads logs
```

This is commonly used in sidecar patterns.

---

# 3. Complete Pod Request Flow

When you create a Pod:

```
kubectl apply -f pod.yaml
             │
             ▼
      Kubernetes API Server
             │
             ▼
      Pod stored in etcd
             │
             ▼
       Scheduler
             │
             ▼
       Selects a Node
             │
             ▼
       Kubelet on Node
             │
             ▼
    Container Runtime
             │
             ▼
       Creates Containers
             │
             ▼
       Pod starts running
```

### Example

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

Flow:

```
pod.yaml
   │
   ▼
kubectl apply
   │
   ▼
API Server
   │
   ▼
Scheduler
   │
   ▼
Worker Node
   │
   ▼
Kubelet
   │
   ▼
Container Runtime
   │
   ▼
NGINX Container
   │
   ▼
Pod IP + Port 80
```

## 4. Pod Architecture vs Container

A common interview question is:

**“Is a Pod a container?”**

**No.**

```
Container
   ↓
Runs the application

Pod
   ↓
Groups and manages one or more containers
   ↓
Provides shared network/storage context
```

### Interview-ready answer

> **“Pod architecture consists of a Pod abstraction running on a single Kubernetes node and containing one or more containers. The containers in a Pod share the Pod's network namespace and can share mounted volumes. Kubernetes schedules the Pod as a unit, while the kubelet and container runtime on the selected node create and manage its containers. A Pod can contain a single application container or multiple tightly coupled containers, such as an application and a sidecar.”**

**Key idea to remember:**

```
Pod = Scheduling Unit
     + Shared Network
     + Shared Storage
     + Container Group
     + Shared Lifecycle
```

Pod structure ( YML )

# Kubernetes Pod Structure — YAML

A Kubernetes **Pod YAML file** is a manifest that tells Kubernetes **what Pod to create and how that Pod should run**.

### Basic Pod YAML structure

```
apiVersion: v1

kind: Pod

metadata:
  name: nginx-pod
  labels:
    app: nginx

spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
```

---

## 1. Pod YAML Structure Diagram

```
Pod YAML
│
├── apiVersion
│
├── kind
│
├── metadata
│   ├── name
│   ├── labels
│   └── annotations
│
└── spec
    │
    ├── containers
    │   ├── name
    │   ├── image
    │   ├── ports
    │   ├── env
    │   ├── resources
    │   ├── volumeMounts
    │   └── probes
    │
    ├── volumes
    │
    ├── restartPolicy
    │
    ├── nodeSelector
    │
    └── serviceAccountName
```

---

# 2. Explain Each Section

## `apiVersion`

```
apiVersion: v1
```

Defines the **Kubernetes API version** used for this resource.

For a basic Pod:

```
apiVersion: v1
```

---

## `kind`

```
kind: Pod
```

Specifies **what Kubernetes object you want to create**.

Here:

```
kind → Pod
```

Other examples include:

```
Deployment
Service
ConfigMap
Secret
Namespace
```

---

## `metadata`

```
metadata:
  name: nginx-pod
  labels:
    app: nginx
```

Contains information that identifies the Pod.

### `name`

```
name: nginx-pod
```

The name of the Pod.

You can check it using:

```
kubectl get pods
```

Output:

```
NAME
nginx-pod
```

### `labels`

```
labels:
  app: nginx
```

Labels are **key-value pairs** used to identify and organize Kubernetes objects.

They are especially important when connecting a Pod to a **Service**.

---

# 3. `spec`

```
spec:
```

`spec` describes the **desired configuration of the Pod**.

This is where you define things such as:

- Containers
- Images
- Ports
- Environment variables
- Volumes
- Resource limits
- Restart policy
- Node selection

---

# 4. `containers`

```
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
```

This is one of the most important sections.

It defines the containers that will run inside the Pod.

```
Pod
 │
 └── containers
      │
      └── nginx-container
```

### `name`

```
name: nginx-container
```

Name of the container.

### `image`

```
image: nginx:latest
```

Specifies the container image that the container runtime should run.

For example:

```
image: nginx:1.27
```

---

# 5. `ports`

```
ports:
  - containerPort: 80
```

Declares the port that the application container uses.

For NGINX:

```
NGINX
  │
  └── Port 80
```

**Important interview point:** `containerPort` does **not by itself expose the application outside the Pod**.

To access a Pod from outside, you typically use a **Service**, Ingress, port-forwarding, or another networking mechanism.

---

# 6. Environment Variables

You can provide environment variables to a container:

```
env:
  - name: APP_ENV
    value: "dev"
```

Example:

```
containers:
  - name: nginx-container
    image: nginx:latest
    env:
      - name: APP_ENV
        value: "dev"
```

Inside the container:

```
APP_ENV=dev
```

---

# 7. Resources

You can define CPU and memory requirements:

```
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "256Mi"
```

Conceptually:

```
             Container
                 │
       ┌─────────┴─────────┐
       │                   │
   Requests              Limits
       │                   │
   Minimum needed       Maximum allowed
```

---

# 8. Volumes

A Pod can define shared storage:

```
volumes:
  - name: app-data
    emptyDir: {}
```

Then mount it into the container:

```
volumeMounts:
  - name: app-data
    mountPath: /data
```

Complete relationship:

```
Pod
│
├── Container
│      │
│      └── /data
│
└── Volume
       │
       └── app-data
```

---

# 9. `restartPolicy`

Example:

```
restartPolicy: Always
```

Controls what Kubernetes should do when a container terminates.

Common values:

```
Always
OnFailure
Never
```

For a normal long-running application Pod, `Always` is commonly used and is also the default.

---

# 10. Complete Intermediate-Level Pod YAML

```
apiVersion: v1

kind: Pod

metadata:
  name: nginx-pod
  labels:
    app: nginx

spec:
  containers:
    - name: nginx-container
      image: nginx:1.27

      ports:
        - containerPort: 80

      env:
        - name: APP_ENV
          value: "dev"

      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "500m"
          memory: "256Mi"

      volumeMounts:
        - name: app-data
          mountPath: /data

  volumes:
    - name: app-data
      emptyDir: {}

  restartPolicy: Always
```

---

# 11. How Kubernetes Processes This YAML

```
pod.yaml
   │
   │ kubectl apply -f pod.yaml
   ▼
API Server
   │
   ▼
Validate Pod definition
   │
   ▼
Store desired state in etcd
   │
   ▼
Scheduler
   │
   ▼
Select Worker Node
   │
   ▼
Kubelet
   │
   ▼
```