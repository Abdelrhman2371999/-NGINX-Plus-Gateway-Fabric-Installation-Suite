# NGINX Plus & Grafana Dashboards on Kubernetes

## Overview

This document describes **how to deploy and expose**:

* **NGINX Plus Dashboard**
* **Grafana Dashboard**

inside a Kubernetes cluster with a **stable IP**, using **MetalLB**, and how to connect NGINX Plus API metrics to dashboards.

---

## Prerequisites

### Cluster & Networking

* Kubernetes cluster (v1.24+ recommended)
* CNI plugin installed (Calico / Flannel / Cilium)
* Working DNS inside cluster (CoreDNS)

### Load Balancer

* MetalLB installed and configured
* IP pools defined and reachable from your network

### NGINX Plus

* Valid **NGINX Plus license**
* Access to `private-registry.nginx.com`
* Docker registry secret created in Kubernetes
* License files available:

  * `*.crt`
  * `*.key`
  * `*.jwt`

### Monitoring Stack

* Prometheus installed (Helm or kube-prometheus-stack)
* Grafana installed

### CLI Tools

* kubectl
* curl

---

## Architecture

```
Browser
  |
  |  (HTTP / LoadBalancer IP)
  v
NGINX Plus Dashboard (Pod)
  |
  |  /api/* (proxy)
  v
NGINX Plus API (unix socket / TCP)

Prometheus  --->  Grafana
```

---

## Part 1: Deploy NGINX Plus

### 1. Create Registry Secret

```bash
kubectl create secret docker-registry nginx-plus-registry-secret \
  --docker-server=private-registry.nginx.com \
  --docker-username=<JWT> \
  --docker-password=<JWT> \
  --namespace=default
```

### 2. Create License Secret

```bash
kubectl create secret generic nplus-license \
  --from-file=license.jwt \
  --from-file=license.crt \
  --from-file=license.key \
  --namespace=default
```

### 3. Deploy NGINX Plus

* Enable **NGINX Plus API**
* Expose API on internal port (example: `8765`)

```nginx
location /api {
  api write=on;
}
```

Verify:

```bash
curl http://127.0.0.1:8765/api/6/nginx
```

---

## Part 2: NGINX Plus Dashboard

### 1. Prepare Dashboard HTML

Download or create dashboard file:

```bash
wget https://raw.githubusercontent.com/nginxinc/nginx-plus-dashboard/master/dashboard.html
```

### 2. Create ConfigMap for HTML

```bash
kubectl create configmap nginx-plus-dashboard-html \
  --from-file=dashboard.html=/full/path/dashboard.html \
  -n default
```

### 3. Create NGINX Dashboard Config

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-plus-dashboard-conf
  namespace: default
data:
  default.conf: |
    server {
      listen 80;

      location / {
        root /usr/share/nginx/html;
        index dashboard.html;
      }

      location /api/ {
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_pass http://nginx-nginx.default.svc.cluster.local:8765/api/;
      }
    }
```

### 4. Deployment for Dashboard

Mount:

* Dashboard HTML
* NGINX config

```yaml
volumeMounts:
- name: dashboard-html
  mountPath: /usr/share/nginx/html/dashboard.html
  subPath: dashboard.html

- name: dashboard-conf
  mountPath: /etc/nginx/conf.d
```

---

## Part 3: Expose Dashboard (Static IP)

### Option A: LoadBalancer (Recommended)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-plus-dashboard-svc
spec:
  type: LoadBalancer
  selector:
    app: nginx-plus-dashboard
  ports:
  - port: 80
    targetPort: 80
```

MetalLB assigns IP:

```bash
kubectl get svc nginx-plus-dashboard-svc -o wide
```

Access:

```text
http://<EXTERNAL-IP>/dashboard.html
```

---

## Part 4: Grafana Dashboard

### 1. Expose Grafana

```bash
kubectl -n monitoring patch svc monitoring-grafana -p '{
  "spec": { "type": "LoadBalancer" }
}'
```

### 2. Get Grafana IP

```bash
kubectl -n monitoring get svc monitoring-grafana -o wide
```

### 3. Login

* Default user: `admin`
* Password from secret:

```bash
kubectl get secret -n monitoring monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 -d
```

---

## Part 5: Connect Prometheus to Grafana

1. Grafana → Settings → Data Sources
2. Add **Prometheus**
3. URL:

```text
http://prometheus-operated.monitoring.svc.cluster.local:9090
```

4. Save & Test

---

## Validation Checklist

* [ ] NGINX Plus Pod running
* [ ] `/api/6/nginx` returns JSON
* [ ] Dashboard pod running
* [ ] Dashboard HTML accessible
* [ ] LoadBalancer IP reachable
* [ ] Prometheus scraping targets
* [ ] Grafana dashboards loading

---

## Common Issues

* **CrashLoopBackOff** → License not mounted
* **404 dashboard.html** → ConfigMap not mounted
* **Empty dashboard** → API path incorrect
* **No external IP** → MetalLB pool issue

---

## Final URLs

* NGINX Plus Dashboard:

```text
http://<LB-IP>/dashboard.html
```

* Grafana:

```text
http://<LB-IP>:3000
```

---

## End of Document
