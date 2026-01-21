````md
# Migration from NGINX Ingress Controller to NGINX Gateway Fabric

## 1. Overview

This document describes the migration process from **NGINX Ingress Controller**
to **NGINX Gateway Fabric** using **Kubernetes Gateway API** and the
**ingress2gateway** conversion tool.

The ingress2gateway tool is an official Kubernetes SIG-Network project.
All tool behavior, scope, and limitations are based on the following reference:

🔗 **Official ingress2gateway Documentation**  
https://github.com/kubernetes-sigs/ingress2gateway/blob/v0.5.0/README.md

---

## 2. Migration Objective

- Migrate from legacy **Ingress** resources to **Gateway API**
- Replace **NGINX Ingress Controller** with **NGINX Gateway Fabric**
- Perform migration with **no downtime**
- Validate traffic before removing legacy Ingress objects

---

## 3. Prerequisites

### Cluster Components
- Kubernetes cluster
- MetalLB installed and configured
- NGINX Gateway Fabric installed
- Gateway API CRDs installed

### Verify Gateway API CRDs
```bash
kubectl get crd | egrep "gatewayclasses|gateways|httproutes"
````

---

## 4. Installing ingress2gateway Tool

The ingress2gateway installation methods are documented in the official README:

🔗 [https://github.com/kubernetes-sigs/ingress2gateway/blob/v0.5.0/README.md](https://github.com/kubernetes-sigs/ingress2gateway/blob/v0.5.0/README.md)

### Example (Go-based installation)

```bash
go install github.com/kubernetes-sigs/ingress2gateway@v0.5.0
```

Verify installation:

```bash
ingress2gateway --help
```

---

## 5. Verify NGINX Gateway Fabric

### Check controller status

```bash
kubectl get pods -n nginx-gateway
```

### Validate GatewayClass

```bash
kubectl get gatewayclass nginx -o yaml
```

Expected:

* `controllerName: gateway.nginx.org/nginx-gateway-controller`
* Status: `Accepted = True`

---

## 6. Inventory Existing Ingress Resources

List all Ingress resources:

```bash
kubectl get ingress -A
```

Create a backup:

```bash
kubectl get ingress -A -o yaml > ingress-backup.yaml
```

---

## 7. Convert Ingress to Gateway API

Ingress resources are converted using **ingress2gateway** as described in the
official README:

🔗 [https://github.com/kubernetes-sigs/ingress2gateway/blob/v0.5.0/README.md#usage](https://github.com/kubernetes-sigs/ingress2gateway/blob/v0.5.0/README.md#usage)

### Conversion command

```bash
ingress2gateway print \
  --providers=ingress-nginx \
  --all-namespaces \
  -o yaml > migrated-gatewayapi.yaml
```

Generated resources:

* Gateway
* HTTPRoute

> ⚠️ Note (from official documentation):
> ingress2gateway does **not** copy annotations from Ingress to Gateway API.
> Manual review is required.

---

## 8. Apply Generated Gateway API Resources

```bash
kubectl apply -f migrated-gatewayapi.yaml
```

Verify:

```bash
kubectl get gateway,httproute -n default
kubectl describe gateway -n default nginx
```

Expected:

* Gateway `PROGRAMMED = True`
* HTTPRoute `Accepted = True`
* References resolved successfully

---

## 9. Validate Dataplane Service (MetalLB)

NGINX Gateway Fabric automatically provisions a dataplane Service
in the same namespace as the Gateway.

```bash
kubectl get deploy,po,svc -n default | egrep -i "nginx|gateway"
```

Expected:

* Deployment running
* Service type: `LoadBalancer`
* External IP assigned by MetalLB

---

## 10. Traffic Validation

Test routing using Host header:

```bash
curl -I http://<GATEWAY_EXTERNAL_IP>/ -H "Host: test.local"
```

Compare with legacy Ingress behavior to ensure identical responses.

---

## 11. Remove Legacy Ingress Resource

After successful validation:

```bash
kubectl delete ingress -n default web
```

Confirm HTTPRoute status:

```bash
kubectl describe httproute -n default web-test-local
```

Expected conditions:

* Accepted = True
* ResolvedRefs = True
* Controlled by nginx-gateway-controller

---

## 12. Cleanup (Optional)

Verify no remaining Ingress resources:

```bash
kubectl get ingress -A
```

Remove legacy ingress-nginx controller:

```bash
kubectl delete ns ingress-nginx
```

---

## 13. References

* ingress2gateway Official README
  [https://github.com/kubernetes-sigs/ingress2gateway/blob/v0.5.0/README.md](https://github.com/kubernetes-sigs/ingress2gateway/blob/v0.5.0/README.md)

* Kubernetes Gateway API Concepts
  [https://gateway-api.sigs.k8s.io/](https://gateway-api.sigs.k8s.io/)

---

## 14. Conclusion

This migration follows the official **ingress2gateway** workflow and aligns with
Kubernetes Gateway API best practices.
The environment now uses **NGINX Gateway Fabric** as the traffic entry point,
providing a scalable and future-proof networking architecture.

```

قولّي 👍
```
