تمام 
`## Before Migration – NGINX Ingress Controller`

---

```md
## Before Migration – NGINX Ingress Controller

### 1. Overview

Before migrating to **NGINX Gateway Fabric**, the cluster was using the
**NGINX Ingress Controller** to expose applications externally.

Traffic routing was defined using **Ingress resources**, and advanced behavior
was controlled mainly through **Ingress annotations**.

---

## 2. Architecture (Before Migration)

### Components Involved
- Client (Browser / API Consumer)
- MetalLB (External IP provider)
- Kubernetes Service (LoadBalancer)
- NGINX Ingress Controller
- Ingress resources
- Backend Kubernetes Service
- Application Pods

---

## 3. Traffic Flow (Ingress-Based)

### Logical Flow
```

Client
↓
MetalLB External IP
↓
Service (LoadBalancer)
↓
NGINX Ingress Controller Pod
↓
Ingress Rules (Host / Path)
↓
Backend Service
↓
Application Pods

````

---

## 4. Where Each Component Lived

### Ingress Controller
- **Namespace:** ingress-nginx
- **Deployment:** ingress-nginx-controller
- **Service:** ingress-nginx-controller (LoadBalancer)
- **Role:**
  - Terminates HTTP connections
  - Parses Ingress rules
  - Routes traffic to backend services

---

### Ingress Resource
- **Namespace:** default
- **Kind:** Ingress
- **Role:**
  - Defines host and path rules
  - References backend services
  - Uses annotations for advanced features

Example:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web
  namespace: default
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
            name: web-svc
            port:
              number: 80
````

---

## 5. How Traffic Was Routed

1. Client sends request:

   ```http
   GET / HTTP/1.1
   Host: test.local
   ```

2. MetalLB assigns an External IP to the Ingress Controller Service

3. Traffic reaches:

   * `ingress-nginx-controller` Service
   * Forwarded to Ingress Controller Pod

4. Ingress Controller:

   * Matches `Host` and `Path`
   * Applies annotations (rewrite, headers, limits, etc.)
   * Proxies traffic to backend service

5. Backend Service load-balances traffic to application pods

---

## 6. Namespace Visibility (Ingress Model)

* Ingress and backend Service usually existed in the **same namespace**
* Cross-namespace routing was:

  * Limited
  * Non-standard
  * Controller-specific
* No native API for multi-team ownership

---

## 7. Traffic Tracking (Ingress Model)

### External Entry

```bash
kubectl get svc -n ingress-nginx
```

### Ingress Resource Status

```bash
kubectl describe ingress -n default web
```

### Ingress Controller Logs

```bash
kubectl logs -n ingress-nginx deploy/ingress-nginx-controller
```

### Backend Service

```bash
kubectl get endpoints web-svc -n default
```

---

## 8. Limitations of the Ingress-Based Model

* Heavy reliance on annotations
* No clear separation between:

  * Infrastructure owners
  * Application owners
* Limited extensibility
* No standardized policy model
* Harder to manage at scale

---

## 9. Migration Motivation

Due to these limitations, the cluster was migrated to:

* **Gateway API**
* **NGINX Gateway Fabric**

To achieve:

* Clear separation of concerns
* Standardized APIs
* Better scalability
* Future-proof traffic management
