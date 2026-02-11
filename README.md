# ☸️ Advanced WordPress Deployment on Kubernetes (Helm)
**High-Availability WordPress + MariaDB Stateful Persistence + Prometheus/Grafana Monitoring**

This project packages a full **2-tier WordPress stack** into a **single Helm chart** and deploys it into a Kubernetes cluster with:
- **High availability** for WordPress (2 replicas)
- **Persistent, stable MariaDB** using a StatefulSet + PVC
- **Ingress-NGINX** as the external entry point (API Gateway)
- **Full observability** via **kube-prometheus-stack** (Prometheus + Grafana)
- A **custom Grafana panel** that shows container up/down health as a steady line at `1`

---

## 📌 Architecture Overview (2-Tier Web App)
### ✅ Frontend: WordPress (Deployment)
- Runs as a **Deployment**
- **2 replicas** for high availability and load balancing
- Stateless by design (easy to scale horizontally)

### ✅ Backend: MariaDB (StatefulSet)
MariaDB is deployed as a **StatefulSet** (not a Deployment) because databases need:
- **Stable network identity** (pod names like `mariadb-0`)
- **Persistent storage** that survives pod restarts
- **Predictable ordering/rollouts** when scaling or updating

### ✅ Networking: Ingress-NGINX (Gateway)
- Cluster stays private
- Only required ports are exposed through the Ingress Controller
- Ingress routes HTTP traffic to WordPress service inside the cluster

### ✅ Observability: Prometheus + Grafana
- Monitoring is provided via **kube-prometheus-stack**
- Dashboards show:
  - Pod/container health
  - Uptime / restarts
  - Cluster resource status

---

## 📂 Helm Chart Structure
This repo converts raw Kubernetes YAML into a clean Helm layout:
wordpress-k8s/
├─ Chart.yaml
├─ values.yaml
└─ templates/
├─ wordpress-deployment.yaml
├─ wordpress-service.yaml
├─ mariadb-statefulset.yaml
├─ mariadb-service.yaml
└─ ingress.yaml


✅ **Why Helm helps:** you manage *one* `values.yaml` instead of editing 10+ YAML files by hand.  
Templates provide reuse, consistency, and repeatable deployments.

---

## ✅ Prerequisites
Make sure you have:

- A Kubernetes cluster (Minikube / Kind / EKS / AKS / GKE)
- **Helm v3+**
- **Ingress-NGINX controller installed**
- Access to the cluster (kubectl configured)

### Install Ingress-NGINX (example)
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```
🚀 Installation (Single Command Helm Deploy)
```
# Clone repo
git clone https://github.com/Hi9841/wordpressk8spr.git
cd wordpressk8spr/wordpress-k8s

# Install chart
helm install my-wordpress ./
```
✅ Verify resources
```
kubectl get pods
kubectl get svc
kubectl get ingress
```
🌐 Accessing Services (Private EC2 / No Public LB)
Because this lab runs on a private EC2 instance, access is done via SSH Port Forwarding.
✅ WordPress (Ingress → Localhost:8080)
Forward port 80 from the ingress controller to your local machine:
```
kubectl -n ingress-nginx port-forward svc/ingress-nginx-controller 8080:80
```
[localhost:8080](http://localhost:8080)

✅ Grafana (Localhost:3000)
If kube-prometheus-stack is installed in monitoring namespace:
```
kubectl -n monitoring port-forward svc/kube-prometheus-stack-grafana 3000:80
```
[localhost:3000](http://localhost:3000)

Grafana creds stored in k8s secret:
```
kubectl -n monitoring get secret kube-prometheus-stack-grafana \
  -o jsonpath="{.data.admin-password}" | base64 --decode; echo
```

🧩 Deployment Flow: How Helm Simplifies YAML Management

Without Helm:
You manually apply many manifests in the correct order
Values (passwords, service names, images) are duplicated everywhere

With Helm:
Templates hold the logic
values.yaml holds configuration
Deployment becomes repeatable:
```
helm install my-wordpress ./
helm upgrade my-wordpress ./
helm uninstall my-wordpress
```

🧪 Testing & Validation Checklist
✅ WordPress has 2 replicas:
```
kubectl get deploy wordpress
kubectl get pods -l app=wordpress
```

✅ MariaDB is a StatefulSet:
```
kubectl get statefulset
```

✅ Ingress routing works:
```
kubectl get ingress
```

✅ Monitoring works:
Prometheus targets are up
Grafana dashboards load
Container health panel stays at 1


📎 Notes / Troubleshooting
WordPress not loading?
Make sure ingress-nginx is running:
```
kubectl -n ingress-nginx get pods
```
Grafana not accessible?
Ensure monitoring stack is installed and services exist:
```
kubectl -n monitoring get svc
```

🏁 Summary

This project demonstrates a real-world Kubernetes pattern:
Stateless app scaled via Deployment
Stateful DB managed via StatefulSet + persistence
Traffic gateway via Ingress
Production-grade observability via Prometheus & Grafana


