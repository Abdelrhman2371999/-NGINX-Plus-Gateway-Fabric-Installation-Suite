# 🚀 NGINX Plus & Gateway Fabric Installation Suite

<div align="center">

### *Enterprise-Grade Kubernetes Traffic, Gateway & Observability Platform*

![NGINX](https://img.shields.io/badge/NGINX-Plus-009639?style=for-the-badge&logo=nginx&logoColor=white)
![GatewayAPI](https://img.shields.io/badge/Gateway_API-NGINX_Fabric-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Production-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Dynatrace](https://img.shields.io/badge/Observability-Dynatrace-1496FF?style=for-the-badge&logo=dynatrace&logoColor=white)
![Docs](https://img.shields.io/badge/Documentation-Hands--On-blue?style=for-the-badge)

> 🚀 **From Ingress to Gateway API — with Full Observability**

</div>



## 📌 Overview

An **end-to-end, production-ready installation and migration suite** for running  
**NGINX Plus**, **Ingress**, and **Gateway API (NGINX Gateway Fabric)** on Kubernetes,  
with **enterprise-grade observability using Dynatrace**.

This repository is designed for **Integrators, DevOps, SREs, and Platform Engineers**  
who want a **clean, repeatable, real-world path** from classic Ingress  
to Gateway API — **with visibility, tracing, and troubleshooting built-in**.

---

## 📑 Table of Contents

- [📌 Overview](#-overview)
- [🧭 Project Scope](#-project-scope)
- [🏗️ High-Level Architecture](#-high-level-architecture)
- [🌐 Request Flow Animation](#-request-flow-animation)
- [📂 Repository Structure](#-repository-structure)
- [🚀 Quick Start Paths](#-quick-start-paths)
- [📊 Observability with Dynatrace](#-observability-with-dynatrace)
- [🛠️ Common Real-World Issues Covered](#-common-real-world-issues-covered)
- [✅ Validation Checklist](#-validation-checklist)
- [🎯 Target Audience](#-target-audience)
- [📌 Use Cases](#-use-cases)
- [📎 Next Enhancements](#-next-enhancements-planned)
- [📜 Status](#-status)

---

## 🧭 Project Scope

This project covers **north–south traffic management** and **full observability**
across the Kubernetes stack:

- External traffic ingress
- Gateway API routing
- Application workloads
- Cluster & application monitoring
- End-to-end request visibility

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

---

## 🌐 Request Flow Animation

The following sequence diagram shows how a request travels from the **end user**
to the **application pod** through **NGINX Gateway Fabric** and Kubernetes services,
and how telemetry is shipped to **Dynatrace**.

```mermaid
sequenceDiagram
    autonumber
    participant U as User / Client
    participant LB as External Load Balancer
    participant GW as NGINX Gateway Fabric
    participant R as Gateway API (HTTPRoute / TLSRoute)
    participant SVC as Kubernetes Service
    participant POD as Application Pod
    participant OA as Dynatrace OneAgent
    participant AG as Dynatrace ActiveGate
    participant DT as Dynatrace SaaS

    U->>LB: HTTPS Request
    LB->>GW: Forward traffic (L4 / L7)
    GW->>R: Match route rules
    R->>SVC: Route to backend Service
    SVC->>POD: Select Pod
    POD-->>SVC: Response
    SVC-->>GW: Response
    GW-->>LB: Response
    LB-->>U: Final Response

    par Observability (Parallel)
        POD-->>OA: Metrics / Traces / Logs
        OA-->>AG: Cluster-local aggregation
        AG-->>DT: Upload telemetry
        DT-->>DT: Correlation & AI analysis
    end
```

### What You Should See in Dynatrace

* **Kubernetes → Overview:** Nodes, Pods, CPU, Memory
* **Service Flow:** Request path & dependencies
* **Problems:** CPU, memory, latency, failures
* **Distributed Traces:** End-to-end request tracing

---

## 📂 Repository Structure

🗂️ Repository Structure – Visual Overview
graph TD
    R[NGINX Plus & Gateway Fabric Installation Suite]

    R --> C[CheckList]
    C --> C1[README.md]

    R --> CC[Custom Configuration<br/>NGINX Gateway Fabric]
    CC --> CC1[README.md]
    CC --> CC2[Advanced Custom Configuration]
    CC2 --> CC21[README.md]

    R --> D[Dashboards]
    D --> D1[README.md]
    D --> D2[Grafana / K8s Dashboards]

    R --> M[IngressController2NGF]
    M --> M1[README.md]
    M --> M2[Before Migration<br/>NGINX Ingress]
    M2 --> M21[README.md]
    M --> M3[NGINX Gateway Fabric<br/>Client Side]
    M3 --> M31[README.md]

    R --> DT[Integrating Kubernetes<br/>with Dynatrace]
    DT --> DT1[README.md]

    R --> I1[Kubernetes NGINX Ingress V1]
    I1 --> I11[Setup.md]

    R --> I2[Kubernetes NGINX Ingress V2]
    I2 --> I21[Setup.md]

    R --> TF[NGINX Gateway Fabric<br/>Traffic Flow & Tracking]
    TF --> TF1[README.md]

    R --> G1[NGF Installation & Testing]
    R --> G2[Advanced NGF Configuration]
    R --> G3[NGINX Plus Installation]
    R --> G4[Kubernetes Cluster Installation]
    R --> G5[Troubleshooting & Issues]

---

## 🚀 Quick Start Paths

### 🔹 Path 1 – NGINX Gateway Fabric Only

1. Prepare Kubernetes cluster
2. Install NGINX Gateway Fabric
3. Deploy HTTPRoute / TLSRoute
4. Validate traffic flow

### 🔹 Path 2 – Ingress → Gateway Migration

1. Install Ingress Controller
2. Migrate using ingress2gateway
3. Deploy NGINX Gateway Fabric
4. Validate routing parity

### 🔹 Path 3 – Full Observability (Recommended)

1. Install Dynatrace Operator
2. Deploy DynaKube (OneAgent + ActiveGate)
3. Observe cluster & services
4. Analyze traffic and problems

---

## 📊 Observability with Dynatrace

This repository includes a **complete real-world guide** for integrating Kubernetes
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

* ActiveGate stuck in `Pending` (missing StorageClass)
* OneAgent `ImagePullBackOff`
* Cluster shows `0 Nodes / 0 Pods`
* Metrics Server TLS issues (kubeadm)
* Network & DNS validation problems

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
✔ Dashboards populated
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

---

**Author:** Abdelrhman Hamed Mousaa  
*Kubernetes • NGINX • Gateway API • Observability*
