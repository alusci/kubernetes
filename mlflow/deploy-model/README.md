# MLflow Model Deployment with KServe and Kubernetes

## 1. Create Environment

```bash
conda create -n mlflow-py310 python=3.10 ipython jupyter
```

## 2. Install Python Dependencies

```bash
pip install mlflow[extras] mlserver mlserver-mlflow
```

## 3. Install Pyenv (Optional)

Refer to the [Pyenv Installation Guide](https://github.com/pyenv/pyenv#installation)

## 4. Train Model

```bash
python model_training.py
```

## 5. Run MLflow UI (Pick Desired Run-ID)

```bash
mlflow ui --port 5000
```

## 6. Test Compatibility with KServe

```bash
mlflow models serve -m runs:/{your-run-id}/model -p 1234 --enable-mlserver
```

## 7. Test Predictions Locally

```bash
curl -X POST -H "Content-Type:application/json" --data '{"inputs": [[14.23, 1.71, 2.43, 15.6, 127.0, 2.8, 3.06, 0.28, 2.29, 5.64, 1.04, 3.92, 1065.0]]}' http://127.0.0.1:1234/invocations
```

---

# Install Kubernetes Locally

## Ubuntu

```bash
# Install kind
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64
sudo mv kind /usr/bin
sudo chmod +x /usr/bin/kind

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Fix Docker socket permissions (if needed)
sudo chmod 666 /var/run/docker.sock

# Useful references
https://kind.sigs.k8s.io/docs/user/quick-start
https://kserve.github.io/website/latest/get_started/
```

## MacOS

```bash
# Install via Homebrew
brew install kubectl
brew install helm
brew install kind

# Create namespace for KServe
kubectl create namespace kserve

# Follow official KServe instructions
https://kserve.github.io/website/latest/get_started/#install-the-kserve-quickstart-environment
```

---

# Build and Deploy the Model

## 1. Build Docker Image

> **Important:** Perform this on a Linux system to avoid architecture compatibility issues.

```bash
mlflow models build-docker -m runs:/b9e3d317bee3400db2158618cf880c8f/model -n alusci/mlflow-wine-classifier --enable-mlserver
```

## 2. Push Image to Docker Hub

```bash
docker push alusci/mlflow-wine-classifier
```

## 3. Create Namespace for Deployment

```bash
kubectl create namespace mlflow-kserve-test
```

## 4. Deploy Model to KServe

```bash
kubectl apply -f model-deployment.yaml
```

## 5. Check InferenceService Status

```bash
kubectl get inferenceservice mlflow-wine-classifier -n mlflow-kserve-test
```

## 6. Test the Prediction System

```bash
# Port-forward Istio Ingress Gateway
INGRESS_GATEWAY_SERVICE=$(kubectl get svc -n istio-system --selector="app=istio-ingressgateway" -o jsonpath='{.items[0].metadata.name}')
kubectl port-forward -n istio-system svc/${INGRESS_GATEWAY_SERVICE} 8080:80

# Retrieve Service Hostname
SERVICE_HOSTNAME=$(kubectl get inferenceservice mlflow-wine-classifier -n mlflow-kserve-test -o jsonpath='{.status.url}' | cut -d "/" -f 3)

# Send Prediction Request
curl -v -H "Host: ${SERVICE_HOSTNAME}" -H "Content-Type: application/json" -d @./test-input.json http://localhost:8080/v2/models/mlflow-model/infer
```

---

# Common Kubernetes Debugging Commands

## Pods and Services

```bash
kubectl get pods -n mlflow-kserve-test
kubectl get services -n mlflow-kserve-test
```

## InferenceService Status

```bash
kubectl get inferenceservices -n mlflow-kserve-test
kubectl describe inferenceservice mlflow-wine-classifier -n mlflow-kserve-test
```

## Logs

```bash
kubectl logs -n mlflow-kserve-test -l serving.kserve.io/inferenceservice=mlflow-wine-classifier
```

## Events and Troubleshooting

```bash
kubectl describe pod <pod-name> -n mlflow-kserve-test
```

## Test Istio Routing

```bash
curl -v -H "Host: ${SERVICE_HOSTNAME}" http://localhost:8080/v2/models
```

## Delete and Retry

```bash
kubectl delete inferenceservice mlflow-wine-classifier -n mlflow-kserve-test
kubectl apply -f model-deployment.yaml
```

---

# References
- [KServe Official Documentation](https://kserve.github.io/website/latest/)
- [Kind Quickstart](https://kind.sigs.k8s.io/docs/user/quick-start/)
- [MLflow Documentation](https://mlflow.org/docs/latest/index.html)

