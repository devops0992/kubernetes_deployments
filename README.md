# ☸️ Kubernetes Deployment Strategies

A hands-on project demonstrating three core Kubernetes deployment strategies using a sample **Online Shop** application. Each strategy is isolated in its own namespace with dedicated manifests.

> 🔗 GitHub Repository: [https://github.com/devops0992/kubernetes_deployments.git](https://github.com/devops0992/kubernetes_deployments.git)

---

## 📁 Project Structure

```
kubernetes_deployments/
│
├── blue-green/
│   ├── blue-green-ns                          # Namespace manifest
│   ├── blue_green_deployment.yaml             # Green deployment (with footer)
│   └── without_footer_blue_green_deployment.yaml  # Blue deployment (without footer)
│
├── recreate/
│   ├── recreate-namespace.yml                 # Namespace manifest
│   ├── recreate-deployment.yml                # Recreate strategy deployment
│   └── recreate-svc.yml                       # NodePort service
│
└── rolling-update/
    ├── rolling-namespace.yaml                 # Namespace manifest
    ├── rolling-update-deployment.yaml         # RollingUpdate strategy deployment
    └── rolling-update-svc.yaml                # NodePort service
```

---

## 🚀 Deployment Strategies Covered

### 1. 🔵🟢 Blue-Green Deployment

**Namespace:** `blue-green-ns`

Blue-Green deployment runs two identical environments simultaneously — **Blue** (current/stable) and **Green** (new version). Traffic is switched from one to the other by updating the service selector, allowing instant rollbacks with zero downtime.

#### How it works in this project:

| Environment | Image | Service Port | Node Port |
|-------------|-------|-------------|-----------|
| **Green** (with footer) | `amitabhdevops/online_shop` | 3000 | 30000 |
| **Blue** (without footer) | `amitabhdevops/online_shop_without_footer` | 3001 | 30001 |

- Both deployments run **4 replicas** simultaneously.
- Each environment has its own dedicated service.
- To switch traffic, update the main load balancer / ingress selector to point to the desired environment.

#### Apply manifests:

```bash
kubectl apply -f blue-green/blue-green-ns
kubectl apply -f blue-green/without_footer_blue_green_deployment.yaml   # Blue
kubectl apply -f blue-green/blue_green_deployment.yaml                  # Green
```

#### Verify:

```bash
kubectl get all -n blue-green-ns
```

---

### 2. ♻️ Recreate Deployment

**Namespace:** `recreate-ns`

The **Recreate** strategy terminates all existing pods before creating new ones. This causes a brief downtime but guarantees that old and new versions never run side by side — useful for applications that cannot handle multiple versions concurrently.

#### Configuration:

```yaml
strategy:
  type: Recreate
```

| Detail | Value |
|--------|-------|
| Image | `amitabhdevops/online-shop` |
| Replicas | 4 |
| Service Type | NodePort |
| Node Port | 30000 |

#### Apply manifests:

```bash
kubectl apply -f recreate/recreate-namespace.yml
kubectl apply -f recreate/recreate-deployment.yml
kubectl apply -f recreate/recreate-svc.yml
```

#### Verify:

```bash
kubectl get all -n recreate-ns
```

#### Simulate an update (to observe recreate behavior):

```bash
kubectl set image deployment/online-shop online-shop=<new-image> -n recreate-ns
kubectl rollout status deployment/online-shop -n recreate-ns
```

---

### 3. 🔄 Rolling Update Deployment

**Namespace:** `rolling-ns`

The **RollingUpdate** strategy gradually replaces old pods with new ones, ensuring the application stays available throughout the update. It's the default Kubernetes strategy and ideal for most production workloads.

#### Configuration:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1         # 1 extra pod above desired count during update
    maxUnavailable: 0   # No pods go down — zero downtime guaranteed
```

| Detail | Value |
|--------|-------|
| Image | `amitabhdevops/online_shop_without_footer` |
| Replicas | 4 |
| Service Type | NodePort |
| Node Port | 30000 |

#### Apply manifests:

```bash
kubectl apply -f rolling-update/rolling-namespace.yaml
kubectl apply -f rolling-update/rolling-update-deployment.yaml
kubectl apply -f rolling-update/rolling-update-svc.yaml
```

#### Verify:

```bash
kubectl get all -n rolling-ns
```

#### Simulate a rolling update:

```bash
kubectl set image deployment/online-shop online-shop=<new-image> -n rolling-ns
kubectl rollout status deployment/online-shop -n rolling-ns
```

#### Rollback if needed:

```bash
kubectl rollout undo deployment/online-shop -n rolling-ns
```

---

## ⚙️ Prerequisites

- Kubernetes cluster (Minikube, Kind, or cloud-based)
- `kubectl` configured and pointing to your cluster

```bash
kubectl version --client
kubectl cluster-info
```

---

## 📊 Strategy Comparison

| Feature | Blue-Green | Recreate | Rolling Update |
|---------|-----------|----------|----------------|
| Downtime | None | Brief downtime | None |
| Rollback speed | Instant (switch selector) | Slow (redeploy) | Moderate (`rollout undo`) |
| Resource usage | Double (two envs live) | Normal | Slightly elevated during update |
| Version coexistence | No (controlled switch) | No | Yes (briefly) |
| Complexity | Medium | Low | Low |
| Best for | Critical apps, A/B testing | Dev/test environments | Most production workloads |

---

## 🔍 Useful Commands

```bash
# Watch pods in real-time
kubectl get pods -n <namespace> -w

# Check deployment rollout history
kubectl rollout history deployment/<name> -n <namespace>

# Describe a deployment
kubectl describe deployment/<name> -n <namespace>

# Scale a deployment
kubectl scale deployment/<name> --replicas=<count> -n <namespace>

# Delete all resources in a namespace
kubectl delete all --all -n <namespace>
```

---

## 🤝 Contributing

Pull requests are welcome! If you'd like to add more deployment strategies (e.g., Canary, A/B testing), feel free to fork the repo and open a PR.

---

## 📄 License

This project is open source and available for educational purposes.
