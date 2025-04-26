
# 📚 Kubernetes and KServe Concepts Summary

## 🧠 Kubernetes Basic Concepts

| Term | Short Explanation |
| :--- | :---------------- |
| **Pod** | Smallest unit in Kubernetes — a Pod runs one or more containers together. Think "a running thing" (like your model container). |
| **Node** | A machine (virtual or physical) that runs your Pods. In local setups like Docker Desktop Kubernetes, the Mac acts as a "single node". |
| **Cluster** | A group of nodes managed together by Kubernetes. Your "environment". |
| **Deployment** | A controller that ensures a specified number of Pods are running and updated. Example: "run 2 copies of my model API." |
| **Service** | A stable network endpoint to access Pods. Load-balances and exposes Pods internally or externally. |
| **Ingress** | Manages external HTTP/S access to Services (via LoadBalancer, NodePort, etc.). |
| **Namespace** | A logical separation inside a Kubernetes cluster. Like folders: you can put different projects in different namespaces. |
| **ConfigMap** | Injects non-sensitive configuration data into Pods. |
| **Secret** | Injects **sensitive** information (passwords, tokens, etc.) into Pods. |
| **PersistentVolumeClaim (PVC)** | Requests storage inside the cluster (used when data needs to persist beyond Pod lifetime). |
| **ReplicaSet** | Ensures a specified number of Pod replicas are running at all times (Deployments manage ReplicaSets). |
| **Container** | A Docker container or similar, running inside a Pod. |

---

## 🔥 KServe-Specific Concepts

| Term | Short Explanation |
| :--- | :---------------- |
| **InferenceService** | Custom resource (CRD) in KServe that defines model serving configuration (model type, URL, scaling). |
| **Predictor** | Serves the model — part of the InferenceService. |
| **Transformer** | (Optional) Pre-process or post-process requests before they reach the Predictor. |
| **Explainer** | (Optional) Attach an explainability service to your model (e.g., SHAP explanations). |
| **Istio** | Service mesh that handles routing, load balancing, security between Services. Required by KServe for ingress traffic. |
| **Knative** | Enables serverless scaling for Pods (scale to zero, then back up on demand). KServe uses Knative Serving underneath. |

---

## 📦 Helm Concepts

| Term | Short Explanation |
| :--- | :---------------- |
| **Helm** | The package manager for Kubernetes — automates installing, upgrading, deleting Kubernetes applications. |
| **Helm Chart** | A collection of YAML templates describing a Kubernetes application (Pods, Services, ConfigMaps, etc.) in a reusable, parameterized way. |
| **Release** | An instance of a Helm Chart installed in a Kubernetes cluster. |
| **Repository** | A place to host Helm charts online (similar to a Python package repository like PyPI). |

---

## 🚀 Typical Kubernetes Deployment Flow (without Helm)

```text
1. Write YAML files manually (Deployment, Service, etc.)
2. Apply them with kubectl apply -f <file>.yaml
3. Manage upgrades, rollbacks manually
```

---

## 🚀 Typical Deployment Flow Using Helm

```text
1. Add Helm repository
2. Pull Helm chart
3. Customize parameters via values.yaml
4. Install chart with helm install
5. Manage upgrade/downgrade easily with helm upgrade or helm rollback
```

Example:

```bash
helm repo add kserve https://kserve.github.io/helm-charts
helm repo update
helm install kserve kserve/kserve --namespace kserve --create-namespace
```

---

## 🎯 Visual Map of Kubernetes Objects

```text
Cluster
 ├── Node
 │    └── Pod (container running your ML model)
 ├── Deployment (manages Pods)
 ├── Service (exposes Pods internally or externally)
 ├── Ingress (exposes Service externally via HTTP/S)
 ├── Namespace (groups all resources logically)
 └── Custom Resources (like InferenceService for KServe)
```

---

# 📜 Final Notes

- **Pods** are the basic runnable units.
- **Deployments** manage the desired state (replicas, updates).
- **Services** expose Pods to the network.
- **Ingress** allows external HTTP access.
- **Helm** simplifies complex deployments.
- **KServe** uses **InferenceService** to handle model serving.
- **Istio** and **Knative** are required under the hood for smart routing and scaling.

---

# ✅ Core Commands to Remember

```bash
kubectl get pods -n <namespace>
kubectl get services -n <namespace>
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace>
kubectl apply -f <file>.yaml

helm install <release-name> <chart-name> [options]
helm upgrade <release-name> <chart-name> [options]
helm uninstall <release-name>
```

---

# 🚀 Ready to Deploy ML Models!
