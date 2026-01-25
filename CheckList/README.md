# Gateway API (v1.3) + NGINX Gateway Fabric  
## Migration Checklist (ingress-nginx ➜ NGF)

This checklist is intended for projects migrating from the community
NGINX Ingress Controller to NGINX Gateway Fabric (NGF) using the
Kubernetes Gateway API v1.3.

---

## 1. Core Foundations
- [ ] GatewayClass created and bound to NGINX Gateway Fabric
- [ ] Gateway resource deployed with correct listeners (HTTP / HTTPS)
- [ ] Listener ports and protocols validated
- [ ] TLS certificates created as Kubernetes Secrets
- [ ] ReferenceGrant configured if TLS Secrets are in a different namespace
- [ ] RBAC aligned with CI/CD and GitOps workflows

---

## 2. Client IP Preservation
**Recommended approach**
- [ ] F5 inserts `X-Forwarded-For` header
- [ ] NGF / NGINX configured to trust XFF only from F5 IP ranges
- [ ] Client IP visibility validated in access logs

**Optional**
- [ ] `externalTrafficPolicy: Local` evaluated
- [ ] Health checks confirmed to prevent traffic drops
- [ ] Pod distribution validated to avoid load imbalance

---

## 3. HTTPRoute – Routing (Core Capabilities)
- [ ] `parentRefs` correctly reference the Gateway and listener
- [ ] `hostnames` defined using valid FQDNs (no IPs or ports)
- [ ] `matches` configured for paths, methods, headers, and query params
- [ ] No overlapping or conflicting routes across namespaces
- [ ] HTTPRoute status shows `Accepted=True`

---

## 4. Filters & Traffic Processing
- [ ] RequestHeaderModifier / ResponseHeaderModifier applied where required
- [ ] URLRewrite or RequestRedirect used (not both in the same rule)
- [ ] Filter compatibility validated (no `IncompatibleFilters`)
- [ ] Any non-core or implementation-specific filter confirmed supported by NGF

---

## 5. Backend References & Traffic Splitting
- [ ] backendRefs point to valid Kubernetes Services
- [ ] Weights configured for canary or blue/green deployments
- [ ] Cross-namespace backends allowed via ReferenceGrant
- [ ] Route status shows `ResolvedRefs=True`

---

## 6. Timeouts & Retries (SLA Alignment)
- [ ] `timeouts.request` aligned with SLA targets
- [ ] `timeouts.backendRequest` ≤ `timeouts.request`
- [ ] Retry policies defined only for idempotent requests
- [ ] Total retry duration does not exceed request timeout

---

## 7. Security Model
- [ ] WAF requirements clearly defined (outside NGF core responsibilities)
- [ ] mTLS configuration validated (if applicable)
- [ ] No unauthorized cross-namespace references
- [ ] Secrets and certificates protected by ReferenceGrant rules

---

## 8. FastCGI Workloads (PHP / Drupal)
- [ ] FastCGI dependencies identified
- [ ] FastCGI handled at application layer (HTTP fronting / sidecar)
- [ ] FastCGI workloads excluded from initial NGF migration if required
- [ ] No assumption of native FastCGI support in Gateway API

---

## 9. Observability & Operations
- [ ] Metrics available (RPS, latency, error rates)
- [ ] Integration with monitoring platform (e.g., Dynatrace / Prometheus)
- [ ] Logs include real client IP
- [ ] Alerts defined for latency and 5xx error thresholds

---

## 10. Migration & Cutover
- [ ] Coexistence period planned (ingress-nginx + NGF)
- [ ] APIs migrated incrementally
- [ ] Cutover scheduled outside business hours
- [ ] Rollback plan documented and tested

---

## 11. Post-Deployment Validation
- [ ] Gateway listeners report Ready=True
- [ ] HTTPRoute Accepted=True
- [ ] No warning events such as `RefNotPermitted` or `IncompatibleFilters`
- [ ] Traffic flow verified end-to-end

---

## Summary
- Prefer **Core and Extended** Gateway API features
- Validate **implementation-specific** behavior with NGF before use
- Preserve client IP via **X-Forwarded-For from F5**
- Treat **API schema validation** as a separate concern from NGF
- Keep **FastCGI** outside the Gateway layer

https://gateway-api.sigs.k8s.io/reference/1.3/spec/?utm_source=chatgpt.com
