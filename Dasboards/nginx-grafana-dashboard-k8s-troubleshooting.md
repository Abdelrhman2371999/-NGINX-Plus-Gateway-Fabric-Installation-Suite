# NGINX Plus & Grafana Dashboards on Kubernetes
## Real-World Deployment & Troubleshooting Guide



## 1. Prerequisites

### Kubernetes
- Kubernetes Cluster (v1.26+)
- Container Runtime: containerd
- CNI: Calico
- At least 1 Worker Node

### Networking
- MetalLB installed and working
- IPAddressPool configured and reachable
- No firewall blocking:
  - 80 / 443
  - 7946 TCP/UDP (MetalLB)
  - NodePort range (30000–32767)

### NGINX
- NGINX Plus license (JWT + cert + key)
- private-registry.nginx.com access
- NGINX Plus API enabled

### Monitoring Stack
- kube-prometheus-stack (Prometheus + Grafana)
- ServiceMonitor enabled

---

## 2. Architecture (Final Working Design)

```

Browser
|
|  (MetalLB External IP)
v
NGINX Plus Dashboard (nginx:alpine)
|
|  /api/ proxy
v
NGINX Plus (Gateway / Ingress)
|
|  NGINX Plus API (:8765)
v
NGINX Plus Metrics

````

---

## 3. Problems Faced & Solutions

---

### ❌ Problem 1: port-forward is not stable

**Issue**
```bash
kubectl port-forward svc/monitoring-grafana 3000:80
````

* Shell must stay open
* Connection drops
* Not production-ready

**Solution**

* Use `NodePort` or `LoadBalancer`
* Prefer MetalLB with fixed IP

---

### ❌ Problem 2: No static IP for Grafana / Prometheus

**Issue**

* Grafana only accessible via port-forward
* No persistent IP

**Fix**

```bash
kubectl patch svc monitoring-grafana -n monitoring -p '{
  "spec": { "type": "NodePort" }
}'
```

or

```bash
type: LoadBalancer
```

MetalLB assigns external IP.

---

### ❌ Problem 3: NGINX Plus Dashboard pod CrashLoopBackOff

**Error**

```
nginx: [emerg] License file is required
```

**Root Cause**

* Used NGINX Plus image for dashboard
* Dashboard **does NOT need NGINX Plus**
* License not mounted

**Fix**

* Use `nginx:alpine` for dashboard
* Proxy API calls to real NGINX Plus

---

### ❌ Problem 4: 404 on /dashboard.html

**Root Cause**

* `dashboard.html` not mounted
* NGINX default root empty

**Fix**

* Mount dashboard.html via ConfigMap

```bash
kubectl create configmap nginx-plus-dashboard-html \
  --from-file=dashboard.html=nginx-endpoints-dashboard.html
```

Mount to:

```
/usr/share/nginx/html
```

---

### ❌ Problem 5: /api returns 404

**Reason**

* API socket is **UNIX socket**
* Not exposed on HTTP by default

**Fix**
NGINX Plus exposes API on **port 8765**

Verify:

```bash
wget -qO- http://127.0.0.1:8765/api/6/nginx
```

---

### ❌ Problem 6: Dashboard container cannot reach API

**Wrong**

```nginx
proxy_pass http://127.0.0.1/api/;
```

**Correct**

```nginx
proxy_pass http://nginx-nginx.default.svc.cluster.local:8765/api/;
```

---

### ❌ Problem 7: Exec confusion (kubectl inside pod)

**Issue**

```bash
kubectl exec ... kubectl ...
```

**Reality**

* kubectl exists **only on host**
* Not inside containers

**Correct**

```bash
kubectl exec -it pod -- /bin/sh
```

---

### ❌ Problem 8: apk / curl not working inside container

**Reason**

* Container runs as non-root

**Fix**

* Don’t install tools inside prod container
* Use `netshoot` for debugging

```bash
kubectl run netshoot --rm -it --image=nicolaka/netshoot
```

---

### ❌ Problem 9: Service has no endpoints

**Check**

```bash
kubectl get endpoints svc-name
```

**Root Cause**

* Pod crashing
* Wrong selector

---

### ❌ Problem 10: MetalLB IP reachable by ping but not HTTP

**Meaning**

* ARP OK
* Pod not ready or crashing
* No endpoints bound

---

## 4. Final Working NGINX Dashboard Config

```nginx
server {
  listen 80;

  location / {
    root /usr/share/nginx/html;
    index dashboard.html;
    try_files $uri $uri/ =404;
  }

  location /api/ {
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header Connection "";
    proxy_pass http://nginx-nginx.default.svc.cluster.local:8765/api/;
  }
}
```

---

## 5. Deployment Summary

| Component  | Image           | Purpose       |
| ---------- | --------------- | ------------- |
| NGINX Plus | nginx-plus-r36  | Traffic + API |
| Dashboard  | nginx:alpine    | UI only       |
| Grafana    | grafana/grafana | Visualization |
| Prometheus | prometheus      | Metrics       |

---

## 6. Validation Checklist

✅ Pod running
✅ Service endpoints exist
✅ API reachable internally
✅ Dashboard loads
✅ No port-forward
✅ Static IP via MetalLB

---

## 7. Final URLs

| Service              | URL                                |
| -------------------- | ---------------------------------- |
| NGINX Plus Dashboard | http://<METALLB-IP>/dashboard.html |
| NGINX Plus API       | http://<NGINX-PLUS-IP>:8765/api/   |
| Grafana              | http://<GRAFANA-IP>                |

---

## 8. Best Practices (Lessons Learned)

* ❌ Don’t use NGINX Plus for dashboard container
* ✅ Separate **UI** from **API**
* ✅ Always verify endpoints
* ✅ Use Service DNS not Pod IP
* ✅ Prefer LoadBalancer over NodePort
* ✅ Use netshoot for debugging
* ✅ Expect real-world issues, not docs-only flows

---

## 9. Next Improvements

* Secure API with allow/deny
* Put dashboard behind Ingress/Gateway
* Add auth to Grafana
* Export NGINX Plus metrics to Prometheus
* Add alerts

---

## Author Notes

This document reflects **real production-like troubleshooting**,
not ideal documentation examples.
