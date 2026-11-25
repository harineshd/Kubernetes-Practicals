# 🏷️ Topic 01: Kubernetes Labels, Selectors & Annotations
## 🏥 MedTrack 3-Tier Healthcare Rollout & Audit Lab

---

## 📖 Table of Contents
1. [Architecture & Cluster Topology](#-architecture--cluster-topology)
2. [Core Concepts: Labels vs Selectors vs Annotations](#-core-concepts-labels-vs-selectors-vs-annotations)
3. [Lab Setup Manifest](#-lab-setup-manifest)
4. [Phase 1: Dynamic Label Operations](#-phase-1-dynamic-label-operations)
5. [Phase 2: Selectors & Filtering (Equality & Set-Based)](#-phase-2-selectors--filtering-equality--set-based)
6. [Phase 3: Production Break & Fix (Service Selector Mismatch)](#-phase-3-production-break--fix-service-selector-mismatch)
7. [Phase 4: Annotations & Auditing](#-phase-4-annotations--auditing)
8. [Summary Cheat Sheet](#-summary-cheat-sheet)

---

## 🏗️ Architecture & Cluster Topology

This lab runs on a multi-node Kubernetes cluster (`revise`) using Kind:
- **Control Plane:** `revise-control-plane`
- **Worker 1:** `revise-worker`
- **Worker 2:** `revise-worker2`

```
                                      +------------------------------------+
                                      |     Kind Cluster: 'revise'         |
                                      +------------------------------------+
                                                        |
                         +------------------------------+------------------------------+
                         |                                                             |
            +---------------------------+                                 +---------------------------+
            |      revise-worker        |                                 |      revise-worker2       |
            +---------------------------+                                 +---------------------------+
            | [Pod] medtrack-web-2      |                                 | [Pod] medtrack-web-1      |
            |   app=medtrack            |                                 |   app=medtrack            |
            |   tier=frontend           |                                 |   tier=frontend           |
            |   environment=prod        |                                 |   environment=staging     |
            |   release=canary          |                                 |                           |
            |                           |                                 | [Pod] medtrack-api-1      |
            | [Pod] medtrack-api-2      |                                 |   app=medtrack            |
            |   app=medtrack            |                                 |   tier=backend            |
            |   tier=backend            |                                 |   environment=dev         |
            |   environment=prod        |                                 |                           |
            |                           |                                 | [Pod] medtrack-db-primary |
            | [Pod] medtrack-db-replica |                                 |   app=medtrack            |
            |   app=medtrack            |                                 |   tier=database           |
            |   tier=database           |                                 |   environment=prod        |
            |   environment=prod        |                                 |   (backup-schedule)       |
            +---------------------------+                                 +---------------------------+
```

---

## 💡 Core Concepts: Labels vs Selectors vs Annotations

| Feature | Labels | Annotations |
| :--- | :--- | :--- |
| **Purpose** | **Identification & Grouping** of objects | **Non-identifying metadata** (Audit, Release, Tools) |
| **Used by Selectors?** | **YES** (Services, Deployments, `kubectl -l`) | **NO** (Cannot be queried via label selectors) |
| **Query Syntax** | Equality (`=`, `!=`) & Set-based (`in`, `notin`) | Read directly via `describe` or JSONPath |
| **Max Size** | 63 characters per value | Up to 256 KB (large configs, certificates, JSON) |
| **Real-world Examples** | `environment: prod`, `tier: frontend` | `git-commit: 8f2a1b`, `backup-schedule: 0 2 * * *` |

---

## 📄 Lab Setup Manifest

The 6 application pods were provisioned using [`lab01-pods.yaml`](./lab01-pods.yaml):
- 2 Frontend Pods: `medtrack-web-1` (dev), `medtrack-web-2` (prod)
- 2 API Backend Pods: `medtrack-api-1` (dev), `medtrack-api-2` (prod)
- 2 Database Pods: `medtrack-db-primary` (prod), `medtrack-db-replica` (prod)

```bash
kubectl apply -f lab01-pods.yaml
```

---

## ⚡ Phase 1: Dynamic Label Operations

Imperative label management allows operational updates on live resources without restarting containers.

### Task 1: Add `release=canary` to `medtrack-web-2`
```bash
kubectl label pod medtrack-web-2 release=canary
```

### Task 2: Add `team=backend-core` to `medtrack-api-2`
```bash
kubectl label pod medtrack-api-2 team=backend-core
```

### Task 3: Overwrite `environment=dev` to `environment=staging` on `medtrack-web-1`
> **Important:** If a label key already exists, `kubectl` rejects modifications unless `--overwrite` is explicitly provided.
```bash
kubectl label pod medtrack-web-1 environment=staging --overwrite
```

### Task 4: Remove `team` label from `medtrack-api-2`
> **Syntax:** To remove a label, append a minus sign (`-`) to the key name.
```bash
kubectl label pod medtrack-api-2 team-
```

---

## 🔍 Phase 2: Selectors & Filtering (Equality & Set-Based)

Selectors allow operators and Kubernetes controllers to query resources precisely.

### Task 5: Multi-Condition Equality (`app=medtrack` AND `environment=prod`)
```bash
kubectl get pods -l app=medtrack,environment=prod
```
**Output:**
```text
NAME                  READY   STATUS    AGE
medtrack-api-2        1/1     Running   25m
medtrack-db-primary   1/1     Running   25m
medtrack-db-replica   1/1     Running   25m
medtrack-web-2        1/1     Running   25m
```

---

### Task 6: Inequality Selector (`environment!=prod`)
Selects non-production pods (`dev`, `staging`):
```bash
kubectl get pods -l 'environment!=prod'
```
**Output:**
```text
NAME             READY   STATUS    AGE
medtrack-api-1   1/1     Running   25m
medtrack-web-1   1/1     Running   25m
```

---

### Task 7: Set-Based `in` Selector (`tier in (frontend, api, backend)`)
Filter pods belonging to specific tiers:
```bash
kubectl get pods -l 'tier in (frontend, backend)'
```
**Output:**
```text
NAME             READY   STATUS    AGE
medtrack-api-1   1/1     Running   25m
medtrack-api-2   1/1     Running   25m
medtrack-web-1   1/1     Running   25m
medtrack-web-2   1/1     Running   25m
```

---

### Task 8: Set-Based `notin` Selector (`environment notin (dev, staging)`)
Filter pods that do NOT belong to development or staging environments:
```bash
kubectl get pods -l 'environment notin (dev, staging)'
```
**Output:**
```text
NAME                  READY   STATUS    AGE
medtrack-api-2        1/1     Running   25m
medtrack-db-primary   1/1     Running   25m
medtrack-db-replica   1/1     Running   25m
medtrack-web-2        1/1     Running   25m
```

---

### Task 9: Key Existence Check (`release` label exists)
Find any pod that has the `release` key, regardless of value:
```bash
kubectl get pods -l 'release'
```
**Output:**
```text
NAME             READY   STATUS    AGE
medtrack-web-2   1/1     Running   25m
```

---

### Task 10: Key Non-Existence Check (`release` label missing)
Find all pods that do NOT have the `release` key:
```bash
kubectl get pods -l '!release'
```
**Output:**
```text
NAME                  READY   STATUS    AGE
medtrack-api-1        1/1     Running   25m
medtrack-api-2        1/1     Running   25m
medtrack-db-primary   1/1     Running   25m
medtrack-db-replica   1/1     Running   25m
medtrack-web-1        1/1     Running   25m
```

---

## 🛠️ Phase 3: Production Break & Fix (Service Selector Mismatch)

### The Incident:
A Kubernetes Service was provisioned to route traffic to the MedTrack backend with the following selector:
```yaml
spec:
  selector:
    app: medtrack
    tier: backend
```
However, the running API pods were initially labeled with `tier: api`. Because of this mismatch, the Service controller found **0 endpoints** (`ENDPOINTS: <none>`), causing total API downtime.

### The Resolution (In-Place Relabeling):
Instead of recreating or restarting the pods, update the label dynamically on both pods:
```bash
kubectl label pod medtrack-api-1 medtrack-api-2 tier=backend --overwrite
```

### Verification:
```bash
kubectl get pods -l tier=backend --show-labels
```
**Output:**
```text
NAME             READY   STATUS    LABELS
medtrack-api-1   1/1     Running   app=medtrack,environment=dev,tier=backend
medtrack-api-2   1/1     Running   app=medtrack,environment=prod,tier=backend
```
Now the Service automatically detects both pods and attaches them to its Endpoints slice without downtime.

---

## 📝 Phase 4: Annotations & Auditing

Annotations attach rich operational metadata for DevOps automation, audit trails, and backup tooling.

### Task 13: Add Audit & Backup Annotations
```bash
# Add backup schedule and owner to DB pod
kubectl annotate pod medtrack-db-primary backup-schedule="0 2 * * *" owner="DBA Team"

# Add git build commit to API pod
kubectl annotate pod medtrack-api-1 build-commit="a1b2c3d"
```

### Task 14: Overwrite / Update Annotation
When a new build is deployed, update the metadata:
```bash
kubectl annotate pod medtrack-api-1 build-commit="f9e8d7c" --overwrite
```

### Task 15: Inspect Annotations
```bash
# Inspect via describe
kubectl describe pod medtrack-db-primary | Select-String -Pattern "Annotations" -Context 0,3

# Or extract clean JSON using JSONPath
kubectl get pod medtrack-db-primary -o jsonpath='{.metadata.annotations}'
```

### Task 16: Remove Annotation
To delete an annotation, append `-` to the annotation key:
```bash
kubectl annotate pod medtrack-db-primary owner-
```

---

## 📌 Summary Cheat Sheet

| Action | Command Example |
| :--- | :--- |
| **Add Label** | `kubectl label pod <pod-name> <key>=<val>` |
| **Overwrite Label** | `kubectl label pod <pod-name> <key>=<val> --overwrite` |
| **Remove Label** | `kubectl label pod <pod-name> <key>-` |
| **Equality Selector** | `kubectl get pods -l app=medtrack,environment=prod` |
| **Inequality Selector** | `kubectl get pods -l 'environment!=prod'` |
| **Set-Based (in)** | `kubectl get pods -l 'tier in (frontend, backend)'` |
| **Set-Based (notin)** | `kubectl get pods -l 'environment notin (dev, staging)'` |
| **Key Exists** | `kubectl get pods -l 'release'` |
| **Key Does Not Exist** | `kubectl get pods -l '!release'` |
| **Add Annotation** | `kubectl annotate pod <pod-name> <key>="<val>"` |
| **Update Annotation** | `kubectl annotate pod <pod-name> <key>="<val>" --overwrite` |
| **Remove Annotation** | `kubectl annotate pod <pod-name> <key>-` |
| **Display Labels** | `kubectl get pods --show-labels` |
