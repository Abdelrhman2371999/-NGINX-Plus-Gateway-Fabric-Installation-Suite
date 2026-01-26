# Integrating Kubernetes with Dynatrace (Operator-based – Trial)

This README documents the **end-to-end steps** to integrate a Kubernetes cluster with **Dynatrace (15-day trial)** using the **Dynatrace Operator + DynaKube** model.

It is written for **integrators / DevOps / SRE engineers** and reflects a **real troubleshooting-based installation**, including common issues and fixes.

---

## 1. Prerequisites

### Kubernetes

* Kubernetes cluster (kubeadm / on‑prem / VM-based)
* At least **1 worker node** (recommended: 2+)
* Cluster access via `kubectl`
* Outbound internet access (HTTPS 443)

### Tools

```bash
kubectl >= 1.24
helm >= 3.x
```

### Dynatrace

* Dynatrace **trial tenant** (15 days)
* Access to Dynatrace UI

---

## 2. High-Level Architecture

```
Kubernetes Cluster
├── Dynatrace Operator
│   ├── Webhook
│   ├── CSI Driver
│
├── DynaKube (CR)
│   ├── OneAgent (DaemonSet)
│   ├── ActiveGate (StatefulSet)
│   └── OTEL Collector
│
└── Dynatrace SaaS (Tenant)
```

---

## 3. Create Dynatrace Tokens

From Dynatrace UI:

1. Go to **Access Tokens**
2. Create:

### Operator Token

Required scopes:

* Kubernetes API
* Installer download
* Settings write

### Data Ingest Token

Required scopes:

* Ingest metrics
* Ingest logs
* Ingest OpenTelemetry traces

---

## 4. Install Dynatrace Operator (Helm)

```bash
helm install dynatrace-operator oci://public.ecr.aws/dynatrace/dynatrace-operator \
  --create-namespace \
  --namespace dynatrace \
  --atomic
```

Verify:

```bash
kubectl -n dynatrace get pods
```

Expected:

* dynatrace-operator
* dynatrace-webhook
* dynatrace-oneagent-csi-driver

---

## 5. Create DynaKube Custom Resource

From Dynatrace UI:

* Kubernetes → Add cluster
* Configure options
* Download `dynakube.yaml`

Apply:

```bash
kubectl apply -f dynakube.yaml
```

Verify:

```bash
kubectl -n dynatrace get dynakube
```

---

## 6. Storage Requirement (ActiveGate PVC)

ActiveGate **requires a StorageClass**.
On bare‑metal / kubeadm clusters, this is commonly missing.

### Install Local Path Provisioner

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
```

### Set as Default StorageClass

```bash
kubectl patch storageclass local-path \
  -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

Delete stuck ActiveGate pod to reschedule:

```bash
kubectl -n dynatrace delete pod -l app.kubernetes.io/name=activegate
```

---

## 7. Verify Pods Status

```bash
kubectl -n dynatrace get pods
```

Expected (eventually):

* ActiveGate → `1/1 Running`
* OneAgent (DaemonSet) → `1/1 Running` per node

---

## 8. Network Validation (Important)

Dynatrace images are pulled from:

```
<tenant>.live.dynatrace.com
```

Validation (401 is expected):

```bash
kubectl run netshoot --rm -it --image=nicolaka/netshoot -- bash -lc \
'curl -I https://<tenant>.live.dynatrace.com/v2/ ; nslookup <tenant>.live.dynatrace.com'
```

Expected:

* HTTP 401 (means reachable)
* DNS resolution works

---

## 9. Metrics Server (Optional but Recommended)

Dynatrace does NOT require metrics-server, but Kubernetes CLI dashboards do.

### Install

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

### Fix TLS Issue (kubeadm clusters)

```bash
kubectl -n kube-system patch deployment metrics-server \
  --type='json' \
  -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
```

Restart:

```bash
kubectl -n kube-system rollout restart deployment/metrics-server
```

Verify:

```bash
kubectl top nodes
```

---

## 10. Dynatrace UI Validation

### Kubernetes

* Kubernetes → Overview
* Set timeframe to **Last 1 hour**

Expected:

* Nodes > 0
* Pods > 0
* CPU / Memory values

### Problems

Dynatrace will automatically create Problems such as:

* CPU saturation
* Memory saturation
* Monitoring unavailable (temporary during setup)

This confirms **data ingestion is active**.

---

## 11. Common Issues & Fixes

### ActiveGate stuck in Pending

Cause:

* No StorageClass

Fix:

* Install local-path provisioner

---

### OneAgent ImagePullBackOff

Cause:

* Internet / proxy issue

Fix:

* Validate HTTPS 443 access

---

### DynaKube stuck in Deploying

Cause:

* ActiveGate not Ready yet

Fix:

* Wait 5–10 minutes
* Check ActiveGate logs

---

## 12. Success Criteria

✔ ActiveGate Running
✔ OneAgent Running on all nodes
✔ Dynatrace Kubernetes Overview populated
✔ Problems auto-detected

---

## 13. Next Steps (Optional)

* Deploy sample app (NGINX)
* Enable namespace injection
* Observe Service Flow & Distributed Tracing
* Create custom dashboards

---

**Status:** Kubernetes successfully integrated with Dynatrace (Trial)
