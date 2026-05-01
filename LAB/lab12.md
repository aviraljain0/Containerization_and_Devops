# Experiment 12 — Study and Analyse Container Orchestration using Kubernetes

## Objective

To learn the basics of Kubernetes by deploying a WordPress containerized application using Minikube. This experiment covers creating Deployments, exposing Services, scaling Pods, and observing Kubernetes self-healing behavior.

---

## Tools & Environment

- **Minikube** v1.38.1 — Local Kubernetes cluster
- **kubectl** — Kubernetes command-line tool
- **OS** — Ubuntu 22.04 (amd64)
- **Container Runtime** — Docker

---

## Theory

**Kubernetes** is an open-source container orchestration platform that automates deployment, scaling, and management of containerized applications.

Key concepts used in this experiment:

| Term | Description |
|------|-------------|
| **Pod** | Smallest deployable unit; wraps one or more containers |
| **Deployment** | Manages a set of identical pods (replicas) |
| **Service** | Exposes pods to the network (inside or outside the cluster) |
| **NodePort** | A service type that exposes a port on each cluster node |
| **ReplicaSet** | Ensures a specified number of pod replicas are always running |
| **kubectl** | CLI tool to interact with the Kubernetes cluster |

---

## Steps Performed

### Step 1 — Create the Deployment YAML File

A YAML manifest file named `wordpress-deployment.yaml` was created using the `nano` text editor.

```bash
nano wordpress-deployment.yaml
```

**File contents (`wordpress-deployment.yaml`):**

```yaml
# wordpress-deployment.yaml
apiVersion: apps/v1          # Which Kubernetes API to use
kind: Deployment             # Type of resource
metadata:
  name: wordpress            # Name of this deployment
spec:
  replicas: 2                # Run 2 identical pods
  selector:
    matchLabels:
      app: wordpress         # Pods with this label belong to this deployment
  template:                  # Template for the pods
    metadata:
      labels:
        app: wordpress       # Label applied to each pod
    spec:
      containers:
      - name: wordpress
        image: wordpress:latest   # Docker image
        ports:
        - containerPort: 80       # Port inside the container
```

> **Screenshot:**
> ![wordpress-deployment.yaml in nano editor](../Screenshots/Lab12_s/lab12.1.png)

---

### Step 2 — Start Minikube and Apply the Deployment

Minikube was started to create a local single-node Kubernetes cluster. Then the deployment manifest was applied using `kubectl`.

```bash
minikube start
kubectl apply -f wordpress-deployment.yaml
```

**Output:**
```
deployment.apps/wordpress created
```

> **Screenshot:**
> ![minikube start and kubectl apply output](../Screenshots/Lab12_s/lab12.2.png)

**Observations:**
- Minikube v1.38.1 started on Ubuntu 22.04 using the Docker driver
- Kubernetes v1.35.1 was provisioned with 2 CPUs and 3072 MB memory
- The deployment `wordpress` was successfully created

---

### Step 3 — Create the Service YAML File

A service manifest file named `wordpress-service.yaml` was created to expose the WordPress deployment externally using a **NodePort**.

```bash
nano wordpress-service.yaml
```

**File contents (`wordpress-service.yaml`):**

```yaml
# wordpress-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-service
spec:
  type: NodePort              # Exposes service on a port of each node (VM)
  selector:
    app: wordpress            # Send traffic to pods with this label
  ports:
  - port: 80                  # Service port
    targetPort: 80            # Pod port
    nodePort: 30007           # External port (range: 30000-32767)
```

> **Screenshot:**
> ![wordpress-service.yaml in nano editor](../Screenshots/Lab12_s/lab12.3.png)

---

### Step 4 — Apply the Service

```bash
kubectl apply -f wordpress-service.yaml
```

**Output:**
```
service/wordpress-service created
```

> **Screenshot:**
> ![kubectl apply service output](../Screenshots/Lab12_s/lab12.4.png)

---

### Step 5 — Verify Running Pods

The pods were verified using `kubectl get pods` to confirm both replicas were running.

```bash
kubectl get pods
```

**Output:**

| NAME | READY | STATUS | RESTARTS | AGE |
|------|-------|--------|----------|-----|
| wordpress-6698dd7d66-cs5hx | 1/1 | Running | 0 | 2m4s |
| wordpress-6698dd7d66-hv455 | 1/1 | Running | 0 | 2m4s |

> **Screenshot:**
> ![kubectl get pods showing 2 running pods](../Screenshots/Lab12_s/lab12.5.png)
Both pods are in **Running** state as expected, matching the `replicas: 2` setting in the deployment.

---

### Step 6 — Verify the Service

The service was verified to confirm it was assigned a ClusterIP and NodePort.

```bash
kubectl get svc
```

**Output:**

| NAME | TYPE | CLUSTER-IP | EXTERNAL-IP | PORT(S) | AGE |
|------|------|------------|-------------|---------|-----|
| kubernetes | ClusterIP | 10.96.0.1 | \<none\> | 443/TCP | 2m47s |
| wordpress-service | NodePort | 10.106.38.175 | \<none\> | 80:30007/TCP | 31s |

> **Screenshot:**
> ![kubectl get svc output](../Screenshots/Lab12_s/lab12.6.png)

The `wordpress-service` is exposed on **port 30007** externally via NodePort.

---

### Step 7 — Scale the Deployment

The deployment was scaled from 2 replicas to 4 replicas to demonstrate horizontal scaling.

```bash
kubectl scale deployment wordpress --replicas=4
```

**Output:**
```
deployment.apps/wordpress scaled
```

> **Screenshot:**
> ![kubectl scale deployment output](../Screenshots/Lab12_s/lab12.7.png)

---

### Step 8 — Verify Scaled Pods

After scaling, `kubectl get pods` was run again to confirm 4 pods are now running.

```bash
kubectl get pods
```

**Output:**

| NAME | READY | STATUS | RESTARTS | AGE |
|------|-------|--------|----------|-----|
| wordpress-6698dd7d66-cs5hx | 1/1 | Running | 0 | 3m26s |
| wordpress-6698dd7d66-hv455 | 1/1 | Running | 0 | 3m26s |
| wordpress-6698dd7d66-n95mf | 1/1 | Running | 0 | 15s |
| wordpress-6698dd7d66-tqvf2 | 1/1 | Running | 0 | 15s |

> **Screenshot:**
> ![kubectl get pods showing 4 running pods](../Screenshots/Lab12_s/lab12.8.png)

Two new pods (`n95mf` and `tqvf2`) were created automatically by the Deployment controller.

---

### Step 9 — Demonstrate Self-Healing (Delete a Pod)

One pod was manually deleted to demonstrate Kubernetes' self-healing capability.

```bash
kubectl delete pod wordpress-6698dd7d66-cs5hx
```

**Output:**
```
pod "wordpress-6698dd7d66-cs5hx" deleted from default namespace
```

> **Screenshot:**
> ![kubectl delete pod output](../Screenshots/Lab12_s/lab12.9.png)

---

### Step 10 — Verify Self-Healing

After deleting the pod, `kubectl get pods` was run to observe that Kubernetes automatically created a new replacement pod.

```bash
kubectl get pods
```

**Output:**

| NAME | READY | STATUS | RESTARTS | AGE |
|------|-------|--------|----------|-----|
| wordpress-6698dd7d66-hv455 | 1/1 | Running | 0 | 4m18s |
| wordpress-6698dd7d66-n95mf | 1/1 | Running | 0 | 67s |
| wordpress-6698dd7d66-tqvf2 | 1/1 | Running | 0 | 67s |
| wordpress-6698dd7d66-zmch7 | 1/1 | Running | 0 | 20s |

> **Screenshot:**
> ![kubectl get pods showing 4 running pods after self-healing](../Screenshots/Lab12_s/lab12.10.png)

The deleted pod `cs5hx` was replaced by a new pod `zmch7` (age: 20s). The total pod count remains **4**, confirming Kubernetes' self-healing behavior.

---

## Commands Summary

| Command | Purpose |
|---------|---------|
| `minikube start` | Start a local Kubernetes cluster |
| `kubectl apply -f <file>.yaml` | Create/update resources from a YAML file |
| `kubectl get pods` | List all running pods |
| `kubectl get svc` | List all services |
| `kubectl scale deployment <name> --replicas=N` | Scale a deployment to N replicas |
| `kubectl delete pod <pod-name>` | Delete a specific pod |
| `kubectl describe deployment <name>` | Show details of a deployment |

---

## Observations

1. **Deployment**: Kubernetes successfully created 2 WordPress pods from the deployment manifest with `replicas: 2`.
2. **Service**: A NodePort service exposed the application on external port `30007`, mapping to container port `80`.
3. **Scaling**: Scaling the deployment to 4 replicas instantly launched 2 additional pods without downtime.
4. **Self-Healing**: When a pod was manually deleted, Kubernetes automatically scheduled a new replacement pod within seconds to maintain the desired replica count of 4.

---

## Conclusion

This experiment demonstrated core Kubernetes concepts in a local Minikube environment:

- Writing and applying **Deployment** and **Service** YAML manifests
- Using `kubectl` to manage and inspect cluster resources
- **Horizontal scaling** of pods via `kubectl scale`
- **Self-healing** — Kubernetes automatically replaces failed or deleted pods to maintain the desired state

Kubernetes abstracts away infrastructure complexity and ensures application availability through its control loop, which continuously reconciles the actual state of the cluster with the desired state defined in the manifests.
