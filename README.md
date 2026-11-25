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
| 01 | **Labels, Selectors & Annotations** | [`01-labels-selectors-annotations/`](./01-labels-selectors-annotations/) | 🟡 In Progress |
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
└── 01-labels-selectors-annotations/         # Topic 01 Lab
    ├── README.md                            # Comprehensive scenario documentation
    └── pods.yaml                            # 5-Pod MedTrack manifest
```
