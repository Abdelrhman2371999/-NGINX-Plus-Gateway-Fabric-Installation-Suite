# 🚀 NGINX Plus & Gateway Fabric Installation Suite

<div align="center">

### *Enterprise-Grade Kubernetes Traffic & Gateway Solutions*

![NGINX](https://img.shields.io/badge/NGINX-Plus-009639?style=for-the-badge&logo=nginx&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Gateway_API-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Docs](https://img.shields.io/badge/Documentation-Hands--On-blue?style=for-the-badge)

> 🚀 **From Ingress to Gateway API — the right way**

</div>


## 📑 Table of Contents

- [📌 What is this Repository?](#-what-is-this-repository)
- [📂 Repository Content Overview](#-repository-content-overview)
  - [NGINX Plus & Kubernetes Foundations](#-nginx-plus--kubernetes-foundations)
  - [Traditional Kubernetes Ingress](#-traditional-kubernetes-ingress-baseline-knowledge)
  - [Ingress → Gateway API Migration](#-migration-ingress--gateway-api)
  - [NGINX Gateway Fabric](#-nginx-gateway-fabric-gateway-api)
  - [Custom & Advanced Configuration](#-custom--advanced-gateway-configuration)
  - [Troubleshooting & Issues](#-troubleshooting--issues)
- [🌐 Request Flow Animation](#-request-flow-animation-gateway-api)
- [🆚 Why Gateway API instead of Ingress?](#-why-gateway-api-instead-of-ingress)
- [🧠 Who Should Use This Repo?](#-who-should-use-this-repo)
- [⭐ Why Star This Repository?](#-why-star-this-repository)
- [✍️ Author](#-author)

---

## 📌 What is this Repository?

This repository is a **complete learning and implementation suite** that explains:

- How traffic enters Kubernetes
- How **NGINX Ingress Controller (v1 & v2)** works
- Why **Ingress is being replaced by Gateway API**
- How **NGINX Gateway Fabric (NGF)** operates internally
- How to build **enterprise-grade Gateway architectures**

🎯 The main goal is to provide **clear, real-world, production-focused documentation** — not just theory.

---

## 📂 Repository Content Overview

> 🔎 Click any link to jump directly to its documentation

---

### 🔹 NGINX Plus & Kubernetes Foundations

- 📄 **[Installing NGINX Plus on Ubuntu](Installing_NGINX_Plus-on-Ubuntu.md)**  
  👉 Install & license NGINX Plus on Ubuntu 22.04  
  👉 Verify binaries, services, and repositories

- 📄 **[Kubernetes Cluster Installation Guide](KubernetesClusterInstallationGuide.md)**  
  👉 Build a Kubernetes cluster (Control Plane + Workers)  
  👉 Networking, container runtime, and cluster readiness

---

### 🔹 Traditional Kubernetes Ingress (Baseline Knowledge)

Understanding Ingress is critical before moving to Gateway API.

#### 📁 Kubernetes Nginx Ingress Controller v1
- 📄 **[Setup Guide](Kubernetes%20Nginx%20Ingress%20ControllerV1/Setup.md)**  
  👉 Classic `Ingress` resources  
  👉 Path-based routing  
  👉 Basic TLS & service exposure

#### 📁 Kubernetes Nginx Ingress Controller v2
- 📄 **[Setup Guide](Kubernetes%20Nginx%20Ingress%20ControllerV2/setup.md)**  
  👉 Newer NGINX Ingress architecture  
  👉 Improved CRDs & controller behavior

---

### 🔄 Migration: Ingress → Gateway API

#### 📁 IngressController2NGF
- 📄 **[Migration Guide](IngressController2NGF/README.md)**  
  👉 Why Ingress is limited  
  👉 Gateway API concepts  
  👉 Mapping:
  - Ingress → Gateway  
  - Rules → HTTPRoute  
  - Annotations → Policy-based config

---

### 🌐 NGINX Gateway Fabric (Gateway API)

- 📄 **[NGF Installation and Testing](NGF-Installation-and-Testing.md)**  
  👉 Install NGINX Gateway Fabric  
  👉 GatewayClass, Gateway, HTTPRoute  
  👉 Validate traffic routing

- 📁 **NGINX Gateway Fabric – Traffic Flow & Tracking**  
  👉 Deep dive into request lifecycle  
  👉 Observability & traffic tracing  
  👉 How requests move inside the cluster

---

### 🧩 Custom & Advanced Gateway Configuration

#### 📁 Custom Configuration in NGINX Kubernetes Gateway Fabric
- 📄 **[Custom Configuration Guide](Custom%20Configuration%20in%20NGINX%20Kubernetes%20Gateway%20Fabric%20(Gateway%20API)/README.md)**  
  👉 Extending Gateway API behavior  
  👉 Fine-grained traffic control

##### 📁 Advanced Custom Configuration (Appendix)
- Enterprise patterns  
- Advanced routing scenarios  
- Real-world edge cases

- 📄 **[Advanced NGF Configuration Guide](Advanced-NGF-Configuration-Guide.md)**  
  👉 Security policies  
  👉 Traffic shaping  
  👉 Production Gateway design

---

### 🛠 Troubleshooting & Issues

- 📄 **[Troubleshooting Guide](Troubleshooting.md)**  
  👉 Common Kubernetes & NGINX problems  
  👉 Debugging traffic failures  
  👉 Gateway & Ingress errors

- 📄 **[Issues & Notes](iusse.md)**  
  👉 Collected implementation issues  
  👉 Practical fixes & lessons learned

---

## 🌐 Request Flow Animation (Gateway API)

```mermaid
sequenceDiagram
    participant Client
    participant Gateway
    participant NGF as NGINX Gateway Fabric
    participant Service
    participant Pod

    Client->>Gateway: HTTP Request
    Gateway->>NGF: Route Match (HTTPRoute)
    NGF->>Service: Forward Traffic
    Service->>Pod: Load Balanced Request
    Pod-->>Client: Response
````

---

## 🔄 Ingress vs Gateway API (Visual Flow)

```mermaid
graph LR
    A[Client] --> B[Ingress Controller]
    B --> C[Service]
    C --> D[Pod]

    A --> E[Gateway API]
    E --> F[NGINX Gateway Fabric]
    F --> C
```

---

## 🆚 Why Gateway API instead of Ingress?

| Feature            | Ingress | Gateway API   |
| ------------------ | ------- | ------------- |
| API Design         | Limited | Role-oriented |
| Traffic Control    | Basic   | Advanced      |
| Extensibility      | Weak    | Strong        |
| Multi-Team Support | ❌       | ✅             |
| Enterprise Ready   | ⚠️      | ✅             |

---

## 🧠 Who Should Use This Repo?

✔ Kubernetes Engineers
✔ DevOps / SRE
✔ Network & Security Engineers
✔ Engineers migrating from Ingress to Gateway API
✔ Anyone learning **NGINX Gateway Fabric** seriously

---

## ⭐ Why Star This Repository?

* Clear Ingress vs Gateway API comparison
* Real-world enterprise Gateway patterns
* Migration-focused documentation
* Visual diagrams & animations
* Production-ready mindset

If this repository helped you understand Kubernetes traffic better, consider giving it a ⭐

---

## ✍️ Author

**Abdelrhman Hamed**
NGINX • Kubernetes • Gateway API • Network Security

> Built with ❤️ to explain Kubernetes traffic *the right way*
