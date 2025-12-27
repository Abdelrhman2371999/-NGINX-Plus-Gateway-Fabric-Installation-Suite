<div align="center">

# 🚀 NGINX Plus & Gateway Fabric Installation Suite

### *Enterprise-Grade Application Delivery Infrastructure*

![NGINX Ecosystem](https://img.shields.io/badge/NGINX-Ecosystem-009639?style=for-the-badge&logo=nginx&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-1.28-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![License](https://img.shields.io/badge/License-Guidance_Only-blue?style=for-the-badge)

*A comprehensive toolkit for deploying and managing **NGINX Plus** and **NGINX Gateway Fabric (NGF)** in production Kubernetes environments.*

</div>

---

## 📊 Quick Navigation

| [🏗️ Architecture](#-architecture-overview) | [📚 Guides](#-documentation-guide) | [⚡ Quick Start](#-quick-start) | [🛠️ Tools](#-tools--technologies) |
|------------------------------------------|-----------------------------------|--------------------------------|-----------------------------------|

---

## 🏗️ Architecture Overview

```mermaid
graph TB
    subgraph "Infrastructure Layer"
        A[Ubuntu 22.04 Server] --> B[NGINX Plus Installation]
        B --> C[Kubernetes Cluster<br/>1 Master + 2 Workers]
    end
    
    subgraph "Ingress Solutions"
        C --> D1[Option 1: Nginx Ingress<br/>Open Source]
        C --> D2[Option 2: NGF<br/>Enterprise]
    end
    
    subgraph "Nginx Ingress Features"
        D1 --> E1[Basic Routing]
        D1 --> E2[SSL/TLS Termination]
        D1 --> E3[Load Balancing]
    end
    
    subgraph "NGF Features"
        D2 --> F1[Gateway API]
        D2 --> F2[Advanced Traffic Mgmt]
        D2 --> F3[Metrics & Monitoring]
        D2 --> F4[Security Policies]
    end
    
    E1 --> G[Production Applications]
    E2 --> G
    E3 --> G
    
    F1 --> G
    F2 --> G
    F3 --> G
    F4 --> G
