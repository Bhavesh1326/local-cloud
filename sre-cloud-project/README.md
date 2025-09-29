# Secure, Observable, and Scalable Kubernetes Platform with GitOps

## Case Study Objective
Showcase a secure, observable, and scalable Kubernetes platform running locally, using GitOps with Argo CD, Istio service mesh, an observability stack, and open-source container security tools for policy enforcement and runtime visibility.

---

## 1. Pre-checks
- Ensure Docker is running and clean:
  ```powershell
  docker ps -a
  docker system prune -af
  netstat -ano | findstr :80
  ```
- Delete any old local Kubernetes clusters:
  ```powershell
  minikube delete
  kind delete cluster --name sre-cloud-project
  k3d cluster delete sre-cloud-project
  ```

---

## 2. Local Kubernetes Cluster Setup
- Create a new Kind cluster:
  ```powershell
  kind create cluster --name sre-cloud-project
  ```
- Create namespaces:
  ```powershell
  kubectl create namespace argocd
  kubectl create namespace istio-system
  kubectl create namespace monitoring
  kubectl create namespace microservices
  ```
- Set up RBAC (example for Argo CD):
  ```powershell
  kubectl create serviceaccount sre-admin -n argocd
  kubectl create clusterrolebinding sre-admin-binding --clusterrole=cluster-admin --serviceaccount=argocd:sre-admin
  ```

---

## 3. Install & Configure Argo CD
- Install Argo CD:
  ```powershell
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  kubectl rollout status deployment/argocd-server -n argocd
  ```
- Port-forward Argo CD UI:
  ```powershell
  kubectl port-forward svc/argocd-server -n argocd 8080:443
  ```
- Get initial admin password:
  ```powershell
  kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
  ```
- Login at [http://localhost:8080](http://localhost:8080) (user: admin, password: above)

---

## 4. GitOps Applications
- Argo CD Applications are in `infrastructure/argocd/applications/`:
  - `istio.yaml` (Istio service mesh)
  - `monitoring.yaml` (Prometheus, Grafana, Loki, Tempo)
  - `microservices.yaml` (Go microservices)
- Sync applications via Argo CD UI or CLI:
  ```powershell
  kubectl apply -f infrastructure/argocd/applications/istio.yaml
  kubectl apply -f infrastructure/argocd/applications/monitoring.yaml
  kubectl apply -f infrastructure/argocd/applications/microservices.yaml
  ```

---

## 5. Microservices Deployment
- Each microservice is in `apps/app1`, `apps/app2`, `apps/app3`:
  - `main.go`, `Dockerfile`, `k8s/deployment.yaml`, `service.yaml`, `hpa.yaml`, `istio-virtualservice.yaml`
- Build and push Docker images:
  ```powershell
  cd apps/app1
  docker build -t bhavesh1326/app1:latest .
  docker push bhavesh1326/app1:latest
  cd ../app2
  docker build -t bhavesh1326/app2:latest .
  docker push bhavesh1326/app2:latest
  cd ../app3
  docker build -t bhavesh1326/app3:latest .
  docker push bhavesh1326/app3:latest
  ```
- Argo CD will deploy microservices automatically.

---

## 6. Observability & Dashboards
- Port-forward Grafana:
  ```powershell
  kubectl port-forward svc/monitoring-stack-grafana -n monitoring 3000:80
  ```
  Access Grafana at [http://localhost:3000](http://localhost:3000) (user: admin, password: prom-operator)
- Port-forward Jaeger/Tempo:
  ```powershell
  kubectl port-forward svc/jaeger-query -n monitoring 16686:16686
  ```
  Access Jaeger at [http://localhost:16686](http://localhost:16686)
- Dashboards and datasources for Istio, microservices, Prometheus, Loki, etc. will be auto-configured.

---

## 7. Security & Policy Enforcement
- Network policies and OPA/Gatekeeper manifests can be added in `infrastructure/security/`.
- Example network policy:
  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  ...
  ```

---

## 8. Re-run & Troubleshooting
- To re-run, repeat pre-checks and setup steps.
- Use Argo CD UI to monitor sync and health.
- Use `kubectl get pods -A` to check all deployments.
- Use port-forwarding for dashboard access.

---

## 9. Project Structure
```
infrastructure/
  argocd/
    applications/
    projects/
  istio/
  monitoring/
  security/
apps/
  app1/
  app2/
  app3/
README.md
```

---

## 10. Notes
- All steps use port-forwarding for local access.
- All manifests and code are GitOps-managed for reproducibility.
- For any issues, check Argo CD UI, pod logs, and dashboard health.