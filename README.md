# ☸️ Kubernetes Practical Labs

Welcome to the **Kubernetes Practical Labs** repository. This repository documents hands-on scenario-based labs designed to master Kubernetes concepts from basics to production troubleshooting.

---

## 🏗️ Architecture & Lab Environment

- **Cluster Name:** `revise` (Local Multi-Node Kind Cluster)
- **Control Plane:** 1 Node (`revise-control-plane`)
- **Worker Nodes:** 2 Nodes (`revise-worker`, `revise-worker2`)
- **Application:** MedTrack (3-Tier Microservices Application: Frontend, Backend, Database)

```
                       +-----------------------------------+
                       |    K8s Cluster: 'revise'         |
                       +-----------------------------------+
                                         |
         +-------------------------------+-------------------------------+
         |                                                               |
+------------------+                                            +------------------+
|  revise-worker   |                                            |  revise-worker2  |
+------------------+                                            +------------------+
| • frontend-2     |                                            | • frontend-1     |
|   (dev, frontend)|                                            |   (dev, frontend)|
| • backend-2      |                                            | • backend-1      |
|   (prod, backend)|                                            |   (dev, backend) |
|                  |                                            | • database-1     |
|                  |                                            |   (prod, db)     |
+------------------+                                            +------------------+
```

---

## 📚 Lab Topics & Progress

| # | Topic | Folder | Status |
|---|---|---|---|
| 00 | **Kubernetes Files & YAML Architecture** | [`00_K8_Files_Structure/`](./00_K8_Files_Structure/) | 🟢 Completed |
| 01 | **Labels, Selectors & Annotations** | [`01-labels-selectors-annotations/`](./01-labels-selectors-annotations/) | 🟢 Completed |
| 02 | **Services + Labels & Selectors** | `02-services-and-networking/` | ⏳ Upcoming |
| 03 | **ReplicaSet & Self-Healing** | `03-replicasets-and-scaling/` | ⏳ Upcoming |
| 04 | **Namespaces & Multi-Tenancy** | `04-namespaces-and-quotas/` | ⏳ Upcoming |
| 05 | **Pod Health Checks & Probes** | `05-liveness-readiness-startup-probes/` | ⏳ Upcoming |
| 06 | **Combined Production Incidents** | `06-production-break-and-fix/` | ⏳ Upcoming |

---

## 🌿 Repository Structure

```text
Kubernetes-Practicals/
├── README.md                                # Root repository guide
├── 00_K8_Files_Structure/                   # Topic 00: YAML & Core Manifest Templates
│   ├── README.md                            # Comprehensive YAML & K8s Architecture Guide
│   ├── pod.yaml                             # Pod manifest template
│   ├── replicaset.yaml                      # ReplicaSet manifest template
│   ├── deployment.yaml                      # Deployment manifest template
│   └── service.yaml                         # Service manifest template
└── 01-labels-selectors-annotations/         # Topic 01 Lab
    ├── README.md                            # Comprehensive scenario documentation
    └── lab01-pods.yaml                      # 6-Pod MedTrack manifest
```
