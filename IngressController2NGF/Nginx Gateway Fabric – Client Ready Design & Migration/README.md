# NGINX Gateway Fabric – Client Ready Design & Migration Guide

## 1. Executive Summary

This document describes the recommended architecture and migration approach for deploying **NGINX Gateway Fabric (NGF)** behind **F5**, covering:

* Client IP preservation
* Rate limiting strategy
* API schema validation placement
* A phased migration from ingress-nginx to NGF

The goal is to achieve **security, scalability, and operational clarity** while minimizing risk during migration.

---

## 2. Target Traffic Flow (End-to-End)

```text
Client
  |
  v
F5 (TLS termination / WAF optional)
  |  - Edge Rate Limiting (DoS / brute force, per IP/URI)
  |  - Adds X-Forwarded-For (real client IP)
  v
NGINX Gateway Fabric (Routing, mTLS, policies)
  |  - Route policies (timeouts/retries/weights)
  |  - Optional simple rate limits (non-critical)
  v
API Gateway / Validation Layer  <── Schema Validation happens here
  |  - Schema validation (enable/bypass per policy)
  |  - Centralized Rate Limiting (per API key/user/token) + shared store (Redis)
  v
Backend Services

```

---

## 3. Decision Matrix (Rate Limiting & Client IP)

| Requirement                       | Recommended Layer                    | Rationale                                | Notes                                |
| --------------------------------- | ------------------------------------ | ---------------------------------------- | ------------------------------------ |
| DoS / brute-force protection      | F5 (Edge)                            | Single entry point, protects cluster     | Security-focused, not business rules |
| Per-consumer / per-API-key limits | Centralized RL (API Gateway / Redis) | Shared state, consistent across replicas | Best for SLA & fairness              |
| Simple per-route limits           | NGF                                  | Close to routing layer                   | Not guaranteed consistent            |
| Preserve real client IP           | X-Forwarded-For (F5 → NGF)           | SNAT-safe, scalable                      | Trust only F5 IP ranges              |
| Network-level source IP           | externalTrafficPolicy: Local         | Preserves L4 source IP                   | Use only if strictly required        |

---

## 4. Recommended Architecture Decisions

### 4.1 Client IP Preservation

* F5 injects **X-Forwarded-For** with the real client IP.
* NGF/NGINX is configured to **trust XFF only from F5 IP ranges**.
* `externalTrafficPolicy: Local` is **optional** and applied only if a hard L4 requirement exists.

### 4.2 Rate Limiting Strategy

* **Edge protection:** F5 handles volumetric attacks and brute-force attempts.
* **API governance:** Centralized rate limiting in the API Gateway / Validation Layer.
* **NGF:** Used only for lightweight, non-critical rate limits.

### 4.3 API Schema Validation

* Schema validation is **not enforced at NGF level**.
* A dedicated **API Gateway / Validation Layer** performs:

  * OpenAPI / schema validation
  * Enable / bypass logic per API or route
* This avoids coupling routing stability with validation complexity.

---

## 5. Migration Plan (Phased Approach)

### Phase 0 – Discovery & Readiness

* Inventory existing ingress-nginx resources and annotations.
* Classify features into:

  * Gateway API core
  * NGF custom policies
  * Unsupported features (e.g., FastCGI)
* Confirm F5 SNAT and XFF behavior.

### Phase 1 – Platform Foundation

* Deploy NGF and GatewayClass/Gateway.
* Enable logging, metrics, and tracing.
* Define global default policies (timeouts, keepalive, basic limits).

### Phase 2 – Networking & Client IP

* Enable X-Forwarded-For on F5.
* Configure NGF to trust XFF only from F5 ranges.
* Decide on `externalTrafficPolicy` (Cluster by default).

### Phase 3 – Route Migration

* Convert Ingress resources to HTTPRoute.
* Replace canary annotations with **traffic weights**.
* Configure per-route timeouts, retries, and header rules.
* Validate Route `Accepted` status.

### Phase 4 – API Validation Layer

* Deploy API Gateway / Validation Layer behind NGF.
* Enable schema validation selectively.
* Implement bypass policies where required.

### Phase 5 – Rate Limiting Rollout

* Configure F5 edge rate limits (DoS / brute force).
* Enable centralized rate limiting (per user / per API key).
* Perform load and consistency testing.

### Phase 6 – Cutover & Cleanup

* Gradual traffic cutover using weights.
* Monitor latency (p95/p99), error rates, and drops.
* Decommission ingress-nginx after stabilization.

---

## 6. Key Benefits of This Design

* Clear separation of concerns (Edge / Routing / Governance)
* Consistent rate limiting across replicas
* Safer schema validation without impacting routing
* Enterprise-ready and multi-team friendly

---

## 7. Final Recommendation

This architecture is **production-grade**, aligns with **Gateway API best practices**, and provides a controlled, low-risk migration path from ingress-nginx to NGINX Gateway Fabric while preserving security, observability, and scalability.
