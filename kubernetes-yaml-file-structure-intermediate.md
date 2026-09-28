# Kubernetes YAML File Structure — Intermediate Level

For **lab practice**, a realistic **Pod manifest** is a useful way to learn Kubernetes YAML structure. This example covers:

- `apiVersion`
- `kind`
- `metadata`
- `labels`
- `annotations`
- `spec`
- `containers`
- `image`
- `ports`
- `env`
- `resources`
- `livenessProbe`
- `readinessProbe`
- `volumeMounts`
- `volumes`
- `restartPolicy`
- `nodeSelector`
- `securityContext`
- `imagePullPolicy`

---

## 1. General Kubernetes YAML Structure

```yaml
apiVersion: <API-VERSION>

kind: <RESOURCE-TYPE>

metadata:
  name: <RESOURCE-NAME>
  namespace: <NAMESPACE>

  labels:
    key: value

  annotations:
    key: value

spec:

  containers:
    - name: <CONTAINER-NAME>
      image: <IMAGE>

      ports:
        - containerPort: <PORT>

      env:
        - name: <VARIABLE>
          value: <VALUE>

      resources:
        requests:
          cpu: <CPU>
          memory: <MEMORY>
        limits:
          cpu: <CPU>
          memory: <MEMORY>

      volumeMounts:
        - name: <VOLUME>
          mountPath: <PATH>

      livenessProbe:
        <PROBE-CONFIGURATION>

      readinessProbe:
        <PROBE-CONFIGURATION>

  volumes:
    - name: <VOLUME>
      <VOLUME-TYPE>

  restartPolicy: <POLICY>

  nodeSelector:
    <KEY>: <VALUE>
```

---

# 2. Best Intermediate Lab Example

```yaml
apiVersion: v1

kind: Pod

metadata:
  name: nginx-intermediate-pod
  namespace: default

  labels:
    app: nginx
    environment: dev
    tier: frontend

  annotations:
    description: "Intermediate Kubernetes Pod for lab practice"

spec:

  containers:

    - name: nginx-container
      image: nginx:1.27
      imagePullPolicy: IfNotPresent

      ports:
        - name: http
          containerPort: 80
          protocol: TCP

      env:
        - name: APP_ENV
          value: "development"

        - name: APP_NAME
          value: "nginx-lab"

      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"

        limits:
          cpu: "500m"
          memory: "256Mi"

      volumeMounts:
        - name: nginx-data
          mountPath: /usr/share/nginx/html

      livenessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 10
        periodSeconds: 10
        timeoutSeconds: 2
        failureThreshold: 3

      readinessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 5
        timeoutSeconds: 2
        failureThreshold: 3

      securityContext:
        allowPrivilegeEscalation: false

  volumes:

    - name: nginx-data
      emptyDir: {}

  restartPolicy: Always

  nodeSelector:
    kubernetes.io/os: linux
```

---

# 3. Understand the Structure

```text
Pod
│
├── apiVersion
│
├── kind
│
├── metadata
│   ├── name
│   ├── namespace
│   ├── labels
│   └── annotations
│
└── spec
    │
    ├── containers
    │   │
    │   └── nginx-container
    │       ├── image
    │       ├── imagePullPolicy
    │       ├── ports
    │       ├── env
    │       ├── resources
    │       │   ├── requests
    │       │   └── limits
    │       ├── volumeMounts
    │       ├── livenessProbe
    │       ├── readinessProbe
    │       └── securityContext
    │
    ├── volumes
    │
    ├── restartPolicy
    │
    └── nodeSelector
```

---

# 4. What Happens When You Apply It?

```text
                  nginx-pod.yaml
                        │
                        │
              kubectl apply -f
                        │
                        ▼
                 API Server
                        │
                        ▼
                Validate YAML
                        │
                        ▼
                     etcd
              Desired State Stored
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
              Container Runtime
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
       Pull NGINX Image       Create Container
             │                     │
             └──────────┬──────────┘
                        ▼
                  Pod Networking
                        │
                        ▼
                  Mount Volume
                        │
                        ▼
              Start NGINX Container
                        │
              ┌─────────┴─────────┐
              ▼                   ▼
       Liveness Probe       Readiness Probe
              │                   │
              └─────────┬─────────┘
                        ▼
                  Pod → Running
```

---

# 5. Practice Each Section

## Step 1 — Create the file

```bash
vi nginx-intermediate-pod.yaml
```

Paste the YAML into it.

## Step 2 — Validate the YAML

```bash
kubectl apply --dry-run=client -f nginx-intermediate-pod.yaml
```

## Step 3 — Create the Pod

```bash
kubectl apply -f nginx-intermediate-pod.yaml
```

Expected:

```text
pod/nginx-intermediate-pod created
```

## Step 4 — Check the Pod

```bash
kubectl get pods
```

Example:

```text
NAME                     READY   STATUS    RESTARTS   AGE
nginx-intermediate-pod   1/1     Running   0          20s
```

## Step 5 — Check Detailed Information

```bash
kubectl describe pod nginx-intermediate-pod
```

You can inspect:

```text
Node
IP
Container
Image
Ports
Environment
Volumes
Probes
Events
```

## Step 6 — Check the YAML Kubernetes Created

```bash
kubectl get pod nginx-intermediate-pod -o yaml
```

## Step 7 — Check Container Logs

```bash
kubectl logs nginx-intermediate-pod
```

## Step 8 — Access NGINX from Your Machine

Because `containerPort: 80` does **not automatically expose the Pod outside the cluster**, use port-forwarding for this lab:

```bash
kubectl port-forward pod/nginx-intermediate-pod 8080:80
```

Then open:

```text
http://localhost:8080
```

Flow:

```text
Browser
   │
   │ localhost:8080
   ▼
kubectl port-forward
   │
   │ 80
   ▼
Pod
   │
   ▼
NGINX Container
```

---

# 6. What You Are Practicing

| YAML Section | What You Learn |
|---|---|
| `apiVersion` | Kubernetes API |
| `kind` | Resource type |
| `metadata` | Object identification |
| `labels` | Object classification/selection |
| `annotations` | Metadata for tools |
| `containers` | Container definition |
| `image` | Container image |
| `ports` | Application port declaration |
| `env` | Environment variables |
| `resources` | CPU/memory requests and limits |
| `volumeMounts` | Mounting storage |
| `volumes` | Pod storage |
| `livenessProbe` | Application health |
| `readinessProbe` | Traffic readiness |
| `securityContext` | Container security settings |
| `restartPolicy` | Container restart behavior |
| `nodeSelector` | Node selection |

---

# 7. Interview-Level Understanding

Remember the hierarchy:

```text
Kubernetes Object
       │
       ▼
   metadata
       │
       ├── name
       ├── namespace
       ├── labels
       └── annotations

       ▼
      spec
       │
       ├── containers
       │     ├── image
       │     ├── ports
       │     ├── env
       │     ├── resources
       │     ├── probes
       │     ├── volumeMounts
       │     └── securityContext
       │
       ├── volumes
       ├── restartPolicy
       └── nodeSelector
```

**Important:** Don't try to memorize the entire YAML as one block. Learn the `metadata → spec → containers` hierarchy, then add features such as probes, resources, volumes, and scheduling one at a time.
