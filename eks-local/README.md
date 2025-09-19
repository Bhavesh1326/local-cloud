# Local Kubernetes Development Environment

This project sets up a local Kubernetes environment with GitOps workflows using Argo CD, service mesh with Istio, and various security and observability tools.

## Prerequisites

1. Docker Desktop (with Kubernetes disabled)
2. kubectl
3. Helm
4. Kind (Kubernetes IN Docker)
5. Git

## Setup Steps

### 1. Install Required Tools

#### Install kubectl
```bash
# For Windows using Chocolatey
choco install kubernetes-cli
```

#### Install Helm
```bash
# For Windows using Chocolatey
choco install kubernetes-helm
```

#### Install Kind
```bash
# For Windows using Chocolatey
choco install kind
```

### 2. Create Kind Cluster

```bash
# Create a cluster configuration file
@'
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
  - containerPort: 30000-32767
    hostPort: 30000-32767
    protocol: TCP
'@ | Out-File -FilePath kind-config.yaml -Encoding UTF8

# Create the cluster
kind create cluster --name eks-local --config kind-config.yaml

# Set kubectl context
kubectl cluster-info --context kind-eks-local
```

### 3. Install Argo CD

```bash
# Create argocd namespace
kubectl create namespace argocd

# Install Argo CD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Install Argo CD CLI (Windows)
choco install argocd

# Wait for all pods to be running
kubectl wait --for=condition=Ready pods --all -n argocd --timeout=300s

# Get initial admin password
$argocdServer = kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | % { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }
Write-Host "Argo CD admin password: $argocdServer"

# Port forward to access Argo CD UI
Start-Process "http://localhost:8080"
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### 4. Install Istio

```bash
# Download Istio
$ISTIO_VERSION=1.16.1
curl -L https://istio.io/downloadIstio | sh -
cd istio-*

# Add istioctl to PATH
$env:PATH += ";$pwd\bin"

# Install Istio with demo profile
istioctl install --set profile=demo -y

# Enable Istio injection for default namespace
kubectl label namespace default istio-injection=enabled

# Verify installation
kubectl get svc -n istio-system
kubectl get pods -n istio-system
```

### 5. Install Observability Stack

#### Install Prometheus and Grafana
```bash
# Add Prometheus community Helm charts
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install kube-prometheus-stack
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring --create-namespace

# Port forward Grafana
kubectl port-forward -n monitoring svc/kube-prometheus-stack-grafana 3000:80
```

### 6. Install Kyverno

```bash
# Install Kyverno
kubectl create -f https://github.com/kyverno/kyverno/releases/download/v1.9.0/install.yaml

# Create a sample policy
@'
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-labels
  annotations:
    policies.kyverno.io/title: Require Labels
    policies.kyverno.io/category: Best Practices
    policies.kyverno.io/severity: medium
spec:
  validationFailureAction: enforce
  rules:
  - name: check-for-labels
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "The 'app' and 'version' labels are required."
      pattern:
        metadata:
          labels:
            app: "?*"
            version: "?*"
'@ | kubectl apply -f -
```

### 7. Install Falco

```bash
# Add Falcosecurity Helm charts
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update

# Install Falco
helm install falco falcosecurity/falco -n falco --create-namespace
```

### 8. Install Trivy

```bash
# Install Trivy using Chocolatey
choco install trivy

# Scan a local image
trivy image --security-checks vuln,config,secret nginx:latest
```

## Next Steps

1. Access Argo CD UI at https://localhost:8080
   - Username: admin
   - Password: (from the output of the `kubectl -n argocd get secret` command)

2. Create your first application in Argo CD

3. Deploy sample applications using the manifests in the `apps` directory

## Cleanup

To delete the Kind cluster:

```bash
kind delete cluster --name eks-local
```
