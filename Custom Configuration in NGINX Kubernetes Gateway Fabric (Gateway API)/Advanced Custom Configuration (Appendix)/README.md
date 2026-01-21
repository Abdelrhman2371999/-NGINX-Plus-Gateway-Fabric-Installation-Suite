
# Advanced Custom Configuration (Appendix)

This section extends the previous document with **advanced enterprise features**
supported by **:contentReference[oaicite:0]{index=0}**.



## Example 3: Rate Limiting Policy

### Use Case
- Protect APIs from abuse
- Prevent DoS / brute-force attacks
- Apply limits per route or per client



### Create a RateLimitPolicy
```yaml
apiVersion: gateway.nginx.org/v1alpha1
kind: RateLimitPolicy
metadata:
  name: api-rate-limit
spec:
  rate: 10r/s
  burst: 20
````

---

### Attach Rate Limit to HTTPRoute

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: api-route
spec:
  parentRefs:
  - name: nginx-gateway
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /api
    backendRefs:
    - name: api-svc
      port: 80
    filters:
    - type: ExtensionRef
      extensionRef:
        group: gateway.nginx.org
        kind: RateLimitPolicy
        name: api-rate-limit
```

---

### Result

* Maximum **10 requests per second**
* Burst up to **20**
* Excess requests receive **HTTP 429**

---

## Example 4: Mutual TLS (mTLS)

### Use Case

* Zero Trust architectures
* Secure internal APIs
* Client certificate authentication

---

### Step 1: Create TLS Profile

```yaml
apiVersion: gateway.nginx.org/v1alpha1
kind: TLSProfile
metadata:
  name: mtls-profile
spec:
  protocols:
  - TLSv1.2
  - TLSv1.3
  client:
    verify: required
    caCertificateRefs:
    - kind: Secret
      name: client-ca
```

> `client-ca` is a Kubernetes Secret containing the CA certificate.

---

### Step 2: Attach TLS Profile to Gateway

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: nginx-gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: https
    port: 443
    protocol: HTTPS
    tls:
      mode: Terminate
      certificateRefs:
      - kind: Secret
        name: server-cert
    allowedRoutes:
      namespaces:
        from: Same
```

---

### Result

* Clients **must present valid certificates**
* Unauthorized clients are rejected during TLS handshake

---

## Example 5: Web Application Firewall (WAF)

### Use Case

* Protect against OWASP Top 10
* SQL Injection, XSS, RCE
* API security

> ⚠️ Requires **NGINX App Protect** (Enterprise)

---

### Create WAF Policy

```yaml
apiVersion: gateway.nginx.org/v1alpha1
kind: AppProtectPolicy
metadata:
  name: waf-policy
spec:
  policy:
    name: default
    enforcementMode: blocking
```

---

### Attach WAF Policy to HTTPRoute

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: secure-route
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
        kind: AppProtectPolicy
        name: waf-policy
```

---

### Result

* Malicious requests blocked before reaching backend
* Full Layer-7 protection

---

## Example 6: Ingress → NGINX Fabric Migration

### Original Ingress (Example)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web
spec:
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
```

---

### Step 1: Create Gateway

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: nginx-gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    port: 80
    protocol: HTTP
```

---

### Step 2: Create HTTPRoute

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-route
spec:
  parentRefs:
  - name: nginx-gateway
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

### Migration Benefits

* Clear separation: **Gateway / Route / Policy**
* No annotations
* Better multi-team governance
* Enterprise-grade extensibility

---

## Final Architecture (Recommended)

```
MetalLB (VIP)
     ↓
NGINX Gateway Fabric
     ↓
Gateway
     ↓
HTTPRoute
     ↓
Policies (Cache / RateLimit / mTLS / WAF)
     ↓
Services
     ↓
Pods
```

---

## Final Notes

* All customization is **declarative**
* No manual reloads
* GitOps friendly
* Production-ready

This makes NGINX Fabric a modern replacement for traditional Ingress controllers.
