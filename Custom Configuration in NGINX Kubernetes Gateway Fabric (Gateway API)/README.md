# Custom Configuration in NGINX Kubernetes Gateway Fabric (Gateway API)

## Overview
This document explains how to create **custom configurations** in  
**:contentReference[oaicite:0]{index=0}**  
using **Gateway API policies** instead of editing `nginx.conf`.

The examples focus on:
- Response caching
- Hiding and modifying HTTP headers



## Why NGINX Fabric?
NGINX Fabric is designed to be:
- Kubernetes-native
- Declarative (YAML-based)
- Safer than editing nginx.conf
- Scalable for multi-team environments

Key concept:
> **No direct nginx.conf editing**  
> All customization is done using **CRDs (Policies)**.



## Architecture Overview
```

Client
|
v
Gateway (NGINX Fabric)
|
HTTPRoute
|
Policies (Cache, Header, RateLimit, TLS)
|
Kubernetes Service
|
Pods

````

---

## Prerequisites
- Kubernetes cluster running
- NGINX Gateway Fabric installed
- Gateway and HTTPRoute already created
- Backend Service exists

Verify:
```bash
kubectl get gateways
kubectl get httproutes
````

---

## Custom Configuration Model

In NGINX Fabric:

* Configuration is applied using **Policy CRDs**
* Policies can be attached to:

  * Gateway
  * HTTPRoute
  * Service

Common policies include:

* CachePolicy
* RequestHeaderModifierPolicy
* ResponseHeaderModifierPolicy
* RateLimitPolicy
* TLSProfile

---

# Example 1: Enable and Configure Caching

## Use Case

* Cache successful responses
* Reduce backend load
* Serve stale content if backend is unavailable

---

## Step 1: Create a Cache Policy

```yaml
apiVersion: gateway.nginx.org/v1alpha1
kind: CachePolicy
metadata:
  name: web-cache
spec:
  cacheZoneName: web-cache
  cacheValidity:
    - responseCodes: [200, 301, 302]
      validity: 10m
  useStale:
    - updating
    - error
    - timeout
```

### What this does

* Creates a cache zone named `web-cache`
* Caches HTTP 200/301/302 responses for 10 minutes
* Serves stale content during errors or timeouts

---

## Step 2: Attach Cache Policy to an HTTPRoute

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-route
spec:
  parentRefs:
  - name: nginx-gateway
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: web-svc
      port: 80
    filters:
    - type: ExtensionRef
      extensionRef:
        group: gateway.nginx.org
        kind: CachePolicy
        name: web-cache
```

---

## Optional: Bypass Cache Based on Headers

```yaml
spec:
  bypassCache:
    requestHeaders:
    - name: Cache-Control
      value: no-cache
```

---

## Result

* Cached responses served from NGINX
* Faster response times
* Backend protected from traffic spikes

---

# Example 2: Hide or Modify HTTP Headers

## Use Case

* Hide internal headers (security)
* Add custom headers for tracing or debugging
* Remove server information

---

## Step 1: Hide Response Headers

```yaml
apiVersion: gateway.nginx.org/v1alpha1
kind: ResponseHeaderModifierPolicy
metadata:
  name: hide-headers
spec:
  remove:
    - Server
    - X-Powered-By
```

### Effect

* Removes sensitive headers from responses
* Improves security posture

---

## Step 2: Add Custom Response Headers

```yaml
apiVersion: gateway.nginx.org/v1alpha1
kind: ResponseHeaderModifierPolicy
metadata:
  name: add-headers
spec:
  add:
    X-Platform: Kubernetes
    X-Gateway: NGINX-Fabric
```

---

## Step 3: Attach Header Policy to HTTPRoute

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-route
spec:
  parentRefs:
  - name: nginx-gateway
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: web-svc
      port: 80
    filters:
    - type: ExtensionRef
      extensionRef:
        group: gateway.nginx.org
        kind: ResponseHeaderModifierPolicy
        name: hide-headers
```

---

## Verify Headers

```bash
curl -I http://test.local
```

Expected:

* No `Server` header
* Custom headers visible (if added)

---

## Best Practices

* Prefer policies over annotations
* Apply policies at HTTPRoute level for fine control
* Avoid mixing Ingress and Gateway API in production
* Version control all YAML files (GitOps)

---

## Common Mistakes

* Trying to edit nginx.conf directly
* Using Ingress annotations with Fabric
* Forgetting to attach policy to a route or gateway

---

## Comparison Summary

| Feature             | Ingress-NGINX | NGINX Plus Ingress | NGINX Fabric |
| ------------------- | ------------- | ------------------ | ------------ |
| Custom nginx.conf   | ❌             | ⚠️ Limited         | ❌            |
| Policy-based config | ❌             | ⚠️                 | ✅            |
| Per-route cache     | ❌             | ⚠️                 | ✅            |
| Gateway API support | ❌             | ❌                  | ✅            |
| Enterprise ready    | ❌             | ✅                  | ✅            |

---

## Conclusion

NGINX Kubernetes Gateway Fabric provides a **clean, safe, and scalable** way
to customize traffic behavior in Kubernetes using:

* Declarative YAML
* Policy-based configuration
* Gateway API standards

This makes it ideal for production, security-focused, and multi-team clusters.

---

## References

* [https://gateway-api.sigs.k8s.io/](https://gateway-api.sigs.k8s.io/)
* [https://docs.nginx.com/nginx-gateway-fabric/](https://docs.nginx.com/nginx-gateway-fabric/)
* [https://kubernetes.io/docs/concepts/services-networking/gateway/](https://kubernetes.io/docs/concepts/services-networking/gateway/)
