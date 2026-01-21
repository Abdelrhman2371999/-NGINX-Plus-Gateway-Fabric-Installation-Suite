
# NGINX Gateway Fabric – Traffic Flow & Tracking Guide

## 1. Overview

This document explains how traffic flows **from outside the Kubernetes cluster**
into applications after migrating from **NGINX Ingress Controller**
to **NGINX Gateway Fabric** using **Gateway API**.

It also provides **practical methods to track and troubleshoot traffic**
at every stage of the flow.


## 2. High-Level Architecture

### Components Involved
- Client (Browser / API Consumer)
- MetalLB (External IP provider)
- Kubernetes Service (LoadBalancer)
- NGINX Gateway Fabric (Data Plane)
- Gateway API objects (Gateway, HTTPRoute)
- Backend Kubernetes Service
- Application Pods

---

## 3. Traffic Flow (From Outside to Inside the Cluster)

### Logical Flow
```

Client
↓
MetalLB External IP
↓
Service (LoadBalancer)
↓
NGINX Gateway Fabric Data Plane
↓
Gateway + HTTPRoute (Routing Logic)
↓
Backend Service
↓
Application Pods

````

---

## 4. Where Each Component Lives (Namespaces)

### Control Plane
- **Namespace:** nginx-gateway
- **Pod:** ngf-nginx-gateway-fabric
- **Role:** 
  - Watches Gateway API objects
  - Generates NGINX configuration
  - Does NOT handle user traffic

---

### Data Plane
- **Namespace:** default
- **Deployment:** nginx-nginx
- **Service:** nginx-nginx (LoadBalancer)
- **Role:**
  - Receives external traffic
  - Applies routing rules
  - Proxies requests to backends

---

### Application
- **Namespace:** default
- **Service:** web-svc
- **Pods:** application pods

---

## 5. Gateway API Object Placement

### Gateway
- Logical routing entry point
- Not a Pod or Service
- Exists as a Kubernetes object

Example:
```yaml
kind: Gateway
metadata:
  name: nginx
  namespace: default
````

---

### HTTPRoute

* Defines routing rules
* Matches host and path
* Points to backend services

Example:

```yaml
kind: HTTPRoute
spec:
  hostnames:
  - test.local
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: web-svc
      port: 80
```

---

## 6. How Namespaces See Each Other

### Important Rule

> Kubernetes namespaces are **not network firewalls**.
> They are logical boundaries for organization.

### Same Namespace

* Gateway, HTTPRoute, Service in same namespace
* No extra configuration needed

### Cross-Namespace Routing

* HTTPRoute can reference a Service in another namespace
* Requires:

  * Explicit `namespace` field
  * `ReferenceGrant` for security

Example:

```yaml
backendRefs:
- name: web-svc
  namespace: test
  port: 80
```

---

## 7. Tracking Traffic (End-to-End)

This section explains how to **observe and debug traffic** at every hop.

---

### 7.1 Track External Entry (MetalLB)

Check LoadBalancer Service:

```bash
kubectl get svc -n default
```

Confirm External IP:

```text
nginx-nginx   LoadBalancer   ...   10.10.0.242
```

Test from client:

```bash
curl -I http://10.10.0.242 -H "Host: test.local"
```

---

### 7.2 Track Gateway Status

Verify Gateway is programmed:

```bash
kubectl get gateway -n default
kubectl describe gateway -n default nginx
```

Expected:

* PROGRAMMED = True

---

### 7.3 Track HTTPRoute Attachment

```bash
kubectl describe httproute -n default web-test-local
```

Expected:

* Accepted = True
* ResolvedRefs = True
* Controller: nginx-gateway-controller

---

### 7.4 Track Data Plane Traffic (NGINX)

Check dataplane pod:

```bash
kubectl get pods -n default | grep nginx-nginx
```

View access logs:

```bash
kubectl logs -n default deploy/nginx-nginx
```

You should see:

* Client IP
* Requested Host
* Requested Path
* Response status

---

### 7.5 Track Backend Service

Confirm service endpoints:

```bash
kubectl get endpoints web-svc -n default
```

Expected:

* One or more Pod IPs listed

---

### 7.6 Track Application Pods

Check pod logs:

```bash
kubectl logs -n default <app-pod-name>
```

This confirms traffic reached the application.

---

## 8. Common Traffic Issues & How to Debug

### Issue: No response from External IP

* Check Service type = LoadBalancer
* Check MetalLB address pool
* Check dataplane pod is running

---

### Issue: 404 from Gateway

* Check HTTPRoute host/path
* Verify Host header in request

---

### Issue: Route Accepted but no traffic

* Check backend service name/port
* Check service endpoints

---

## 9. Verification Checklist

* [ ] Gateway PROGRAMMED = True
* [ ] HTTPRoute Accepted & ResolvedRefs = True
* [ ] Dataplane Service has External IP
* [ ] curl with Host header works
* [ ] Backend pods receive traffic

---

## 10. Summary

* Traffic enters the cluster via **MetalLB**
* LoadBalancer Service forwards traffic to **NGINX Gateway Fabric dataplane**
* Routing decisions are made using **Gateway API**
* Backend services receive traffic using standard Kubernetes networking
* Traffic can be traced at every layer using kubectl and logs

---

## 11. Conclusion

This setup provides:

* Clear separation of control plane and data plane
* Standardized traffic management using Gateway API
* Improved observability and extensibility compared to legacy Ingress

The cluster is now fully migrated and traffic is successfully handled
by **NGINX Gateway Fabric**.
