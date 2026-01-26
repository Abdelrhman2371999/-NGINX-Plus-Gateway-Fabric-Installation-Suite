# NGINX Plus & Gateway Fabric Installation Suite

An **end-to-end, production-ready installation and migration suite** for running
**NGINX Plus**, **Ingress**, and **Gateway API (NGINX Gateway Fabric)** on Kubernetes,
with **full observability using Dynatrace**.

This repository is designed for **Integrators, DevOps, SREs, and Platform Engineers**
who want a **clean, repeatable, and well-documented path** from classic Ingress
to Gateway API with enterprise-grade monitoring.



## 📌 What This Repository Covers

- Kubernetes cluster preparation (on-prem / kubeadm / VM)
- NGINX Plus Ingress Controller installation
- Migration from **Ingress → Gateway API**
- NGINX Gateway Fabric installation & validation
- Advanced routing (HTTPRoute, TLSRoute)
- Traffic flow tracking & troubleshooting
- **Observability & monitoring using Dynatrace**
- Real-world issues and fixes (storage, networking, image pulls, metrics)

---

## 🧭 Project Scope

This project covers **north-south traffic management** and **full observability**
across the Kubernetes stack:

- External traffic ingress
- Gateway API routing
- Application workloads
- Cluster & application monitoring

---

## 🏗️ High-Level Architecture

```mermaid
flowchart LR
    U[End Users / Clients]

    U --> LB[External Load Balancer]

    subgraph Kubernetes_Cluster[Kubernetes Cluster]
        LB --> GW[NGINX Gateway Fabric]
        GW --> RT[Gateway API\nHTTPRoute / TLSRoute]
        RT --> SVC[Kubernetes Services]
        SVC --> PODS[Application Pods]

        subgraph Observability
            OA[Dynatrace OneAgent\nDaemonSet]
            AG[Dynatrace ActiveGate]
        end

        PODS --- OA
        OA --> AG
    end

    AG --> DT[Dynatrace SaaS Platform]

    DT --> DASH[Dashboards]
    DT --> PROB[Problems & Davis AI]
    DT --> TRACE[Distributed Tracing]
````

## 🌐 Request Flow Animation

The following sequence diagram shows how a request travels from the **end user**
to the **application pod** through **NGINX Gateway Fabric** and Kubernetes services,
and how telemetry is shipped to **Dynatrace**.

```mermaid
sequenceDiagram
    autonumber
    participant U as User/Client
    participant LB as External Load Balancer
    participant GW as NGINX Gateway Fabric
    participant R as Gateway API (HTTPRoute/TLSRoute)
    participant SVC as Kubernetes Service
    participant POD as Application Pod
    participant OA as Dynatrace OneAgent
    participant AG as Dynatrace ActiveGate
    participant DT as Dynatrace SaaS

    U->>LB: HTTPS Request
    LB->>GW: Forward traffic (L4/L7)
    GW->>R: Match route rules
    R->>SVC: Route to backend Service
    SVC->>POD: Select Pod (Endpoints)
    POD-->>SVC: Response
    SVC-->>GW: Response
    GW-->>LB: Response
    LB-->>U: Final Response

    par Observability (in parallel)
        POD-->>OA: Metrics/Traces/Logs
        OA-->>AG: Send telemetry (cluster-local)
        AG-->>DT: Upload to Dynatrace tenant
        DT-->>DT: Correlate (Service Flow / Problems)
    end

```
What you should see in Dynatrace

Kubernetes → Overview: Nodes, Pods, CPU/Memory

Services / Service Flow: Request path & dependencies

Problems: CPU/Memory saturation, failures, latency anomalies

Traces: End-to-end request tracing (if tracing is enabled)

### Architecture Explanation

* **Ingress / Gateway Layer**

  * NGINX Gateway Fabric acts as the **north–south entry point**
  * Gateway API resources control routing and TLS

* **Application Layer**

  * Kubernetes Services route traffic to backend Pods

* **Observability Layer**

  * Dynatrace OneAgent runs on every node
  * ActiveGate aggregates cluster telemetry

* **Dynatrace Platform**

  * Dashboards, Problems, Service Flow, Distributed Tracing

---

## 📂 Repository Structure

```
.
├── IngressController2NGF/
│   └── Ingress to Gateway API migration guide
│
├── Dynatrace/
│   └── Integrating Kubernetes with Dynatrace (Operator-based – Trial)
│
├── NGF-Installation-and-Testing.md
├── Advanced-NGF-Configuration-Guide.md
├── Troubleshooting.md
└── README.md (this file)
```

---

## 🚀 Quick Start Paths

### 🔹 Path 1 – NGINX Gateway Fabric Only

1. Prepare Kubernetes cluster
2. Install NGINX Gateway Fabric
3. Deploy HTTPRoute / TLSRoute
4. Validate traffic flow

### 🔹 Path 2 – Ingress → Gateway Migration

1. Install Ingress Controller
2. Migrate resources using ingress2gateway
3. Deploy NGINX Gateway Fabric
4. Validate routing parity

### 🔹 Path 3 – Full Observability (Recommended)

1. Install Dynatrace Operator
2. Deploy DynaKube (OneAgent + ActiveGate)
3. Observe cluster, services, and traffic
4. Use dashboards and Problems for analysis

---

## 📊 Observability with Dynatrace

This repository includes a **complete, real-world guide** for integrating Kubernetes
with Dynatrace using the **Operator + DynaKube** model.

### Features Enabled

* Node & Pod metrics
* Kubernetes cluster visibility
* Application & service flow
* Automatic problem detection
* Distributed tracing
* Logs & telemetry ingestion

📘 Documentation:

```
Dynatrace/Integrating Kubernetes with Dynatrace (Operator-based – Trial).md
```

---

## 🛠️ Common Real-World Issues Covered

* ActiveGate stuck in `Pending` (StorageClass missing)
* OneAgent `ImagePullBackOff`
* Cluster shows `0 Nodes / 0 Pods`
* Metrics server TLS issues (kubeadm)
* Network & DNS validation

Each issue includes:

* Root cause
* Verification commands
* Fix steps

---

## ✅ Validation Checklist

✔ NGINX Gateway Fabric running
✔ Gateway API routes functional
✔ Traffic flowing end-to-end
✔ Dynatrace OneAgent on all nodes
✔ ActiveGate running
✔ Kubernetes dashboards populated
✔ Problems automatically detected

---

## 🎯 Target Audience

* Kubernetes Integrators
* Platform Engineers
* DevOps / SRE
* Network & Security Engineers
* Solution Architects

---

## 📌 Use Cases

* Customer PoC / Demo
* Production readiness validation
* Interview / technical showcase
* Internal platform documentation
* Enterprise migration projects

---

## 📎 Next Enhancements (Planned)

* mTLS with Gateway API
* WAF integration
* Rate limiting & security policies
* Multi-cluster observability
* CI/CD validation pipelines

---

## 📜 Status

**Production-ready documentation & lab-tested integration**

---

> Maintained as a practical, real-world reference for Kubernetes
> traffic management and observability.

