
# NGINX Ingress Controller on Kubernetes (Bare-Metal Cluster)

## Overview
This document explains how to deploy and use **NGINX Ingress Controller (Community Edition)** on a Kubernetes cluster with:
- 1 Control Plane (Master)
- 2 Worker Nodes
- Bare-metal environment (no cloud LoadBalancer)
- Optional MetalLB for external IPs

The guide covers:
- Ingress concepts
- Installing NGINX Ingress
- Exposing it (hostNetwork or MetalLB)
- Deploying an application
- Load balancing traffic between worker nodes


## Prerequisites
- Kubernetes cluster is running
- `kubectl` configured
- Cluster networking (CNI) is working
- (Optional) MetalLB installed for LoadBalancer services

Check cluster:
```bash
kubectl get nodes
````

---

## What is Ingress?

Ingress is a Kubernetes API object that manages **external HTTP/HTTPS access** to services inside the cluster.

Ingress provides:

* Host-based routing (`example.com`)
* Path-based routing (`/api`, `/app`)
* TLS termination
* Load balancing to backend Pods

Ingress requires an **Ingress Controller** (NGINX in this case).

---

## Step 1: Install NGINX Ingress Controller (Bare-Metal)

Install the official community Ingress:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/baremetal/deploy.yaml
```

Verify:

```bash
kubectl get pods -n ingress-nginx
```

Expected:

* ingress-nginx-controller → Running

---

## Step 2 (Option A): Expose Ingress Using hostNetwork

Useful when:

* No LoadBalancer available
* You want Ingress to listen directly on node IPs (80/443)

Patch the controller:

```bash
kubectl -n ingress-nginx patch deployment ingress-nginx-controller \
  --type='json' \
  -p='[
    {"op":"add","path":"/spec/template/spec/hostNetwork","value":true},
    {"op":"add","path":"/spec/template/spec/dnsPolicy","value":"ClusterFirstWithHostNet"}
  ]'
```

Result:

* Ingress listens on **worker node IPs**
* Access via: `http://<worker-ip>`

---

## Step 2 (Option B): Expose Ingress Using MetalLB (Recommended)

If MetalLB is installed, change service type:

```bash
kubectl -n ingress-nginx patch svc ingress-nginx-controller \
  -p '{"spec":{"type":"LoadBalancer"}}'
```

Check external IP:

```bash
kubectl -n ingress-nginx get svc ingress-nginx-controller
```

Example:

```
EXTERNAL-IP: 10.10.0.241
```

This IP becomes your **Ingress VIP**.

---

## Step 3: Deploy a Sample Application

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: test-app
  template:
    metadata:
      labels:
        app: test-app
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

Apply:

```bash
kubectl apply -f test-app-deploy.yaml
```

---

## Step 4: Create a Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: test-app-service
spec:
  selector:
    app: test-app
  ports:
  - port: 80
    targetPort: 80
```

Apply:

```bash
kubectl apply -f test-app-svc.yaml
```

Verify endpoints:

```bash
kubectl get endpoints test-app-service
```

---

## Step 5: Create Ingress Resource

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: test.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: test-app-service
            port:
              number: 80
```

Apply:

```bash
kubectl apply -f test-ingress.yaml
```

Verify:

```bash
kubectl get ingress
```

---

## Step 6: DNS / Hosts Configuration (Local Test)

On your client machine:

```txt
10.10.0.241   test.local
```

(Use worker IP if using hostNetwork)

---

## Step 7: Test Access

```bash
curl -H "Host: test.local" http://10.10.0.241
```

Or from browser:

```
http://test.local
```

---

## Load Balancing Behavior

### Backend Load Balancing

* Kubernetes Service distributes traffic across Pods
* Pods may run on different worker nodes

### Ingress High Availability

* Multiple ingress-nginx replicas can run
* MetalLB provides a floating VIP
* Traffic is balanced across worker nodes

Scale ingress:

```bash
kubectl -n ingress-nginx scale deployment ingress-nginx-controller --replicas=2
```

---

## Common Issues & Solutions

### 404 Not Found from Ingress

Cause:

* Host header does not match Ingress rule

Fix:

```bash
curl -H "Host: test.local" http://<ingress-ip>
```

---

### MetalLB memberlist timeout

Cause:

* Port `7946/tcp` and `7946/udp` blocked between nodes

Fix:

* Open ports on all nodes
* Check connectivity between workers

---

## Summary

* Ingress provides HTTP routing into Kubernetes
* NGINX Ingress is production-ready and widely used
* MetalLB enables LoadBalancer on bare-metal clusters
* Services + Ingress = scalable and resilient traffic routing

---

## References

* [https://kubernetes.io/docs/concepts/services-networking/ingress/](https://kubernetes.io/docs/concepts/services-networking/ingress/)
* [https://kubernetes.github.io/ingress-nginx/](https://kubernetes.github.io/ingress-nginx/)
* [https://metallb.universe.tf/](https://metallb.universe.tf/)

```
